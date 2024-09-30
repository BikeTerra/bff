# Biketerra Fitness Format (.bff)
A lightweight JSON format for storing routes or activities.

## Why?

`GPX` doesn't support activities. `TCX` and GPX are XML-based: lots of data repetition = larger filesizes.

`FIT` is a binary format. It's very efficient, but the data is difficult to view or modify without special tools.

`BTT` aims to be the "sweet spot" between data efficiency and readability.

## Structure

| Prop | Type | Required | Purpose |
| :--- | :--- | :------- | :------ |
| name | text | Y | The route or activity title |
| meta.description | text || Additional details |
| meta.author | text || The author name |
| meta.{field} | mixed || Custom meta, e.g. `totalDistance`
| keys | array | Y | row schemas; `[[<type>, [<fields>]], ...]` |
| rows | array | Y | row data; `rows[N][0]` is the schema index |

## Key types
The first element of each `keys` array is the `type`:

| Key type | Purpose |
| :------- | :------ |
| event | An event (stop, lap, session) |
| row | A data row |

## Event types

| Event Type | Purpose |
| :--------- | :------ |
| stop | Stop activity recording |
| lap | Split an activity into chunks |
| session | Separate activities |

## Data fields

| Field | Datatype | Unit |
| :---- | :------- | :--- |
| timestamp | int | Unix epoch (ms) |
| type | text | mainly used for events |
| latitude | float | WGS 84 |
| longitude | float | WGS 84 |
| altitude | int | cm |
| distance | int | cm |
| speed | int | cm/s |
| grade | int | grade * 100 |
| heartRate | int | BPM |
| cadence | int | RPM |
| power | int | watts |

## Examples

### Route (minimal)

```json
{
    "name": "My route",
    "keys": [
        ["row", ["latitude", "longitude", "altitude"]]
    ],
    "rows": [
        [0, 34.052235, -118.243683, 8900],
        [0, 34.052500, -118.244000, 9500]
    ]
}
```

### Activity (minimal)

```json
{
    "name": "My activity",
    "keys": [
        ["row", ["timestamp", "latitude", "longitude", "altitude", "heartRate", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [0, 1727599260000, 34.052235, -118.243683, 8900, 80, 90, 100000, 500, 200, 150],
        [0, 1727599320000, 34.052500, -118.244000, 9500, 85, 95, 150000, 520, 210, 180],
        [0, 1727599560000, 34.052700, -118.244500, 10000, 90, 100, 200000, 550, 220, 200]
    ]
}
```

### Activity (with a stop)

```json
{
    "name": "My activity",
    "keys": [
        ["event", ["timestamp", "type"]],
        ["row", ["timestamp", "latitude", "longitude", "altitude", "heartRate", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [1, 1727599260000, 34.052235, -118.243683, 8900, 80, 90, 100000, 500, 200, 150],
        [1, 1727599320000, 34.052500, -118.244000, 9500, 85, 95, 150000, 520, 210, 180],
        [0, 1727599320000, "stop"],
        [0, 1727599560000, "start"],
        [1, 1727599560000, 34.052700, -118.244500, 10000, 90, 100, 200000, 550, 220, 200]
    ]
}
```

### Triathalon (advanced)

An activity split into 3 sessions (running, swimming, biking). The biking section is split into laps.

```json
{
    "name": "Triathlon Activity",
    "keys": [
        ["event", ["timestamp", "type", "sport"]],
        ["row", ["timestamp", "latitude", "longitude", "altitude", "heartRate", "cadence", "distance", "speed", "grade"]],
        ["row", ["timestamp", "latitude", "longitude", "heartRate", "distance"]],
        ["row", ["timestamp", "latitude", "longitude", "altitude", "hr", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [0, 1727656800000, "session", "running"],
        [1, 1727657100000, 51.5074, -0.1278, 1500, 140, 80, 50000, 450, 120],
        [1, 1727657400000, 51.5075, -0.1279, 2000, 150, 85, 100000, 480, 150],
        [0, 1727658900000, "session", "swimming"],
        [2, 1727659200000, 51.5076, -0.1280, 130, 10000],
        [2, 1727659800000, 51.5077, -0.1281, 140, 20000],
        [0, 1727660700000, "session", "biking"],
        [3, 1727661600000, 51.5078, -0.1282, 3000, 145, 90, 300000, 2000, 250, 100],
        [3, 1727662500000, 51.5079, -0.1283, 3500, 150, 95, 400000, 2200, 260, 120],
        [0, 1727662500000, "lap"],
        [3, 1727663400000, 51.5080, -0.1284, 4000, 155, 100, 500000, 2400, 270, 140],
        [0, 1727663400000, "lap"],
        [3, 1727664300000, 51.5081, -0.1285, 4500, 160, 105, 600000, 2600, 280, 160]
    ]
}
```
