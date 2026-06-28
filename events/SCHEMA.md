# events/ — output folder

Each regional run writes `YYYY-MM-DD-<REGION>.csv` (e.g. `2026-06-25-EAST.csv`).
A no-region run writes `YYYY-MM-DD.csv`. One file per region — runs never share a file.

## Columns

| column | meaning |
|---|---|
| date | event start, `YYYY-MM-DD` |
| end_date | event end if multi-day, else blank |
| event_name | name of event |
| event_type | expo / health fair / senior day / craft fair / festival / fair / health walk / civic / faith / other |
| venue | building or place |
| address | street address if listed |
| city | city |
| county | MO county (matches territory.csv) |
| state | MO |
| audience | who attends (seniors, general public, 55+, etc.) |
| tabling_confirmed | `yes` / `implied` / `unknown` |
| tabling_details | booth cost, table size, vendor deadline |
| cost | vendor/booth cost if stated |
| contact_name | event organizer |
| contact_email | organizer email |
| contact_phone | organizer phone |
| registration_url | vendor signup / event page |
| source_url | page the event was found on (required) |
| date_found | run date |
| territory_status | `confirmed` or `verify` (from territory.csv confidence) |
| notes | anything useful |

Open in Excel or Google Sheets. Filter `tabling_confirmed = yes`, sort by `date`.
