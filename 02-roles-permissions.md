# Roles & Permissions

Three roles: **Employee**, **Manager**, **Admin**. Admin is a superset of Manager. UI density scales with the role — Admin sees the most, Employee sees the least.

| Action | Employee | Manager | Admin |
|---|:---:|:---:|:---:|
| Create job | ✕ | ✓ | ✓ |
| Import job (from Excel) | ✕ | ✓ | ✓ |
| Choose/add work items for a job | ✕ | ✓ | ✓ |
| Assign work item (to any stage, any time) | ✕ | ✓ | ✓ |
| Reassign work item | ✕ | ✓ | ✓ |
| Start own assigned work | ✓ | — | — |
| Stop/pause own assigned work | ✓ | — | — |
| Start/stop work **on behalf of** an employee | ✕ | ✓ | ✓ |
| Update own work item (activity update) | ✓ (own only) | ✓ (own only) | ✓ (own only) |
| Submit/release own work | ✓ | — | — |
| Raise urgent flag on own work item | ✓ | — | — |
| Approve submitted work | ✕ | ✓ | ✓ |
| Reject submitted work | ✕ | ✓ | ✓ |
| Return work item to a previous stage | ✕ | ✓ | ✓ |
| View own assigned work | ✓ | ✓ | ✓ |
| View all jobs / full board | ✕ | ✓ | ✓ |
| View employee workload | ✕ | ✓ | ✓ |
| Add comment | ✓ (own work items) | ✓ (any) | ✓ (any) |
| Upload document | ✓ (own work items) | ✓ (any) | ✓ (any) |
| Close job | ✕ | ✓ | ✓ |
| Delete job | ✕ | ✕ | ✓ |
| Manage users / invite | ✕ | ✕ | ✓ |
| Manage roles | ✕ | ✕ | ✓ |
| Manage workflow templates | ✕ | ✕ | ✓ |
| Manage org settings / integrations | ✕ | ✕ | ✓ |

## Enforcement
Enforce this **twice**: in the UI (hide actions that aren't available — don't just disable them) and in Supabase Row Level Security (RLS), which is the actual security boundary. Example RLS shape for `work_items`:

```sql
-- Employees can only update work items assigned to them, and only specific columns
-- (status transitions handled via RPC functions, not raw UPDATE — see 03-workflow-engine.md)
create policy employee_view_own on work_items
  for select using (assignee_id = auth.uid());

create policy manager_view_all on work_items
  for select using (
    exists (select 1 from users u where u.id = auth.uid() and u.role in ('manager','admin'))
  );
```

Never trust client-side role checks alone — every write path (especially status transitions) goes through the RPC functions defined in `03-workflow-engine.md`, which re-verify role server-side before mutating anything.
