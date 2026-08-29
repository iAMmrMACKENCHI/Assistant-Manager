# Onboarding Flow

This is the actual first-run experience. It replaces "pick your industry" with "teach TaskGrid how you work." Three steps, no integrations, no sales call.

## Step 1 — Organization
```
Assistant Manager
Your company's operational brain.

[ Start Free ]   [ Sign In ]
```
Signup form: Name, Work email, Company name, Password. No credit card.

```
Welcome to TaskGrid. Let's set up how your company works.

Company Name    [___________]
Industry        [___________]  (free text, not a picklist — don't gate on this)
Country / Timezone
```
Creates `organizations` row + the signing-up user as `role = 'admin'`.

## Step 2 — People
```
Add your team

[ Add manually ]   [ Upload Excel ]
```
Manual: Name, Email, Department (free text), Role (admin / manager / employee).
Excel: same four columns, mapped via the import layer (`01-data-model.md` / existing scheduling-import logic) — don't require an exact column-name match, map loosely (fuzzy header match + manual confirm step if ambiguous).

Each row creates a `users` invite. Sending the invite is what will later consume an Employee Credit (`10-monetization-credits.md`) — but **don't charge or block at this step**; invites during onboarding come out of the free starter balance.

## Step 3 — "Tell TaskGrid how you work"
This is the screen that replaces integrations as the onboarding centerpiece.

```
What kind of work do you manage?

[ FTTB Installation ]   [ Solar Installation ]   [ Field Service ]   [ Start from scratch ]
```
Picking a starter option **clones a system-level template** (see `09-metadata-schema-model.md`) into the org's own editable `work_type` — pre-filled fields and stages, not a blank canvas. "Start from scratch" gives an empty Work Type with one starter stage to edit.

**3a — Name it**
```
Work Type name: [ FTTB Installation ]
```

**3b — What information do you need?**
```
Fields for this work type

Customer          Text      Required
Project ID        Text      Required
Address           Text      Required
Priority          Select    (Low / Normal / High)
FDP               Text
MMP               Text
POE               Text

[ + Add field ]
```
Field types: text, number, date, select, photo, document. Cap at ~12 fields in the onboarding builder — if they need more, that's a signal for a follow-up conversation, not a reason to expand the UI.

**3c — What happens next?**
```
Request  →  Survey  →  Design  →  QC  →  Customer Approval  →  Permit  →  Construction  →  Billing  →  Closed

[ + Add stage ]        [ drag to reorder ]        [ assign a role to each stage ]
```
Each stage: label + which role normally works it (employee role, not a named person — actual assignment happens per job). Cap at ~10 stages in the builder. No conditional branching UI in v1 — the data model supports parallel stages (`parallel_group`), but don't expose branching logic in onboarding; that's a v2 conversation once real usage shows the need.

**Done:**
```
You're set up. Create your first job.
```

## Design constraint (important)
Doc's caution applies directly: don't let this become a 400-option configurator. Five to seven objects total — Organization, People, Work Type, Fields, Stages — and opinionated defaults (the starter templates) doing most of the work. If an org needs something the builder can't express, that's a manual/support conversation for now, not a feature to rush into the UI.
