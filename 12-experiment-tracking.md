# Experiment Tracking

Don't optimize pricing yet — instrument first, decide later. This file lists what must be tracked from day 1 so the A/B test in a few months has real data to run on, not a backfill problem.

## The three variants to eventually test
- **A** — Free credits → paid credits (pure pay-as-you-go)
- **B** — Free credits → monthly subscription
- **C** — Free credits → credits + a premium AI tier

Don't build the variant-switching logic now. Just make sure every org's signup records which cohort it would belong to (a `signup_cohort` column on `organizations`, even if every org is cohort A for now) so a retroactive analysis is possible.

## Events to log (in addition to the operational `events` table)
A lightweight `product_events` table, separate from the operational audit trail — this is product analytics, not the job timeline:

```sql
create table product_events (
  id uuid primary key default gen_random_uuid(),
  org_id uuid references organizations(id),
  user_id uuid references users(id),
  event text not null,   -- see list below
  properties jsonb default '{}',
  created_at timestamptz default now()
);
```

Required `event` values, fired at these exact moments:
| Event | Fired when |
|---|---|
| `signup_completed` | Step 1 of onboarding finishes |
| `first_workflow_created` | First `work_type` saved |
| `first_job_created` | First `jobs` row for the org |
| `fifth_job_created` | Org's 5th job — the "real usage" threshold |
| `employee_invited` | Any invite sent |
| `employee_activated` | Invited user's first login |
| `credits_low_25` / `credits_low_10` / `credits_exhausted` | Balance crosses each threshold |
| `credits_purchased` | Successful credit purchase |
| `auto_recharge_enabled` | Explicit opt-in only |
| `referral_sent` | Referral link/invite sent |
| `referral_org_activated` | Referred org crosses the "active" bar (`10-monetization-credits.md`) |

## Metrics computed from the above (not stored directly — derive them)
- Signup → first workflow (time delta between `signup_completed` and `first_workflow_created`)
- First workflow → first job
- First job → 5 jobs
- 7-day and 30-day retention (org has ≥1 operational `events` row in that trailing window)
- Employees invited vs. activated (ratio)
- Credits consumed / org / month
- Payment conversion (`credits_exhausted` → `credits_purchased`, with time delta)
- Referral rate (`referral_sent` → `referral_org_activated`)
- Jobs processed / month / org

## North star
Restated from `10-monetization-credits.md`: **active organizations processing real work weekly.** Every dashboard built on this data should make that number impossible to miss and everything else secondary.
