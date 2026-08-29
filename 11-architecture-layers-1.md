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

## Data storage principle
Native storage is primary, not optional. Every table in `01-data-model.md` is owned by the app in Supabase Postgres — the platform is fully functional with **zero** external connections. External sources (Excel now, APIs/MS365/Salesforce/ERP later) are one-way ingestion into that native store via `integrations` + `import_batches`, never a live dependency the app reads from at request time. This is why Excel import works day one without waiting on any "real" integration to be built.

## What "build now" actually means
- **Data/Events** and **Workflow Engine**: the full RPC-driven state machine, org-configurable via `work_types`/`work_type_fields`/`work_type_stages`. This is the whole product for MVP.
- **Manager UI / Engineer UI**: per `04-manager-dashboard.md` / `05-employee-dashboard.md`.
- **External Systems**: ingestion only, and it's fully defined now — the `integrations` and `import_batches` tables in `01-data-model.md`. Excel import (`10-monetization-credits.md`'s onboarding, `07-build-plan.md` Step 12) uses `import_batches` with `source = 'excel'` and no `integration_id` at all. A real connector (MS365, Salesforce, ERP) is just a future `integrations` row with `connected = true` and a scheduled job writing into the same `import_batches` pipeline — the schema doesn't change when that gets built.

## What "not built yet" means precisely
Don't build a UI, a model call, or a scoring algorithm for these. **Do** keep the data they'd need flowing correctly from day one:

- **Knowledge Layer** — no clustering, no "problem → cause → resolution" auto-tagging. But `events.payload`, `work_items.note` (rejection/return reasons), and `comments.body` should already be structured enough (typed event `type`, free-text `reason` fields) that this layer can be built later without a data migration.
- **AI Copilot** — no chat interface, no suggestions, no automation. Nothing to build; just don't paint yourself into a corner where a future copilot can't read the event log cleanly.

## Guardrail
If a feature request doesn't clearly belong to Data/Events, Workflow Engine, or the two UIs — it's Knowledge Layer or AI Copilot territory, and it waits. This list is the scope fence for the MVP; refer back to it before adding anything new.
