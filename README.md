# EFA Standby Coverage

One question, asked the night before a standby:

> I'm on a V9 out of Sydney on Thursday. What could they actually call me out for?

**Open it at <https://efa-rosterallowance.github.io/Route-map/>**, or open `index.html` straight off
disk. Pick your base, fleet, standby code and day, and it lists every operating freighter departing
your base inside the window. The flights are drawn on a map, and each one opens the full pairing you
would be flying.

## What you tell it

- **Base**: MEL, SYD or BNE. A Sydney standby also covers **Western Sydney (WSI)** departures, so SYD
  includes them.
- **Fleet**: A330 or A320.
- **Standby / reserve code**: V1 to V65, listed in V-number order, each with its window.
- **Day of the week.** The freighter timetable repeats weekly, so a weekday is all it needs.

## What counts as covered

A call-out is bounded **from the start of your standby to the end of the duty it gives you** — 16 hours
on two crew, 20 on three. Your standby's own end time does not come into it. A V-code running
12:00–15:00 still reaches a duty signing off at 04:00 the next morning, and the header shows both
ceilings as **FDP by**.

The limit is measured against the pairing's **sign-off**, not the flight's arrival, because that is when
the duty actually ends: QF7345 lands Melbourne at 00:25 but signs off in Perth at 04:20, nearly four
hours later. Each flight shows the sign-off it was judged against. Where no pairing has been loaded yet
there is only the scheduled arrival, which under-reads any duty with more than one sector, so those read
**(est)** rather than being quietly trusted.

**Positioning has no limit.** A pax sector carries no duty of its own, so every positioning flight
leaving your base that day, from the moment your standby starts, is listed.

The 16 hours reach into the next morning, so early departures the day after are listed too — **after**
the evening's flights, with a `+1` on the departure time and the day they fly on a badge. Tapping one
opens that day's pairing.

## Patterns

Tap any flight, in the list or on the map, and the pattern panel opens with the whole pairing. For
each day it shows every leg, with sign-on and sign-off, block and ground times, and where the rest or
layover is. The map redraws to show just that pairing, with the flight you tapped highlighted.
Operating legs are marked **OP** and positioning legs **PAX**.

Thirteen pairings are loaded, covering the SYD and MEL A330 international and domestic flying. Where a
pairing has a **Z crew** (3rd pilot) variant, a Crew toggle in the panel switches between the two.
QF7525, QF7526 and QF7535 are the flights operated augmented, marked **3 crew** in the list — that is
what gives them the 20-hour limit rather than 16.
Some pairings begin by positioning, such as QF470 MEL–SYD, QF127 SYD–HKG or QF81 SYD–SIN, so those
positioning flights appear in the list too and open the same way. A flight without a loaded pairing
still opens the panel, which says so.

## The map

| | |
|---|---|
| Orange dot | Your base |
| Blue dot | Other ports served |
| Solid green line | Operating sector |
| Dashed amber line | Positioning |

The **Light / Dark** switch changes the map tiles only. The map needs a connection, since it draws
OpenStreetMap tiles through CARTO with Leaflet. **Without one, everything else still works**: the
map box says it could not load, and the coverage list and pattern panel carry on. Nothing is stored or
sent anywhere, and a reload starts fresh.

## Where the data comes from

The flights and pairings are transcribed from the Qantas Freight timetable and pairing sheets into
`Roster_Flight_Numbers.xlsx`, which has four sheets: *Operated (freighter)*, *Positioning*, *Patterns*
and *Notes*. `index.html` carries the same data, and the two are kept in step. The pairings here are
also the source of truth for the other EFA tools, so a change here has to be carried through to them.

## Limits of the tool

It knows the published weekly timetable, not the day's operation. It cannot see ad hoc charters,
cancellations, swaps or a schedule change that has not been transcribed yet, and it does not know
whether Crewing would actually call you, only what departs inside your reach. Check anything that
matters against your roster and Crewing.

Unofficial, and not affiliated with or endorsed by any airline, operator, employer or pilot
association. © 2026 Thomas Pappin. All rights reserved.
