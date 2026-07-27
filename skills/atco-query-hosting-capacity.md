---
name: Query ATCO Electric DER hosting capacity
description: >-
  Find how much distributed energy resource (solar, storage, micro-generation) capacity ATCO
  Electric's Alberta distribution grid can accept on a given feeder, substation or map area,
  using ATCO's public, anonymous ArcGIS REST feature service.
api: openapi/atco-electric-hosting-capacity-openapi.yml
base_url: https://services7.arcgis.com/cw2emabghNLkoYlB/arcgis/rest/services/AGO_HostingCapacity/FeatureServer
auth: none
operations:
  - getFeatureServiceInfo
  - getHostingCapacityLayerInfo
  - queryHostingCapacity
  - queryHostingCapacityByPost
  - queryTopHostingCapacityFeatures
generated: '2026-07-27'
method: generated
---

# Query ATCO Electric DER hosting capacity

ATCO Electric publishes DER hosting capacity for its Alberta distribution system as a public Esri
ArcGIS REST feature service. One polyline feature per feeder segment, 880,623 segments across 135
substations, each carrying the capacity in kilowatts that the segment can host. This is the data a
micro-generation applicant needs to know whether the grid has room where they want to connect.

## Before you start

- **No credentials.** No key, token, signup, referer or licence acceptance. Just call it.
- **Always send `f=json`** (or `f=geojson`). Without `f`, the service returns an HTML page, not JSON.
- **Errors come back as HTTP 200.** Check the body for an `error` member before treating a response
  as success. See `errors/atco-problem-types.yml`.
- **This data is unversioned and uncommitted.** No SLA, no deprecation policy, no status page, no
  published licence. Do not build something load-bearing on it without your own caching.

## Step 1 — confirm the layer and its freshness

`getHostingCapacityLayerInfo` — `GET /0?f=json`

Read two things from the response:

- `fields[]` — the authoritative field list: `FEEDER` (alias CIRCUIT), `PHASE`, `SUB_NAME`,
  `SUB_NUM`, `DATELOADED`, `DER_CAP_KW`, `DER_CAP_KW1`, `PROJECT_AREA`, `Shape__Length`.
- `editingInfo.dataLastEditDate` — an epoch-milliseconds timestamp. This is the only refresh signal
  the API has; there is no changelog. It read `1761939339179` (2025-10-31) on 2026-07-27.

Use `getFeatureServiceInfo` (`GET /?f=json`) if you also need `maxRecordCount` and the layer list.

## Step 2 — pick the substation

`queryHostingCapacity` with `returnDistinctValues`:

```
GET /0/query?where=1%3D1&outFields=SUB_NAME&returnDistinctValues=true&returnGeometry=false&f=json
```

Returns 135 distinct substation names. `SUB_NAME` is the layer's display field and the natural
top-level grouping. Match the user's location to a substation name before querying segments —
querying 880,623 features unfiltered is never the right move.

## Step 3 — size the result before you fetch it

`queryHostingCapacity` with `returnCountOnly`:

```
GET /0/query?where=SUB_NAME%3D%27BONNYVILLE%27&returnCountOnly=true&f=json
→ {"count":34009}
```

Always count first. A single substation can hold tens of thousands of segments.

## Step 4 — answer the question with aggregation, not raw features

Prefer statistics over feature dumps. `queryHostingCapacity` supports `outStatistics` and
`groupByFieldsForStatistics`, with COUNT, SUM, AVG, MIN, MAX, STDDEV and percentiles:

```
GET /0/query
  ?where=SUB_NAME%3D%27BONNYVILLE%27
  &groupByFieldsForStatistics=FEEDER
  &outStatistics=[{"statisticType":"max","onStatisticField":"DER_CAP_KW","outStatisticFieldName":"max_kw"}]
  &returnGeometry=false
  &f=json
```

For "where is there the most room", use `queryTopHostingCapacityFeatures`. It needs **both**
`topFilter` and `where` — omitting `where` returns the "No where clause specified" error:

```
GET /0/queryTopFeatures
  ?where=1%3D1
  &topFilter={"groupByFields":"SUB_NAME","topCount":1,"orderByFields":"DER_CAP_KW DESC"}
  &outFields=SUB_NAME,DER_CAP_KW
  &returnGeometry=false
  &f=json
```

## Step 5 — fetch segments, paginated

When you genuinely need the features:

```
GET /0/query
  ?where=SUB_NAME%3D%27BONNYVILLE%27+AND+DER_CAP_KW+%3E+1000
  &outFields=OBJECTID,FEEDER,PHASE,SUB_NAME,DER_CAP_KW
  &orderByFields=OBJECTID
  &returnGeometry=false
  &resultOffset=0
  &resultRecordCount=2000
  &f=json
```

- `maxRecordCount` is **1000**, but `returnGeometry=false` raises the ceiling to **32,000**.
- Keep advancing `resultOffset` while `exceededTransferLimit` is `true` in the response.
- Always set `orderByFields` to a stable field (`OBJECTID`) or paging is not deterministic.
- Use `f=geojson` when you need RFC 7946 output for a map; use `returnGeometry=true` only then —
  the geometry is in NAD83 / Alberta 3TM (wkid 102185), so add `outSR=4326` for WGS84.

## Step 6 — spatial questions

For "what's near this point", use the spatial filter instead of an attribute filter:

```
GET /0/query
  ?geometry=<xmin,ymin,xmax,ymax>
  &geometryType=esriGeometryEnvelope
  &inSR=4326
  &spatialRel=esriSpatialRelIntersects
  &where=1%3D1
  &outFields=FEEDER,SUB_NAME,DER_CAP_KW
  &returnGeometry=false
  &f=json
```

Supported relationships include `esriSpatialRelIntersects`, `Contains`, `Crosses`, `Within`,
`Touches`, `Overlaps`, `Disjoint` and `EnvelopeIntersects`.

## Long queries

If a `where` clause or geometry exceeds practical URL length, use `queryHostingCapacityByPost` —
`POST /0/query` with the same parameters as `application/x-www-form-urlencoded`. It is still a
read; the service advertises capabilities `Query` only and has no write surface.

## What to tell the user, honestly

- `DER_CAP_KW` and `DER_CAP_KW1` carry the same alias and were equal in every sampled feature.
  ATCO publishes no data dictionary, so do not assert a difference between them.
- `DATELOADED` is a **string** field (`'20251031'`), not a date field — you cannot filter it with
  date semantics, only string comparison.
- The values come from a CYME hosting capacity study, refreshed on ATCO's own schedule with no
  published cadence. Report the load date alongside any number you give.
- Hosting capacity is indicative. It is not an interconnection approval. Point the user at
  https://electric.atco.com/en-ca/connections-services/new-connections-changes/micro-generation.html
  for the actual application process.
