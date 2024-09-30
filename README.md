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
        ["row", ["time", "lat", "lng", "alt", "speed", "hr"]]
    ],
    "rows": [
        [0, 1727599260, 34.052235, -118.243683, 89, 5.0, 120],
        [0, 1727599320, 34.052500, -118.244000, 95, 5.2, 125],
        [0, 1727599560, 34.052700, -118.244500, 100, 5.5, 130]
    ]
}
```

### Activity (with a stop)

```json
{
    "name": "My activity",
    "keys": [
        ["event", ["time", "type"]],
        ["row", ["time", "lat", "lng", "alt", "speed", "hr"]]
    ],
    "rows": [
        [1, 1727599260, 34.052235, -118.243683, 89, 5.0, 120],
        [1, 1727599320, 34.052500, -118.244000, 95, 5.2, 125],
        [0, 1727599320, "stop"],
        [0, 1727599560, "start"],
        [1, 1727599560, 34.052700, -118.244500, 100, 5.5, 130]
    ]
}
```

### Triathalon (advanced)

An activity split into 3 sessions (running, swimming, biking). The biking section is split into laps.

```json
{
  "name": "Triathlon Activity",
  "keys": [
    ["event", ["time", "type", "sport"]],
    ["row", ["time", "lat", "lng", "alt", "hr"]],
    ["row", ["time", "lat", "lng", "hr"]]
  ],
  "rows": [
    [0, 1727656800, "session", "running"],
    [1, 1727657100, 51.5074, -0.1278, 15, 140],
    [1, 1727657400, 51.5075, -0.1279, 20, 150],
    [0, 1727658900, "session", "swimming"],
    [2, 1727659200, 51.5076, -0.1280, 130],
    [2, 1727659800, 51.5077, -0.1281, 140],
    [0, 1727660700, "session", "biking"],
    [1, 1727661600, 51.5078, -0.1282, 30, 145],
    [1, 1727662500, 51.5079, -0.1283, 35, 150],
    [0, 1727662500, "lap"],
    [1, 1727663400, 51.5080, -0.1284, 40, 155],
    [0, 1727663400, "lap"],
    [1, 1727664300, 51.5081, -0.1285, 45, 160]
  ]
}
```
