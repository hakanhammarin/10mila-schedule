# 10mila Körschema – web one-pager

A single-page web view of the 10mila TA / Mål / Växel schedule. The page reads
its data from `schedule.csv`, which is generated from the source Excel file so
the schedule can keep being maintained in Excel as before.

## What the page does

- **TV-studio digital clock** at the very top — rolling seconds, red on black
  with a soft glow and a blinking colon, locked to the browser's local time.
- **Auto-focus on the current minute.** The event whose start time is `≤ now`
  (the latest one) is highlighted and pinned at **20 %** from the top of the
  viewport.
- **Auto-scroll** — when the current event changes, the page smoothly
  re-scrolls so the new "now" event is back at the 20 % mark. State is
  re-evaluated every 15 s and re-pinned every 30 s.
- **Past events dimmed** to 70 % intensity (a slight gray-out, not a heavy
  fade) so future events stay easy to scan.
- **Color → category** — Excel cell fill colors on column C are pivoted into
  a readable `category` column in the CSV (and shown as a colored badge and
  side-stripe on the page). See the mapping below.
- **Live CSV reload** — the page re-fetches `schedule.csv` every 5 minutes
  with a cache-buster, so updates pushed from Excel show up without a manual
  refresh.

## File layout

```
.
├── TA - Körschema 2026 ver 1.xlsx   # source of truth (edit in Excel)
├── convert.py                       # xlsx → schedule.csv
├── schedule.csv                     # generated; checked in for convenience
├── index.html                       # the one-pager (HTML + CSS + JS, no build)
└── README.md
```

## Updating the schedule from Excel

1. Edit the `.xlsx` in Excel as usual (times, tasks, notes, colors).
2. Regenerate the CSV:
   ```bash
   pip install openpyxl     # one-time
   python3 convert.py
   ```
   This rewrites `schedule.csv` from whatever `.xlsx` it finds next to the
   script.
3. Commit/deploy `schedule.csv` (and the `.xlsx` if you want to keep the
   source under version control). The web page picks up the new CSV
   automatically within ~5 minutes, or immediately on a manual reload.

## Running the page

`index.html` loads `schedule.csv` via `fetch`, which browsers refuse to do
from `file://`. Serve the folder over HTTP — anything works:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

Or drop the three files (`index.html`, `schedule.csv`, optionally
`convert.py`) onto any static host (GitHub Pages, Netlify, S3, an internal
nginx, …).

## CSV format

`schedule.csv` is plain UTF-8 CSV with a header row. Columns:

| column          | example                                | notes                                                                 |
| --------------- | -------------------------------------- | --------------------------------------------------------------------- |
| `time`          | `2026-05-02T14:30`                     | Local time, no timezone. Used as the event's start time.              |
| `adjusted_time` | `2026-05-02T14:32`                     | Optional. If set, overrides `time` when sorting/highlighting.         |
| `category`      | `Skylt`                                | Derived from the Excel cell color (see below). Drives the badge color. |
| `task`          | `Skylta upplopp för Herr sträcka 1`    | Free text.                                                            |
| `notes`         | `1001-1099, 1100-1199`                 | Free text. Multi-line OK (quote properly per RFC 4180).               |
| `responsible`   | `Christer Johansson`                   | Free text.                                                            |
| `location`      | `Mål/Växel`                            | Free text.                                                            |

You can hand-edit the CSV directly if you don't want to round-trip through
Excel — the page only depends on the CSV.

## Color → category mapping

The Excel uses cell fill on column C as a quick visual coding. `convert.py`
flattens that into the `category` text column:

| Excel fill (ARGB)                       | Visual          | `category`   |
| --------------------------------------- | --------------- | ------------ |
| `FFFFFF00`                              | yellow          | `Skylt`      |
| `FFFFC000`                              | orange          | `Förvarning` |
| `FFFF0000`                              | red             | `Mål`        |
| `FF00B0F0`                              | light blue      | `Växling`    |
| `FF0070C0`                              | dark blue       | `Jaktstart`  |
| theme 3, light tint (`~0.9`)            | light green     | `Admin`      |
| (no fill)                               | —               | `Tidplan`    |

If the schedule starts using a new color, add it to `COLOR_CATEGORIES` in
`convert.py` and to the `--cat-*` CSS variables / `.event[data-cat="..."]`
rules in `index.html`.

## Tweaking the layout

All the knobs live in the `:root` block at the top of `index.html`:

- `--past-opacity` — how dim past events are. Default `0.7` (70 % intensity
  remaining).
- `--focus-offset` — where the current event sits. Default `20vh`.
- `--accent` / `--accent-glow` — clock color and glow.
