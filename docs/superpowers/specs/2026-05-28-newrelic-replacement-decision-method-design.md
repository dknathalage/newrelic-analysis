# NewRelic Replacement — Decision Method + Consultation Form

**Date:** 2026-05-28
**Owner:** Platform team (bears NewRelic cost, makes the decision)
**Goal:** Replace NewRelic with a cheaper alternative. Make a *quantitative* feature-parity-to-cost decision. Target product is not yet chosen.

---

## Core principle

The platform team bears the cost and owns the decision. Engineering teams consume observability but do not weigh cost — asking them cost tradeoffs produces garbage data (everything is "critical," keep everything). So:

- **Telemetry drives all numbers.** Pulled from the NewRelic API, it covers 100% of users objectively.
- **The form is a qualitative overlay only.** It captures what telemetry cannot see: severity of loss, what would be rebuilt, hidden lock-in, training gaps.

This removes sample bias: a small number of form responses (e.g. ~10) is not a statistical liability because we are not counting votes — we are collecting flags layered on full-population telemetry.

---

## Two data sources

### A. Telemetry (NewRelic API) — quantitative engine

Pulled directly via API access (confirmed available):

- **Per-data-type ingest GB/mo:** APM spans, logs, metrics, custom events, traces + seat count.
  → reveals *where the money goes today* (e.g. 70% logs → log-volume-priced tools win).
- **Entity / host / series / dashboard / alert / synthetic counts.**
  → migration volume + input to candidate-tool cost modeling.
- **Actual feature usage across all users:** who queries NRQL, who fires/owns alerts, who views which dashboards.
  → **quantitative demand** per feature, full population.

**Demand metric (the primary axis — one formula):**

> **demand(capability) = distinct active users touching it in last 90d ÷ eligible population**

Reach, normalized to a % — comparable across all feature types ("how many people would this hurt if cut"). **Intensity (raw event volume) is a tiebreaker only**, used to separate two capabilities with similar reach; never the primary number, so one power user can't fake broad demand.

**Eligible population = all licensed NewRelic users** — one fixed denominator for every capability (not per-capability access). Niche capabilities (e.g. Mobile) showing low reach is correct, not a denominator artifact; the form-only marking + blocker flags catch genuine-but-narrow needs.

**Capability → telemetry signal map** (defines what feeds demand per feature; without a signal a capability is form-only):

| Capability | Telemetry signal for "active user" | Source |
|---|---|---|
| APM | owns/views an APM service entity | entity + view events |
| Infrastructure | views infra host data | view events |
| Logs | runs log query / views log UI | query events |
| Distributed tracing | views traces | view events |
| Dashboards/NRQL | authors/runs NRQL, views dashboard | query + view events |
| Alerts/AI | owns/ack's an alert condition | alert policy ownership |
| Errors Inbox | views Errors Inbox | view events |
| Kubernetes | views K8s cluster data | view events |
| Synthetics | owns a synthetic monitor | monitor ownership |
| Serverless | views Lambda/serverless entity | entity + view events |
| Browser/RUM | owns/views a browser app entity | entity + view events |
| Mobile | owns/views a mobile app entity | entity + view events |
| Network | views network monitoring | view events |

Capabilities where no usable telemetry signal exists are marked **form-only** in the decision table and flagged as an asymmetry — demand comes from the form overlay alone for those, with lower confidence.

**Join key:** MS Forms (restricted to tenant) auto-captures respondent Microsoft email; NewRelic SSO uses the same email → per-user join, no extra form question. This powers the telemetry cross-check (inflation filter #3).

**Criticality weighting:** OPEN ITEM. Self-reported tier is garbage (all engineers touch critical apps). Objective signal (incident/pageable-alert history, throughput, SLO presence) to be assessed later by the platform team. Demand scoring works without it; weighting plugs in when ready.

### B. Consultation form — qualitative overlay

Per-user, all hands-on NewRelic users. Captures only what telemetry can't.

**Build mechanism:** The form is authored as a **Markdown source file** that maps 1:1 to native Microsoft Forms question types. Someone recreates it by hand in the MS Forms UI from the Markdown (MS Forms has no reliable Markdown import; Graph has no supported Forms-authoring API). Markdown = single source of truth + copy-paste text. No conditional branching needed — the overlay model dropped drill-down, so the form is flat.

**Markdown → MS Forms question-type mapping:**

| Section | MS Forms question type |
|---|---|
| Team/squad | Choice (dropdown) or Text |
| Role | Choice (single) |
| Service(s) owned | Text (long) |
| Feature rating (capabilities × scale) | **Likert** (rows = capabilities, columns = dependence scale) |
| Forced top-3 must-haves | 3 separate Choice (dropdown) slots, each = pick one capability — hard-caps at 3, no free text |
| Lock-in / rebuild | Text (long) |
| Pain (hate to lose / frustrates) | Text (long) ×2 |
| Catch-all | Text (long) |

The feature-rating section is the one that matters most — MS Forms **Likert** renders the whole capability list × dependence scale as one compact grid, which is exactly the 1:1 fit.

**Intro framing:** "Platform team is cutting observability tooling cost. Tell us what you actually use and can't lose — or we might cut it."

**Sections:**

1. **Metadata** *(mandatory)* — team/squad (fixed dropdown, not free text) · role (dev/SRE/on-call/mgr). Respondent email auto-captured by MS Forms = join key to telemetry; service-level join derived from NewRelic entity ownership, not asked.

2. **Feature rating** *(mandatory)* — every NewRelic capability (user-facing NewRelic names — see taxonomy note), rated on one dependence scale:
   - `Can't work without it` (blocked if gone)
   - `Use often — would hurt to lose`
   - `Occasional — could cope`
   - `Never use`
   - `Didn't know it existed`

   Capabilities listed: APM · Infrastructure · Logs · Distributed tracing · Browser/RUM · Mobile · Synthetics · Alerts/AI · Dashboards/NRQL · Errors Inbox · Kubernetes · Serverless · Network.

3. **Forced top-3 must-haves** *(mandatory)* — "Of everything you marked blocking, pick the 3 truest." Inflation filter.

4. **Lock-in / rebuild** *(optional)* — "Which custom queries / dashboards would you rebuild if you lost them?" Separates live (real port cost) from abandoned. Telemetry supplies the raw count; this supplies which matter.

5. **Pain** *(optional)* — "What would you hate to lose?" + "What frustrates you today?" Hate-to-lose = requirement; frustrates = bonus win if a cheaper tool fixes it.

6. **Catch-all** *(optional)* — "Anything we missed?"

---

## Inflation control — three filters

Engineers over-claim need. Three decreasing-inflation filters:

1. **Free rating** — honest signal, no cap on "can't work without it."
2. **Forced top-3** — over-claimers self-demote (8 blockers but top-3 names only 3 → other 5 downgrade).
3. **Telemetry cross-check** — even top-3 validated against what they actually touch.

---

## Scoring rubric — FROZEN before any response is read

No post-hoc tuning (prevents fitting weights to a predetermined tool choice).

- **Quantitative demand** = the reach formula above (distinct active users ÷ eligible pop, 90d). This is the only number that drives the decision.
- **Form overlay = annotation flags on each feature row, NOT aggregated into the demand number** (keeps the axes separate, prevents ad-hoc mixing at decision time):
  - `can't work without it` → blocker flag
  - `use often` → "often" flag
  - `occasional` → "occasional" flag
  - `never use` → no flag
  - `didn't know it existed` → **training-gap flag — routed OUT of the cut decision** (excluded from cut candidates pending enablement review, so a genuinely-needed-but-undiscovered feature isn't cut as if unused)

---

## Internal eval layer (not user-facing)

Users see NewRelic vocabulary (NRQL, Errors Inbox) for response quality. The platform team maintains a hidden **NewRelic → neutral-capability map** for cross-tool comparison, plus the differentiating axes users won't volunteer:

- query-language lock-in
- metric cardinality limits
- data retention window
- trace sampling
- session replay

These live in the eval matrix, not the form.

---

## Deliverable — the decision table

Decision unit is the **feature** (form is per-user; this is the pivot). Kept lean — one row per capability, no compound scores:

| feature | demand (reach %, 90d) | overlay flags (blocker / often / training-gap / form-only) | data-type cost share | migration volume (live items) | → keep / replace / drop |
|---|---|---|---|---|---|

- **Cost is NOT attributed per feature.** NewRelic prices by data type (log GB, ingest GB, seats), so cost sits at the **data-type** level; each capability maps to its data type(s) for a *cost share*, not an independent per-feature dollar figure (avoids fabricated numbers).
- **Migration volume** = count of live items (dashboards/queries the user said they'd rebuild) from telemetry + form, not an effort estimate.

**Cut candidate = reach < 10% of licensed users AND no blocker flag AND not training-gap-flagged.** This replaces the vague "gap = savings": a feature is a savings target only when telemetry shows few people touch it *and* nobody flags it must-have *and* it isn't an undiscovered-but-needed capability. The 10% threshold is a fixed value frozen before any data is pulled (adjustable by the platform team up front, never after seeing results).

---

## Open items

1. **Criticality weighting** — objective signal (incidents/pages/throughput/SLO) to be chosen by platform team.
2. **Candidate products** — not yet selected; eval matrix axes defined so candidates can be scored when chosen.
