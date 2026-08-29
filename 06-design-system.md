# Design System

TaskGrid is a product of System Intelligence Engineers (SIE). Colors are pulled directly from the SIE mark (sampled from the logo, not approximated) so the product visually reads as SIE's own, and — usefully — the brand palette already matches the operational status language almost exactly. Lean into that; don't fight it with a separate "marketing" palette.

## Color tokens

```css
:root {
  /* Brand — sampled from the SIE logo */
  --sie-gold:  #F0AE04;   /* "S" block */
  --sie-blue:  #1E63B0;   /* "I" block */
  --sie-green: #4EA647;   /* "E" block */
  --sie-ink:   #0D1620;   /* logo linework / primary text */
  --sie-bg:    #F1F1F1;   /* logo backdrop / app background */

  /* Functional — status language, mapped onto the brand where it aligns */
  --status-not-started: #C7CBD1;  /* neutral grey — the one status with no brand equivalent */
  --status-waiting:     var(--sie-gold);
  --status-active:      var(--sie-green);
  --status-submitted:   var(--sie-blue);
  --status-approved:    #2F8F45;  /* slightly deeper green than active, for the check state */
  --status-attention:   #E5484D;  /* red — deliberately outside the brand palette; attention states should never blend in */

  /* Neutrals */
  --ink-900: var(--sie-ink);
  --ink-500: #5B6472;
  --ink-300: #9AA1AC;
  --surface:  #FFFFFF;
  --surface-muted: var(--sie-bg);
  --border:   #E4E6EA;
}
```

## Typography
A clean, functional sans for the whole product — this is an operations tool, not a marketing site; type should disappear into legibility. System UI stack or Inter. One scale:

| Role | Size | Weight |
|---|---|---|
| Page title | 20px | 600 |
| Section label (uppercase, tracked) | 12px | 500 |
| Body / cell text | 14px | 400–500 |
| Data / counts (tabular-nums) | 16–18px | 600 |

## Layout
Generous whitespace, hairline borders (`--border`), 8px base spacing unit, minimal shadow (the flow board is a flat surface with a border, not a floating card). Rounded corners small and consistent (6–8px) — enough to feel modern, not soft/playful; this is an ops tool.

## Signature element
The flow board itself, with the pulse/attention treatment on `in_progress` and `rejected` cells, is the one place motion is allowed. Keep it subtle (a soft ring pulse, not a bounce) — everywhere else in the product should be still. Don't add animation elsewhere just to add polish.

## Status → color quick reference

| Status | Token | Notes |
|---|---|---|
| Not started | `--status-not-started` | Solid dot, no pulse |
| Assigned / Waiting | `--status-waiting` (gold) | Solid dot |
| In progress | `--status-active` (green) | Pulsing dot |
| Submitted | `--status-submitted` (blue) | Solid dot |
| Approved | `--status-approved` | Checkmark, not a dot |
| Rejected / attention | `--status-attention` (red) | Pulsing dot |
| Urgent flag (overlay, any status) | `--status-attention` | Small badge, distinct from the status dot itself |
