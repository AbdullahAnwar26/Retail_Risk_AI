# 16b - Diagram and Link Validation

Date: 2026-11-09. Scope: all `.md` files in `G:\Retail_Risk_AI\planning\` present at run time (00-15 and 17; file 16 did not exist yet, see R-20). No planning file was edited.

## 1. Mermaid diagrams

### Method
- Node v24.22.1 and npm 11.19.1 are present.
- Installed `@mermaid-js/mermaid-cli@11.16.0` (published 2026-06-29, more than two weeks old) locally in the scratch folder only (not global, not in the project).
- npm did not run puppeteer's postinstall (npm `allowScripts` policy), so no Chromium download was attempted. This means I did not test whether that download is blocked. `mmdc` was pointed at the system Google Chrome (`C:/Program Files/Google/Chrome/Application/chrome.exe`) through a puppeteer config file.
- Each block was extracted to its own `.mmd` file, then rendered with `mmdc` to SVG. Rendering needs a successful parse, so a clean render counts as validated and rendered. No output contained "Syntax error".
- Scratch folder: `C:\Users\WINDOW~1\AppData\Local\Temp\claude\739dbc0da06b8c85\9ce98d40-ce39-48cd-a708-8c30f7501447\scratchpad\mermaid-check\`
  - blocks: `blocks\*.mmd`
  - rendered SVGs: `out\*.svg`
  - tool install: `tool\`
- Manual-review fallback was not needed. PNG output was not produced because SVG was sufficient.

### Results: 13 blocks, 13 PASS, 0 FAIL

| # | File | Block | Fence line | Type | Result | Error | Fix |
|---|---|---|---|---|---|---|---|
| 1 | 04-standalone-vs-plugin-architecture.md | 1 | 121 | flowchart TB | PASS | none | none |
| 2 | 05-high-level-system-architecture.md | 1 | 42 | flowchart LR | PASS | none | none |
| 3 | 05-high-level-system-architecture.md | 2 | 82 | flowchart TB | PASS | none | none |
| 4 | 05-high-level-system-architecture.md | 3 | 126 | flowchart LR | PASS | none | none |
| 5 | 05-high-level-system-architecture.md | 4 | 165 | flowchart TB | PASS | none | none |
| 6 | 05-high-level-system-architecture.md | 5 | 195 | flowchart TB | PASS | none | none |
| 7 | 05-high-level-system-architecture.md | 6 | 233 | sequenceDiagram | PASS | none | none |
| 8 | 07-database-and-data-flow-design.md | 1 | 67 | erDiagram | PASS | none | none |
| 9 | 07-database-and-data-flow-design.md | 2 | 218 | flowchart LR | PASS | none | none |
| 10 | 07-database-and-data-flow-design.md | 3 | 241 | flowchart LR | PASS | none | none |
| 11 | 14-jurisdiction-policy-layer-design.md | 1 | 76 | erDiagram | PASS | none | none |
| 12 | 14-jurisdiction-policy-layer-design.md | 2 | 237 | sequenceDiagram | PASS | none | none |
| 13 | 14-jurisdiction-policy-layer-design.md | 3 | 406 | stateDiagram-v2 | PASS | none | none |

No corrected blocks are needed.

Caveat: this checks syntax and successful rendering. I did not visually inspect the SVGs for layout or readability. The two erDiagrams (07-1, 14-1) produced large SVGs (about 230-260 KB), so they may be dense on screen.

## 2. Link and reference checks

### Method
- There are no Markdown links `[text](target)` in any file. The only parentheses with filenames are plain-text mentions, so check (a) has nothing to resolve.
- Filenames mentioned in text:
  - `13-...`, `14-...`, `15-...`, `02-...` and `08-...` all match real files.
  - `16-validation-review-report.md`, `16b-...` and `17-...` are named in file 00.
  - `prd.md`, `bootstrap.py`, `project-context.md` and the like are intentionally external or not-provided names, not planning files.
- Section references were checked by script: every "file NN §X", "file NN section X" and bare "§X" was matched against the real numbered headings. All resolve. The three script hits were regex artefacts and were checked by hand:
  - `03:244`: "file 13, file 14 §9" is fine.
  - `07:199`: "file 14 §13.2" is fine.
  - `08:8`: "`14-...md` §5" is fine.
- ID families checked against their definitions:
  - `CTL-*`: all 23 referenced IDs are defined in file 14 §5.
  - `SEC-01..18`, `PRV-01..13`, `CMP-01..19` and `NYC-01..11` in file 08 are contiguous and every reference resolves.
  - `Q-C1..8`, `Q-P1..9` and `Q-A1..3` in file 13 §2 all exist.
  - File 13 rows `F01-F18`, `S01-S17` and `C01-C07` all exist, and no reference goes beyond them.
  - File 13 §6 `Q1..Q24` exist and the mapping in file 12 §G2 uses valid numbers.
  - File 14 `J1..J9`, `EXCL-*`, `POL-*` and file 15 `C4` resolve.
  - Stories `S1.1-S19.4`, epics `E1-E19` and phases `P1-P12`, `P2A`, `P3A`, `P4A`, `P8A` exist.
  - Gates `G1-G12`, `G2A`, `G4A` and `G8A` are defined. `G3A` is not (R-02).
- Heading numbering: no duplicate or skipped `##` or `###` numbers in any file. File 14 starts at §0 on purpose. Files 10, 12 and 06 use E-, letter- and M-numbering and are consistent.

### Problems found (file | line | reference | problem | proposed fix)

| ID | File | Line | Reference | Problem | Proposed fix |
|---|---|---|---|---|---|
| R-01 | 11-implementation-roadmap.md | 18 vs 28-38 | G4, G5, G9 | The phase table uses gates G4 (data sign-off), G5 (demo of slice) and G9 (AI safety review). The §3 "Approval gates" table lists no decision-maker for any of them. It names only G1, G2, G3, G6/G8/G12, G2A/G4A/G8A, G7, G10 and G11. | Add G4, G5 and G9 rows to §3 with decision-makers. |
| R-02 | 11-implementation-roadmap.md | 78, 79, 89 | G3A | G3A (policy layer review) appears in the P3A row and in the P4A entry criteria, but not in §3 or in the §7 "Legal-review gates summary". | Add a G3A row to §3 with its decision-maker. |
| R-03 | 11, 12, 14 | 11:31-36, 12:61, 14:24-31 | G1-G8 | The same labels mean three things. In file 11 they are approval gates, in file 14 §1 they are design goals/constraints, and in file 12 "G1/G2/G3" are section headings (G1 Manager decisions, G2 legal questions, G3 superseded items). This is ambiguous in cross-file prose. For example "G1 passed" (gate) and "Section G" both appear in the same lines. | Rename the file 14 goals to `DG1..DG8` and the file 12 subsections to `G.1/G.2/G.3`, or always write "gate G1" and "goal G1". |
| R-04 | 05, 11 | 05:12-20, 11:10-21 | P1-P9 | `P1..P9` mean architectural principles in file 05 §1, and phases in files 11, 10 and 12 (for example "Needed by P3" in 12). Same label, different meanings. | Rename the file 05 principles to `AP1..AP9`, or always write "principle P5" and "phase P5". |
| R-05 | 08, 12 | 08:190-200, 12:9-17, 12:93 | D1-D9 | Two different decision lists share these IDs. File 08 §4 has D1-D11 (D1 adults 18+, D7 charter answers, D8 NY CDPA, ...). File 12 §A has D1-D9 (D1 next-best offers owner, D7 audit scope, D8 US/UK installs, ...). The same ID names different items. File 12 §G3 says "D8 superseded", which is true only for file 12's D8. | Prefix the file 08 IDs as `SD1..SD11`, or `08-D1..`. In file 12 §G3 write "file 12 D8". |
| R-06 | 01, 08, 13 | 01:24,45,76,85,103,105 and others | S0-S5 vs S01-S17 | `S0..S5` are the credit-platform segments (file 01, 02, 06, 07, 12 D3). `S01..S17` are NY State rows in file 13 (also in 03, 08, 10, 12). `S12` and `S13` also match story IDs `S12.x/S13.x` in file 10. All are close lookalikes. | State "NY State row S12" or "story S12.2" on first use in each file, or write story IDs as `ST12.2`. |
| R-07 | 13, 14, 15 | e.g. 14 §13 vs F14 | F14 vs "file 14" | `F14` (federal row, model risk SR 26-2) reads like "file 14". Mostly clear in context (for example 03:116 "File 13 row F14"), but a bare "F14" could be misread. | Always write "row F14" or "F-14", and say "file 14" in full. |
| R-08 | 08, 13 | 08:125-163 vs 13:87-135 | F01-S17 | File 08 §3 re-lists F/S/C IDs as pointers. Each ID appears as a definition in both files. The IDs match, but they are duplicated, so a change in file 13 can drift. | Keep it, but say that file 13 is the sole owner. A one-line note at 08 §3 start would do. |
| R-09 | 12-open-questions-and-decisions.md | 79 | "Federal pack (F05, F14)" for Q3 (CFPB supervision / asset thresholds) | F14 is model risk management. The CFPB-posture row is F18. F05 (UDAAP/CFPB supervision) is right. | Change `F05, F14` to `F05, F18`. |
| R-10 | 12-open-questions-and-decisions.md | 75 | Q-numbers | The column is headed "File 13 Q#" and uses bare `Q1..Q23`. File 13 has two schemes, `Q-C/Q-P/Q-A` in §2 and numbered 1-24 in §6, and bare `Q4` is also used in 08:107 and 08:197. | Write the bare numbers as `§6 Q4`, or rename the §6 list to `LQ1..LQ24`. |
| R-11 | 00-work-log-and-methodology.md | 90 | "13 ... (pending) | 14 ... (pending) | 15 ... (pending)" | The file index is stale. Files 13-15 exist and were written. File 16, 16b and 17 are not listed. | Update the index: 13-15 drafted, add 16, 16b and 17. |
| R-12 | 00-work-log-and-methodology.md | 66 | `16-validation-review-report.md` | Named as an output but not present in the folder at validation time (the run was still "Running"). | Re-check once it is written. |
| R-13 | 00-work-log-and-methodology.md | 70 | "N1-N9" | The wording is unclear. N1-N9 exist only in file 12 §G1. | Write "file 12 §G1 decisions N1-N9". |
| R-14 | 11-implementation-roadmap.md | 66 | "(S15.5)" | Resolves (S15.5 is the add-a-jurisdiction rehearsal), but the story ID is written without "story" or file 10. A reader may take it as a file 13 S-row (S15 is the NY Privacy Act row). | Write "story S15.5 (file 10)". |
| R-15 | 10-bmad-epics-and-stories.md | 129-138 | S18.x, S19.x | Story IDs `S18.x/S19.x` sit beside file 13 rows `S12/S13`, and the S18.4 text lists file 13 rows `S12, S13, F07, F12`. Same ambiguity as R-06. | Same fix as R-06. |
| R-16 | 14-jurisdiction-policy-layer-design.md | 527-533 | control names without prefix | The worked example writes controls as `CREDIT-REFERRAL-ONLY`, `MINOR-PROTECT`, `HUMAN-REVIEW` etc. without the `CTL-` prefix. They map one-to-one onto catalogue IDs. | Use the full `CTL-...` names, or note "CTL- prefix omitted". |
| R-17 | 14-jurisdiction-policy-layer-design.md | 74 | §3.2 diagram vs entity names | The ER diagram (14 §3.2) and file 07 §2.2 both model the policy entities. Both render, but I did not compare entity and field names between them. | Do a name-for-name comparison in the next review. |
| R-18 | 08, 13 | 08:105-115 | "NYC-01..11" | `NYC-xx` control IDs in file 08 §2.4 share the prefix `NYC-` with `POL-US-NY-NYC-0007`/`0009` in file 14. They do not clash, but they look alike. | Rename to `NYX-01..11` (or similar) if you want to avoid confusion. |
| R-19 | 03-technology-compatibility-report.md | 209, 245, 264, 267 | "file 14 J8", "J1", "J9", "J7" | These resolve to the open items in file 14 §16 (J1-J9). The wording "file 14 J8" omits "§16". | Optional: "file 14 §16 J8". |
| R-20 | 00-work-log-and-methodology.md | 97 | "File 17" | File 17 exists. The reference is OK. Listed only because file 16 is still missing. | None. |

Counts: 20 items logged, of which 0 broken file or section pointers.
- Real content errors: 1 (R-09).
- Missing definitions or stale status: 4 (R-01, R-02, R-11, R-12).
- Ambiguous or colliding ID namespaces: 7 (R-03 to R-07, R-10, R-18).
- Style or readability notes: 8 (R-08, R-13 to R-17, R-19, R-20).

No broken filenames, no unresolved section references, no duplicate or skipped heading numbers, no undefined `CTL`/`SEC`/`PRV`/`CMP`/`NYC`/`Q-*`/`F`/`S`/`C`/`J` IDs.

## 3. What was not checked
- URLs inside file 13 were not fetched.
- Whether a cited section's content supports the claim was checked only on spot samples (file 12 §G2 row mapping, file 08 F/S pointer table, file 03 F/S citations), not exhaustively.
- Diagram content versus prose was not compared, and the SVGs were not viewed.
