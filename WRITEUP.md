# AI Proposal Triage Design: Answers

**Reference:** MATACS-TH-2026-02

## 1. AI vs. rules

The admin sets the category, mandatory conditions, scoring thresholds and weights through a versioned configuration screen, not hard-coded logic. A reviewer selects that category and uploads proposals. A normal PDF parser retrieves text and page references; OCR is an optional fallback for scans. AI converts unstructured proposal text into a defined JSON schema with citations, explicit `null` for missing facts and warnings for conflicting or suspicious content. Plain code validates the schema, verifies mandatory statements against cited evidence, applies weighted scores and sorts eligible proposals. A reviewer checks uncertain facts and makes the award decision. AI is suitable for interpreting varied wording, but hard requirements and arithmetic must be deterministic and auditable.

## 2. Consistency and explainability

Each evaluation stores a document hash, facts with evidence, rules version, subscores and output. The same **validated facts and rules version** produce the same score and order; rerunning AI extraction alone could vary, so approved facts are persisted and not silently overwritten. The UI shows pass/fail/unknown status, evidence location, each criterion's 0–5 subscore, weighted contribution, total and reason. Equal totals are displayed as ties; proposal letter only stabilizes their display order. Missing evidence is not treated as proof of compliance.

### Defined scoring policy (design assumption, not provided by MATACS)

Mandatory: bidder-held Ontario electrical contractor licence and confirmed WSIB clearance and CAD 5M liability insurance. Explicit bidder noncompliance is ineligible; uncertain mandatory evidence is flagged. A missing timeline is flagged pending clarification, rather than guessed. Ranking includes only fully scoreable, eligible proposals. The CAD $260,000 budget ceiling and 16-week limit are scoring/project constraints; exceeding either is flagged for reviewer review, not silently redefined as an additional mandatory licence-style rule.

Scores use `(subscore / 5) × weight`, summed to 100. Weights: experience 30, price 25, timeline 15, warranty 15, team 15. The configured weights must total 100.

| Criterion | Explicit example rule for a 0–5 subscore |
| --- | --- |
| Experience | Count eligible commercial installs >=100 kW in the last five years: 12+ = 5; 8–11 = 4; 5–7 = 3; 2–4 = 2; 1 = 1; none = 0. |
| Price (CAD) | <=200k = 5; >200–220k = 4; >220–240k = 3; >240–260k = 2; above ceiling = 0 plus reviewer flag. |
| Duration (weeks) | <=10 = 5; 11–12 = 4; 13–14 = 3; 15–16 = 2; >16 = 0 plus reviewer flag; unknown = no score. |
| Warranty | Average of panel and workmanship subscores. Panel years: >=25=5, 20–24=4, 15–19=3, 10–14=2, 1–9=1, none=0. Workmanship years: >=12=5, 10–11=4, 5–9=3, 2–4=2, 1=1, none=0. |
| Team/approach | 1.25 points each for stated named project lead, stated crew, safety plan, and performance-monitoring plan. Progress reports alone do not count as performance monitoring. |

For the sample ranking, A and E's year-level experience claims are provisionally accepted. C's reported experience spans beyond the five-year lookback, so the count used for illustration must be checked against dated references before final award. A real system would request dated project evidence, revise the experience score as needed and retain the change log.

### Illustrative ranking result

A: `30 + 15 + 12 + 13.5 + 15 = 85.5`, rank 1.  
E: `24 + 10 + 9 + 15 + 15 = 73`, rank 2.  
C: `24 + 10 + 9 + 13.5 + 7.5 = 64`, rank 3, conditional on experience verification.  
D: `flagged`, no score because the timeline is unknown.  
B: `ineligible`, no score because the bidder has not established its own required licence; a proposed future subcontractor is not a bidder-held licence.

## 3. Human in the loop

A reviewer confirms extracted evidence, resolves missing/conflicting information, reviews warning flags and approves or overrides the recommendation. Evidence-backed corrections trigger a fresh deterministic score. The final award remains human-controlled; a change to the final decision does not secretly edit the calculated score. Log actor ID, timestamp, proposal ID, source document hash/page, old/new values, justification, criteria version, score breakdown and approval/override action. Restrict overrides to authorized roles.

## 4. Changing criteria

An admin edit creates a **new criteria version**. Previously approved evaluations remain immutable with their original criteria and scores. A new evaluation runs under the new version, compares score breakdowns and requests fresh human approval. Validate that weights add to 100 and no required criterion is missing before activating a new version.

## 5. Risks and safeguards

- **Bad extraction or hallucination:** strict JSON schema, page-cited evidence, `null` for missing fields, rule validation and reviewer verification.
- **Prompt injection:** treat uploaded proposal text as data, not instructions. Proposal C contains a request to award itself first place; ignore that instruction, log the attempt, and assess its factual content normally.
- **Missing/unverified facts:** flag D's absent completion schedule; confirm licences and dated project experience before award. Do not infer an unstated timeline or bidder-held licence.
- **Bias and hidden scoring:** apply the same admin-approved rubric to every eligible bidder, show score breakdown and provide an override audit trail.
- **Criteria drift:** version configurations, preserve historical runs and recompute only under explicit re-evaluation.
- **Cost/security:** bound document size and AI calls, cache validated extraction by document hash, minimize sensitive data, protect access and never put secrets in the repository.

**Scope assumption:** Only proposal letters and derived assessments are published here; no employer brief or sample-proposal text is reproduced. All rankings are demonstrations of the stated scoring policy, not independent verification of bidder claims.
