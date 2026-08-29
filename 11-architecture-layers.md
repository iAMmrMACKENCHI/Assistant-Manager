# Architecture — Target Layers & MVP Scope

```
                 TASKGRID
                     │
             ┌───────┴───────┐
             │   AI COPILOT  │   ← not built yet
             └───────┬───────┘
                     │
             ┌───────┴───────┐
             │ KNOWLEDGE LAYER│   ← not built yet
             └───────┬───────┘
                     │
             ┌───────┴───────┐
             │ WORKFLOW ENGINE│   ← build now (03-workflow-engine.md, driven by work_type_stages)
             └───────┬───────┘
                     │
             ┌───────┴───────┐
             │  DATA / EVENTS │   ← build now (01 + 09-data-model.md)
             └───────┬───────┘
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    Manager       Engineer      External
      UI             UI          Systems
   (build now)   (build now)    (stub only)
```

## What "build now" actually means
- **Data/Events** and **Workflow Engine**: the full RPC-driven state machine, org-configurable via `work_types`/`work_type_fields`/`work_type_stages`. This is the whole product for MVP.
- **Manager UI / Engineer UI**: per `04-manager-dashboard.md` / `05-employee-dashboard.md`.
- **External Systems**: interfaces only — e.g. an `import_jobs(source, mapping)` function signature and an `integrations` table (`org_id`, `provider`, `config jsonb`, `connected boolean`) that exists but has no real connector wired to it yet. This keeps the door open without spending build time on Microsoft 365/Salesforce/ERP now.

## What "not built yet" means precisely
Don't build a UI, a model call, or a scoring algorithm for these. **Do** keep the data they'd need flowing correctly from day one:

- **Knowledge Layer** — no clustering, no "problem → cause → resolution" auto-tagging. But `events.payload`, `work_items.note` (rejection/return reasons), and `comments.body` should already be structured enough (typed event `type`, free-text `reason` fields) that this layer can be built later without a data migration.
- **AI Copilot** — no chat interface, no suggestions, no automation. Nothing to build; just don't paint yourself into a corner where a future copilot can't read the event log cleanly.

## Guardrail
If a feature request doesn't clearly belong to Data/Events, Workflow Engine, or the two UIs — it's Knowledge Layer or AI Copilot territory, and it waits. This list is the scope fence for the MVP; refer back to it before adding anything new.
