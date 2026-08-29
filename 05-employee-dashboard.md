# Employee Dashboard

Mobile-first, single column, one clear call-to-action per job at a time. No analytics, no workflow controls, no admin surface — an employee should never see a screen with more than one obvious next action.

## Home screen — "Today's Work"
List of the employee's own assigned work items, most urgent/active first:

```
AZ-1234
Site Survey
Assigned Today
[ START ]
```

```
AZ-1277
Inspection
Due Tomorrow
[ VIEW ]
```

Sections, in order: **In Progress** (anything they've started), **Assigned / Waiting**, **Returned / Rejected** (needs their attention — show these near the top, not buried at the bottom, since they represent required action).

## In progress
```
AZ-1234
Site Survey
IN PROGRESS · Started 09:14

[ UPDATE ]
[ STOP / PAUSE ]
[ SUBMIT ]
```

## Update
Simple form, not a general-purpose builder. Base fields: Hours Spent, Notes. Role-specific fields come from `workflow_steps.field_schema` for that stage — e.g. NDS Design / Layout / TCP show Splice Count + Footage; SSP QC shows GLM + WFMT. **Only render fields relevant to that employee's current stage** — never show every possible field to everyone.

Include an **Urgent** toggle on the update form — off by default. When on, the update is flagged for the manager dashboard immediately (transition #10 in `03-workflow-engine.md`) and the employee should see a short confirmation that it's been flagged, not just a generic "saved."

## Rejected / Returned
```
AZ-1234
Site Survey
NEEDS ATTENTION

"Missing pole photos — please reshoot bays 3-4."
— Dana, 40 min ago

[ RESUME ]
```
The reason is always shown, never hidden behind a tap. Resume moves it back to `in_progress` (transition #7) so the employee can rework and resubmit.

## What never appears here
Assignment controls, reassignment, approve/reject buttons, other employees' work, workflow configuration, analytics/charts. If a screen would need any of these, it belongs on the manager dashboard, not here.
