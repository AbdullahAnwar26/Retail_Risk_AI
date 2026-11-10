<!-- bmad:context -->
<!-- Verified 2026-11-09; no SHA (G:\Retail_Risk_AI is not a git repository). Managed by bmad-project-context (setup, greenfield); edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## Retail_Risk_AI

This is the Retail Banking AI Intelligence Subsystem: an advisory retail-intelligence core that runs standalone or as a host plugin, in two tracks (DEC-50, 2026-11-10). Track A "Conventional" covers NYC and London (US/US-NY/US-NY-NYC and GB/GB-ENG/GB-ENG-LDN) with separate US and UK data cells. Track B "Islamic" covers an Islamic bank, assumed UAE onshore, as a separate bank-hosted deployment that never shares data or models with Track A. Both tracks use one shared core and one contract. It is in planning only; no application code exists yet. The planned stack is Python/FastAPI, PostgreSQL, React and OpenAPI (DEC-05, developer-ratified). The planning pack is in `planning/` (00-21). The BMAD artefacts are in `_bmad-output/planning-artifacts/`; start from its `README.md`.

## Policy

- Do not write application code, infrastructure or tests until the developer explicitly approves implementation. GATE-1/2/3 batches are developer-ratified on delegated authority (12 §H.7; manager countersignature recommended). Until then, produce planning documents only.
- Do not run `bmad-build`, `bmad-build-auto`, `bmad-agent-dev` or `bmad-qa-generate-e2e-tests`. Use the planning skills (`bmad-prd`, `bmad-architecture`, `bmad-ux`, `bmad-create-epics-and-stories`, `bmad-sprint-planning` in readiness mode).
- You may edit `planning/` files, but every edit needs a change-log line in that file. Log each pass in `00-work-log-and-methodology.md` §5B and its change log. Files 19 (UK) and 20 (UAE/Sharia) were being written by other agents on 2026-11-10; check they are finished before editing them.
- Use synthetic data and synthetic catalogues only. Never connect to a real bank system (12 §H.4 condition 1).
- Never design, name or emit a credit decision: approve, decline, limit, pricing, affordability or adverse action. Credit appears only as `CREDIT_REFERRAL`. Every output carries `not_for_credit_decisioning` (DEC-42).
- Apply the two-tier minors block list in `planning/14-jurisdiction-policy-layer-design.md` §13.3 everywhere. Never relax Tier 1.
- Never state that a law applies. Mark legal points `[LEGAL-CONFIRM]` and cite file 13 (US), 19 (UK) or 20 (UAE) row IDs. Keep every policy pack `PENDING_LEGAL_REVIEW`, enforced as BLOCK, until a named approver is recorded (interim approvers are roles only, DEC-63).
- Never state that a product, term or merchant category is Sharia-compliant. Mark Sharia points `[SHARIA-CONFIRM]`. The Islamic overlay only tightens (DEC-54), and Track B artefacts need Sharia approval (DEC-55). Never infer religion in any track.
- Keep tracks apart. Put no track or market literals in core logic (DEC-51). Never mix Track A and Track B data, models or keys (DEC-53), and never let US and UK personal data cross cells (DEC-52).
- Mark DECIDED-DEFAULT items for manager ratification at their gate. Record delegated items as "DECIDED (developer-delegated)" and recommend a manager countersignature. Never invent external facts such as a charter, licence or named person. Cite decisions as `DEC-xx` / `BQ-x` from file 12 §H, which overrides the §A status column (§H.7 is the latest).
- Stop if anyone asks to use real data, activate a pack, consume PD/grade, or enable a hosted LLM before its blocking item is resolved (12 §H.4 no-go triggers).

## Where things are

- Decision register and statuses: `planning/12-open-questions-and-decisions.md` §H (authoritative; §H.7 = two-track and delegated decisions).
- Two-track design (deployment profile, cells, Sharia overlay, catalogues, demos): `planning/21-two-track-deployment-design.md`. UK and UAE/Sharia matrices: `planning/19-...`, `planning/20-...` (written 2026-11-10).
- Phase and gate IDs (`PH-x`, `GATE-x`): `planning/11-implementation-roadmap.md`.
- PRD: `_bmad-output/planning-artifacts/prds/prd-Retail_Risk_AI-2026-11-09/prd.md`.
- Architecture invariants (AD-1..AD-27): `_bmad-output/planning-artifacts/architecture/architecture-Retail_Risk_AI-2026-11-09/ARCHITECTURE-SPINE.md`.
- UX contract: `_bmad-output/planning-artifacts/ux-designs/ux-Retail_Risk_AI-2026-11-09/` (EXPERIENCE.md is complete; DESIGN.md tokens are open).
- Epics and stories: `_bmad-output/planning-artifacts/epics.md`.
- Readiness verdict: `_bmad-output/planning-artifacts/implementation-readiness.md`.
- BMAD reads project knowledge from `planning/` through `_bmad/custom/config.toml`. Do not edit `_bmad/config.toml`; the installer manages it.
- `bmad-create-epics-and-stories` step 1 globs `planning-artifacts/*prd*.md` and `*architecture*.md` at the top level. The PRD and spine live in run subfolders, so give their paths explicitly when it asks for documents.

## Running and verifying

- TODO: there are no build, test or run commands yet. Add them on the first refresh after code exists, using the stack pinned at GATE-2.
- Lint the architecture spine with `uv run .claude/skills/bmad-architecture/scripts/lint_spine.py --workspace <spine run folder>`.
- Write BMAD memlogs only through `uv run _bmad/scripts/memlog.py`, never by hand.

## Conventions that differ from defaults

- Cite file 13 rows as "file 13 row S12", legal questions as "file 13 §6 Q4" and stories as "story S8.5". PDF income segments S0-S5 are credit-platform data, not file 13 rows.
- PRD FR numbers are local. Cite the missing platform prd.md IDs as "platform FR-39".

<!-- /bmad:context -->
