# ATCO (atco)

ATCO Ltd. (TSX: ACO.X) is a Calgary, Alberta diversified global corporation and the controlling shareholder of Canadian Utilities Limited, through which it runs the regulated energy businesses that make it one of western Canada's largest utility groups. It sits across several tiers of the value chain at once: ATCO Electric owns and operates electricity transmission and distribution across Alberta, ATCO Gas and ATCO Pipelines distribute and transmit natural gas in the province, ATCO EnPower builds storage, renewables and hydrogen assets, ATCO Australia operates gas distribution infrastructure in Western Australia, and ATCO Energy is a competitive retailer selling electricity, natural gas and home services to Alberta customers. Its API posture is the mirror image of the Ontario utilities: no consumer energy data mandate reaches it at all. Alberta has no Green Button regulation, ATCO is not on the Green Button Alliance member list, and it does not appear in the public Australian CDR energy data holder register — so there is no consumer data API, and a customer's usage and billing data lives only behind the My Account login on a Salesforce commerce portal. What ATCO does publish, unmandated and without any signup at all, is real open grid data: ATCO Electric's DER hosting capacity map is served from a public, anonymously queryable Esri ArcGIS REST feature service carrying 880,623 feeder segment features, linked directly from ATCO Electric's own micro-generation and grid-connection pages. Open grid data, closed consumer data, no developer portal, and no OpenAPI anywhere on the estate.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/atco/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/atco/refs/heads/main/apis.yml)

## Tags

- Energy
- Canada
- Utilities
- Electricity
- Gas
- Grid
- Distribution
- Transmission
- DER
- Solar
- Renewables
- Open Data
- Geospatial
- Alberta

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### ATCO Electric Hosting Capacity Feature Service

ATCO Electric's distributed energy resource (DER) hosting capacity data, published as a public Esri ArcGIS REST feature service and surfaced through the "DER HOSTING CAPACITY MAP" web application that ATCO Electric links from its micro-generation and connect-to-the-grid pages. A single polyline feature layer, "Hosting Capacity (KW)", exposes FEEDER, PHASE, SUB_NAME, SUB_NUM, DATELOADED, DER_CAP_KW, DER_CAP_KW1 and PROJECT_AREA fields. An anonymous, unauthenticated query against layer 0 returned a feature count of 880,623 on 2026-07-27. Service capabilities are "Query" only, maxRecordCount 1000. There is no ATCO documentation page, no OpenAPI definition, and no published usage terms — the contract is the self-describing ArcGIS REST service descriptor itself.

- **Human URL:** [https://electric.atco.com/en-ca/connections-services/new-connections-changes/micro-generation.html](https://electric.atco.com/en-ca/connections-services/new-connections-changes/micro-generation.html)
- **Base URL:** `https://services7.arcgis.com/cw2emabghNLkoYlB/arcgis/rest/services/AGO_HostingCapacity/FeatureServer`

#### Tags

- Grid
- DER
- Hosting Capacity
- Distribution
- Geospatial
- Open Data
- Micro-generation

#### Properties

- [API Reference](https://services7.arcgis.com/cw2emabghNLkoYlB/arcgis/rest/services/AGO_HostingCapacity/FeatureServer?f=json)
- [API Reference](https://services7.arcgis.com/cw2emabghNLkoYlB/arcgis/rest/services/AGO_HostingCapacity/FeatureServer/0?f=json)
- [Service Descriptor](openapi/atco-electric-hosting-capacity-featureserver.esri.json)
- [Service Descriptor](openapi/atco-electric-hosting-capacity-layer-0.esri.json)
- [Documentation](https://electric.atco.com/en-ca/connections-services/new-connections-changes/micro-generation.html)
- [Documentation](https://electric.atco.com/en-ca/connections-services/new-connections-changes/connect-grid.html)
- [Application](https://atco.maps.arcgis.com/apps/webappviewer/index.html?id=fa9c52c758b64f90ada01c9d27609d8d)
- [Standard](https://developers.arcgis.com/rest/services-reference/enterprise/feature-service/) — ArcGIS REST Feature Service

## Common Properties

- [Website](https://www.atco.com/)
- [About](https://www.atco.com/en-ca/about-us.html)
- [Website](https://electric.atco.com/en-ca.html) — ATCO Electric
- [Website](https://gas.atco.com/en-ca.html) — ATCO Gas
- [Website](https://www.atcoenergy.com/) — ATCO Energy (retail)
- [Sign In](https://store.atco.com/ccrz__CCSiteLogin) — My Account (customer login)
- [LinkedIn](https://www.linkedin.com/company/atco-group)
- [Regulator](https://www.auc.ab.ca/) — Alberta Utilities Commission
- [Regulator](https://www.aeso.ca/) — Alberta Electric System Operator

## Mandate Posture

| Field | Value |
| --- | --- |
| Home market | Canada (Alberta) |
| Mandate regime | `none` |
| Mandate status | `not-applicable` |
| Data standard | no energy data standard reference found |
| Consumer data API | No |
| Open market/grid data | Yes |
| Access gate | `self-serve` (anonymous — no signup at all) |
| Auth model | None; anonymous HTTPS |
| Developer portal | None |
| OpenAPI / Swagger | None found |

Ontario Regulation 633/21 binds Ontario local distribution companies; ATCO distributes in Alberta, not Ontario. The public Australian CDR energy data holder register was queried directly on 2026-07-27 and returned 84 brands, none matching ATCO. The Green Button Alliance member list contains no ATCO entry, and the phrase "green button" appears nowhere on ATCO's Canadian web properties.

## Maintainers

- Kin Lane — kin@apievangelist.com
