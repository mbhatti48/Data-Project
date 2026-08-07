# Toronto Cafe Location Scoring

An end-to-end data engineering pipeline that scores every location in Toronto as a place to open a cafe. The city is divided into 5,747 hexagons of roughly 250 metres each. Every hexagon receives a transparent 0 to 100 score built from competition, transit access, residential density, and income, plus a letter grade and a machine-learned location type. The final output is an interactive map of the whole city.

**Pipeline:** MongoDB Atlas (raw landing) -> Databricks (bronze / silver / gold on Delta Lake, Spark SQL) -> H3 spatial indexing -> weighted scoring model -> k-means segmentation -> interactive Folium map.

## The output

**[Open the live interactive map](https://mbhatti48.github.io/Data-Project/toronto-cafe-location-scoring/toronto_cafe_map.html)**, or download `toronto_cafe_map.html` and open it in any browser. Either way an internet connection is needed, for the basemap tiles. Features:

- All 5,747 hexagons coloured by score, red to green
- Hover tooltip: score, grade, location type, cafe and transit counts
- Search box that jumps to any of Toronto's 158 neighbourhoods
- A toggleable second view colouring the city by its five location types

## Key results

- **1,693 cafes** in Toronto (774 chain, 919 independent, a 45.7% chain share), counted from OpenStreetMap
- **88% of the city's hexagons contain no cafe at all.** The analysis is built so those empty hexagons stay visible, because underserved areas are the point of a location model
- **Household income runs inverse to cafe viability in Toronto.** The richest neighbourhoods (Bridle Path at $164k median after tax) are car dependent with no walkable retail; the poorest (Kensington-Chinatown at $56k) are dense and full of thriving cafes. Income is therefore modelled as a floor rather than a ladder, and residential density carries the demand signal, with 54x variation across the city against income's 3.2x
- **Unsupervised clustering independently rediscovered the transit network.** Two of the five location types are statistically identical in income and density and differ only 7x in transit. The 1,193 transit-served hexagons with no cafes are the model's opportunity space, and they trace the bus and streetcar routes visibly on the map
- The top scoring hexagons land in genuine cafe territory (West Queen West, the Danforth, High Park North, Yonge-Eglinton), validated against the city rather than against the model's own assumptions

## Architecture

```
OpenStreetMap (Overpass API)  --\
TTC GTFS feed (CKAN API)      ---->  MongoDB Atlas   ---->  Databricks
City of Toronto boundaries    --/    (raw landing)          bronze: raw documents as landed
2021 Census profiles  ----------------------------------->  silver: typed tables, H3 hex grid, counts
                                                            gold:   factors and scores per hexagon
                                                                |
                                              k-means segmentation (scikit-learn)
                                                                |
                                              Folium interactive map (toronto_cafe_map.html)
```

Design decisions worth noting:

- **A document database as the landing zone** because the source data is genuinely semi-structured: every OpenStreetMap cafe carries a different set of tags. The clean tabular census bypasses it and enters at the silver layer, since routing structured data through a document store adds a step and no value
- **H3 hexagons instead of a geometry engine.** Assigning every point a hexagon address turns "what is within 250 metres" into a GROUP BY. No PostGIS, and the approach scales unchanged from one city to a country
- **Medallion layering on Delta Lake**, so raw data stays faithful and every downstream table can be rebuilt from it
- **Data quality gates at every seam:** row counts, null checks, duplicate hexagon checks, and a join round trip verifying that 18-digit hexagon ids survived a Spark to pandas conversion intact

## The scoring model

Five factors, each normalised to 0 to 100, combined as a weighted average. All weights and constants are visible in the notebook and tunable.

| Factor | Weight | Shape |
| --- | --- | --- |
| Market validation | 25% | Peaks at 3 to 5 existing cafes: proven demand with room left. Zero cafes scores 30, not 0, and saturation (10+) scores 20 |
| Transit access | 25% | Log curve against the citywide maximum, damping interchange overcounting |
| Residential density | 20% | Percentile rank of people per hexagon |
| Competition headroom | 20% | Smooth decay on effective competition, where a chain counts as 0.4 of an independent. Zero competitors scores a neutral 50: absence of competitors is absence of information, not evidence of opportunity |
| Income adequacy | 10% | A floor with a gentle ramp, flat above $70k. No bonus for extreme wealth |

The model took three calibration rounds. The first version ranked empty suburban bus loops at the top (emptiness was rewarded twice, a formula bug). The second favoured any hexagon holding a single chain cafe (a data gap: no demand signal, fixed by joining the census). The third was validated against ground truth: known cafe districts score high without topping the list, which is correct behaviour since their best corners are already taken. The lesson that generalises: formula bugs are fixed with arithmetic, data gaps are fixed with new sources, and no amount of weight tuning conjures information the data does not contain.

### Grade

Each hexagon also gets a letter grade, computed with `PERCENT_RANK()` over `location_score` rather than a fixed score threshold: a grade shows how a hexagon ranks against every other hexagon in the city, not an absolute cutoff.

| Grade | Percentile rank |
| --- | --- |
| A | Top 5% (95th and up) |
| B | 80th-95th |
| C | 40th-80th |
| D | 15th-40th |
| F | Bottom 15% |

Grades are relative, not absolute: an A means better than 95% of Toronto, not "guaranteed good."

## Repository contents

| File | What it is |
| --- | --- |
| `toronto_cafe_pipeline.ipynb` | The main Databricks notebook: bronze through gold, scoring, clustering, map. Fully narrated |
| `ingest_neighbourhoods.ipynb` | Local ingest: City of Toronto boundary file -> MongoDB |
| `ingest_osm.ipynb` | Local ingest: cafes from the Overpass API -> MongoDB |
| `ingest_ttc.ipynb` | Local ingest: TTC GTFS feed via the CKAN API -> MongoDB |
| `toronto_cafe_map.html` | The interactive map. Open in a browser |
| `DATA_SOURCES.md` | Every source, its licence, and its vintage |
| `neighbourhood_census_2021.csv` | The reshaped census table (158 rows), included so the pipeline can be reproduced without redoing the extract |
| `.env.example` | Template for the MongoDB connection string |

## Reproducing it

1. Create a free MongoDB Atlas cluster and put its connection string in a local `.env` (copy `.env.example`)
2. Run the three ingest notebooks locally (Python 3.11+, `pip install -r requirements.txt`) to land the raw data
3. Import `toronto_cafe_pipeline.ipynb` into Databricks (built on Free Edition serverless), add the connection string, run top to bottom
4. The reshaped census table ships in this repository (`neighbourhood_census_2021.csv`, derived from the open StatCan profiles); upload it to Databricks as `silver_census` with the neighbourhood code read as text so the leading zeros survive. The extraction itself is documented inside the pipeline notebook

Note on the Databricks tier: Free Edition serverless does not permit custom JARs, so the pipeline reads Mongo through the Python driver rather than the Spark connector. At this data size (a few thousand documents) a single-node read is comfortable; on a standard cluster the connector is a drop-in swap for parallel reads.

## Known limitations

Stated rather than hidden, with the planned fix where one exists:

1. **Counting stops at hexagon borders.** A cafe 20 metres into the next hexagon is invisible to its neighbour. Fix: count over each hexagon plus its ring of six neighbours (H3 grid_disk)
2. **No daytime population.** Census counts where people sleep, so office district demand is understated. Fix: StatCan place-of-work data
3. **Density and income are neighbourhood level, not hexagon level.** Population per hexagon is the neighbourhood total divided evenly across its hexagons, and median income is inherited the same way, so every hexagon in a neighbourhood carries identical values for both. Across 5,747 hexagons that is only 156 distinct density values and 72 distinct income values. The practical consequence: two cafe-free hexagons in the same neighbourhood differ only in transit access, because the two cafe factors are also constant wherever there are no cafes. Neighbourhood profiles are the finest granularity Statistics Canada publishes; finer resolution would need dissemination-area data and a spatial crosswalk
4. **"Independent" means no brand recorded in OpenStreetMap.** A reasonable inference, not a verified registry
5. **No zoning layer.** The model can score parkland or industrial hexagons where a cafe cannot open
6. **Cafes just outside the city limit are not counted** as competition for border hexagons

## Data sources and licences

All data is open. Full details in `DATA_SOURCES.md`.

| Source | Licence |
| --- | --- |
| OpenStreetMap via the Overpass API | ODbL, (c) OpenStreetMap contributors |
| City of Toronto Open Data (neighbourhoods, TTC GTFS) | Open Government Licence - Toronto |
| Statistics Canada, 2021 Census neighbourhood profiles | Statistics Canada Open Licence |

Census figures are from 2021, the most recent available: 2026 Census releases begin in February 2027.

## Licence

MIT. See `LICENSE`.
