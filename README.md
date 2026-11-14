# aaja-data

Public calendar data for [Aaja](https://github.com/), a Bikram Sambat calendar
app for iOS. Hosted as static JSON via GitHub Pages so the app can pick up new
BS years, holidays, tithis (or corrections to any of them) without an App
Store update.

## Data provenance

BS year, holiday, and tithi data comes only from sources reviewed and
approved by the app's maintainer. A new year is only added once its calendar
has been published and approved by the Nepal Panchanga Nirnayak Bikash
Samiti.

**BS 2083 is currently provisional.** It was cross-checked internally for
arithmetic consistency (month lengths sum to 365, AD date ranges contiguous,
weekday progression consistent) but has **not** yet been verified against the
official Samiti PDF.

**`v1/calendars/holidays.json` and BS 2082's `tithis` in `v1/years.json`
are currently placeholder content, not real data.** `holidays.json` is
filled with obviously-fabricated entries (e.g. "Placeholder Regional
Holiday 1"). 2082's `tithis` just cycle 1, 2, 3, ..., 15, 30, 1, 2, ... across
days with no regard for which tithi actually falls on which real calendar
day — so the app's fetch/cache/render pipeline could be built and tested
end-to-end before real, approved per-day data exists. **Every placeholder
entry must be replaced with real Samiti-approved day assignments before
shipping** — nothing in them should be mistaken for a real holiday or tithi
occurrence.

BS 2083's `tithis` are real day assignments transcribed from a printed
calendar, but not yet cross-checked against the official Samiti PDF (hence
`verified: false`). 32 of its 365 days were inferred from neighbouring dates
rather than read directly, and still need checking.

`v1/years.json`'s `holidays` field (2083 only so far) is a real day-number
list transcribed directly off a printed calendar -- it's complete for the
days it lists, just not yet cross-checked against the official Samiti PDF
(hence `verified: false`).

## Layout

Everything lives under one versioned root, `v1/`, but split into two tiers
that mirror how the app treats the data:

- **`v1/years.json`** — the entire core dataset: month lengths, which days
  are holidays, and tithis, all in one per-year record. Always on, never
  optional or user-toggleable (the app shows Saturdays and these holidays in
  red unconditionally, and a tithi under every single day) -- so the app
  also ships a bundled baseline for it, independent of this repo being
  reachable at all. One file, one request, one shared `verified` flag for
  everything in a year's record.
- **`v1/calendars/`** — the app's optional, user-toggleable "Calendars"
  layer, the same concept as iOS's own Settings → Calendar → Calendars list
  (Birthdays, Holidays, subscribed calendars you can show or hide). Starts
  with `holidays.json`; more named calendars can be added here later
  without disturbing the core tier above.

Each file has its own independent `schemaVersion` — if one file's schema
ever needs a breaking change, only that file moves to a new `v2/` path (e.g.
`v2/years.json`); the others keep working unchanged against `v1/`.

- `v1/years.json` — BS year records: month lengths, Baisakh 1 in AD, which
  days are holidays, tithi numbers, and a `verified` flag covering all of
  it (see schema below). This is the *only* source of which dates render in
  red and which tithi falls on which day -- there's no separate "core
  holidays" or "tithis" file. A day can be red with no name attached to it
  yet; the day-number list is often known well before every day has a
  researched English/Nepali name, so there's nothing to block the redness
  on. Likewise tithis can be filled in month by month as they're worked out.
- `v1/calendars/holidays.json` — holidays (regional, community,
  cultural, etc. -- anything not in the core dataset, and never shown in
  red). Each occurrence references a holiday by `id` rather than carrying
  its name directly, since the same holiday (e.g. Dashain) recurs across
  years on different dates but with the same name -- see the `names` table
  in the schema below.

## Schema

**`v1/years.json`**
```json
{
  "schemaVersion": 1,
  "years": [
    {
      "year": 2083,
      "baisakh1": "2026-04-14",
      "monthLengths": [31, 31, 32, 31, 31, 31, 30, 29, 30, 29, 30, 30],
      "holidays": [
        [1, 18],
        "... 10 more month arrays, one per month, each listing that month's holiday day-numbers ..."
      ],
      "tithis": [
        [27, 28, 29, 30, 1, 2, "... one number per day of month 1 ..."],
        "... 11 more month arrays ..."
      ],
      "verified": false
    }
  ]
}
```
`holidays` and `tithis` are both positionally parallel to `monthLengths`
(index 0 = month 1), one array per month. Both are optional, independently
of each other and of `monthLengths`/`baisakh1` -- a year, or any month
within it, can simply not have holiday or tithi data yet without that being
a data error.

- **`holidays`**: a day appearing in its month's array is a holiday -- shown
  in red, the same weight as Saturday -- regardless of whether any name for
  it exists anywhere (see `holidays.json` for names). Every
  day-number must fall within that month's real length (from `monthLengths`
  in the same record). A missing month's array simply has no holidays yet.
- **`tithis`**: a tithi *number*, not a name — its position in the lunar
  month, 1 through 30, so the number also carries the paksha:

  | Number | Paksha | Tithi (printed number) |
    |---|---|---|
  | 1–14 | Shukla (waxing) | Pratipada (1) … Chaturdashi (14) |
  | 15 | Shukla | Purnima (15) |
  | 16–29 | Krishna (waning) | Pratipada (1) … Chaturdashi (14) |
  | 30 | Krishna | Amavasya / Aunsi (30) |

  A printed Krishna tithi is stored as its printed number + 15 (e.g. Krishna
  Dwadashi, printed 12, is 27). The printed 1–14 alone can't tell the two
  halves apart, which is why the paksha is encoded here rather than inferred.
  Day to day, the number advances by 1, or stays the same when a tithi spans
  two sunrises, or advances by 2 when one is skipped -- never by more.
  The English/Nepali name for each number is fixed and
  lives in the app (`BSTithiNames` in `BSCalendarKit`), not here -- unlike
  holiday names, tithi names are a closed, permanent set that never grows
  or needs correcting independently of an app release. Unlike `holidays`,
  a month's `tithis` array must either be empty (no tithi data for that
  month yet) or have *exactly* as many entries as that month has real days
  -- one tithi per day, no gaps, since every day has exactly one, not just
  "zero or more" like holidays. A wrong count would silently shift every
  later tithi in the month by a day.

**`v1/calendars/holidays.json`**
```json
{
  "schemaVersion": 1,
  "names": {
    "dashain": { "nameEnglish": "Vijaya Dashami", "nameNepali": "विजया दशमी" }
  },
  "years": [
    {
      "year": 2083,
      "verified": false,
      "holidays": [
        { "month": 7, "day": 10, "id": "dashain" }
      ]
    }
  ]
}
```
Every holiday occurrence references a key in the top-level `names` table by
`id` rather than carrying `nameEnglish`/`nameNepali` directly -- the same
holiday recurs across years on different dates but with the same name, so
the name is written once in `names` instead of being repeated on every
occurrence. `id` is an arbitrary stable string key (e.g. `"dashain"`), not a
display value. `month`/`day` must fall within that year's real month lengths
(see `v1/years.json`) once the year is published there. Unlike
`v1/years.json`'s `holidays` field, an entry here is never shown in red --
this file is purely for the optional "Calendars" layer.

## How to consume this

These are static files on GitHub Pages, not a dynamic API — there's no query
syntax or parameters, just a plain `GET` per file, e.g.:

```
GET https://saujanreshmi.github.io/aaja-data/v1/years.json
GET https://saujanreshmi.github.io/aaja-data/v1/calendars/holidays.json
```

- **Conditional requests are supported and expected**, per file. GitHub Pages
  sets an `ETag` on each response; send it back as `If-None-Match` on
  subsequent requests to that same file and you'll get a `304 Not Modified`
  instead of the full body when nothing's changed. A well-behaved client
  checks at most once a day, independently per file, and always uses this.
- **Versioning is path-based, not header-based**, per file. `v1/` means "this
  exact schema," for that one file. If a file's schema ever needs a breaking
  change, that file ships as `v2/` alongside the others' `v1/` rather than
  changing `v1/`'s shape — so don't assume new fields won't appear in a
  `v1/` file additively, but do assume existing fields won't be removed or
  repurposed within that file's `v1/`. (One exception, made before any app
  release: `years.json`'s `tithis` changed on 2026-11-14 from printed numbers
  1–15/30 to 1–30 with paksha encoded. Clients built before then reject the
  new numbers and keep their cached data.)
- **Every file is a complete snapshot, never a diff.** A client refreshing a
  file should replace everything it has cached for that file with whatever
  the latest fetch returns — there's no merge-by-year or upsert-by-date
  semantics, since a republished file is always the full, current truth
  (including e.g. a holiday being retracted, which a partial/merge update
  couldn't express).
- **No rate limiting beyond being a reasonable client.** These are static
  files behind GitHub's CDN (Fastly) — no auth, no API key. Just don't poll
  more often than the data could plausibly change.
- **Treat `verified: false` as provisional**, wherever it appears (per BS
  year in both files). It means that year's data hasn't been checked
  against the official Samiti PDF yet — don't present it to end users as
  authoritative without surfacing that.
