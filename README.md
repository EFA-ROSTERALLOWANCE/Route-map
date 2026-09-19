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

A flight is on the list if it departs your base on that day **between the start of your standby and
five hours after its end**. The five hours are the call-out lead time: a standby ending at 17:00 can
still be called for a flight that pushes at 22:00. The header shows that reach as **dep by**.

A window whose reach runs past midnight also catches the next morning's early departures. Those are
listed **after** the evening's flights, with a `+1` on the departure time and the day they fly on a
badge, and tapping one opens that day's pairing.

## Patterns

Tap any flight, in the list or on the map, and the pattern panel opens with the whole pairing. For
each day it shows every leg, with sign-on and sign-off, block and ground times, and where the rest or
layover is. The map redraws to show just that pairing, with the flight you tapped highlighted.
Operating legs are marked **OP** and positioning legs **PAX**.

Thirteen pairings are loaded, covering the SYD and MEL A330 international and domestic flying. Where a
pairing has a **Z crew** (3rd pilot) variant, a Crew toggle in the panel switches between the two.
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
