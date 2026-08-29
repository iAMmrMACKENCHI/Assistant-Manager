# Data Model

Keep **Job** and **Work Item** separate. A Job can contain any number of Work Items, and its set of Work Items is decided per-job (from a template, or built ad hoc by a manager) — never a fixed universal pipeline.

## Entities
`Organization → User → Role`, `WorkflowTemplate → WorkflowStep`, `Job → WorkItem → Assignment`, `Event`, `ActivityUpdate`, `Approval`, `Comment`, `Document`.

## Schema (Postgres / Supabase)

```sql
create extension if not exists "pgcrypto";

create table organizations (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  created_at timestamptz default now()
);

create table users (
  id uuid primary key references auth.users(id),
  org_id uuid references organizations(id),
  name text not null,
  email text not null,
  role text not null check (role in ('admin','manager','employee')),
  created_at timestamptz default now()
);

-- Reusable pipelines, e.g. "FTTB Standard"
create table workflow_templates (
  id uuid primary key default gen_random_uuid(),
  org_id uuid references organizations(id),
  name text not null,
  description text
);

-- Master list of possible stages for a template. A job only uses a subset.
create table workflow_steps (
  id uuid primary key default gen_random_uuid(),
  template_id uuid references workflow_templates(id),
  key text not null,              -- 'site_survey', 'nds_design', etc.
  label text not null,
  sort_order int not null,
  parallel_group text,            -- steps sharing a group can run concurrently (e.g. NDS + Layout)
  field_schema jsonb default '{}' -- role-specific update fields, e.g. {"splice_count": "number", "footage": "number"}
);

create table jobs (
  id text primary key,            -- human-facing code, e.g. 'AZ-1234'
  org_id uuid references organizations(id),
  template_id uuid references workflow_templates(id),
  customer text,
  market text,
  address text,
  priority text default 'normal',
  status text not null default 'open' check (status in ('open','closed')),
  created_by uuid references users(id),
  created_at timestamptz default now()
);

-- The actual unit of work on a job. Only rows that exist are "in play" —
-- absence of a row for a given step means that step is not part of this job.
create table work_items (
  id uuid primary key default gen_random_uuid(),
  job_id text references jobs(id) on delete cascade,
  step_key text not null,
  label text not null,
  status text not null default 'assigned'
    check (status in ('not_started','assigned','in_progress','submitted','approved','rejected')),
  assignee_id uuid references users(id),
  urgent boolean default false,
  urgent_note text,
  started_at timestamptz,
  submitted_at timestamptz,
  approved_at timestamptz,
  last_update_at timestamptz default now(),
  note text,                       -- rejection reason / general note
  created_at timestamptz default now()
);

-- History of who has held a work item (supports reassignment audit trail)
create table assignments (
  id uuid primary key default gen_random_uuid(),
  work_item_id uuid references work_items(id) on delete cascade,
  user_id uuid references users(id),
  assigned_by uuid references users(id),
  assigned_at timestamptz default now(),
  unassigned_at timestamptz
);

-- Daily/per-session work log entries employees submit
create table activity_updates (
  id uuid primary key default gen_random_uuid(),
  work_item_id uuid references work_items(id) on delete cascade,
  user_id uuid references users(id),
  date date default current_date,
  hours_spent numeric,
  notes text,
  metrics jsonb default '{}',      -- role-specific: {"splice_count": 40, "footage": 1200}
  created_at timestamptz default now()
);

-- Immutable log. Every meaningful action writes here. This is the timeline.
create table events (
  id uuid primary key default gen_random_uuid(),
  job_id text references jobs(id) on delete cascade,
  work_item_id uuid references work_items(id) on delete set null,
  type text not null,              -- 'created','assigned','started','paused','submitted',
                                    -- 'approved','rejected','reassigned','returned_to_stage',
                                    -- 'urgent_flag_raised','urgent_flag_cleared','closed'
  actor_id uuid references users(id),
  payload jsonb default '{}',      -- free-form context, e.g. {"from_stage": "...", "reason": "..."}
  created_at timestamptz default now()
);

create table approvals (
  id uuid primary key default gen_random_uuid(),
  work_item_id uuid references work_items(id) on delete cascade,
  decided_by uuid references users(id),
  decision text not null check (decision in ('approved','rejected')),
  reason text,
  decided_at timestamptz default now()
);

create table comments (
  id uuid primary key default gen_random_uuid(),
  job_id text references jobs(id) on delete cascade,
  work_item_id uuid references work_items(id) on delete set null,
  author_id uuid references users(id),
  body text not null,
  mentions uuid[] default '{}',
  created_at timestamptz default now()
);

create table documents (
  id uuid primary key default gen_random_uuid(),
  job_id text references jobs(id) on delete cascade,
  work_item_id uuid references work_items(id) on delete set null,
  uploaded_by uuid references users(id),
  filename text,
  url text,
  created_at timestamptz default now()
);
```

## Notes
- `work_items.status` is the six-state machine described in `03-workflow-engine.md` — treat it as the single source of truth for the flow board's colors.
- `field_schema` on `workflow_steps` drives which activity-update fields an employee sees (Splice Count / Footage for NDS/Layout/TCP, GLM/WFMT for SSP QC, etc.) — don't hardcode role fields in the UI, read them from this.
- A job's active stages = `select distinct step_key from work_items where job_id = ?`. Never derive the board columns from the template's full step list.
