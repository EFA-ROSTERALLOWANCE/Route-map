# CLAUDE.md

Guidance for Claude Code when working in the Route-map repo.

## What this repo is

**EFA Standby Coverage**: pick a base, fleet, standby (V) code and weekday, and it lists the operating
freighters that standby could be called out for, drawn on a map. Tap a flight and it opens that flight's
full pairing, with every leg drawn. `index.html` is the whole app: hand-written HTML/CSS/JS, one file, no
build, no framework, no tests. `Roster_Flight_Numbers.xlsx` is the spreadsheet the data is transcribed
from. That is all there is.

**This repo is public, and GitHub Pages serves it** at `https://efa-rosterallowance.github.io/Route-map/`
straight from `main` at the root. So every file committed here, this one included, is published at a
URL. Nothing private goes in: no company documents, no pairing sheets, and no details from the private
sibling repos beyond their names. Pages deploys on every push to `main`, so a push is a release.

## It is the source of truth for what EFA flies, and two other repos read it

This is the part that is easy to break from inside this repo, because nothing here shows it.

- **Live-duty-limits** generates its `PATTERNS` from this file with
  `node tools/patterns-from-routemap.mjs ../route-map/index.html`, run from a clone sitting beside this
  one. The generator does not parse JavaScript. It finds `const PATTERNS = {`, `const PATTERN_OF = {` and
  `const PATTERN_Z_OF = {` and reads up to the first `\n};`, **a `};` at column 0**, then `eval`s each one
  **on its own**. So those three must stay plain object literals: no references to other constants, no
  function calls, no spread, and nothing inside that closes with `};` at column 0. It reads each day's
  `wd`, `rpt`, `rls` and `legs`, each leg's `code`, `dep`, `arr`, `depT`, `arrT`, `pax` and `grnd`, the
  pattern's `title`, the flight half of each `PATTERN_OF` key, and which patterns `PATTERN_Z_OF` names.
  Rename any of those and it breaks there. It works out block time itself from the clocks, using its
  own table of standard UTC offsets, so **a new port here also needs an entry in that table**, or the
  generator fails with `unknown port`.
- **bidding-system** derives each SYD pattern's meal allowance from the layover structure here
  (`rpt`/`rls` and `restPort`), and checks its pattern spans against these pairings.

**So after changing a pairing here, regenerate Live-duty-limits' `PATTERNS`, and tell the user that
bidding-system's allowance figures need re-deriving.** Neither happens by itself: nothing downstream
fetches this file at run time.

Live-duty-limits carries corrections for two gaps in the data here, and the fixes belong upstream:

- **Pairing sheets that disagree with the flight table.** Some pairings carry their own times, not the
  ones `FLIGHTS` prints: QF7525 SYD–HKG at 13:20 in `P7525SUN` against the table's 12:20, and QF7526
  HKG–SYD at 23:40 in `P7525WED` against 23:45. They may be real schedule variants or transcription
  errors. **Ask the user; don't align them yourself.** It is a judgement about the flying.
- **Z crew is marked on only one pairing.** QF7525, QF7526 and QF7535 are flown with three pilots, but
  `PATTERN_Z_OF` has only `'7535|Thu'`, so every other copy of those flights reads as two-crew. Adding Z
  variants here is the durable fix, and it turns Live-duty-limits' `AUGMENTED_ONLY` rule into a no-op
  rather than a correction.

## The data tables

All sit in the single `<script>`. The first four are at the top; the pattern tables follow the rendering
code, about two-thirds of the way down.

| Table | What it holds |
|---|---|
| `STANDBY` | V-code windows `{code, start, end}`. **Kept in V-number order**, because that is the picker's order. |
| `FLIGHTS` | The freighter timetable: `{flight, dep, arr, fleet, net, std, sta, days}`. `M4`/`T4`/`T3` are weekday shorthands defined just above it. |
| `BASE_PORTS` | Which departure ports a base's standby covers. A SYD standby also covers WSI. |
| `AIRPORTS` | Lat/lon and name for every port drawn. A port missing here is silently not drawn. |
| `PATTERNS` | Full pairings, keyed `P<flight><DAY>` (e.g. `P7345WED`), with suffixes for crew-base and Z variants (`P7535WED_MEL`, `P7535THU_Z`). |
| `PATTERN_OF` | `'<flight>\|<Day>'` → pattern id, or `'<flight>\|*'` if the pairing does not depend on the day. |
| `PATTERN_Z_OF` | Same keys, pointing at the Z-crew (3rd pilot) variant. |

**Fleet naming.** `FLIGHTS` and the buttons' `data-fleet` say `A321`, but the UI prints **A320** for it
everywhere, on purpose. Keep the data key `A321` and the label A320.

A pattern is `{title, days:[{wd, rpt, rls, rest, restPort, legs:[…]}], footer?}`. A leg is
`{code, pax, dep, depT, arr, arrT, eq, blk, grnd}`. `code` is a bare number for QF flights (`legLabel`
prefixes `QF`), or a full code for anything else (`CX377`, `CAR001` for a car transfer). `rpt`/`rls`
are omitted on a day that only positions home. `rest` is shown when present, `restPort` alone reads as
a layover, and `blk` is shown only on operating legs.

## Adding or changing a pairing

1. Add the `PATTERNS` entry, then register it in `PATTERN_OF`, and in `PATTERN_Z_OF` if it has a Z
   variant. The Crew toggle in the pattern panel appears on its own when a Z key exists.
2. **A pattern that starts with positioning also needs that flight in `FLIGHTS`**, with `pax:true`,
   `net:'Positioning start'` and an `ac` type. Otherwise it never appears in the list and the pattern
   cannot be reached. QF470, QF127 and QF81 are the existing examples.
3. Every port a leg touches needs an `AIRPORTS` entry, or that leg is not drawn.
4. Update `Roster_Flight_Numbers.xlsx` in step (sheets *Operated (freighter)*, *Positioning*,
   *Patterns*, *Notes*). History keeps the two together. A commit that only touches the spreadsheet
   uses the subject prefix `Spreadsheet:`.
5. Then do the downstream work in the section above.

A flight with no pattern still opens the panel, with an empty-state message asking for its pairing
sheet. That is the intended state for flights nobody has sent a sheet for yet, not a bug.

## The coverage rule

`coveredFlights()`: a flight counts if it is on the chosen fleet, departs one of the base's
`BASE_PORTS`, and has its `std` inside `[standby start, standby end + LEAD_MIN]` on the chosen day.
`LEAD_MIN = 300` is the 5-hour call-out lead time. The same test runs again against the next weekday,
shifted by 24 h, so a window that runs past midnight still catches an early departure. All times are
local wall-clock `HH:MM`, with no dates or time zones: the timetable repeats every week.

Each flight it returns is a copy carrying `depDay` (the weekday it actually departs) and `depMin` (minutes
from the start of the standby day, so a next-morning departure is `1440` or more). **Sort, display and
pattern lookup all use those, never `std` and `sel.day`.** Sorting on `std` put a 01:55 next-morning
flight at the top of the list, and opening it with `sel.day` looked up the standby day's pairing, not the
day it flies. The row carries `data-day` for the same reason, and a next-day row shows its departure as
`+1` with the weekday on a badge.

## Network, and what happens without it

This is the only EFA app that needs a connection. Leaflet 1.9.4 comes from unpkg, and the basemap tiles
from CARTO (`light_all` / `dark_all`, OpenStreetMap data), requested from the **keyed** endpoint
`basemaps.cartocdn.com/rastertiles/<style>/…?key=` with the key in `CARTO_KEY`. Without the key, CARTO
draws "API KEY REQUIRED" across every tile. **The key is public by nature**: it travels in every tile
request from every visitor's browser, so there is nothing to hide, and moving it anywhere else in this
file does not help. Its protection is a domain restriction in the CARTO dashboard. If the watermark
comes back, suspect the key (revoked, over quota, or the domain restriction not matching the Pages URL)
before the code. **The map is optional by design**: if `L` is
undefined, the map box shows a notice and the coverage list and pattern panel still work. Keep it that
way. Anything that depends on Leaflet goes behind the `if(!map) return` guard.

The page is dark-themed throughout. The **Light / Dark** switch changes only the basemap tiles, not the
page. There is no persistence (no `localStorage`) and no fetch of any kind: everything resets on reload.

## Verifying

There are no automated tests here. Open `index.html` (it runs from `file://`, but the map needs a
connection) and work through base → fleet → V code → day, open a pattern, and try the Z-crew toggle on
QF7535 Thursday. **Check it at phone width.** It is used on an iPhone, where every browser is WebKit, so
a narrow desktop window is not a full substitute. After changing any of the three pattern literals, run
the Live-duty-limits generator against this file as the real check that they still slice. Its test suite
then checks every duty's computed sign-off against the `rls` printed here.

## Commits

Every commit is authored as **`Claude <noreply@anthropic.com>`**, one convention across all the EFA
repos. Set it in the clone's local config before the first commit, since there is no global identity to
fall back on:

```
git config user.name "Claude" && git config user.email "noreply@anthropic.com"
```

`git log --format='%an <%ae>'` is the check. Subjects are short and imperative and describe user-visible
behaviour ("Order standby codes by V number in the picker"). **Never merge a pull request through the
GitHub API or the green merge button**: GitHub attributes the merge commit to the authenticated account,
so it lands under the owner's name. Merge locally instead:
`git checkout main && git merge --no-ff <branch> && git push origin main`.
