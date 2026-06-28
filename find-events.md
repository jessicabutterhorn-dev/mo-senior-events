# MO Senior Events Finder — Agent Instructions

You find **public events in Jessica's Missouri territory where Medicare-eligible
seniors gather AND a broker can set up a vendor/sponsor table** to generate leads.

Audience served: Medicare-eligible (65+) and turning-65 population.
End user: Aetna Account Manager placing Independent & Captive brokers at events.

## Step 1 — Load territory

Read `territory.csv`. Search every county where `confidence` is `confirmed` or
`verify`. (The `verify` rows are unconfirmed boundary counties — still search
them; Jessica will prune later.)

**Region sharding:** the territory is split into regions via the `region` column
(NORTHEAST, EAST, CENTRAL, SOUTHEAST, SOUTHWEST) so each run covers ~15–19
counties and avoids timeouts. If the run prompt specifies a `REGION`, search
ONLY rows where `region` matches it; otherwise search all rows.

## Step 2 — Search the web

**First, compute the date window and write it down before searching:**
- `EARLIEST` = run date **+ 21 days**
- `LATEST` = run date **+ 120 days**
- Example: run date 2026-06-28 → EARLIEST **2026-07-19**, LATEST **2026-10-26**.

State both dates explicitly at the start of your run, then use them as hard
bounds. An event qualifies on date ONLY if its `date` is on/after EARLIEST and
on/before LATEST.

For each county, run web searches for events dated between EARLIEST and LATEST
(at least 21 days out, no more than 120). Jessica needs ~20 days' lead to get a
booth/event approved — anything sooner than 21 days is unusable, so do not return
it. Senior gatherings are not only fairs — lead with standing senior venues so
recall does not collapse onto county fairs.

**Lead with standing senior venues** (run these first, every county):

- `<county> Missouri senior center activities calendar 2026`
- `<largest city> MO senior living / 55+ community open house events`
- `<county> Missouri Area Agency on Aging events 2026`
- `<largest city> MO library senior program / older adult events`
- `<county> senior day OR 55+ expo OR resource fair OR health fair vendor`

**Then fairs / festivals** (cap ~2 queries per county — they over-return):

- `<county> County Missouri fair / festival 2026 vendor application`
- `<county> MO craft fair / farmers market vendor application`

**Then health walks** (region-wide, not per-county):

- `<region> Missouri 2026 charity health walk vendor/sponsor` — Walk to End
  Alzheimer's, AHA Heart Walk, Sista Strut (usually October), diabetes/cancer/
  stroke/kidney/heart walks.

First read `recurring-leads.csv` — these annual senior events are already known.
Skip re-searching them; if one falls in the run's 90-day window, copy it
straight to output instead of searching.

Also check recurring sources: county senior centers, Area Agencies on Aging,
libraries, hospitals/health systems, churches with senior ministries, Chambers
of Commerce event calendars, county fair boards, SHIP/CLAIM (MO Medicare info)
events, and city Parks & Rec 55+ programs.

## Step 3 — Qualify each event

KEEP an event only if BOTH are true:
1. **Senior-skewing audience** — expo, senior day/expo, health fair, resource
   fair, senior center event, 55+ dance, bingo, craft fair for seniors, county
   fair, farmers market, festival, church senior group, civic club (Rotary/
   Lions), library senior program, **charity health walks/runs** (Walk to End
   Alzheimer's, AHA Heart Walk, Sista Strut breast-cancer walk — often October;
   diabetes/cancer/stroke/kidney walks), and any **event supporting senior
   health issues** — anywhere the 65+ crowd shows up.
2. **Tabling opportunity** — the listing mentions (or clearly implies) vendor
   tables, exhibitor/booth space, sponsorship, or vendor applications. If it
   only *implies* (e.g. a large public expo), set `tabling_confirmed = implied`.

DROP: virtual-only events; events dated **fewer than 21 days from the run date**
(can't get approved in time) or **more than 120 days out**; private/invite-only
with no vendor access; and anything outside the territory counties.
Worked example: run date 2026-06-28 → keep only events with `date` on or after
**2026-07-19** (21 days out).

## Step 4 — Output: append to this week's CSV

Write to `events/YYYY-MM-DD-<REGION>.csv` when a region is specified (e.g.
`events/2026-06-25-CENTRAL.csv`), else `events/YYYY-MM-DD.csv`. Use today's run
date. One file per region keeps parallel regional runs from clobbering each
other. Columns:

```
date,end_date,event_name,event_type,venue,address,city,county,state,audience,tabling_confirmed,tabling_details,cost,contact_name,contact_email,contact_phone,registration_url,source_url,date_found,territory_status,notes
```

Rules:
- A row's `county` MUST belong to the region being run. Drop any row whose
  county is outside the run's region before writing. Never mix regions in one file.
- One row per event. **Dedupe** by venue + county + overlapping date range
  (NOT exact event_name — the same fair appears under variant names like
  "Montgomery County Fair" and "Montgomery County Fair 2026"). If two rows
  share a venue/county and their date ranges overlap, keep one.
  - **Exception:** a `senior day` / senior-specific sub-event of a larger fair
    (e.g. "Montgomery County Fair – Senior Citizens Day") is its OWN row, even
    though it shares the venue/dates. It is the prime Medicare-table day and
    must not be collapsed into the parent fair.
- `territory_status`: copy the county's `confidence` from territory.csv
  (`confirmed` | `verify`). `verify` = boundary county Jessica must confirm
  is hers before placing a broker.
- `date` / `end_date`: ISO `YYYY-MM-DD`. Single-day → leave `end_date` blank.
- `tabling_confirmed`: `yes` | `implied` | `unknown`.
- `tabling_details`: booth cost, table size, vendor deadline if stated.
- `source_url`: the page you found it on (required — no source, no row).
- `date_found`: the run date.
- Empty fields left blank, never guessed.
- ANY field containing a comma MUST be wrapped in double quotes (audience,
  tabling_details, notes routinely contain commas). Verify before writing.
- Trim leading/trailing whitespace from every field (no ` 636-931-7697`).
- Sort by `date` ascending.

**FINAL DATE CHECK (do this before saving — #1 mistake):** go row by row and
compare each `date` against the EARLIEST/LATEST cutoffs from Step 2. Delete any
row with `date` before EARLIEST (too soon — can't get approved) or after LATEST.
Do not skip this — a single sub-21-day row makes the whole file untrustworthy.

## Step 5 — Summary

After writing the CSV, print a short summary: total events found, count by
county, and the soonest 5 events with dates. Note any county that returned zero
results (may need a manual look).

## Guardrails

- **Real sources only.** Every row traces to a real URL you actually found. Do
  not invent events, dates, contacts, or venues. Blank > guess.
- This tool only *surfaces public opportunities*. It does not give CMS/Medicare
  marketing-compliance advice — whether an event is a permissible educational vs
  marketing/sales event is the broker's/AM's call.
