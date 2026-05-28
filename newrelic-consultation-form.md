# NewRelic Consultation Form — MS Forms build source

> Single deliverable produced from the two specs in `docs/superpowers/specs/`:
> - `2026-05-28-newrelic-replacement-decision-method-design.md` — the method
> - `2026-05-28-newrelic-survey-analysis-guide.md` — how to use responses
>
> **How to use this file:** recreate the questions below in Microsoft Forms by hand. Each is annotated with its MS Forms question type in `[brackets]`. Restrict the form to your organization so respondent email is auto-captured (the join key — do **not** add an email question). Mark `*` questions Required.

## What the platform team needs to do

**Before the form goes out**
- [ ] Freeze the decision rule (per method spec): eligible population = all licensed NewRelic users; demand = distinct active users ÷ that pop, 90d; cut threshold = reach < 10%. Write it down dated — never change after seeing data.
- [ ] Confirm the 13-capability list below matches what your org actually runs on NewRelic; add/remove rows.
- [ ] Fill the real team names into Q1's dropdown.
- [ ] Build the form in MS Forms, restricted to the tenant (auto-captures email = join key).

**While the form is open**
- [ ] In parallel, pull telemetry via the NewRelic API: per-capability distinct-users (reach) + intensity, per-data-type ingest GB + seats (cost), dashboard/alert/synthetic/custom-event counts (migration volume).
- [ ] Don't chase a high response rate — telemetry is the number, the form is a qualitative overlay.

**After responses are in**
- [ ] Join responses to telemetry on email; build the per-feature decision table; classify DROP / KEEP / REPLACE / PARK by the frozen rule (analysis guide, Steps 3–4).
- [ ] Chase the expensive data type for savings, not feature count (guide Step 5).
- [ ] When candidate products are chosen, model recurring + migration cost against current spend (guide Step 6).
- [ ] Use the frozen rule to defend every cut against pushback (guide Step 7).

**Open item:** pick the criticality weighting signal (incidents/pages/SLOs) when ready — applies to everyone via the same signal, not by team claim.

---

> **Form title:** Observability Tooling — What You Actually Use
>
> **Description (intro):** The platform team is reviewing our observability tooling cost. Tell us what you actually use and can't lose — or we might cut it. Takes ~3 minutes. Your honest "never use" answers are as useful as the must-haves.

---

## Section 1 — About you

**1. Which team are you on?** `*` `[Choice — dropdown]`
<!-- Replace with your real team list. Fixed dropdown, NOT free text — needed to join responses to telemetry. -->
- Team A
- Team B
- Team C
- Team D
- (…fill in actual teams…)

**2. What's your primary role?** `*` `[Choice — single select]`
- Developer
- SRE / Platform
- On-call / Incident responder
- Engineering manager / Lead

---

## Section 2 — How much do you depend on each capability?

**3. For each NewRelic capability, pick the option that best fits you.** `*` `[Likert]`

Rows (statements / sub-questions = the capabilities):
- APM (application performance monitoring)
- Infrastructure monitoring
- Logs
- Distributed tracing
- Browser monitoring / RUM
- Mobile monitoring
- Synthetics (uptime / synthetic checks)
- Alerts & AI (alert conditions, anomaly detection)
- Dashboards / NRQL (custom queries & dashboards)
- Errors Inbox
- Kubernetes monitoring
- Serverless / Lambda monitoring
- Network monitoring

Columns (the dependence scale — same options for every row):
1. Can't work without it
2. Use often — would hurt to lose
3. Occasional — could cope
4. Never use
5. Didn't know it existed

---

## Section 3 — Your true must-haves

> Of everything you marked "Can't work without it," name the 3 you'd fight hardest to keep. Three slots — no more.

**4. Must-have #1** `*` `[Choice — dropdown]`
**5. Must-have #2** `[Choice — dropdown]`
**6. Must-have #3** `[Choice — dropdown]`

<!-- Each dropdown lists the same 13 capabilities as Section 2, plus "— none —". Three separate dropdowns hard-cap the answer at 3; do not use a free-text or multi-select field. -->

Dropdown options (for Q4–Q6):
- APM
- Infrastructure monitoring
- Logs
- Distributed tracing
- Browser / RUM
- Mobile
- Synthetics
- Alerts & AI
- Dashboards / NRQL
- Errors Inbox
- Kubernetes
- Serverless
- Network
- — none —

---

## Section 4 — Switching cost (only you know this)

**7. Which custom dashboards or NRQL queries would you actually rebuild if you lost them?** `[Text — long answer]`
> Name or link the ones you genuinely rely on. Leave blank if none — we don't need a list of everything, just what's load-bearing.

---

## Section 5 — Pain points

**8. What does NewRelic do that you'd hate to lose?** `[Text — long answer]`

**9. What frustrates you about NewRelic today?** `[Text — long answer]`
> Be specific — if a cheaper tool fixes it, that's a win, not just a cost cut.

---

## Section 6 — Anything else

**10. Anything we missed?** `[Text — long answer]`

---

### Build checklist
- [ ] Form restricted to organization (auto-captures email = join key)
- [ ] Q1 team dropdown filled with real team names
- [ ] Q3 built as a single Likert grid (13 rows × 5 columns above)
- [ ] Q4–Q6 are three separate dropdowns (hard cap of 3)
- [ ] Required (`*`) set on Q1, Q2, Q3, Q4
- [ ] Intro text set to the framing above
