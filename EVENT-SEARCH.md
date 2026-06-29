# EVENT SEARCH

Runbook for the MO Senior Events finder — how it runs, what it returns, and how to operate it.

## Status — 2026-06-28 (end of day)

- ✅ **Date filter (21–120 days) hardened and WORKING.** `find-events.md` now makes the
  run compute/echo EARLIEST (run+21) and LATEST (run+120) up front and do a row-by-row
  date check before saving. Verified on CENTRAL — all 24 rows in window (2026-07-20 →
  2026-10-22), zero sub-21-day violations.
- ✅ **CENTRAL runs fine** — committed in ~2min, 24 events. The earlier 42min stall was a
  one-off transient, not systemic. No split needed.
- ✅ **NORTHEAST** ran clean (auto, earlier). **Cloud unattended path proven.**
- ⚠️ **EAST needs a re-run.** Its run used the pre-hardening prompt and kept 4 sub-21-day
  rows (`2026-06-28-EAST.csv`). Re-run EAST → should come back clean. Verify dates ≥ floor.
- ⬜ **Delete 5 duplicate Routines** in the UI (Monday-clustered 7:00–7:12 dupes). API can't
  delete. Keep the 5 on their own weekday (Sun/Mon/Tue/Wed/Thu).
- ⬜ **Re-verify EAST** after re-run.

Pick up here next session.

## What it does

Web-searches Jessica's Missouri territory for **public events where Medicare-eligible
(65+ / turning-65) seniors gather AND a broker can get a vendor/sponsor table**.
Output = dated CSVs of actionable leads (one row per event) plus a digest.

- End user: Aetna Account Manager placing Independent & Captive brokers at events.
- Repo: `jessicabutterhorn-dev/mo-senior-events` (local: `~/mo-senior-events`).

## How it runs

| Setting | Value |
|---|---|
| **Cadence** | **MANUAL** (as of 2026-06-28). Not auto-scheduled — fire each region yourself. |
| **Model** | Sonnet 4.6 (`claude-sonnet-4-6`) — right tier for search + extraction + formatting. |
| **Where** | Cloud Routines (claude.ai → Code → Routines). Each clones the repo, runs `find-events.md`, commits the CSV to GitHub. |
| **Output** | `events/YYYY-MM-DD-<REGION>.csv`, committed to `main`. |

### Why manual (not weekly)

Weekly auto-runs re-discovered the same events every week → duplicate rows + wasted
searches. Manual run = you control cadence, no duplication. Run a region only when you
want fresh leads for it.

## Search rules (in `find-events.md`)

- **Window: 21–120 days out.** Only events dated **≥ 21 days** from the run date
  (you need ~20 days to get an event/booth approved — anything sooner is unusable)
  and **≤ 120 days** (pre-booking ceiling). Enforced by making the run echo
  EARLIEST/LATEST cutoffs up front + a row-by-row date check before saving (LLM
  date math needs this scaffolding or it slips).
- **Lead with standing senior venues** (senior centers, 55+/senior living, Area
  Agencies on Aging, library senior programs) — not just county fairs.
- **Dedupe** by venue + county + overlapping date range. A `senior day` sub-event of
  a fair is kept as its own row (prime Medicare-table day).
- **Real sources only** — every row traces to a real URL. Blank > guess.
- CSV: quote any field containing a comma; trim whitespace.
- Reads `recurring-leads.csv` first (4 known annual expos) to skip re-searching them.

## Territory (5 regions, in `territory.csv`)

Sharded so each run covers ~12–19 counties (one combined run timed out). One Routine
per region, each on its own day (schedule only matters if re-enabled — currently all
Paused/manual):

| Region | Run day (legacy) | Counties |
|---|---|---|
| NORTHEAST | Sunday | 19 |
| EAST | Monday | 12 |
| CENTRAL | Tuesday | 16 (12 confirmed + 4 verify) |
| SOUTHEAST | Wednesday | ~18 |
| SOUTHWEST | Thursday | ~17 |

`verify` counties (Pettis/Sedalia, Saline, Benton, Hickory) are unconfirmed boundary —
searched but flagged; confirm they're yours before placing a broker.

## How to run a region

1. claude.ai → **Code → Routines**.
2. Click the region's Routine (the one on its own weekday — NOT the Monday-clustered
   dupes) → **Run now**.
3. Wait for it to commit `events/<date>-<REGION>.csv` to GitHub, then `git pull` locally.
4. **Run one region at a time** — avoids timeout/concurrency limits.

## Verify a run worked

- New `events/<date>-<REGION>.csv` committed on `main`.
- Every row `date` is **≥ 21 days** after the run date (e.g. run 2026-06-28 → all
  dates ≥ 2026-07-19).
- Open in Excel/Sheets, filter `tabling_confirmed = yes`, sort by `date`.

## Known issues

- **CENTRAL stall was a one-off.** One early manual run hung ~42min with no commit, but
  a later run finished clean in ~2min (24 events). Treat as transient. If it recurs:
  check the UI session log, or split CENTRAL into two ~8-county runs. NOT a permissions
  problem — the unattended path is proven (NORTHEAST cron ran + committed clean).
- **Duplicate Routines.** There were 2 per region (5 real + 5 Monday-clustered dupes).
  Delete the dupes in the UI (API can't delete). Keep the one on each region's own day.
- **No cross-run dedupe.** Running the same region twice inside 120 days can re-find
  events. Within-file dedupe + recurring-leads seeding reduce but don't eliminate it.

## Permissions

`.claude/settings.json` (committed) allows the headless cloud run to use WebSearch,
WebFetch, Write/Edit, and git commit/push without prompts. The cloud env reads this
from the repo — your local `~/.claude/settings.local.json` does NOT travel to the cloud.
