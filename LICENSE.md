# OSM-derived files

Three files in this repository are derived from the [OpenStreetMap](https://www.openstreetmap.org/): the SUMO network file `scenario/bus_roads_only.net.xml`, the bus stop definitions in `scenario/gtfs_pt_stops.add.xml`, and the trip and route definitions in `scenario/gtfs_pt_vehicles.add.xml`. Each of those files, taken as a whole, is a database made available under the Open Database License: http://opendatacommons.org/licenses/odbl/1.0/. The licence applying to the individual contents of those databases differs between the network file and the other two, and is stated in the subsections below.

Copyright (C) OpenStreetMap contributors; derived database (C) 2026 Toyota Motor Corporation.

These files contain information from OpenStreetMap, which is made available under the ODbL.

None of them carries an inline `SPDX-License-Identifier`, because one identifier cannot express a licence on a file as a whole that differs from the licence on its contents.

Each of the three repeats the notices that apply to it in a comment at the top of the file, so a file that travels on its own still carries them; redistribution has to keep them.

## The SUMO network file

`scenario/bus_roads_only.net.xml` was originally imported from the OpenStreetMap; only the ways carried by the `route=bus` relations of the Nagoya City Bus were extracted. Any rights in the individual contents of that database are licensed under the Database Contents License: http://opendatacommons.org/licenses/dbcl/1.0/.

## Stop, trip and route definitions

The bus stop definitions in `scenario/gtfs_pt_stops.add.xml` and the trip and route definitions in `scenario/gtfs_pt_vehicles.add.xml` are databases assembled by Toyota Motor Corporation from third-party open data. They are derived from the OpenStreetMap through the network in `scenario/bus_roads_only.net.xml`: they reference its edges and lanes by identifiers built from the OpenStreetMap way identifiers, they measure positions along its lane geometry, and they reflect parts of its structure. Which values come from which source is summarised below.

The individual contents of those databases, other than any OpenStreetMap data they carry, are licensed under the Creative Commons Attribution 4.0 International License: http://creativecommons.org/licenses/by/4.0/. Those contents are the ones derived from the City of Nagoya GTFS-JP open data described in the subsection below, together with Toyota Motor Corporation's own contribution to those files.

To the extent that these files carry OpenStreetMap data, that data is content of the OpenStreetMap database and remains subject to the DbCL 1.0 cited above.

The two files mix material from both sources at the attribute level. The lane and edge identifiers are built from OpenStreetMap way identifiers and the stop positions are measured on OpenStreetMap lane geometry; those are the OpenStreetMap data the paragraph above refers to. The stop names, the arrival times, and the trip and route identifiers derive from the GTFS-JP data described in the subsection below. Everything else is this scenario's own rather than the City of Nagoya's: the romanized route names and trip headsigns, the English service name inside the identifiers together with the stop index and suffix conventions of the `gtfs2pt.py` tooling that produced them, the stop dwell time, the bus stop length, and the vehicle type. None of those last values are surveyed or published figures.

### Source: Nagoya City Bus GTFS-JP data (`scenario/gtfs_pt_stops.add.xml`, `scenario/gtfs_pt_vehicles.add.xml`)

The stop names, arrival times, trip and route identifiers, and stop coordinates that the two files reproduce are derived from the Nagoya City Bus GTFS-JP data (【名古屋市】市バスGTFS-JPデータ / 市バスGTFS-JPデータ 2026年3月28日改正), published as open data by the Transportation Bureau of the City of Nagoya (名古屋市 交通局) at the [BODIK open data catalog](https://data.bodik.jp/dataset/231002_7109030000_bus-gtfs-jp). Copyright (C) City of Nagoya (Transportation Bureau). That dataset is published under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/) ([Japanese](https://creativecommons.org/licenses/by/4.0/deed.ja)), per the [City of Nagoya open data terms of use](https://www.city.nagoya.jp/somu/page/0000056954.html), which require the source to be credited in a prescribed form and data that has been edited or processed to be marked as such.

The two files were produced by processing that dataset. Anyone redistributing them must carry the following notice:

```
このシナリオは、以下の著作物を改変して利用しています。
市バスGTFS-JPデータ 2026年3月28日改正、名古屋市、クリエイティブ・コモンズ・ライセンス 表示4.0 国際
```

In English: this scenario adapts the following work, the *Nagoya City Bus GTFS-JP data, revised 2026-03-28* (City of Nagoya), CC BY 4.0, https://data.bodik.jp/dataset/231002_7109030000_bus-gtfs-jp.

The City of Nagoya's terms of use additionally impose the following condition: 「編集・加工等を行ったデータを、あたかも本市が作成したかのような態様で公表・利用することを禁止します。」 — data that has been edited or processed must not be published or used in a manner suggesting that the City produced it. The notice above and the disclaimer that follows are how this repository meets that condition; anyone redistributing these files, or any of the values they carry, has to meet it too.

The two files are not published by, endorsed by, or attributable to the City of Nagoya or any other organ of the Japanese state, and must not be presented as if the City or any of them had produced them. See [README.md](README.md) for the download URL and checksum of the exact GTFS archive these files were built from.

# Container build files

Everything under `docker/` (e.g. `docker/Dockerfile`) is made available under the [MIT License](https://opensource.org/licenses/MIT).

```
MIT License

Copyright (c) 2026 Toyota Motor Corporation

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Note that the image built from `docker/Dockerfile` installs the
[Eclipse SUMO](https://eclipse.dev/sumo/) runtime, which is distributed under the
[EPL 2.0](https://www.eclipse.org/legal/epl-2.0/). The MIT license above covers
only the build files in this repository, not SUMO itself or the other
dependencies the image pulls in.

# All other files

All other files are licensed under the Creative Commons Attribution 4.0 International License: http://creativecommons.org/licenses/by/4.0/. Copyright (C) 2026 Toyota Motor Corporation. At present they are `README.md`, `scenario/nagoya_bus.sumocfg`, `scenario/vtypes.xml` and `.gitignore`, together with any file added later that no section above covers.

This section does not apply to the OSM-derived files — `scenario/bus_roads_only.net.xml`, `scenario/gtfs_pt_stops.add.xml` and `scenario/gtfs_pt_vehicles.add.xml` — whose terms are set out under *OSM-derived files* above. Nothing in this section grants rights in the OpenStreetMap- or GTFS-JP-derived material those files carry.

This section does not apply to `LICENSE.md` itself: it is the licence notice for the files above, not one of the works being licensed.
