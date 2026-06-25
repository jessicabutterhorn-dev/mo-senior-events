# MO Senior Events

Finds public events in my Missouri territory where Medicare-eligible seniors
gather **and a broker can table** to generate leads. Built for placing
Independent & Captive brokers in front of the 65+ / turning-65 population.

## Files

- **`territory.csv`** — my purple 2026 counties. Edit this to fix the territory.
  The `confidence` column flags counties I still need to verify against the map.
- **`find-events.md`** — instructions the agent follows each run.
- **`events/`** — weekly output. One `YYYY-MM-DD.csv` per run. See `events/SCHEMA.md`.

## How to run

**On demand (now):** open this repo in Claude Code and say:
> Follow find-events.md and search my territory for events.

**Weekly (automatic):** a scheduled cloud agent runs `find-events.md` every week
and commits a fresh CSV to `events/`. (Set up via the schedule routine.)

## What counts as an event

Any senior-skewing public gathering with a vendor/sponsor table option: senior
expos, health fairs, senior days, resource fairs, 55+ dances, bingo, craft
fairs, county fairs, farmers markets, festivals, church senior groups, civic
clubs (Rotary/Lions), library senior programs, Area Agency on Aging events.

## Territory to verify

The red/purple county boundary was read from a map image. Rows marked `verify`
in `territory.csv` need a quick check — especially **Greene (Springfield)** and
the **St. Louis metro**, which are major markets. Fix the CSV and the agent
picks up the change next run.
