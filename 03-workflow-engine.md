# Workflow Engine — States & Transitions

This is the core logic. Every other screen is a view onto this state machine.

## The six states (this is also the color language — see `06-design-system.md`)

| Status | Color | Meaning |
|---|---|---|
| `not_started` | Grey | Work item exists on the job but hasn't been assigned/started yet |
| `assigned` | Gold | Someone is assigned; hasn't started (or has been paused) |
| `in_progress` | Green | Actively being worked |
| `submitted` | Blue | Employee released it; waiting on manager/QC decision |
| `approved` | Green check | Signed off. Effectively read-only from here |
| `rejected` | Red | Sent back; needs rework, with a reason attached |

A work item that simply doesn't exist on a job is **not** a state — it renders as `—` (not applicable) and is never treated as "pending."

## Transitions

| # | Action | From | To | Who | What happens |
|---|---|---|---|---|---|
| 1 | **Assign** | (no row) | `assigned` | Manager/Admin | Creates the `work_items` row for that job + step. Doesn't require any other step to exist or be complete — this is how a manager can hand a job straight to NDS Design with no Scheduling/Site Survey ever created. Event: `assigned`. |
| 2 | **Start** | `assigned` | `in_progress` | Employee (own) or Manager (on behalf) | Sets `started_at`. Event: `started`. |
| 3 | **Stop / Pause** | `in_progress` | `assigned` | Employee (own) or Manager (on behalf) | Resumable — Start again picks it back up. Event: `paused`. |
| 4 | **Submit / Release** | `in_progress` | `submitted` | Employee (own only) | Sets `submitted_at`. Event: `submitted`. |
| 5 | **Approve** | `submitted` | `approved` | Manager/Admin | Sets `approved_at`. Writes an `approvals` row. Event: `approved`. |
| 6 | **Reject** | `submitted` | `rejected` | Manager/Admin | **Requires a reason** (stored on `work_items.note` and the `approvals` row). Event: `rejected`. |
| 7 | **Resume after rejection** | `rejected` | `in_progress` | Employee (own) or Manager (on behalf) | Same employee reworks the *same* stage. Event: `started` (payload notes `resumed_from: rejected`). |
| 8 | **Reassign** | any status | unchanged | Manager/Admin | Only `assignee_id` changes. Closes the current `assignments` row (`unassigned_at`), opens a new one. Event: `reassigned`. |
| 9 | **Return to previous stage** | any status | see below | Manager/Admin only | See dedicated section below. |
| 10 | **Raise urgent flag** | `assigned` or `in_progress` | unchanged | Employee (own) | Sets `urgent = true` + `urgent_note`. Event: `urgent_flag_raised`. |
| 11 | **Clear urgent flag** | any | unchanged | Manager/Admin | Sets `urgent = false`. Event: `urgent_flag_cleared`. |
| 12 | **Close job** | job-level | `jobs.status = closed` | Manager/Admin | Only allowed with no `submitted` or `rejected` items outstanding, unless force-closed with a confirmation + reason. |

## Reject vs. Return to previous stage — these are different actions

It's tempting to conflate these. Don't.

- **Reject** = *this stage's own output is wrong.* Same employee, same stage, redo it. (`submitted → rejected → in_progress`, no other work item touched.)
- **Return to previous stage** = *a downstream check found the problem actually originates earlier in the job.* Example: QC on Routing discovers the NDS Design itself was wrong. The current stage can't proceed until the earlier one is redone.

**Return to previous stage behavior:**
1. Manager picks the target stage from the job's existing (or template) step list.
2. If a work item for that stage already exists, it's reopened: status → `assigned` (same or reassigned employee).
3. If it doesn't exist yet, a new work item is created at that stage (status `assigned`) — same as a fresh Assign.
4. The stage the manager returned *from* goes to `not_started` (its downstream work is void until the upstream fix lands) — it does not stay `in_progress`.
5. A required reason is logged as an event on **both** work items: `returned_to_stage` (payload includes `from_stage`, `to_stage`, `reason`).
6. The employee on the returned-to stage sees this on their dashboard like any other assignment — plus the reason, surfaced the same way a rejection reason is.

## Urgent updates

An employee can flag an update as urgent from their own work item (a toggle when they hit Update) — for something that needs manager eyes *now*, independent of the stage's normal status. This doesn't change the status color; it adds a distinct badge (see `06-design-system.md`) that stays visible on the manager's flow board and job detail until a manager explicitly clears it. Don't let a normal status change (e.g., Submit) silently clear an urgent flag — clearing is a manager action.

## Manager assigning directly into the middle of a pipeline

This is the default, not a special case. A job's work items are whatever the manager assigns — there's no requirement to "complete" Scheduling before Site Survey exists, or to touch Site Survey at all before assigning NDS Design. The board only ever renders columns for stages that have an actual `work_items` row on that job (see `01-data-model.md`). When a manager assigns a stage directly, the assigned employee's dashboard shows exactly that job at exactly that stage — full job context (customer, address, priority), nothing about "missing" earlier stages, because nothing is missing; those stages simply aren't part of this job.

## Implementation note

Implement each transition as a Postgres RPC function (`assign_work_item`, `start_work_item`, `submit_work_item`, `approve_work_item`, `reject_work_item`, `return_to_stage`, etc.), each one: (a) re-checks the caller's role server-side, (b) validates the current status allows the transition, (c) updates `work_items`, (d) inserts the `events` row — all inside one transaction. Never let the client set `work_items.status` directly via a raw update; that bypasses the audit trail and the permission check.
