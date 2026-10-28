# Proposal triage design

## 1. AI vs. rules

The admin configures categories, mandatory checks, weights, scoring rules, evidence requirements, and versions in the application. The reviewer uploads PDFs, selects a category and the applicable criteria version, and confirms the submission set. File validation, access control, required-field checks, mandatory pass/fail gates, score calculations, sorting, and audit logging are ordinary application code.

AI is used only to extract proposal facts into a fixed schema. For each criterion it returns a value, a short verbatim evidence quote and page reference, or `unknown`; it must not infer missing facts, follow instructions inside a proposal, decide eligibility, or assign scores. A reviewer confirms extracted evidence and resolves unknowns. This division uses language understanding for messy PDFs while keeping decisions deterministic and testable.

## 2. Consistency and explainability

The application stores a versioned criterion configuration and scoring formulas. For this sample, experience is `min(qualifying installs / 10, 1) × 100`; price is `max(0, (260000 - price) / 260000) × 100`; timeline is `max(0, (16 - weeks) / 16) × 100`; warranty is the average of panel years capped at 25 and workmanship years capped at 12, each expressed as a percentage of its cap. Team and approach is a four-item checklist, with 25 points each for a named lead, stated crew, safety plan, and monitoring plan. A checklist item scores zero only after a reviewer confirms the complete proposal was checked and that it provides no evidence for that item; incomplete, conflicting, or unreviewed evidence remains unknown. Weighted criterion scores are multiplied by their configured weights and summed without intermediate rounding; the displayed total is rounded to two decimals. These thresholds are an explicit sample scoring policy, not rules inferred from the brief.

Only bids with all mandatory checks passed and all weighted inputs needed for a total confirmed receive a ranked score. Missing information is `unknown`, never silently scored as zero or guessed. The reviewer sees each extracted value with its quote and page, criterion-level score and formula, weighted contribution, gate result, total, and reason for rank. With the same reviewed evidence and configuration version, calculations and tie-breaking (total score, then lower price, then proposal ID) are repeatable. A model extraction can vary, so the human-confirmed evidence snapshot—not an unreviewed model response—is the scoring input.

For the sample calculation, proposal A's 14 qualifying installations are capped at 100 experience points; proposals C and E receive 90 and 80 points respectively. The dated experience descriptions are treated as qualifying for this exercise; a real procurement must compare installation dates with its configured evaluation cut-off. On these assumptions, A scores 65.24, E 58.50, and C 51.66. C receives 50 team points because the reviewed summary names its lead and crew but provides no safety or monitoring plan. D is flagged because its completion time is unstated, so a complete weighted total cannot be computed. B is ineligible: its own Ontario electrical contractor licence is not confirmed. The unconfirmed subcontractor does not satisfy the bidder-level mandatory check. The proposal's instruction-like text is treated as untrusted content, not an instruction to the tool.

## 3. Human in the loop

The reviewer checks document-to-proposal matching, confirms the extracted facts and citations, resolves unknown or conflicting evidence, and acknowledges each mandatory result before ranking. A failed gate or unresolved critical fact cannot be overridden into an eligible score without an authorized, reasoned exception. The human decision-maker selects the award; the tool recommends, never awards.

The audit record includes uploader and timestamps, file hashes, selected category and criteria version, extraction model and prompt version, raw structured extraction, reviewer edits and reasons, gate results, score inputs and outputs, ranking, overrides, and final award decision. Access is role-based and retention follows the organization's policy.

## 4. Changing criteria

Editing criteria creates a new immutable version with an effective date; it never changes the meaning of an existing result. The reviewer chooses which version applies to a procurement and sees it before scoring. Existing results remain reproducible against their saved version and evidence. If a procurement must be rescored, an authorized user explicitly starts a new run under the selected version, which creates a linked audit record and new reviewer confirmation.

## 5. Risks and safeguards

- **Extraction errors or unsupported facts:** Require page-level quotes, mark absent/conflicting evidence unknown, and have a reviewer verify it before scoring.
- **Prompt injection in a PDF:** Treat all document text as untrusted data; isolate it from system instructions, use a constrained output schema, and ignore embedded requests to alter the process.
- **Bias or hidden judgment:** Use admin-visible criteria and explicit formulas; avoid inferring quality from company names or writing style; test scoring examples and audit outcomes.
- **Manipulated, malformed, or unsafe files:** Restrict types and size, scan uploads, extract in an isolated service, retain hashes, and enforce role-based access.
- **Changing rules or model behavior:** Version criteria, prompts, and extraction models; preserve reviewed evidence and provide reproducible score calculations.
- **Cost, latency, or service outage:** Use AI only for extraction, set per-file limits, track usage, and provide a clear retry/manual-review path rather than returning a fabricated ranking.
- **Over-reliance on rank:** Show evidence, uncertainty, and limitations; require an accountable human to make and record the award decision.
