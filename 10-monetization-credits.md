# Monetization — Credits System

Core rule from both source docs: **never trap or delete data to force payment.** Degrade gracefully; keep trust.

## Schema

```sql
create table credit_balances (
  org_id uuid primary key references organizations(id),
  balance int not null default 0,
  starter_grant int not null default 0,
  updated_at timestamptz default now()
);

create table credit_transactions (
  id uuid primary key default gen_random_uuid(),
  org_id uuid references organizations(id),
  type text not null check (type in ('grant','purchase','consume','referral_bonus','refund')),
  amount int not null,             -- positive for credit-in, negative for consume
  reason text,                     -- 'employee_activation', 'starter_grant', 'referral:org_xyz', ...
  related_id uuid,                 -- e.g. the user_id activated, or the referred org_id
  created_at timestamptz default now()
);

create table credit_packages (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  credit_amount int not null,
  price_usd numeric not null,
  is_active boolean default true
);

create table referrals (
  id uuid primary key default gen_random_uuid(),
  referrer_org_id uuid references organizations(id),
  referred_org_id uuid references organizations(id),
  status text default 'pending' check (status in ('pending','active','bonus_awarded')),
  created_at timestamptz default now()
);
```

## MVP consumption model
Keep it simple until real usage data exists — **don't try to price AI/automation/knowledge usage yet, those aren't built.**

- 1 active employee (a `users` row with `role = 'employee'` or `manager'` that has logged in / has an assignment in the last 30 days) draws a fixed **N credits/month**, deducted as a `consume` transaction.
- Admin seats and inactive invited-but-never-logged-in users don't consume.
- Don't hardcode `N` in application logic — store it as an org-level or platform-level setting so it can be tuned per the A/B tests in `12-experiment-tracking.md` without a deploy.

## Warning levels (copy)

**25% remaining**
```
You're using TaskGrid heavily.
You have {X} credits remaining.
```

**10% remaining**
```
You're almost out of credits.
Add credits to keep your team working without interruption.
[ Buy Credits ]  [ Enable Auto-Recharge ]
```

**0% — the conversion moment**
```
Your free capacity has been exhausted.

Keep TaskGrid running:

Buy Credits          $XX → XXXX credits
Auto-Recharge        Automatically top up at 10% (opt-in only — never enabled by default)
Talk to us           For larger organizations
```

## What happens at 0% — graceful degradation, not a wall

| Capability | At 0% |
|---|---|
| View existing jobs, history, documents, timeline | ✓ Fully available |
| Employees continue existing assigned/in-progress work items (start, update, submit) | ✓ Available — don't strand someone mid-job |
| Create a **new** job | ✕ Blocked |
| Activate a **new** employee | ✕ Blocked |
| AI Copilot / Knowledge Intelligence / Automation (once they exist) | ✕ Locked |
| Existing data | Never deleted, never hidden |

Think **Read-only-for-new-work → Pay → Full operation**, not **Pay or lose everything.**

## Referral loop
```
Invite another organization → earn 500 credits
```
Bonus fires only when the referred org becomes **active** (define: processes its first 5 jobs, or has 3+ active employees within 30 days — pick one and instrument it, don't leave it vague). Both `referrer_org_id` and `referred_org_id` get the bonus as a `referral_bonus` transaction.

## Internal (founder) product-intelligence dashboard
Separate from any customer-facing screen — admin/founder-only.
```
Organizations: {n}        Active employees: {n}
Jobs processed: {n}       Events: {n}
Weekly active users: {n}  Jobs/user: {n}

Most common delay:        {top delay reason, from rejected/returned events}
Most common problem:      {top rejection/return reason text, clustered}
Most common workflow:     {most-used work_type across orgs}
Most requested capability: {from a lightweight in-app feedback capture — build the capture now, not the dashboard}
```
This dashboard reads from `events`, `credit_transactions`, and a simple `feature_requests` table (`org_id`, `user_id`, `text`, `created_at`) — don't build clustering/NLP for "most common problem" yet; a manual weekly review of `rejected`/`returned_to_stage` event payloads is enough until volume justifies automation.

## North star metric
**Active organizations that process real work every week** (a job has at least one event in the last 7 days). Not signups, not downloads, not seats provisioned.
