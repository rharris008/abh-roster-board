# ABH Shift Roster Board

Single-file drag-and-drop roster board for ABH Pureau production at Erskine Park.
Three shifts, 17 roles each, 51 slots total.

Live: https://rharris008.github.io/abh-roster-board/

## Data protection

**No staff data is in this repository.** `index.html` contains zero names. People and
assignments are fetched from Supabase at runtime.

The publishable key in the page source is meant to be public. Protection is enforced
server side, not by hiding the key:

| Table | anon SELECT | anon INSERT | anon UPDATE | anon DELETE |
|---|---|---|---|---|
| `roster_people` | yes, `active` rows only | no (401) | `emp_type` column only | no |
| `roster_assignments` | yes | yes | yes | yes |

`roster_people` holds `id`, `name`, `line`, `cohort`, `emp_type`, `active`, `sort_order`.
It has **no email, phone, address, pay or HR column, and none may ever be added.**
Anyone who opens the page can see the 43 names and which line they work. That is the
accepted exposure. Nothing beyond that is reachable with this key.

The column-level grant means a visitor can set someone's employment type but cannot
rename a person, move them to another line, deactivate them, or add or remove anyone.
Verified: `PATCH {"name":...}` returns 401, `PATCH {"emp_type":...}` returns 204.

`roster_assignments` is deliberately writable, because the board is a shared working
tool. The worst a visitor can do is shuffle the roster, which is fully recoverable.

### Before adding any column

If you ever need a field that is not safe to publish, put it in a separate table with
no anon policy and join it server side. Do not add it to `roster_people`.

## Roles

| Area | Count | Roles |
|---|---|---|
| Shift Management | 1 | Shift Leader |
| HMPS | 3 | HMPS Operator, HMPS Pack Off x2 |
| Line A / B | 4 | Line A Operator, Line B Operator, Line AB Pack Off / Overpacker x2 |
| Bottle Line | 5 | Bottle Filler, Filling Machine Operator, Labeller / Shrink Operator, Bottle Pack Off x2 |
| Forklift | 3 | Production Feed Forklift, Pallet Wrap Forklift x2 |
| Float | 1 | 2L Overpacker |

## Using it

- **Desktop:** drag a card from the pool into a role slot. Drag between slots to move
  someone. Drag back to the pool, or press the red x, to unassign.
- **Phone:** tap a person to select, then tap a vacant slot to place them.
- **Employment type:** click the type chip on any card to cycle
  not confirmed, permanent, temp, contractor. It saves for everyone.
- **Pool filter:** defaults to "Production floor". Search by name or line.
- **Refresh** re-reads Supabase. **Clear shift** empties one shift.
- **Export CSV** produces a UTF-8 BOM CSV of all 51 slots. **Print** drops the pool panel.

Every drag saves immediately. The lamp in the header reads Live, Saving, or Offline.
A failed save rolls the board back rather than showing a state that was not stored.

## Where the people came from

Seeded from a live Workforce Ready pull on 02/09/2026 via
`~/.claude/tools/_output/workforce_ready_integration.py`.

- `line` is the most frequent cost centre across that person's last 28 days of time entries
- `cohort` is their WFR work-schedule grouping
- `emp_type` starts as `unknown` for everyone

Linus Velso is not in WFR (contractor) and was added manually.
Richard Magin was excluded: he left on 14/04/2026 but WFR has no termination date set.

### Two known gaps

1. **Shift names are unknown.** WFR returns a work-schedule ID per employee but not its
   name: `/config/work-schedules/{id}` returns 404 and the company `/schedules` endpoint
   returns 403. The floor splits into two cohorts, labelled `Sched A` (11 people) and
   `Sched B` (8 people). Which is Day and which is Afternoon has to be read off the WFR
   admin console. IDs: 17781778, 17781779, 17788555, 18326475, 18326481, 18326492,
   18326494, 18326495.
2. **Employment type is unavailable from WFR.** Permanent / temp / contractor is not on
   any working endpoint, so it is set by hand in the board and stored in Supabase.

## Re-seeding after a headcount change

Re-run the WFR pull and upsert into `roster_people` using the service role, which
bypasses RLS. The page never writes to that table beyond `emp_type`.

## Brand

Arial throughout. Navy `#1B2A4A`, Pureau Blue `#0E7CCE`, Red `#C0392B`, Amber `#E67E22`,
Green `#27AE60`. Light and dark themes both defined.
