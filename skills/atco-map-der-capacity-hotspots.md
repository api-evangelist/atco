---
name: Map ATCO Electric DER capacity hotspots for site selection
description: >-
  Rank ATCO Electric's Alberta substations and feeders by available DER hosting capacity and
  produce a GeoJSON shortlist, to help site a solar, storage or micro-generation project where
  the distribution grid has the most room.
api: openapi/atco-electric-hosting-capacity-openapi.yml
base_url: https://services7.arcgis.com/cw2emabghNLkoYlB/arcgis/rest/services/AGO_HostingCapacity/FeatureServer
auth: none
operations:
  - getHostingCapacityLayerInfo
  - queryHostingCapacity
  - queryTopHostingCapacityFeatures
generated: '2026-07-27'
method: generated
---

# Map ATCO Electric DER capacity hotspots for site selection

A siting workflow over the same public feature service: go from 880,623 feeder segments to a small
GeoJSON shortlist of the highest-capacity locations, without ever pulling the whole layer.

## Step 0 — record the vintage

`getHostingCapacityLayerInfo` — `GET /0?f=json`. Capture `editingInfo.dataLastEditDate` and carry it
through to your output. Every number in this workflow is only as current as that timestamp, and
there is no changelog to check against.

## Step 1 — rank substations by peak capacity

`queryHostingCapacity` with grouped statistics:

```
GET /0/query
  ?where=1%3D1
  &groupByFieldsForStatistics=SUB_NAME
  &outStatistics=[
     {"statisticType":"max","onStatisticField":"DER_CAP_KW","outStatisticFieldName":"max_kw"},
     {"statisticType":"avg","onStatisticField":"DER_CAP_KW","outStatisticFieldName":"avg_kw"},
     {"statisticType":"count","onStatisticField":"OBJECTID","outStatisticFieldName":"segments"}]
  &orderByFields=max_kw DESC
  &returnGeometry=false
  &f=json
```

135 substations, one row each. This is the whole grid summarized in a single request — the correct
shape for this API, which will otherwise happily try to hand you a million polylines.

Add a `having` clause to filter on the aggregate (the layer advertises `supportsHavingClause: true`),
for example `having=MAX(DER_CAP_KW) > 15000` — the top class break on ATCO's own map renderer.

## Step 2 — drop down to the best feeder in each candidate substation

`queryTopHostingCapacityFeatures`:

```
GET /0/queryTopFeatures
  ?where=SUB_NAME IN ('<A>','<B>','<C>')
  &topFilter={"groupByFields":"FEEDER","topCount":1,"orderByFields":"DER_CAP_KW DESC"}
  &outFields=SUB_NAME,SUB_NUM,FEEDER,PHASE,DER_CAP_KW
  &returnGeometry=false
  &f=json
```

Both `topFilter` and `where` are required. Quote string literals with single quotes.

## Step 3 — pull the shortlist as GeoJSON

Only now ask for geometry, and only for the segments that survived:

```
GET /0/query
  ?where=SUB_NAME%3D%27<SUB>%27 AND DER_CAP_KW %3E 15000
  &outFields=OBJECTID,SUB_NAME,FEEDER,PHASE,DER_CAP_KW,DATELOADED
  &returnGeometry=true
  &outSR=4326
  &orderByFields=OBJECTID
  &resultRecordCount=1000
  &f=geojson
```

`outSR=4326` reprojects from the layer's native NAD83 / Alberta 3TM (wkid 102185) to WGS84 so the
result drops straight into any web map. Page on `properties.exceededTransferLimit`.

## Step 4 — bound the search area geographically

If the project has a fixed area of interest, filter spatially instead of by substation:

```
GET /0/query
  ?geometry=<xmin,ymin,xmax,ymax>
  &geometryType=esriGeometryEnvelope
  &inSR=4326
  &spatialRel=esriSpatialRelIntersects
  &where=DER_CAP_KW %3E 5000
  &outFields=SUB_NAME,FEEDER,DER_CAP_KW
  &returnCountOnly=true
  &f=json
```

Count first, then re-run without `returnCountOnly` once you know the result is a sane size.

## Caveats to carry into the output

- Hosting capacity is a CYME study result, not an interconnection offer, and not a queue position.
  Two projects can both "fit" on paper and not both connect.
- Capacity is per segment. A high figure on one segment says nothing about the upstream feeder or
  transformer.
- ATCO publishes no licence or attribution requirement (the ArcGIS item has `licenseInfo: null` and
  `accessInformation` empty) and no usage terms — but also no permission grant. Attribute ATCO
  Electric anyway.
- Cross-check the shortlist against ATCO Electric's own map application before acting:
  https://atco.maps.arcgis.com/apps/webappviewer/index.html?id=fa9c52c758b64f90ada01c9d27609d8d
