# Summary

Each map covers the metropolitan area around one or more major Taiwanese cities. Metropolitan-area bounds are drawn from a curated Functional Urban Area (FUA) registry built on Ministry of Interior 村里界圖 (Village & Borough Boundary) polygons; sub-municipal population, employment, and commute data drive resident and worker placement at the 里 (village/borough) grain.

## Features

- High level of detail, with sub-村里 population placement driven by the DGBAS 最小統計區 (Minimum Statistical Area) census polygons — the official sub-里 census product carrying per-cell population direct from the household register — and refined within each cell by NLUI Level-2 residential floor area (pure 0502 + mixed-use 0503) summed from NLSC LoD1 building footprints.
- Spatial realism — points are assigned in a manner that is aware of water features, mountain-indigenous reserve areas, and density-weighted placement surfaces.
- Special demand from several sources is modeled where Taiwanese open data is available.
  - **Airports**
    - Demand based on annualized passenger statistics from the 民用航空局 (Civil Aeronautics Administration / CAA), 2024 throughput across the 6 international and domestic terminals.
  - **Institutions of Learning**
    - Students in post-secondary (大學 / 專科學校) sized from official daytime (日間) per-school enrollment data published by the 教育部 (Ministry of Education / MOE), filtered to the latest 學年度 (academic year). University locations are recovered via a layered OSM exact-name campus + Wikidata + manual-curation pipeline.
  - **Tourism Attractions**
    - Annual visitor counts at major 觀光遊憩據點 (scenic & recreation sites) from the 交通部觀光署 (Tourism Administration) national rollup (data.gov.tw/dataset/8116), filtered to facility-anchored counting methods (門票數 / 人工計數器 / 柵欄式計數設備 / AI 精密人流偵測 / 電子計數器). Per the [Tourism Administration 主要觀光遊憩據點統計作業要點](https://stat.taiwan.net.tw/) methodology spec, the 9 approved counting methods include both facility-level (門票數, 人工計數器, gate, AI, electronic) and area-level (電信數據 telco, 廟方估計 temple self-report, 停車數概估 parking-based estimation, 自動車流監視 traffic monitoring). Area-level entries are auto-skipped to avoid double-counting district pass-through traffic already captured by residential gravity. Methodology cross-check uses the [data.tainan.gov.tw 觀光遊憩月報](https://data.tainan.gov.tw/) per-county feed which preserves the 備註 (methodology notes) column dropped from the national rollup. Coordinates resolved via the Tourism Administration Attraction Registry V2.1 (data.gov.tw/dataset/7777).
- Buildings are sourced from NLSC LoD1 (內政部國土測繪中心 3D 精緻建物模型), with surveyed per-building height and floor count for the entire island.
- OSRM routing is included, with a scaling time penalty based on commute distance and metropolitan area size.
- Building depth defaults to −10 m, with train-related infrastructure exempt.

## High-Level Methodology

Township-level resident and worker totals are taken directly from official census products — residence-side from the MOI 戶政司 monthly per-village register (ODRP014) and DGBAS 2020 census township-level employment-status counts (working-age population, employed persons by agriculture / industry / services sector), workplace-side from the 2021 工商業普查 (Industry & Services Census) per-township persons-engaged. These township totals are conserved through the disaggregation to 里 (village/borough). Within each 里, residents are placed using the DGBAS 最小統計區 (Minimum Statistical Area, MSA) sub-里 census polygons — the official census product carrying a per-cell Dec-2024 P_CNT population field direct from the household register — then refined within each MSA cell by a 100 m residential floor-area signal built from NLSC LoD1 buildings under the NLUI Level-2 residential classification (0502 pure + 0503 mixed-use). Workers are placed using the NLUI Level-2 raster combined with NLSC LoD1 floor area per workplace land-use class, calibrated by a bounded-Tikhonov LSQ with per-county census sector constraints from the 2021 工商業普查 per-county persons-engaged by industry major (17 majors × 22 counties, consolidated into 8 sector groups inside the solve with a white-collar / blue-collar 職員/工員 split). Population counts are conserved at township control totals, while workplace-side calibration via a per-county Option A scaling reconciles the universe mismatch between the 2021 工商業普查 (9.61M workplace persons-engaged) and the 2020 census all-employed universe (11.56M, including agriculture and public-sector employment).

A gravity model augmented with the DGBAS 2020 census commute O/D matrices (workplace × residence cross-tabs at per-county and per-township resolution) reproduces macro-level commute patterns. A cordon-snap pass injects the out-of-bundle commute mass from neighbouring counties as anchored sinks at county centroids; a Generalized Iterative Proportional Fit (GIPF) pass then pins each township's self-loop share to the per-township census ground truth for same-township commutes.

Agriculture employment is dropped at the township grain using the DGBAS census employment-status counts (agricultural-sector employed persons per township) to avoid placing rural-residence farmers in workplace buildings, since the 2020 census records work location by residence rather than workplace for the agricultural cohort.

## Primary Data Sources

- DGBAS — 主計總處 — 2020 Population & Housing Census (township-level employment-status counts, per-county industry counts, workplace × residence cross-tabs, work-commute O/D matrices), 最小統計區 (Minimum Statistical Area) sub-里 census polygons with per-cell population, and 2021 工商業普查 (Industry & Services Census — workplace persons-engaged by township × industry and by county × industry-major) (stat.gov.tw)
- MOI — 內政部 — 戶政司 monthly village population (ODRP014 / ris.gov.tw) and 村里界圖 administrative boundary polygons (data.moi.gov.tw)
- NLSC — 內政部國土測繪中心 — 國土利用調查 (NLUI) Level-2 land-use raster served via public WMS (maps.nlsc.gov.tw), and the 3D 精緻建物模型 LoD1 building footprint + height + floor count product (3dmaps.nlsc.gov.tw)
- MOE — 教育部 — 大專校院校別資料 (per-school × 日間/進修 × degree-level enrollment) and institutional registry (stats.moe.gov.tw)
- CAA — 民用航空局 — terminal-level annual passenger statistics (caa.gov.tw)
- Tourism Administration — 交通部觀光署 — historical monthly visitor counts at major scenic & recreation sites (data.gov.tw/dataset/8116) + Attraction Registry V2.1 (data.gov.tw/dataset/7777). Per-county methodology cross-check via 臺南市開放資料平臺 (data.tainan.gov.tw).

## Issues/Questions

Please raise an issue on this repository for incorrect manifests, broken download links, or release-page problems. Suggestions are welcome — especially pointers to Taiwanese open datasets that could improve existing bundles or unlock new categories (hospital / military / sport / cultural / attractions are scheduled for Phase E expansion).

## Known Issues

- The published commute O/D matrix is the 2020 DGBAS census, which post-dates a partial pandemic disruption. The per-township GIPF calibration anchors the self-loop share to the census ground truth, but the cross-township flow structure inherits whatever residual COVID-era bias remains in the source.
- Individual post-secondary education institutions are placed at a single point matching the registered institutional address, which may not reflect the actual spatial distribution of students across multiple campuses or satellite facilities. Per-faculty placement is planned for a future update.
- Inland-water bathymetry (rivers / lakes / reservoirs) is included via a flat −4 m depth index. **Coastal bathymetry (Taiwan Strait + Pacific)** is scheduled for a v2 release once an offshore depth source is registered.
- Special demand currently covers airports, post-secondary education, and tourism attractions. Hospitals (NHI register), military bases (MND garrison roster), sport venues, and cultural centers will be added as Phase E data lands.
- Tourism attraction visitor counts auto-skip area-level methodologies (電信數據 / 廟方估計 / 停車數概估 / 自動車流監視) per the 9-method counting spec. Methodology is only directly verifiable for ~30 Tainan facilities via the per-county feed; other counties rely on a name-pattern heuristic validated against the Tainan distribution. False positives (legitimately ticketed entries the heuristic flagged as district pass-through) can be re-added via the operator-curated override CSV.
- ~~Some coastal tiles on map edges do not render correctly due to a land mask polygon mismatch between the map boundaries and the "unvierse" tiles~~ **(Resolved in 0.1.1)**
- ~~IPF based synthetic O/D is asymmetric (resident-side constraints) and therefore concentrates error in the workplace-side distribution (e.g. in Wanhua district)~~ **(Resolved in 0.1.1)**

# Changelog

## 0.1.1 (2026-05-26)

### Updated Cities

- `TPE` - 臺北 / Taipei
- `RMQ` - 臺中 / Taichung
- `KHH` - 高雄 / Kaohsiung
- `TNN` - 臺南 / Tainan
- `HSZ` - 新竹 / Hsinchu
- `CYI` - 嘉義 / Chiayi

### Bugfixes

- Coastline rendering — fixed grid-like artifacts and rectangular intrusions at the map's land/ocean boundary, most visible along the western edge of southwest Pingtung county.
- Inland water and rivers — rivers that cross the bundle edge now extend past the map boundary instead of being sharply cut off; small redundant river polygons inside larger estuarine water bodies are no longer rendered as incongruous fragments at river mouths.
- Map labels — city, town, and locality labels from outside the bundle no longer bleed through (e.g. Kaohsiung's major label no longer appears in Tainan, Tainan's no longer in Chiayi).
- Inland depth-index — pockets of inland farmland are no longer mis-marked as ocean in the runtime depth lookup, eliminating spurious "ocean over land" cells.

### New Features

- The synthetic origin/destination commute pass now keys the township-level constraint on the workplace side, removing the resident-side asymmetry that previously concentrated error in residential townships.
- **Taipei (TPE)** — designated-city scope now anchors on multiple city centres across the metropolitan area for improved label centering.

## 0.1.0 (2026-05-24)

### Initial Cities

- `TPE` - 臺北 / Taipei
- `RMQ` - 臺中 / Taichung
- `KHH` - 高雄 / Kaohsiung
- `TNN` - 臺南 / Tainan
- `HSZ` - 新竹 / Hsinchu
- `CYI` - 嘉義 / Chiayi

### New Features

- First release of the Taiwan maps.
- 里-level (village/borough) resident and worker placement for all six bundles, calibrated against the DGBAS 2020 census and the 2021 工商業普查.
- NLSC LoD1 is the canonical TW building source, providing surveyed per-building height and floor count for the entire island.
- Phase E special demand for airports and post-secondary education institutions (universities + 專科學校) are included with a small haircut applied to enrollment counts so the modeled on-site demand matches more realistically the trips generated by the daytime (日間) cohort.
- Cordon-snap injection of out-of-bundle commute mass plus a GIPF pass that pins each township's self-loop share to the per-township DGBAS census ground truth for same-township commutes.

# Planned Updates

- Phase E expansion to cover hospitals, military bases, primary/secondary education, tourism attractions, sport venues, and cultural centers as Taiwanese open data is registered.
- Coastal bathymetry (Taiwan Strait + Pacific) via EMODnet or GEBCO 2024.
- Per-faculty placement for multi-campus universities (currently single-point at the registered institutional address).
- Continued refinement of attraction coverage and point placement.
