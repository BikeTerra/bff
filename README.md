# Biketerra Fitness Format (.bff)
A lightweight JSON format for storing routes or activities.

## Why?

The other main formats are GPX, TCX, and FIT.

`GPX` doesn't support activities. `TCX` and GPX are XML-based -- there's lots of data repetition, resulting in larger filesizes.

`FIT` is a binary format. It's very efficient, but the data is difficult to view or modify without special tools.

`BTT` aims to be the "sweet spot" between data efficiency and readability.

## Structure

| Prop | Type | Required | Purpose |
| :--- | :--- | :------- | :------ |
| name | text | Y | The route or activity title |
| meta.description | text |||
| meta.author | text |||
| meta.{field} | mixed || Extra data, e.g. `totalDistance`
| keys | array | Y | the data structure: `[[<type>, [<fields>]], ...]` |
| rows | array | Y | data rows. `rows[N][0]` is the data structure key

## Event types

| Event Type | Purpose |
| :--------- | :------ |
| start | Start activity |
| stop | Stop activity |
| lap | Split an activity into chunks |
| session | Separate activities |

## Examples

### Route (minimal)

```json
{
    "name": "My route",
    "keys": [
        ["row", ["lat", "lng", "alt"]]
    ],
    "rows": [
        [0, 34.052235, -118.243683, 89],
        [0, 34.052500, -118.244000, 95]
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
        [0, 1727599260, 34.052235, -118.243683, 89, 80, 1000, 5.0, 200, 1.5],
        [0, 1727599320, 34.052500, -118.244000, 95, 85, 1500, 5.2, 210, 1.8],
        [0, 1727599560, 34.052700, -118.244500, 100, 90, 2000, 5.5, 220, 2.0]
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
        [1, 1727599260, 34.052235, -118.243683, 89, 80, 1000, 5.0, 200, 1.5],
        [1, 1727599320, 34.052500, -118.244000, 95, 85, 1500, 5.2, 210, 1.8],
        [0, 1727599320, "stop"],
        [0, 1727599560, "start"],
        [1, 1727599560, 34.052700, -118.244500, 100, 90, 2000, 5.5, 220, 2.0]
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
    [0, 1727656800, "session", "running"],
    [1, 1727657100, 51.5074, -0.1278, 15, 140, 80, 500, 4.5, 1.2],
    [1, 1727657400, 51.5075, -0.1279, 20, 150, 85, 1000, 4.8, 1.5],
    [0, 1727658900, "session", "swimming"],
    [2, 1727659200, 51.5076, -0.1280, 130, 100],
    [2, 1727659800, 51.5077, -0.1281, 140, 200],
    [0, 1727660700, "session", "biking"],
    [3, 1727661600, 51.5078, -0.1282, 30, 145, 90, 3000, 20.0, 250, 1.0],
    [3, 1727662500, 51.5079, -0.1283, 35, 150, 95, 4000, 22.0, 260, 1.2],
    [0, 1727662500, "lap"],
    [3, 1727663400, 51.5080, -0.1284, 40, 155, 100, 5000, 24.0, 270, 1.4],
    [0, 1727663400, "lap"],
    [3, 1727664300, 51.5081, -0.1285, 45, 160, 105, 6000, 26.0, 280, 1.6]
  ]
}
```
