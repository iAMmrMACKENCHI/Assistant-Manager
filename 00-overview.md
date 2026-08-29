# TaskGrid — Build Spec

TaskGrid is a **configurable Operations OS**, built by System Intelligence Engineers (SIE). It is not a project-management app, not a Kanban board, and not a fixed telecom/construction tool — FTTB is the first template, not the product. Every organization defines its own Work Types, Fields, and Workflow during onboarding (`08-onboarding-flow.md`); the platform stays the same underneath (`09-metadata-schema-model.md`).

Purpose: give a manager an instant visual read on every job — where it sits in its workflow, who's on it, and where attention is needed right now — while employees see only "what do I need to do." Business model: free signup → org configures its own operations → team adopts it → usage-based credits fund the ongoing platform (`10-monetization-credits.md`). See `11-architecture-layers.md` for how this stays extensible toward AI/Knowledge layers without building them yet.

Read these files in order. Each one is a complete, standalone reference — implement in the order given in `07-build-plan.md`.

| File | Covers |
|---|---|
| `01-data-model.md` | Entities + Postgres/Supabase schema |
| `02-roles-permissions.md` | What each role can/can't do |
| `03-workflow-engine.md` | Statuses, transitions, who can trigger what — the core logic |
| `04-manager-dashboard.md` | Flow board UI spec |
| `05-employee-dashboard.md` | Mobile-first employee UI spec |
| `06-design-system.md` | Colors, type, spacing (SIE brand-derived) |
| `07-build-plan.md` | Step-by-step implementation order |
| `08-onboarding-flow.md` | How an org configures itself on signup |
| `09-metadata-schema-model.md` | Configurable Work Types/Fields/Stages — supersedes fixed template framing in `01`/`03` |
| `10-monetization-credits.md` | Credit system, warnings, referrals, founder dashboard |
| `11-architecture-layers.md` | Target 6-layer architecture, MVP scope fence |
| `12-experiment-tracking.md` | What to instrument now for pricing decisions later |

## Tech stack
Next.js (App Router) + TypeScript, Supabase (Postgres + Auth + Realtime), Tailwind CSS. Mobile-first for employees, desktop-first for managers/admins. Architecture should leave room for later Microsoft 365 (Excel/Outlook/Teams/SharePoint) connectors — don't hard-couple core logic to any one integration.

## Non-negotiable principles
1. **A job doesn't require a fixed pipeline.** Some jobs are one stage. Some skip straight to a middle stage. Never render an unused stage as "pending" — it simply isn't shown for that job.
2. **Employees don't manage workflow.** They start, pause, update, and submit *their own* assigned work. Nothing else.
3. **Managers/Admins control structure.** Assignment, reassignment, approval, rejection, and moving work backward are manager-only actions.
4. **Every state change is an event.** Never overwrite status silently — append to an immutable event log.
5. **Attention-based hierarchy.** Surface what needs action; fade what's done. The UI should be understandable in a few seconds, not requiring analysis.
6. **Don't build yet:** AI copilot, predictive scoring, native mobile apps, full MS365 integration, workflow marketplace. Build the operational foundation first.

## Out of scope for this pass
Multi-tenant billing, SSO, native push notifications (use in-app + email stub), full MS365 sync (leave connector interfaces stubbed).
