# aaja-data

Public calendar data for [Aaja](https://github.com/), a Bikram Sambat calendar
app for iOS. Hosted as static JSON via GitHub Pages so the app can pick up new
BS years (or corrections to existing ones) without an App Store update.

## Data provenance

BS year data — month lengths, holidays, tithis — comes only from sources
reviewed and approved by the app's maintainer. Nothing here is invented or
guessed. A new year is only added once its calendar has been published and
approved by the Nepal Panchanga Nirnayak Bikash Samiti.

**BS 2083 is currently provisional.** It was cross-checked internally for
arithmetic consistency (month lengths sum to 365, AD date ranges contiguous,
weekday progression consistent) but has **not** yet been verified against the
official Samiti PDF.

## Layout

- `calendar/v1/years.json` — BS year records (month lengths, Baisakh 1 in AD,
  a `verified` flag). See `schemaVersion` in the file; a future breaking
  change to this format will be published under a new `v2/` path so older app
  versions keep working against `v1/`.

## Schema

```json
{
  "schemaVersion": 1,
  "years": [
    {
      "year": 2083,
      "baisakh1": "2026-04-14",
      "monthLengths": [31, 31, 32, 31, 31, 31, 30, 29, 30, 29, 30, 30],
      "verified": false
    }
  ]
}
```
