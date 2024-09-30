# Biketerra Fitness Format (.bff)
A lightweight JSON format for storing routes or activities.

## Why?

`GPX` doesn't support activities. `TCX` and GPX are XML-based: lots of data repetition = larger filesizes.

`FIT` is a binary format. It's very efficient, but the data is difficult to view or modify without special tools.

`BFF` aims to be the "sweet spot" between data efficiency and readability.

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
| power | int | watts |
| cadence | int | RPM |
| heartRate | int | BPM |

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
        ["row", ["timestamp", "latitude", "longitude", "altitude", "distance", "speed", "grade", "power", "cadence", "heartRate"]]
    ],
    "rows": [
        [0, 1727599260000, 34.052235, -118.243683, 8900, 100000, 500, 150, 200, 90, 80],
        [0, 1727599320000, 34.052500, -118.244000, 9500, 150000, 520, 180, 210, 95, 85],
        [0, 1727599560000, 34.052700, -118.244500, 10000, 200000, 550, 200, 220, 100, 90]
    ]
}
```

### Activity (with a stop)

```json
{
    "name": "My activity",
    "keys": [
        ["event", ["timestamp", "type"]],
        ["row", ["timestamp", "latitude", "longitude", "altitude", "distance", "speed", "grade", "power", "cadence", "heartRate"]]
    ],
    "rows": [
        [1, 1727599260000, 34.052235, -118.243683, 8900, 100000, 500, 150, 200, 90, 80],
        [1, 1727599320000, 34.052500, -118.244000, 9500, 150000, 520, 180, 210, 95, 85],
        [0, 1727599320000, "stop"],
        [1, 1727599560000, 34.052700, -118.244500, 10000, 200000, 550, 200, 220, 100, 90]
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
        ["row", ["timestamp", "latitude", "longitude", "altitude", "distance", "speed", "grade", "cadence", "heartRate"]],
        ["row", ["timestamp", "latitude", "longitude", "distance", "heartRate"]],
        ["row", ["timestamp", "latitude", "longitude", "altitude", "distance", "speed", "grade", "power", "cadence", "heartRate"]]
    ],
    "rows": [
        [0, 1727656800000, "session", "running"],
        [1, 1727657100000, 51.5074, -0.1278, 1500, 50000, 450, 120, 80, 140],
        [1, 1727657400000, 51.5075, -0.1279, 2000, 100000, 480, 150, 85, 150],
        [0, 1727658900000, "session", "swimming"],
        [2, 1727659200000, 51.5076, -0.1280, 10000, 130],
        [2, 1727659800000, 51.5077, -0.1281, 20000, 140],
        [0, 1727660700000, "session", "biking"],
        [3, 1727661600000, 51.5078, -0.1282, 3000, 300000, 2000, 100, 90, 145],
        [3, 1727662500000, 51.5079, -0.1283, 3500, 400000, 2200, 120, 95, 150],
        [0, 1727662500000, "lap"],
        [3, 1727663400000, 51.5080, -0.1284, 4000, 500000, 2400, 140, 100, 155],
        [0, 1727663400000, "lap"],
        [3, 1727664300000, 51.5081, -0.1285, 4500, 600000, 2600, 160, 105, 160]
    ]
}
```
