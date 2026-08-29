# Metadata Schema Model

Read alongside `01-data-model.md` — this file replaces that file's `workflow_templates` / `workflow_steps` framing with a per-organization, configurable version. Don't run both systems; this supersedes them.

## Why
Every org defines its own Work Types, Fields, and Stages during onboarding (`08-onboarding-flow.md`). The core database schema must **not** change per customer — the customer's schema is data, not DDL.

## Schema additions

```sql
-- System-level starter templates (not org-scoped) — what onboarding clones from
create table system_templates (
  id uuid primary key default gen_random_uuid(),
  name text not null,              -- 'FTTB Installation', 'Solar Installation', 'Field Service'
  fields jsonb not null,           -- array of field defs, same shape as work_type_fields rows
  stages jsonb not null            -- array of stage defs, same shape as work_type_stages rows
);

-- An org's own configurable work type (cloned from a system_template, or built from scratch)
create table work_types (
  id uuid primary key default gen_random_uuid(),
  org_id uuid references organizations(id),
  name text not null,
  cloned_from uuid references system_templates(id),
  is_active boolean default true,
  created_at timestamptz default now()
);

-- Dynamic fields captured on a job of this work type
create table work_type_fields (
  id uuid primary key default gen_random_uuid(),
  work_type_id uuid references work_types(id) on delete cascade,
  key text not null,               -- stable identifier, e.g. 'fdp' — used as the jsonb key on jobs.field_values
  label text not null,             -- display label, e.g. 'FDP'
  field_type text not null check (field_type in ('text','number','date','select','photo','document')),
  options jsonb default '[]',      -- choices, for 'select' type
  required boolean default false,
  sort_order int not null
);

-- Ordered stages for this work type — replaces workflow_steps
create table work_type_stages (
  id uuid primary key default gen_random_uuid(),
  work_type_id uuid references work_types(id) on delete cascade,
  key text not null,
  label text not null,
  sort_order int not null,
  parallel_group text,                       -- shared value = can run concurrently (data-model support; not exposed in onboarding UI yet)
  assignable_role text default 'employee',
  required_field_keys text[] default '{}'    -- must be non-null on jobs.field_values before this stage's work item can be Submitted
);
```

## Changes to `jobs` (from `01-data-model.md`)

```sql
alter table jobs add column work_type_id uuid references work_types(id);
alter table jobs add column field_values jsonb default '{}';
-- e.g. {"customer": "ABC", "project_id": "1458", "address": "Phoenix", "fdp": "FDP-22"}
```

Drop `template_id` in favor of `work_type_id`. `work_items.step_key` now resolves against `work_type_stages.key` scoped to the job's `work_type_id`, instead of a generic template chain.

## Enforcement, not just convention
- **Submit** transition (`03-workflow-engine.md`, #4) must check `work_type_stages.required_field_keys` against `jobs.field_values` server-side, inside the RPC — not just as a client-side form validator. A job missing a required field for that stage should not be submittable, full stop.
- Use `jsonb` for `field_values` rather than an EAV table (`job_field_key`, `job_field_value` rows) at this scale — it's simpler to read/write and Postgres's `jsonb` indexing (`GIN`) is enough for MVP query needs. Revisit only if cross-job field querying/reporting becomes a real bottleneck.

## Customer-facing naming (don't leak internal terms)
| Internal | Customer-facing |
|---|---|
| `organizations` | Organization |
| `users` | People |
| `work_types` | Work Types |
| `work_type_stages` (ordered) | Workflow |
| `work_type_fields` | Fields |
| `work_items.status` / current stage | Stages (as in "where is this job") |
| future SLA/automation config | Rules |
| `users.role` + `02-roles-permissions.md` matrix | Roles |

Never show the word "schema," "template ID," or "jsonb" in the UI. Managers configure "Fields" and "Workflow," not a data model.
