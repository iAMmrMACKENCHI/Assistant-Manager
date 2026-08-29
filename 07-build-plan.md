# Build Plan

Execute in this order. Each step has a definition of done — don't move to the next step until it's met.

## Step 0 — Read the pivot
Before scaffolding: TaskGrid is now a configurable Operations OS, not a fixed-schema tool. `01-data-model.md` and `03-workflow-engine.md` describe the core mechanics correctly, but their `workflow_templates`/`workflow_steps` tables are **superseded** by `09-metadata-schema-model.md`'s `work_types`/`work_type_fields`/`work_type_stages`. Build against `09`, not the older template tables.

## Step 1 — Scaffold
Create Next.js (App Router) + TypeScript project. Install `@supabase/supabase-js`, `@supabase/ssr`, `tailwindcss`, `lucide-react`. Set up `.env.local` with `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
**Done when:** project boots, Tailwind classes render.

## Step 2 — Database
Run the schema from `01-data-model.md` **plus** the metadata layer from `09-metadata-schema-model.md` (use `work_types`/`work_type_fields`/`work_type_stages`, not `workflow_templates`/`workflow_steps`) in the Supabase SQL editor or as migrations under `supabase/migrations/`. Enable RLS on every table.
**Done when:** tables exist and RLS is on (deny-by-default) even before policies are written.

## Step 3 — Auth & roles
Wire Supabase Auth (email/password is enough for MVP). On signup/invite, create the matching `users` row with `role`. Middleware or a server helper that reads the current user's role for use everywhere else.
**Done when:** a logged-in user's role is available server-side and client-side without an extra query per page.

## Step 4 — Permissions (RLS policies)
Write RLS policies per `02-roles-permissions.md` for every table. Employees: select/update scoped to their own `assignee_id`. Managers/Admins: broader per the matrix.
**Done when:** an employee querying `work_items` only ever gets rows where they're the assignee; a manager gets the org's full set.

## Step 5 — Workflow engine (RPC functions)
Implement each transition in `03-workflow-engine.md` as a Postgres RPC function (`assign_work_item`, `start_work_item`, `stop_work_item`, `submit_work_item`, `approve_work_item`, `reject_work_item`, `resume_work_item`, `reassign_work_item`, `return_to_stage`, `raise_urgent_flag`, `clear_urgent_flag`, `close_job`). Each function re-checks role, validates the from-status, mutates `work_items`, and writes an `events` row — atomically.
**Done when:** calling any function outside its allowed status/role fails cleanly; calling it correctly leaves both `work_items` and `events` updated in one transaction.

## Step 5b — Onboarding flow
Build the three-step onboarding in `08-onboarding-flow.md`: org creation, people (manual + Excel), and the "tell TaskGrid how you work" Work Type/Fields/Stages builder, including the system-template cloning (`system_templates` → org's own `work_types`).
**Done when:** a brand-new signup can go from empty account to "You're set up, create your first job" using only the built-in FTTB starter template, with zero manual database work.

## Step 6 — Manager dashboard
Build `/dashboard`. Port the structure of `taskgrid-manager-board.jsx` (provided prototype) onto live Supabase queries: fetch jobs + their work items, derive board columns from stages actually present (never the full template list), render per `04-manager-dashboard.md`. Wire the side panel's action buttons to the Step 5 RPCs. Add a Supabase Realtime subscription on `work_items` scoped to the org so the board updates live.
**Done when:** a manager can assign, reassign, approve, reject, and return-to-stage entirely from the board, and it updates without a page refresh when the underlying data changes.

## Step 7 — Employee dashboard
Build `/my-work` per `05-employee-dashboard.md`. Fetch only the logged-in user's assigned work items. Wire Start/Stop/Update/Submit/Resume to the Step 5 RPCs. Render the Update form's fields dynamically from `workflow_steps.field_schema` for that item's stage. Include the urgent toggle.
**Done when:** an employee can go from Assigned → Start → Update → Submit, and see a manager's rejection reason with a working Resume action — all on a single-column mobile layout.

## Step 8 — Job detail
Build the job drill-in: info panel, work items, documents, comments, and the activity timeline rendered from `events` (human-readable, chronological). Reachable from any board row.
**Done when:** every action taken in Steps 6–7 shows up correctly ordered in that job's timeline.

## Step 9 — Design system
Apply `06-design-system.md` tokens as CSS variables / Tailwind theme extension across both dashboards. Replace any placeholder colors from the prototype with the real brand-derived tokens.
**Done when:** status colors, type scale, and spacing match the spec exactly — no default Tailwind blues/greens left un-mapped.

## Step 10 — Excel import
Build the import layer described in the original product brief: accept an Excel file, map source columns (Arizona and Colorado formats differ) into the common `jobs`/`work_items` shape via a configurable column-mapping step, not a hardcoded parser per market.
**Done when:** both the Arizona-format and Colorado-format sample sheets import into the same internal structure without code changes between them.

## Step 11 — Seed & QA
Seed demo data resembling the FTTB examples (mixed workflows — some jobs short, some running the full Scheduling → Billing pipeline). Walk the MVP success criteria:
- Manager: create/import a job, choose work items, assign, watch it update live through start → submit → approve/reject → return-to-stage, read the full activity history.
- Employee: see assigned work, start, update, submit, and correctly handle a rejection.
**Done when:** every one of those flows works end to end against the real database, not mock data.

## Step 12 — Credits & instrumentation
Implement `10-monetization-credits.md`'s schema and the 25%/10%/0% warning states, and wire the `product_events` firing points from `12-experiment-tracking.md` at every listed moment (signup, first workflow, first job, invites, credit thresholds, referrals). Founder/product-intelligence dashboard can be a simple internal-only page reading from these tables — no polish needed yet.
**Done when:** a full trial run (signup → onboarding → invite → job → exhaust free credits → see the 0% state) never blocks access to existing data, and every `product_events` row in the table above fires at the right moment.
