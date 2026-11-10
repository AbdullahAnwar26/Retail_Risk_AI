# 17 - BMAD Installation Recommendation (research only, nothing installed)

Date: 2026-11-09. Status: RESEARCH ONLY. No installer, npx or download of BMAD was run. Installation needs explicit user approval.

## 0. Sources and whether opened

| Source | URL | Opened? |
|---|---|---|
| GitHub repo README | https://github.com/bmad-code-org/BMAD-METHOD | Yes (via fetch summary) |
| GitHub releases | https://github.com/bmad-code-org/BMAD-METHOD/releases | Yes |
| package.json at tag v6.12.4 | https://raw.githubusercontent.com/bmad-code-org/BMAD-METHOD/v6.12.4/package.json | Yes |
| Docs home | https://docs.bmad-method.org/ | Yes |
| Docs: install (v7 preview page) | https://docs.bmad-method.org/start/install-bmad/ | Yes |
| Docs: choose a planning path | https://docs.bmad-method.org/plan/choose-a-planning-path/ | Yes |
| Docs: add modules | https://docs.bmad-method.org/customize/add-modules/ | Yes |
| npm package page | https://www.npmjs.com/package/bmad-method | NO (HTTP 403). package.json used instead |
| Older v6 install/CLI pages (zh-cn, mintlify) | various | NO (404 / 410) |
| Web search snippets (third-party, unofficial) | e.g. aicoding.csdn.net, mintlify mirror | Snippets only; treated as unverified |

Caveat: page text was obtained through a summarising fetch tool, not raw reading, so wording below is paraphrase unless in quotes. Re-verify before acting.

## 1. What BMAD is

- Open-source framework for AI-driven agile development ("Breakthrough Method for Agile AI Driven Development"): planning, architecture and implementation, with ceremony scaled to the size of the change.
- Stable: **v6.12.4**, released 14 Oct (year not shown on the page; presumably 2026). npm `bmad-method` 6.12.4, MIT, Node >=20.12.0 (package.json at tag).
- V7 is in **preview alpha** (full release said to be expected around mid-November 2026); docs warn of frequent breaking changes. The live docs site now describes v7, so docs and the stable v6 installer differ.
- License: MIT. "BMad" / "BMAD-METHOD" are trademarks of BMad Code, LLC.
- Maintainers: published under GitHub org `bmad-code-org`; contact contact@bmadcode.com. Individual maintainers not verified on the pages read.
- package.json: no `postinstall` script; `prepare` runs husky only if present. Runtime deps include @clack/prompts, commander, glob, js-yaml, xml2js, yaml, etc. (13 listed).

## 2. Install method per docs

**Stable v6.12 (recommended path):** `npx bmad-method install`
- Requirements (v6.12): Node.js 20.12+, Python 3.10+, uv.
- Documented flags (v6.12, from add-modules page): `--directory`, `--custom-source`, `--tools`, `--yes`, `--action` (`quick-update` or `update`). Example from docs: `npx bmad-method install --directory . --custom-source /path/to/my-module --tools claude-code --yes`.
- Release notes: `install --yes` without `--directory` installs into the current directory instead of prompting (v6.12.2).
- Interactive prompts (module selection, IDE/tool selection incl. Claude Code, channel picker stable/pin): channel picker is documented in release notes; the full prompt list was NOT verified (v6 install page not reachable). Third-party snippet mentions `--modules bmm` - unverified, and the official add-modules page says it does not mention `--modules`.
- Modules listed on README: BMad Method, Builder, Creative Intelligence Suite, Test Architect, Loop, Game Dev Studio.

**V7 preview alpha (NOT recommended now):** `npx skills add bmad-code-org/BMAD-METHOD --skill '*'`, then ask the `bmad` skill to run `bmad setup`; `bmad status` to verify. Needs Node, npm, Git, uv. Shows a "Security Risk Assessments" table (Gen Agent Trust Hub, Socket, Snyk) during install. Claude Code plugin route: `/plugin marketplace add bmad-code-org/bmad-plugins`.

## 3. What an install changes

| Item | Verified from docs? |
|---|---|
| `_bmad/` folder with shared runtime/scripts, `_bmad/scripts/`, `_bmad/custom/` (v7 page: "Setup installs the shared runtime and module scripts under `_bmad/`") | Yes for v7; v6 presumed similar, unverified |
| `_bmad/custom/config.toml` and `config.user.toml` overrides survive setup refreshes | Yes (release notes/v7 page) |
| Output folder `_bmad-output` for documents/tickets | Yes (v7 page); third-party snippet says default in v6 too |
| Claude Code skills folder (`.claude/skills/`) | NOT verified in official docs (v7 page only says the coding tool discovers skills; a third-party snippet mentions `.claude/skills/`) |
| Modifies existing files | NOT stated. Unverified |
| Network needed | Yes in practice: `npx` fetches the package; v7 `npx skills add` fetches from GitHub. Module/updates also fetch |
| Writes outside project | Not stated. v7 page mentions "project and global installation scopes" so global (user-level) scope exists in v7; npx cache is written outside the project. v6 behaviour unverified |
| Creates git state | Not stated; v7 requires Git installed. Unverified whether it commits/inits |
| Uninstall docs | None found on v7 page. Unverified for v6 |

Treat all "unverified" rows as unknown until a dry run in a scratch copy (section 6).

## 4. Agents, workflows, mapping

Current docs (v7 preview) name:
- Agents: Mary (Business Analyst, `bmad-agent-analyst`), John (Product Manager, `bmad-agent-pm`), Sofia (Product Owner, `bmad-agent-po`).
- Skills: `bmad-architecture`, `bmad-ux`, `bmad-brainstorming`, `bmad-forge-idea`, `bmad-deep-recon` (research), `bmad-build`, `bmad-build-auto`, `bmad-retrospective`, `bmad-project-context`, `bmad-create-epics-and-stories` (named in v6.12.1 notes), `bmad-walkthrough` (renamed from `bmad-checkpoint-preview`), `bmad` (help/setup/status), `bmad-toolsmith`.
- Docs refer to "skills" rather than slash commands; exact invocation syntax in Claude Code (e.g. `/bmad-...`) is NOT verified.
- Epics: Sofia slices requirements into stories recorded in the epic's `tickets.toml` (v7).
- NOT verified in current docs: architect/UX persona names, a dedicated "implementation readiness check", "sprint planning", separate Dev/QA agents, and "party mode" (docs only link a "Run Multi-Agent Discussions" page; skill name unknown). Older BMAD versions had these (e.g. Winston, Sally, `/bmad-bmm-*` commands, party mode) but that is from memory, not verified here; do not rely on it.

Planning-path guidance: our project is "project-sized" (several epics, shared product/UX/technical docs).

Mapping of our planning pack to BMAD artefacts (artefact names generic; verify exact filenames after install):

| Our file | Feeds BMAD artefact |
|---|---|
| 01 project research analysis | Product brief / research input (`bmad-deep-recon` output, analyst context) |
| 02 retail subsystem scope | Product brief scope; PRD scope section |
| 03 technology compatibility report | Architecture inputs/constraints |
| 04 standalone vs plugin architecture | Architecture decision record |
| 05 high-level system architecture | Architecture document (`bmad-architecture`) |
| 06 module responsibility matrix | Architecture (components) |
| 07 database and data flow design | Architecture (data) |
| 08 security/privacy/compliance | PRD NFRs; architecture security section; project context |
| 09 MVP scope and prioritisation | PRD (MVP scope, priorities) |
| 10 BMAD epics and stories | Epics and stories (draft; Sofia / `bmad-create-epics-and-stories` to reconcile) |
| 11 implementation roadmap | Epic sequencing / build order |
| 12 open questions and decisions | PRD open items; decision log |
| 13 NYC regulatory matrix, 14 jurisdiction policy layer, 15 integration contracts | PRD compliance reqs; architecture (policy layer, contracts) |
| 16 | Not present in planning folder at time of research |
| 17 (this file) | Process only |
| UX | No UX file in pack yet; `bmad-ux` would produce one |

## 5. Recommended workflow

1. Snapshot, install (after approval) into a scratch copy first, inspect, then decide.
2. Place planning pack as read-only inputs (copy, not move) and write `project-context.md` (via `bmad-project-context`) pointing to files 01-15 and stating rules: "no application code until approved", stack and compliance constraints.
3. Phases: (a) Analyst (Mary): brief from 01/02 - GATE 1; (b) PM (John): PRD from 08/09/12/13 - GATE 2; (c) UX (`bmad-ux`) - GATE 3; (d) Architecture (`bmad-architecture`) from 03-07, 14, 15 - GATE 4; (e) Epics/stories (Sofia) reconciled with file 10/11 - GATE 5; (f) readiness review (manual cross-check if no skill exists) - GATE 6.
4. Each gate: human reviews and signs off in a short approval note (date, reviewer, files/hash) stored in `planning/approvals/`. No next phase until the note exists.
5. "No code until approval": do not run `bmad-build` / `bmad-build-auto`; add a rule in project context and `CLAUDE.md`, and a deny permission rule for the build skills in `.claude/settings.json` until the user lifts it. Keep Edit/Write limited to docs paths.
6. After approval of all gates, enable build one story at a time, never `bmad-build-auto` at first.

## 6. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Version drift: docs site is v7 preview while stable is v6.12.4; skill names differ | Pin `npx bmad-method@6.12.4 install`; read the v6.12.4 tag docs, not the live site |
| Supply chain (npx runs remote code, 13 deps) | Pin exact version; inspect package.json/installer on the tag first; no postinstall found; use `npm view bmad-method@6.12.4 dist.integrity`; run on a clean machine/folder first |
| Overwriting files | Install into a clean subfolder or copy; snapshot first; compare diff after; do not run `update`/`quick-update` unattended |
| Agent permissions / unattended build skills | Deny `bmad-build*` until approval; review skill files for tool grants |
| Writes outside project / global scope | Use `--directory` explicitly; check %USERPROFILE%\.claude and npx cache before/after |
| Empty workspace vs subfolder | Subfolder or sibling copy for trial; skills discovery by Claude Code may need the project root, so final install must be at the root |
| v7 alpha breaking changes | Do not use v7 until stable release and re-check |
| Rollback | Pre-install zip/copy of `G:\Retail_Risk_AI`; after install, delete `_bmad/`, `_bmad-output/` and the skill/command folders created (found by diffing file lists); restore snapshot. Official uninstall steps not found |

## 7. Pre-install checklist and recommendation

Checklist:
- [ ] User gives explicit written approval to install.
- [ ] Zip/copy of `G:\Retail_Risk_AI` made; file listing with hashes saved.
- [ ] Node >=20.12, Python >=3.10, uv present (`node -v`, `python --version`, `uv --version`).
- [ ] Decide v6.12.4 (pinned) and re-confirm it is still "Latest stable" that day.
- [ ] Open the v6.12.4 tag README/docs and confirm prompts, folders, uninstall.
- [ ] Trial in a scratch copy folder; diff before/after; check for changes outside the folder and for git state.
- [ ] Add `bmad-build*` deny rule and "no code" rule.
- [ ] Planning pack finalised (open questions in 12 resolved or logged).

Recommendation: **Install later, not now** (after user approval and the scratch-copy trial). Use stable **v6.12.4**, pinned, not v7 alpha (re-evaluate when v7 goes stable, expected mid-November 2026). First trial in a scratch copy of the workspace (e.g. `G:\Retail_Risk_AI_bmad_trial`), then, if clean, install at `G:\Retail_Risk_AI` root with Claude Code as the tool, the core Method module only (no Test Architect/CIS/Loop/Game Dev yet), interactive prompts rather than `--yes`. Installation is NOT being done now.
