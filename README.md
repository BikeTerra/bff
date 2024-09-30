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
| keys | array | Y | the data structure: `row` or `event`, followed by an array of fields |
| rows | array | Y | data rows. `rows[N][0]` is the data structure from the appropriate `keys` element

## The purpose of `keys`

The `keys` prop stores an array of data structures. Each data structure is a 2-element array: `[<type>, [<fields>]]`

The first element in every `rows` item is an integer reference to a data structure in the `keys` array.

## Event types

| Event Type | Purpose |
| :--------- | :------ |
| start | Start activity |
| stop | Stop activity |
| lap | Split an activity |
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
        ["event", ["time", "type"]],
        ["row", ["time", "lat", "lng", "alt", "speed", "hr"]]
    ],
    "rows": [
        [0, 1727599200, "start"],
        [1, 1727599260, 34.052235, -118.243683, 89, 5.0, 120],
        [1, 1727599320, 34.052500, -118.244000, 95, 5.2, 125],
        [0, 1727599500, "lap"],
        [1, 1727599560, 34.052700, -118.244500, 100, 5.5, 130],
        [0, 1727601600, "stop"]
    ]
}
```
