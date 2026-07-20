---
name: regatta-scheduler
description: Build and revise compact rowing regatta heat sheets from RegattaCentral boats and boat-athletes exports. Use when asked to create, import, deconflict, retime, validate, or review a rowing regatta schedule or Google Sheet using RegattaCentral exports, boat entries, athlete rosters, lane limits, lunch breaks, flight grouping, duplicate-boat checks, athlete conflict windows, age/category grouping, or before/after schedule reports.
license: MIT
metadata:
  author: pealco
  version: "1.0.0"
---

# Regatta Scheduler

## Overview

Use this skill to turn rowing entry exports into a race-ready heat sheet that is compact, readable, and free of avoidable athlete conflicts while staying close to the original schedule.

Always treat the boats file and boat-athletes file as the authoritative inputs. Both are exports from RegattaCentral, where clubs enter boats and submit lineups. If a live Google Sheet or previous heat sheet exists, treat it as the current scheduling state to preserve unless the user says to rebuild from scratch.

## Workflow

1. Load context and constraints.
   - Use the spreadsheet and Google Sheets skills when reading or editing workbooks or live sheets.
   - Identify the boats file, boat-athletes file, target Google Sheet or output path, and any existing schedule.
   - Extract or ask for only missing hard configuration: race date, lane count, first race time, flight interval, lunch start/restart times, athlete conflict window, and any known shell/equipment conflicts.
   - Default only when the user has implied the value in the conversation; otherwise state assumptions before applying them.

2. Normalize the inputs.
   - Join boats to athletes by `Boat ID`.
   - Preserve stable identifiers: boat ID, event ID/order, event label/description, club, status, gender, race type, age/average age, seed, and all sheet columns not being intentionally changed.
   - Normalize event names into classification fields: boat class, oar count, gender category, age category, adaptive/para status, novice/open/masters/junior status, and club.
   - Flag duplicates before scheduling: duplicate boat IDs in the same event, duplicate scheduled boat rows, missing athlete rows, missing boat rows, and placeholder athletes such as TBD.

3. Build candidate flights.
   - Respect the lane limit as a hard constraint; split events with more entries than lanes into multiple flights.
   - Sort split flights by age or seed when available so similar crews race together.
   - Combine sparse events only when boat classes are compatible, preferably same class first and then similar oar-count classes.
   - Avoid mixing extreme boat types, such as singles with eights, unless the user explicitly accepts it.
   - Keep pure flights when they are reasonably full; avoid singleton flights where a compatible combination exists.

4. Optimize the schedule.
   - Use hard constraints first: no duplicate boats, no more boats than lanes, no same-flight duplicate athletes, no athlete conflicts inside the configured window, valid times, and required breaks.
   - Use soft constraints next: preserve original order, preserve morning/afternoon side of lunch, minimize moved flights, keep similar age/category crews together, keep the schedule compact, and preserve visually meaningful groupings.
   - When a conflict cannot be solved without tradeoffs, produce the smallest change set and report the compromise before editing live data.

5. Write the heat sheet.
   - For Google Sheets, edit only the intended ranges and preserve unrelated formatting/formulas.
   - Fill time, flight/combine, lane/bow, status, and formatting columns consistently with the existing sheet conventions.
   - Use alternating fill colors by flight when asked to improve readability.
   - Do not alter handicaps, statuses, or race classifications unless the request includes that scope.

6. Verify and report.
   - Re-export or reread the final sheet after edits.
   - Run conflict analysis from the final sheet plus boat-athletes file.
   - Check lane counts, duplicate boat IDs, missing athletes, same-flight duplicate athletes, cross-flight athlete conflicts, lunch timing, and any user-provided shell constraints.
   - Provide a concise final summary plus an audit report for changed flights, lunch-boundary moves, accepted conflicts, and unresolved risks.

## Detailed Model

Read [references/scheduling-model.md](references/scheduling-model.md) before actually building or changing a schedule. It contains the constraint definitions, optimization scoring guidance, and required reports.

## Guardrails

- Never silently accept an athlete conflict. Either fix it, list it as an accepted exception from the user, or call it unresolved.
- Do not treat event labels alone as ground truth when the schedule has richer descriptions; use current heat-sheet descriptions to classify scheduled boats.
- Do not infer shell/equipment conflicts unless the data explicitly contains equipment IDs or the user provides the relationship. If shell data is missing, say that athlete conflicts are solved but equipment conflicts are unverified.
- Keep edits scoped. A deconflict request should not rewrite handicaps, statuses, event names, or unrelated formatting.
- Preserve a before/after snapshot or export before making live edits so the change report can be reconstructed.
