# Scheduling Model

## Inputs

Required (both exported from RegattaCentral):
- Boats file: one row per boat entry with `Boat ID`, event/order/label, club, status, gender, race type, age or average age, and optional seed/result fields.
- Boat-athletes file: one row per athlete-seat assignment with `Boat ID`, athlete name or ID, birthdate when available, gender, affiliation, and event label/order.

Optional but valuable:
- Existing heat sheet or Google Sheet URL.
- Race date and event start time.
- Lane count.
- Flight spacing in minutes.
- Lunch or other fixed breaks.
- Athlete conflict window in minutes.
- Known shared-shell/equipment constraints.
- Hard event blocks that must stay before or after lunch.

## Normalized Fields

Create an internal row for every scheduled boat:
- `boat_id`
- `event_id` or `event_order`
- `event_label`
- `description`
- `club`
- `athletes`: stable athlete IDs when present; otherwise normalized names plus birthdates.
- `gender`: men, women, mixed, open, or unknown.
- `race_type`: junior, masters, novice, open, adaptive/para, inclusion, etc.
- `boat_class`: `1x`, `2x`, `2-`, `4+`, `4x`, `8+`, etc.
- `oar_count`: sculling classes count two oars per athlete; sweep classes count one oar per rower. For example `2x` has four oars, `4+` has four oars, `4x` has eight oars, and `8+` has eight oars.
- `age_value`: average age or event age when available.
- `original_time`, `original_order`, and `original_half`: before lunch, after lunch, or unknown.

Use the heat-sheet `Description` for scheduled classification when roster event labels are numeric or ambiguous.

## Hard Constraints

A final schedule must satisfy these unless the user explicitly accepts an exception:
- Every non-scratched boat appears exactly once.
- No duplicate `Boat ID` appears in multiple flights.
- Each flight has at most the configured lane count.
- A flight cannot contain the same athlete twice.
- Any two flights containing the same athlete must be at least the conflict-window minutes apart. Treat the boundary as allowed unless the user says otherwise: with a 45-minute window, 44 minutes conflicts and 45 minutes is okay.
- Flights must have valid, monotonic times except for explicit breaks.
- Lunch or fixed break rows must be respected when configured.
- Known shell/equipment turnaround conflicts must be respected when provided.

## Soft Constraints

Optimize these after hard constraints are satisfied:
- Keep the schedule as compact as possible.
- Preserve the original relative order of flights.
- Preserve before-lunch and after-lunch placement when possible.
- Minimize the number of moved boats/flights and the distance moved.
- Prefer pure flights: same event, same boat class, same gender/category.
- If combining is necessary, combine similar oar-count and race-style classes before less similar ones.
- Keep ages close within split events and combined flights.
- Avoid singleton flights unless no compatible combination exists.
- Avoid moving a large, well-formed flight to solve a small conflict when moving a smaller compatible entry would work.

## Compatibility Heuristics

Use compatibility as a scoring heuristic, not as a replacement for user judgment:
- Best: same event and boat class.
- Good: same boat class, related gender/category, similar age.
- Often acceptable: same oar count and similar race style, such as `2x` with `4+` when the user accepts four-oar matching.
- Usually poor: different oar counts or strongly different racing dynamics, such as `1x` with `8+`.
- Do not assume `4x` is equivalent to `4+`; `4x` has eight oars and `4+` has four.
- Adaptive/para/inclusion events may need separate grouping or status treatment. Ask before combining them with non-adaptive events if the source schedule does not already do so.

## Optimization Loop

1. Preserve the existing schedule as the baseline.
2. Run validation and list all hard-constraint failures.
3. For each conflict, generate local moves first:
   - Move one boat into a compatible nearby flight with spare lanes.
   - Swap compatible boats between flights.
   - Split an overfull event by age/seed.
   - Move a sparse compatible flight across lunch only if same-half moves fail.
4. Score candidates:
   - Reject hard-constraint failures.
   - Penalize lane overages, athlete conflicts, and duplicate boats as effectively infinite.
   - Penalize lunch-boundary moves heavily but not infinitely unless the user says it is hard.
   - Penalize time displacement, event-order inversion, and boat-class impurity.
   - Reward compactness and reduced singleton count.
5. Apply the smallest complete solution, then rerun full validation from the written sheet.

## Retiming Rules

- Time flights, not rows: all boats in the same flight get the same event time.
- Advance by the configured interval between flights.
- If lunch is active, stop before lunch and restart at the configured post-lunch time.
- Keep break rows visible and do not schedule races into them.
- When moving a boat or event to another flight, retime only if the flight order changes or the user asked for retiming.

## Google Sheet Conventions

Adapt to the sheet's actual columns, but common columns include:
- `Event ID`
- `Event Time`
- `Event`
- `Combine`
- `Bow/Lane#`
- `Description`
- `Boat ID`
- `Avg Age`
- `Handicap`
- `Entry Status`
- `Status`

For combined flights, use the sheet's existing convention for `Combine` or flight ID. Preserve blank/combine semantics already established in the workbook.

Use stable lane numbers from 1 through lane count inside each flight. When combining or splitting flights, ensure no duplicate lane numbers in a flight.

## Required Reports

After a schedule change, provide:
- Final conflict count and exact accepted exceptions, if any.
- Lane-overage and duplicate-boat checks.
- List of flights or boats moved, with old time/flight and new time/flight.
- Lunch-boundary changes: originally before lunch now after lunch, and originally after lunch now before lunch.
- Singleton flights remaining and whether they are exhibition or intentional.
- Any unverified risk, especially shell/equipment conflicts when no equipment IDs were provided.

For larger changes, create an HTML report that groups findings by severity and includes athlete names, boat IDs, clubs, event descriptions, and time gaps.

## Verification Checklist

Before saying the schedule is complete:
- Re-read or export the final sheet after writing.
- Confirm all scheduled boat IDs are unique.
- Confirm all active boats from the boats file are present, and no inactive/scratched boats were added unless requested.
- Confirm each flight has no more than the lane count.
- Confirm same-flight athlete duplicates are zero.
- Confirm cross-flight athlete conflicts are zero or explicitly accepted.
- Confirm lunch timing and flight spacing.
- Confirm formatting changes did not hide or corrupt data.
- Confirm change reports are generated from before/after snapshots, not memory.
