# How to Use the Survey Responses — Platform Team Guide

**Date:** 2026-05-28
**Audience:** Platform team (you) — bears NewRelic cost, owns the decision.
**Companion to:** `2026-05-28-newrelic-replacement-decision-method-design.md`

This is the runbook from "responses are in" to "decision made and defended." Read the method spec first; this operationalizes it.

---

## Step 0 — Freeze before you look (do this BEFORE opening any response)

The whole method is defensible only if these are fixed before you see data. Write them down, dated, and don't touch them after:

- **Eligible population** = all licensed NewRelic users (one number, same denominator for every capability).
- **Demand** = distinct active users touching a capability in last 90d ÷ eligible population.
- **Cut threshold** = reach < 10%.
- **Cut rule** = `reach < 10% AND no blocker flag AND not training-gap-flagged`.

If you tune any of these after seeing results, every cut becomes "you rigged it." Freeze = your armor.

---

## Step 1 — Pull telemetry (the numbers; form is NOT the number)

From the NewRelic API, per capability (use the capability→signal map in the spec):

- distinct active users, last 90d → compute **reach %**
- raw event volume → keep as **intensity** (tiebreaker only)
- per-data-type ingest GB/mo (APM, logs, metrics, custom events, traces) + seat count → **cost side**
- counts: dashboards, alerts, synthetics, custom event types → **migration volume**

Telemetry covers 100% of users. This is your demand axis. The form never overrides these numbers — it annotates them.

---

## Step 2 — Join form responses to telemetry

Key = respondent Microsoft email = NewRelic SSO email. For each respondent, attach their telemetry footprint to their stated answers. This powers the cross-check in Step 4.

If a respondent's email doesn't match a NewRelic user, flag it — don't silently drop. Usually means a shared/service account or a lapsed license.

---

## Step 3 — Build the per-feature decision table

One row per capability. Keep it lean — no compound scores:

| feature | reach % | intensity | overlay flags | data-type cost share | migration volume |
|---|---|---|---|---|---|

Overlay flags from the form, per feature:
- **blocker** — anyone rated "can't work without it" OR named it in their top-3
- **training-gap** — anyone marked "didn't know it existed"
- **form-only** — capability has no telemetry signal (demand comes from form alone, lower confidence)

---

## Step 4 — Classify each feature

Apply the frozen rule mechanically. Do not eyeball.

- **DROP** → `reach < 10% AND no blocker flag AND not training-gap-flagged`. Nobody uses it, nobody flags it, it's not undiscovered. Safe cut.
- **KEEP** → `reach ≥ 10%` OR has a blocker flag. People depend on it; replacement must cover it.
- **REPLACE** → kept feature that a cheaper candidate tool also covers (resolved in Step 6).
- **PARK (enablement)** → training-gap-flagged but low reach. Don't cut yet — someone needs it but didn't know it exists. Decide separately whether to enable or retire after a usage review.

Cross-check trap: if someone's top-3 names a feature their telemetry shows they never touch, weight it down — stated need without usage is aspiration, not dependence. Note it; don't let one loud voice flip a DROP.

---

## Step 5 — Cost side (where the money actually is)

NewRelic bills by data type, not feature. So:

- Rank data types by ingest GB/$ — find the expensive one (usually logs).
- Map each kept feature to its data type(s) for a **cost share**, not a fake per-feature dollar.
- The win is concentrated: if logs are 70% of spend, a cheaper log path beats trimming ten low-cost features. Chase the data type, not the feature count.

---

## Step 6 — Model candidate tools (once products are chosen)

For each candidate, using your eval-layer neutral-capability map + differentiating axes (query-language lock-in, cardinality limits, retention, sampling, session replay):

1. Does it cover every KEEP feature? Gaps = hard blockers or DIY cost.
2. Model its recurring cost from your raw counts (hosts/series/GB/seats) on its pricing.
3. Add one-time **migration cost** = the live items from Step 1 + the "would rebuild" answers (Q7). Custom NRQL/dashboards are the real switching tax.
4. Total cost = recurring + amortized migration. Compare against current NewRelic spend.

A candidate that's cheaper recurring but forces rebuilding 200 live dashboards may not actually be cheaper year one. Say so.

---

## Step 7 — Decision-defense playbook (surviving the pushback)

You're cutting tooling others didn't ask to lose. Expect "you cut my feature." The frozen rubric is your defense — use it, don't argue taste.

**"You cut X, I need it."**
→ "X had reach below 10% and no one flagged it must-have, per the rule we froze before reading any responses. You had the survey. If that's wrong, show me the usage — we'll re-run the same rule." Shifts it from your opinion to a shared, pre-committed rule.

**"The survey only had ~10 responses, it's not representative."**
→ "Correct — that's why the survey didn't decide demand. Demand came from telemetry across all users, 100% coverage. The survey only added severity flags on top. Your usage was counted whether you responded or not."

**"My team is special / our app is critical."**
→ Criticality weighting is an open item, deliberately. If there's an objective signal (incidents/pages/SLOs) we'll fold it in — but it applies to everyone via the same signal, not by claim.

**"This breaks my 3am workflow."**
→ That's exactly what the blocker flag and top-3 capture. If it was flagged, it's a KEEP and the replacement must cover it — escalate if a candidate can't. If it wasn't flagged, that's signal the workflow isn't as load-bearing as it feels.

**Leadership ask: "show me the savings."**
→ Lead with the data-type cost concentration (Step 5), then the DROP list with reach numbers, then candidate total cost incl. migration. One table, frozen rule cited.

---

## Traps to avoid

- **Don't let the form outvote telemetry.** 10 loud responses ≠ population. Form flags, telemetry counts.
- **Don't re-tune the threshold to spare a feature.** That's how you lose the whole defense. Park it as enablement instead.
- **Don't attribute cost per feature.** Pricing is per data type. Per-feature dollars are fabricated and will be challenged.
- **Don't forget migration cost.** Cheaper recurring + huge port effort can net more expensive.
- **Don't cut training-gap features silently.** Low usage + "didn't know it existed" = enablement question, not a cut.
