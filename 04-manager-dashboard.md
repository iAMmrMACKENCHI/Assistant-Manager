# Manager Dashboard

Desktop-first. This is the primary interface — a manager should read the state of the whole operation in a few seconds. A working reference prototype (mock data, interactive) is provided as `taskgrid-manager-board.jsx` — port its structure to real data per this spec rather than starting from scratch.

## Layout
1. **Header** — product name + a small "by SIE" mark, using the brand colors from `06-design-system.md`.
2. **Summary row** — four small numbers, not cards that dominate the screen: **Active** (green count), **Waiting** (gold count), **Delayed** (red/stalled count), **Needs Action** (blue submitted + red rejected). These are glanceable, not a BI widget.
3. **Flow board** — the main surface. Rows = jobs. Columns = the union of stages actually in use across the visible jobs (not a fixed master list — see `03-workflow-engine.md`). Cell = status dot/check for that job's stage, or a muted `—` if that job doesn't have that stage.
4. **Legend** — small, at the bottom, not competing with the board.

## Cell interactions
Clicking any real cell (not a `—`) opens a right-side panel:
- Job ID, stage name
- Current status + who's assigned
- Started time / last update
- Rejection or return-to-stage reason, if present (highlighted, not buried)
- **Urgent badge** if `urgent = true`, with a way to clear it
- Contextual action buttons — only the ones valid for the current status per the transition table in `03-workflow-engine.md`:
  - `not_started` → Assign
  - `assigned` → Start on behalf, Reassign
  - `in_progress` → Stop, Reassign
  - `submitted` → Approve, Reject
  - `rejected` → Reassign, view reason
  - `approved` → view only (+ comment)
- Comment box, always available

## Assigning into a stage with no prior work items
Clicking a `—` cell (a stage not yet part of that job) opens the same panel in an "Assign" state — picking an employee here creates the work item at that stage directly, per transition #1 in `03-workflow-engine.md`. No requirement to fill in earlier stages first.

## Urgent flags
A work item with `urgent = true` gets a small distinct indicator on its cell (e.g. a dot in the corner) that persists regardless of the cell's status color — a green "in progress" cell with an urgent flag still needs to visually stand out from a normal green cell.

## Job detail (drill-in from a row)
Job info (customer, market, address, priority) → current work items → assignees → documents → comments → **activity timeline** (every event from the `events` table, chronological, human-readable — "09:14 Site Survey started by Scott," etc.). The timeline is not optional — it's the audit trail managers will actually rely on.

## Realtime
Subscribe to `postgres_changes` on `work_items` (and `events`, for the timeline) scoped to the org. An employee starting or submitting work should update the board with no manual refresh.
