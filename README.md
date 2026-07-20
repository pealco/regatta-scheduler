# regatta-scheduler

An [Agent Skill](https://agentskills.io) that builds and revises rowing regatta heat sheets: compact, readable, and free of avoidable athlete conflicts.

Give your coding agent the boat entries and lineup rosters exported from [RegattaCentral](https://www.regattacentral.com), plus (optionally) an existing schedule. It produces a race-ready heat sheet that respects the constraints that matter on race day:

- **Hard constraints:** lane limits per flight, no duplicate boats, no rower in two flights closer together than the conflict window, lunch breaks, valid monotonic times.
- **Soft constraints:** stay close to the original schedule, keep events on their side of lunch, keep similar crews racing together, and move the smallest thing the smallest distance.

The full constraint model, optimization loop, and verification checklist live in [references/scheduling-model.md](references/scheduling-model.md).

Backstory: [By hand, by annealer, by agent](https://pedroalcocer.com/blog/scheduling-a-regatta/), the story of three years of scheduling the same regatta and how this skill came out of it.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills), which works with Claude Code, Codex, Cursor, and most other agents:

```bash
npx skills add pealco/regatta-scheduler
```

Or manually, by cloning into your agent's skills directory:

```bash
# Claude Code
git clone https://github.com/pealco/regatta-scheduler.git ~/.claude/skills/regatta-scheduler

# Codex
git clone https://github.com/pealco/regatta-scheduler.git ~/.codex/skills/regatta-scheduler
```

## Use

Ask your agent things like:

- "Build a heat sheet from `boats.csv` and `boat-athletes.csv`: 5 lanes, first race at 8:00, 7-minute flights, lunch at 12:00, 45-minute athlete conflict window."
- "Deconflict this schedule using the current sheet and these rosters. Move as little as possible."
- "Validate the final schedule and report any rower conflicts, lane overages, or duplicate boats."

## Inputs

The boats and boat-athletes files are RegattaCentral exports, where clubs enter boats and submit lineups.

| File | Contents |
|---|---|
| Boats file | One row per entry: `Boat ID`, event, club, status, gender, race type, age |
| Boat-athletes file | One row per seat: `Boat ID`, athlete, birthdate, gender, affiliation |
| Existing schedule (optional) | Google Sheet or export; treated as the baseline to preserve |

## What it guards against

- Silently accepted athlete conflicts: each one gets fixed, explicitly accepted, or reported, never ignored
- Shredded schedules: the existing order is a baseline to preserve, not raw material
- Domain traps, like combining a 4x with a 4+ (eight oars vs. four)
- Unverified output: the final sheet is re-read and validated after every write

## License

[MIT](LICENSE)
