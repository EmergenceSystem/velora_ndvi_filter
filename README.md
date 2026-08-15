# velora_ndvi_filter

An [Emergence](https://github.com/EmergenceSystem) filter that turns a text
query into an **NDVI / vegetation index**, computed by a
[velora](https://github.com/roquess/velora) node.

It is a thin adapter: it advertises a vegetation capability vector on the
em_pop gossip mesh and, on `POST /agent/query`, forwards the query to velora
with intent `ndvi`. velora does the real work — classify the query, geocode a
place name through the mesh, find a least-cloud Sentinel-2 scene via STAC,
compute the NDVI raster — and returns an NDVI card with vegetation `stats`
(mean/min/max), which this filter relays.

## Contract

```
POST /agent/query
{"query": "beauce"}               -> {"results": [{"type":"ndvi","id":...,
                                                   "stats":{"mean":N,"min":N,"max":N},
                                                   "bounds":[[s,w],[n,e]]}]}
```

A raster URL works directly (`{"query":"https://host/scene.tif"}`); a place
name requires velora to have a geocoder and STAC configured. If velora is
unreachable the filter returns `{"results": []}`.

## Configuration (`config/sys.config`)

| key | default | meaning |
|-----|---------|---------|
| `pop_port`   | 9212 | em_pop gossip port |
| `query_port` | 9213 | Cowboy HTTP query port |
| `pop_seeds`  | `[{"localhost",9100}]` | em_pop seed peers |
| `velora_url` | `http://127.0.0.1:8080/agent/query` | velora endpoint |

## Build & test

```
rebar3 compile
rebar3 ct
```

Apache-2.0.
