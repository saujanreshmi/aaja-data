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

## How to consume this

This is a static file on GitHub Pages, not a dynamic API — there's no query
syntax or parameters, just a plain `GET`:

```
GET https://saujanreshmi.github.io/aaja-data/calendar/v1/years.json
```

- **Conditional requests are supported and expected.** GitHub Pages sets an
  `ETag` on the response; send it back as `If-None-Match` on subsequent
  requests and you'll get a `304 Not Modified` instead of the full body when
  nothing's changed. A well-behaved client checks at most once a day and
  always uses this — there's no reason to re-download an unchanged file.
- **Versioning is path-based, not header-based.** `v1/` means "this exact
  schema." If the schema ever needs a breaking change, it ships as `v2/`
  alongside `v1/` rather than changing `v1/`'s shape — so don't assume new
  fields won't appear in `v1/` additively, but do assume existing fields
  won't be removed or repurposed within `v1/`.
- **No rate limiting beyond being a reasonable client.** It's a static file
  behind GitHub's CDN (Fastly) — no auth, no API key. Just don't poll more
  often than the data could plausibly change (this is updated at most a
  handful of times a year).
- **Treat `verified: false` as provisional.** It means the year hasn't been
  checked against the official Samiti PDF yet — don't present it to end users
  as authoritative without surfacing that.
