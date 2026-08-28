# Nagoya City Bus SUMO Simulation Scenario

A SUMO (Simulation of Urban MObility) scenario for the route buses operated in the City of Nagoya.
It combines the OpenStreetMap road network with GTFS-JP bus operation data to run a full-day bus simulation.

## File Structure

This repository contains everything needed to run the simulation. Everything the
simulation itself reads lives in `scenario/`; the rest of the repository is
documentation and the container build:

```
.
├── README.md                         # This file
├── scenario/
│   ├── nagoya_bus.sumocfg            # Simulation configuration
│   ├── bus_roads_only.net.xml        # SUMO network (bus route roads)
│   ├── vtypes.xml                    # Vehicle type definitions
│   ├── gtfs_pt_stops.add.xml         # Bus stop location definitions
│   └── gtfs_pt_vehicles.add.xml      # Trip/route definitions with per-trip timetables
├── .gitignore
├── docker/
│   └── Dockerfile                    # Headless SUMO 1.19.0 runtime image
└── LICENSE.md                        # License
```

## Running the Simulation

### Requirements

Either of the following:

- **SUMO 1.19.0** on the host. 
- **Docker**, using the image described in
  [Running in a Container](#running-in-a-container-docker). 

### Full-day simulation (0:00 – 24:00)

```bash
sumo -c scenario/nagoya_bus.sumocfg
```

The simulation runs without producing FCD output.

### Enabling FCD output

The configuration file contains no output settings. Pass the FCD options on the command line:

```bash
# Enable FCD output from the command line
sumo -c scenario/nagoya_bus.sumocfg --fcd-output out/nagoya_bus_fcd.xml --fcd-output.geo true --device.fcd.period 1
```

The output file `out/nagoya_bus_fcd.xml` records the position, speed, and heading of every bus in WGS84 longitude/latitude (at 1-second intervals with `--device.fcd.period 1`).

These are host paths, relative to this directory. Inside the container the
output path is different — see
[Writing output files to the host](#4-writing-output-files-to-the-host).

---

## Running in a Container (Docker)

`docker/Dockerfile` builds a minimal headless SUMO 1.19.0 image, so you can run
the scenario without installing SUMO on the host. The scenario itself is **not**
baked into the image. Mount the `scenario/` directory at `/scenario` at run time.
That keeps the image small and reusable across scenario revisions.

### 1. Build the image

Run from this directory (the build context is `docker/`; the scenario files are
not part of it):

```bash
docker build -f docker/Dockerfile -t nagoya-bus-sumo:1.19.0 docker/
```

The image installs the `eclipse-sumo==1.19.0` wheel, which pins the exact SUMO
version this scenario was built for. The wheel is architecture-specific but
selected automatically, so the image builds on both `linux/amd64` and
`linux/arm64`.

### 2. Full-day simulation (0:00 – 24:00)

Mount `scenario/` read-only at `/scenario`. The default entrypoint is
`sumo -c nagoya_bus.sumocfg`, so no command is needed:

```bash
docker run --rm -v "$PWD/scenario:/scenario:ro" nagoya-bus-sumo:1.19.0
```

### 3. Overriding SUMO options

Any SUMO options appended after the image name are passed through to `sumo`:

```bash
# Stop the simulation at 06:00
docker run --rm -v "$PWD/scenario:/scenario:ro" nagoya-bus-sumo:1.19.0 --end 21600
```

### 4. Writing output files to the host

The `:ro` mount above makes `/scenario` read-only, so output goes to a separate
writable mount at **`/out`**. Use `--user` so the generated files are owned by you
instead of `root`:

```bash
mkdir -p out
docker run --rm --user "$(id -u):$(id -g)" \
    -v "$PWD/scenario:/scenario:ro" \
    -v "$PWD/out:/out" \
    nagoya-bus-sumo:1.19.0 \
    --fcd-output /out/nagoya_bus_fcd.xml --fcd-output.geo true --device.fcd.period 1
```

Always use the absolute `/out/...` path for container output. A relative path
such as `out/nagoya_bus_fcd.xml` resolves inside the read-only `/scenario` and
fails with `Could not build output file (Read-only file system)`. `/out` does not
exist in the image either, so forgetting the second mount fails immediately with
`No such file or directory` rather than quietly discarding the output when the
container exits.

The host-side `out/` directory is listed in `.gitignore`, so simulation output is
not committed.

## Data Sources

### Map Data (OpenStreetMap)

| Item | Details |
|------|---------|
| Source | [OpenStreetMap](https://www.openstreetmap.org/) — © OpenStreetMap contributors, [ODbL 1.0](https://opendatacommons.org/licenses/odbl/1-0/) |
| Note | Only ways belonging to `route=bus` relations along the bus routes of the Nagoya City Bus were extracted. |


### Bus Operation Data (GTFS-JP)

| Item | Details |
|------|---------|
| Source | 市バスGTFS-JPデータ 2026年3月28日改正, City of Nagoya (Transportation Bureau), [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — [BODIK dataset page](https://data.bodik.jp/dataset/231002_7109030000_bus-gtfs-jp) |
| Download URL | `https://data.bodik.jp/dataset/c5794008-8053-42ab-99b9-ee7f6fdf9a9e/resource/125a1d12-7df6-489c-abde-911856e05d1b/download/20260328_bus-gtfs-jp.zip` |
| SHA-256 | `4684cfddf127281738efe5e3e602f931479c8dcccb7ff8cb012ea5315e9d4c61` (3,682,760 bytes) |
| Format | GTFS-JP (Japanese standard bus information format) |
| Data validity period | 2026-03-28 – 2027-04-30 |
| Simulated day | 2026-03-30, a Monday (`gtfs2pt.py --date 20260330`) |
| Stop names | Official English names from the dataset's `translations.txt` |

## Simulation Specifications

### Raw GTFS Statistics

| Item | Value |
|------|-------|
| Routes (routes.txt) | 185 routes |
| Trips (trips.txt) | 30,417 trips (includes weekday, Saturday, and Sunday/holiday schedules) |
| Stops (stops.txt) | 3,886 stops |

### SUMO Network Statistics

| Item | Value |
|------|-------|
| Edges (road links) | 3,253 (plus 11,613 internal junction edges) |
| Nodes (intersections) | 1,317 — 646 traffic-light, 625 priority, 33 right-before-left, 13 dead-end (plus 2,668 internal junctions) |

### Speed Limits

OSM does not tag `maxspeed` on most of these streets, and SUMO's
`osmNetconvert.typ.xml` then falls back to generic defaults that are motorway
values for a city (tertiary 80 km/h, primary/secondary/trunk 100 km/h). Buses
built to those speeds outrun the GTFS timetable and then idle at every stop, so
the shipped network is capped:

| Road class | Cap applied |
|------------|-------------|
| `highway.tertiary`, `highway.tertiary_link` | 40 km/h |
| All other classes | 60 km/h |

Edges already signposted at or below the cap keep their surveyed OSM value, and
`highway.service` is built at 20 km/h by the type override described above, so
the road lanes still span 20 to 60 km/h
(20, 30, 40, 50 and 60 km/h). Internal junction lanes go lower — down to about
10 km/h — but those are turn-speed limits SUMO derives, not surveyed values.

### Mapped Simulation Trips

| Item | Value |
|------|-------|
| SUMO routes | 677 routes — 672 carry vehicles; the remaining 5 are unused |
| SUMO vehicles (trips) | 11,948 trips |
| Bus stops | 3,847 `<busStop>` elements |
| Timetable | Per trip — 228,264 `<stop>` elements, each timed from its own GTFS trip |

### Simulation Settings

| Item | Value |
|------|-------|
| Simulation time | 0 s – 86,400 s (0:00 – 24:00) |
| FCD output | Not configured (enable from the command line with `--fcd-output` / `--fcd-output.geo true`) |
| Teleport | Disabled (`time-to-teleport=-1`) |
| Dwell time | 10 s per stop, with that trip's own GTFS arrival time as `until` (except the last stop of each trip) |
| Other traffic | None — the network carries the modeled buses only |

> **Note:** because the network contains no general traffic, congestion here comes
> only from buses interacting with each other and with traffic lights. Conversely,
> since a halted bus occupies its lane and 63.3% of the bus stops sit on
> single-lane edges (55.3% of all stop events), followers queue behind a dwelling
> bus. The modeled delay at busy stops is pessimistic relative to real bus bays.

## Licenses

The SUMO network file `scenario/bus_roads_only.net.xml`, the bus stop definitions in `scenario/gtfs_pt_stops.add.xml` and the trip and route definitions in `scenario/gtfs_pt_vehicles.add.xml` are databases derived from the OpenStreetMap and from Japanese open data. Each of those files, taken as a whole, is a database made available under the [ODbL 1.0](http://opendatacommons.org/licenses/odbl/1.0/), and different terms apply to their individual contents. The container build files under `docker/` are licensed under the [MIT License](https://opensource.org/licenses/MIT), and all other files under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/).

See [LICENSE.md](LICENSE.md) for the authoritative terms, including the attribution notices that redistribution has to carry.

### OpenStreetMap attribution

Those three files contain information from OpenStreetMap, which is made available under the ODbL.

```
© OpenStreetMap contributors
```

### Nagoya City Bus GTFS-JP attribution

The two additional files carry data adapted from the Nagoya City Bus GTFS-JP feed, which the City of Nagoya (Transportation Bureau) publishes under CC BY 4.0. The City's terms of use prescribe the form of the notice:

```
このシナリオは、以下の著作物を改変して利用しています。
市バスGTFS-JPデータ 2026年3月28日改正、名古屋市、クリエイティブ・コモンズ・ライセンス 表示4.0 国際
```

