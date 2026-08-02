# Data Sources

Every source feeding the pipeline: what it provides, how it is accessed, how fresh it is, and its licence. All sources are open and free. No API keys and no paid services.

| # | Source | Provides | Access | Licence |
| --- | --- | --- | --- | --- |
| 1 | OpenStreetMap (Overpass API) | Cafe locations, names, and brand tags for Toronto | Live API query in `ingest_osm.ipynb` | ODbL, (c) OpenStreetMap contributors |
| 2 | City of Toronto Open Data: Neighbourhoods | Official 158 neighbourhood boundary polygons | Manual GeoJSON download | Open Government Licence - Toronto |
| 3 | City of Toronto Open Data: TTC Routes and Schedules | Every TTC stop record (subway, streetcar, bus) as a GTFS feed | Fetched programmatically through the city's CKAN API in `ingest_ttc.ipynb`, so each run pulls the current feed | Open Government Licence - Toronto |
| 4 | Statistics Canada, 2021 Census (City of Toronto Neighbourhood Profiles) | Population and median after-tax household income per neighbourhood | Manual download, reshaped from wide to one row per neighbourhood | Statistics Canada Open Licence |

## Links

- Neighbourhoods: https://open.toronto.ca/dataset/neighbourhoods/
- Neighbourhood Profiles (census): https://open.toronto.ca/dataset/neighbourhood-profiles/
- TTC Routes and Schedules (GTFS): https://open.toronto.ca/dataset/ttc-routes-and-schedules/
- Overpass API: https://overpass-api.de/api/interpreter

## Freshness notes

- **OpenStreetMap** is community maintained and continuously edited; the Overpass mirror lags the live map by minutes. Every landed record carries an ingestion timestamp
- **TTC GTFS** is republished roughly every six weeks (each board period). Because the ingest asks the CKAN API for the current file, re-running always fetches the latest feed
- **Neighbourhood boundaries** changed to the 158-area model in 2021 and are deliberately stable
- **Census figures are 2021**, the most recent that exist anywhere in Canada. The 2026 Census begins publishing in February 2027, with income detail later. The pipeline is built so a refreshed profile file is a drop-in replacement

## Coverage caveats

- OpenStreetMap cafe coverage in Toronto is strong but community sourced; treat counts as representative rather than exhaustive. Cafes mapped as building outlines rather than points are not captured by the current query
- A cafe counts as a chain when a brand tag is recorded. Untagged small chains are counted as independents
- The transit feed records every direction and platform separately, so stop counts measure how densely transit is recorded, not the number of distinct places to board
