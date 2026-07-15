# subwaybuilder-tw-maps

## Summary

Each map covers the metropolitan area around one or more major Taiwanese cities. Metropolitan-area bounds are drawn from a curated Functional Urban Area (FUA) registry built on Ministry of Interior 村里界圖 (Village & Borough Boundary) polygons; sub-municipal population, employment, and commute data drive resident and worker placement at the 里 (village / borough) grain.

## Features

- High level of detail, with sub-村里 population placement driven by the DGBAS 最小統計區 (Minimum Statistical Area) census polygons — the official sub-里 census product carrying per-cell population direct from the household register — and refined within each cell by NLUI Level-2 residential floor area (pure 0502 + mixed-use 0503) summed from NLSC LoD1 building footprints.
- Spatial realism — points are assigned in a manner that is aware of water features, mountain-indigenous reserve areas, and density-weighted placement surfaces.
- Special demand from several sources is modeled where Taiwanese open data is available — covering airports, universities and colleges, tourism attractions, hospitals, ports, convention centers, sports venues, cultural centers, and libraries. See [Special Demand Details](#special-demand-details) below for the per-category breakdown.
- Buildings are sourced from NLSC LoD1 (內政部國土測繪中心 3D 精緻建物模型), with surveyed per-building height and floor count for the entire island.
- OSRM routing is included, with a scaling time penalty based on commute distance and metropolitan-area size.
- Building foundation depth is modeled per building from height and footprint width — deeper for tall, slender towers up to a cap and shallow for low-rise — with freestanding towers held shallow and train-related infrastructure exempt.
- Coastal seafloor depth is modeled from GEBCO global bathymetry, smoothed and aligned to the rendered coastline.

## High-Level Methodology

Township-level resident and worker totals are taken directly from official census products — residence-side from the MOI 戶政司 monthly per-village register (ODRP014) and DGBAS 2020 census township-level employment-status counts (working-age population, employed persons by agriculture / industry / services sector), workplace-side from the 2021 工商業普查 (Industry & Services Census) per-township persons-engaged. These township totals are conserved through the disaggregation to 里 (village / borough). Within each 里, residents are placed using the DGBAS 最小統計區 (Minimum Statistical Area, MSA) sub-里 census polygons — the official census product carrying a per-cell Dec-2024 population field direct from the household register — then refined within each MSA cell by a 100 m residential floor-area signal built from NLSC LoD1 buildings under the NLUI Level-2 residential classification (0502 pure + 0503 mixed-use). Workers are placed using the NLUI Level-2 raster combined with NLSC LoD1 floor area per workplace land-use class, calibrated by a bounded least-squares fit with per-county census sector constraints from the 2021 工商業普查 per-county persons-engaged by industry major (consolidated into sector groups with a white-collar / blue-collar 職員 / 工員 split). Population counts are conserved at township control totals, while a per-county workplace-side scaling reconciles the universe mismatch between the 2021 工商業普查 workplace persons-engaged and the 2020 census all-employed universe (which includes agriculture and public-sector employment).

A gravity model augmented with the DGBAS 2020 census commute origin/destination matrices (workplace × residence cross-tabs at per-county and per-township resolution) reproduces macro-level commute patterns. A cordon-snap pass injects the out-of-bundle commute mass from neighbouring counties as anchored sinks at county centroids; a Generalized Iterative Proportional Fit (GIPF) pass then pins each township's self-loop share to the per-township census ground truth for same-township commutes.

Agriculture employment is dropped at the township grain using the DGBAS census employment-status counts (agricultural-sector employed persons per township) to avoid placing rural-residence farmers in workplace buildings, since the 2020 census records work location by residence rather than workplace for the agricultural cohort.

## Primary Data Sources

- **DGBAS 主計總處 — 2020 Population & Housing Census + 2021 工商業普查** (township employment-status counts, per-county industry counts, workplace × residence commute O/D matrices, 最小統計區 sub-里 census polygons with per-cell population, and Industry & Services Census workplace persons-engaged by township × industry) — [stat.gov.tw](https://www.stat.gov.tw/)
- **MOI 內政部** (戶政司 monthly village population ODRP014, and 村里界圖 administrative-boundary polygons) — [ris.gov.tw](https://www.ris.gov.tw/) · [data.moi.gov.tw](https://data.moi.gov.tw/)
- **NLSC 內政部國土測繪中心** (國土利用調查 NLUI Level-2 land-use raster via public WMS, and the 3D 精緻建物模型 LoD1 building footprint + height + floor-count product) — [maps.nlsc.gov.tw](https://maps.nlsc.gov.tw/) · [3dmaps.nlsc.gov.tw](https://3dmaps.nlsc.gov.tw/)
- **MOE 教育部** (大專校院校別資料 per-school × 日間 / 進修 × degree-level enrollment, and the institutional registry) — [stats.moe.gov.tw](https://stats.moe.gov.tw/)
- **CAA 民用航空局** (terminal-level annual passenger statistics) — [caa.gov.tw](https://www.caa.gov.tw/)
- **Tourism Administration 交通部觀光署** (historical monthly visitor counts at major scenic & recreation sites + the Attraction Registry V2.1; per-county methodology cross-check via 臺南市開放資料平臺) — [觀光遊憩據點 (data.gov.tw)](https://data.gov.tw/dataset/8116) · [Registry V2.1](https://data.gov.tw/dataset/7777) · [data.tainan.gov.tw](https://data.tainan.gov.tw/)
- **National Central Library 國家圖書館** (public-library circulation statistics from the 公共圖書館統計系統, used to size and distribute library demand) — [publibstat.ncl.edu.tw](https://publibstat.ncl.edu.tw/)
- **GEBCO** (global ocean bathymetry — the coastal seafloor depth gradient) — [gebco.net](https://www.gebco.net/data_and_products/gridded_bathymetry_data/)

## Issues/Questions

Please raise an issue on this repository for incorrect manifests, broken download links, or release-page problems. Suggestions are welcome — especially pointers to Taiwanese open datasets that could improve existing bundles or unlock new categories (military bases and primary / secondary education are still scheduled for a future release).

## Known Issues

- The published commute O/D matrix is the 2020 DGBAS census, which post-dates a partial pandemic disruption. The per-township GIPF calibration anchors the self-loop share to the census ground truth, but the cross-township flow structure inherits whatever residual COVID-era bias remains in the source.
- Individual post-secondary institutions are placed at a single point matching the registered institutional address, which may not reflect the actual spatial distribution of students across multiple campuses or satellite facilities. **(Partially resolved in 0.2.0** — cross-county multi-campus institutions are now split across campuses by MOE enrollment shares; per-faculty placement within a single campus is still planned for a future update.**)**
- Special demand currently covers airports, post-secondary education, tourism attractions, hospitals, ports, convention / exhibition centers, stadiums and arenas, and cultural centers. Military bases (MND garrison roster) and primary / secondary education will be added as data sources are curated.
- Tourism attraction visitor counts auto-skip area-level methodologies (電信數據 / 廟方估計 / 停車數概估 / 自動車流監視) per the 9-method counting spec. Methodology is only directly verifiable for a subset of Tainan facilities via the per-county feed; other counties rely on a name-pattern heuristic validated against the Tainan distribution. False positives (legitimately ticketed entries the heuristic flagged as district pass-through) can be re-added via the operator-curated override list.
- The NLSC LoD1 building model is a LiDAR + photogrammetric hybrid product, and some building footprints carry artifacts (e.g. fusing a podium and tower into a single polygon, or having certain areas of the footprint mischaracterized in height).
  - In addition, a single building may be a composite of multiple, often overlapping parts; each of these parts is persisted independently into the buildings index and treated separately for collisions.
- ~~Some coastal tiles on map edges do not render correctly due to a land-mask polygon mismatch between the map boundaries and the "universe" tiles.~~ **(Resolved in 0.1.1)**
- ~~The synthetic O/D is asymmetric (resident-side constraints) and therefore concentrates error in the workplace-side distribution (e.g. in Wanhua district).~~ **(Resolved in 0.1.1)**

## Changelog

### 0.2.0 (2026-07-15)

#### Updated Cities

- `TPE` - 臺北 (Taipei)
- `RMQ` - 臺中 (Taichung)
- `KHH` - 高雄 (Kaohsiung)
- `TNN` - 臺南 (Tainan)
- `HSZ` - 新竹 (Hsinchu)
- `CYI` - 嘉義 (Chiayi)

#### New Features

- **Coastal bathymetry.** Coastal seafloor depth is now modeled from real global bathymetric soundings (GEBCO 2026) as an independently interpolated depth gradient, replacing the previous flat −5 m default — depth-banded contours are smoothed and aligned to the coastline, and the seafloor is rebuilt from the same geometry as the rendered water so the two stay consistent at every zoom.
  - Taiwan is modeled from the GEBCO global grid rather than the Japanese J-EGG500 survey used for the Japan maps; the coastline follows the OpenStreetMap water boundary the maps already render, the nearshore and offshore seafloor are built as one continuous surface so depth no longer steps at the shelf edge, and a hand-curated override fills a few estuary shoals that every global source mis-reads at the shore.

- **Per-building foundation depth.** A building's foundation — the below-ground volume a subway tunnel must clear — is now modeled per building from its height and footprint width rather than the previous flat −10 m default; mid- and high-rise foundations deepen with height and slenderness up to a cap, while low-rise buildings sit at a shallow minimum, and freestanding towers are detected and held at the minimum rather than given a deep foundation.

- **Libraries.** Public libraries are now modeled as daily-commute demand points, sized from National Central Library circulation statistics. System-level circulation is distributed across individual branches, with a footfall correction that reconciles systems whose visit counts are recorded on a different basis so no single branch is over- or under-credited relative to its true share.

#### Other Features

- **Updated buildings index.** The buildings index for each map is now packaged in both `.bin` and `.json` formats for compatibility with the most recent versions of the simulation engine.

- **Added compatibility for the bridges/tunnels layer.** The sim can now distinguish bridges and tunnels via a `structure` field carried on the road output.

### 0.1.3 (2026-06-03)

#### Updated Cities

- `TPE` - 臺北 (Taipei)
- `RMQ` - 臺中 (Taichung)
- `KHH` - 高雄 (Kaohsiung)
- `TNN` - 臺南 (Tainan)
- `HSZ` - 新竹 (Hsinchu)
- `CYI` - 嘉義 (Chiayi)

#### New Features

- **Track-collision now applies to small inland water.** Previously the inland-water depth index sampled at ~300 m grid spacing and missed small streams, ponds, and irrigation lakes.
- **Traditional municipal flags.** Taipei (臺北), Taichung (臺中), and Tainan (臺南) now display their traditional municipal flag designs.

#### Bugfixes

- **Subterranean canals no longer rendered as surface water and no longer block track.** Hydropower diversion tunnels, anti-silt drains, and culverted irrigation channels were previously drawn as visible water polygons within the simulation — these features are now filtered out of the water layer.
- **Administrative-boundary parks no longer paint green over settled villages.** OSM admin-boundary polygons (`nature_reserve`, `national_park`, `protected_area`, `allotments`) used to render as continuous green areas covering the residential areas inside their administrative footprints.
- **Coastline alignment between landuse and water.** Landuse and water now share the exact same simplified vertex positions at every zoom level, eliminating thin sub-pixel overlap ribbons.

### 0.1.2 (2026-06-01)

#### Updated Cities

- `TPE` - 臺北 (Taipei)
- `RMQ` - 臺中 (Taichung)
- `KHH` - 高雄 (Kaohsiung)
- `TNN` - 臺南 (Tainan)
- `HSZ` - 新竹 (Hsinchu)
- `CYI` - 嘉義 (Chiayi)

#### New Features

- **Broader rendered landuse coverage.** Greenery rendering now resolves a wider palette of OSM landuse kinds.
  - The NLUI raster layer is still used for the workplace dasymetric upstream.
- **Coastal landuse no longer covers the ocean.** The rendered landuse layer is now clipped to the authoritative MOI 村里界圖 land polygon, so coastal parks, forest, and other land features no longer extend offshore past the rendered ocean boundary.
- **Multi-campus universities.** Post-secondary institutions that operate across multiple campuses are now split across those campuses using the MOE cross-county student-distribution table, instead of being placed at a single registered institutional address. Total enrollment per institution is preserved.
- **New special-demand categories.** Added across all bundles:
  - **Hospitals** — daily commute demand at inpatient and outpatient facilities. Per-township acute bed counts from MOHW are allocated across hospitals within each township by NLUI Level-2 health-classification building floor area, anchored to the NHI 健保特約醫事機構 national provider register, with outpatient demand sized from per-county outpatient visit rates.
  - **Ports** — demand at the main commercial ports and busiest passenger-ferry routes, sized from Taiwan International Ports Corporation (TIPC) annual passenger statistics and operator-published route reports.
  - **Convention & exhibition centers** — demand at major MICE venues, sized from operator-published ESG and annual reports.
  - **Stadiums & arenas** — demand at professional sport venues including CPBL baseball stadiums and PLG / T1 basketball arenas, sized from published league attendance reports.
  - **Cultural centers** — demand at county and city 文化局 / 藝文中心 facilities.
- **Tourism attraction coverage expanded.** The base 交通部觀光署 dataset is supplemented with operator-published visitor counts from county and city tourism portals.
  - Some multi-anchor scenic complexes are now split across constituent sub-locations rather than concentrated on a single point.
  - A tier-based haircut calibrates the visit-count-to-commute-demand conversion for area-level methodology entries (telco, parking-based estimation, traffic monitoring) that were previously over-credited.
- **Building-classification refinements.** Malformed source building footprints that aggregated multiple separate facilities into a single oversized polygon are excluded from both workplace calibration and the final exported tiles.

#### Bugfixes

- **Residential coverage for newly created villages.** Recent post-2020 administrative splits and the 里 created as a result previously received zero residents because the MOI 村里界圖 boundary shapefile used at build time post-dates the village-level population data.
  - The build now uses dual-vintage MOI 戶政司 population data — the 2020 DGBAS census anchor combined with the latest matched monthly release — and rescales the newer distribution against the census anchor at the township grain.
  - Residents are restored to all affected villages while township totals remain pinned to the official census anchor.
- **Spurious NLSC building artifacts removed.** A small fraction of the NLSC LoD1 3D building model carried photogrammetric artifacts that surfaced as visible bugs in earlier renders. Two classes are now filtered:
  - _Ribbon footprints_ — improbably long thin polygons (likely at the seams of a scan) are excluded.
  - _Corrupted floor counts_ — individual records carrying impossibly high floor values (e.g. a floor count encoding as a 32 km-tall building) are normalized; the building footprint is preserved, only the height and floor fields are reset.

### 0.1.1 (2026-05-26)

#### Updated Cities

- `TPE` - 臺北 (Taipei)
- `RMQ` - 臺中 (Taichung)
- `KHH` - 高雄 (Kaohsiung)
- `TNN` - 臺南 (Tainan)
- `HSZ` - 新竹 (Hsinchu)
- `CYI` - 嘉義 (Chiayi)

#### New Features

- **Workplace-side O/D constraint.** The synthetic origin/destination commute pass now keys the township-level constraint on the workplace side, removing the resident-side asymmetry that previously concentrated error in residential townships.
- **Taipei (TPE) multi-centre labeling.** Designated-city scope now anchors on multiple city centres across the metropolitan area for improved label centering.

#### Bugfixes

- **Coastline rendering.** Fixed grid-like artifacts and rectangular intrusions at the map's land / ocean boundary, most visible along the western edge of southwest Pingtung county.
- **Inland water and rivers.** Rivers that cross the bundle edge now extend past the map boundary instead of being sharply cut off; small redundant river polygons inside larger estuarine water bodies are no longer rendered as incongruous fragments at river mouths.
- **Map labels.** City, town, and locality labels from outside the bundle no longer bleed through (e.g. Kaohsiung's major label no longer appears in Tainan, Tainan's no longer in Chiayi).
- **Inland depth-index.** Pockets of inland farmland are no longer mis-marked as ocean in the runtime depth lookup, eliminating spurious "ocean over land" cells.

### 0.1.0 (2026-05-24)

#### Initial Cities

- `TPE` - 臺北 (Taipei)
- `RMQ` - 臺中 (Taichung)
- `KHH` - 高雄 (Kaohsiung)
- `TNN` - 臺南 (Tainan)
- `HSZ` - 新竹 (Hsinchu)
- `CYI` - 嘉義 (Chiayi)

#### New Features

- **First release of the Taiwan maps.** 里-level (village / borough) resident and worker placement for all six bundles, calibrated against the DGBAS 2020 census and the 2021 工商業普查.
- **NLSC LoD1 buildings.** The canonical TW building source, providing surveyed per-building height and floor count for the entire island.
- **Special demand for airports and post-secondary education.** Universities and 專科學校 included, with a haircut applied to enrollment counts so modeled on-site demand better matches the trips generated by the daytime (日間) cohort.
- **Commute calibration.** Cordon-snap injection of out-of-bundle commute mass plus a GIPF pass that pins each township's self-loop share to the per-township DGBAS census ground truth for same-township commutes.

## Planned Updates

- **Military bases** are scheduled for a future release, once the garrison roster and per-base personnel estimates are curated.
- **Primary and secondary education** demand is scheduled for a future release.

## Special Demand Details

Per-category breakdown of the modeled demand-point categories beyond residence and workplace commute. Each category is geocoded against the relevant authoritative source and sized from operator- or government-published visitor / passenger / enrollment / bed-count figures.

- **Airports**
  - Demand based on annualized passenger statistics from the 民用航空局 (Civil Aeronautics Administration / CAA), across the international and domestic terminals.
- **Institutions of Learning**
  - Post-secondary students (大學 / 專科學校) sized from official daytime (日間) per-school enrollment published by the 教育部 (Ministry of Education / MOE), filtered to the latest academic year. Campus locations are recovered via a layered OSM exact-name + Wikidata + manual-curation pipeline; multi-campus institutions are split across campuses by MOE enrollment shares.
- **Tourism Attractions**
  - Annual visitor counts at major 觀光遊憩據點 (scenic & recreation sites) from the 交通部觀光署 (Tourism Administration) national rollup, filtered to facility-anchored counting methods (門票數 / 人工計數器 / 柵欄式計數設備 / AI 精密人流偵測 / 電子計數器). Area-level methods (telco, temple self-report, parking-based estimation, traffic monitoring) are auto-skipped to avoid double-counting district pass-through traffic already captured by residential gravity; a methodology cross-check uses the per-county 觀光遊憩月報 feed. Coordinates resolve via the Tourism Administration Attraction Registry V2.1.
- **Hospitals**
  - Daily commute demand at inpatient and outpatient facilities. Per-township acute bed counts from MOHW are allocated across hospitals by NLUI Level-2 health-classification building floor area, anchored to the NHI national provider register, with outpatient demand sized from per-county outpatient visit rates.
- **Ports**
  - Demand at the main commercial ports and busiest passenger-ferry routes, sized from Taiwan International Ports Corporation (TIPC) annual passenger statistics and operator route reports.
- **Convention & Exhibition Centers**
  - Demand at major MICE venues, sized from operator-published ESG and annual reports.
- **Sports Venues**
  - Demand at professional sport venues including CPBL baseball stadiums and PLG / T1 basketball arenas, sized from published league attendance reports.
- **Cultural Centers**
  - Demand at county and city 文化局 / 藝文中心 facilities.
- **Libraries**
  - Demand at public libraries, sized from National Central Library (國家圖書館) circulation statistics. System-level circulation is distributed across branches, with a footfall correction reconciling systems whose visit counts are recorded on a different basis.

## License

All maps are released under the [GNU General Public License v3.0](https://github.com/ahkimn/subwaybuilder-tw-maps/blob/main/LICENSE).

## Credits

All maps authored by [Yukina-](https://subwaybuildermodded.com/credits/)
