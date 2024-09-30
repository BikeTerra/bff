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
| rows | array | Y | row data; `rows[N][0]` is the data structure index |

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
| lat | float | WGS 84 |
| lng | float | WGS 84 |
| alt | int | mm |
| hr | int | BPM |
| cadence | int | RPM |
| distance | int | mm |
| speed | int | mm/s |
| power | int | watts |
| grade | int | grade * 100 |

## Examples

### Route (minimal)

```json
{
    "name": "My route",
    "keys": [
        ["row", ["lat", "lng", "alt"]]
    ],
    "rows": [
        [0, 34.052235, -118.243683, 89000],
        [0, 34.052500, -118.244000, 95000]
    ]
}
```

### Activity (minimal)

```json
{
    "name": "My activity",
    "keys": [
        ["row", ["timestamp", "lat", "lng", "alt", "hr", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [0, 1727599260000, 34.052235, -118.243683, 89000, 80, 90, 1000000, 5000, 200, 150],
        [0, 1727599320000, 34.052500, -118.244000, 95000, 85, 95, 1500000, 5200, 210, 180],
        [0, 1727599560000, 34.052700, -118.244500, 100000, 90, 100, 2000000, 5500, 220, 200]
    ]
}
```

### Activity (with a stop)

```json
{
    "name": "My activity",
    "keys": [
        ["event", ["timestamp", "type"]],
        ["row", ["timestamp", "lat", "lng", "alt", "hr", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [1, 1727599260000, 34.052235, -118.243683, 89000, 80, 90, 1000000, 5000, 200, 150],
        [1, 1727599320000, 34.052500, -118.244000, 95000, 85, 95, 1500000, 5200, 210, 180],
        [0, 1727599320000, "stop"],
        [0, 1727599560000, "start"],
        [1, 1727599560000, 34.052700, -118.244500, 100000, 90, 100, 2000000, 5500, 220, 200]
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
        ["row", ["timestamp", "lat", "lng", "alt", "hr", "cadence", "distance", "speed", "grade"]],
        ["row", ["timestamp", "lat", "lng", "hr", "distance"]],
        ["row", ["timestamp", "lat", "lng", "alt", "hr", "cadence", "distance", "speed", "power", "grade"]]
    ],
    "rows": [
        [0, 1727656800000, "session", "running"],
        [1, 1727657100000, 51.5074, -0.1278, 15000, 140, 80, 500000, 4500, 120],
        [1, 1727657400000, 51.5075, -0.1279, 20000, 150, 85, 1000000, 4800, 150],
        [0, 1727658900000, "session", "swimming"],
        [2, 1727659200000, 51.5076, -0.1280, 130, 100000],
        [2, 1727659800000, 51.5077, -0.1281, 140, 200000],
        [0, 1727660700000, "session", "biking"],
        [3, 1727661600000, 51.5078, -0.1282, 30000, 145, 90, 3000000, 20000, 250, 100],
        [3, 1727662500000, 51.5079, -0.1283, 35000, 150, 95, 4000000, 22000, 260, 120],
        [0, 1727662500000, "lap"],
        [3, 1727663400000, 51.5080, -0.1284, 40000, 155, 100, 5000000, 24000, 270, 140],
        [0, 1727663400000, "lap"],
        [3, 1727664300000, 51.5081, -0.1285, 45000, 160, 105, 6000000, 26000, 280, 160]
    ]
}
```
