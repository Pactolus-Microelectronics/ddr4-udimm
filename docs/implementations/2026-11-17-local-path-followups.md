# Fix the broken timing-diagram link, the label name, and a wrong claim about `baseline/`

Date: 2026-11-17

## Problem / Motivation

A sweep of every tracked file on `main` for local paths, after [#39](https://github.com/Pactolus-Microelectronics/ddr4-udimm/pull/39) cleaned the exports, found three things to fix in the documents and two to record:

- **A broken image link.** `docs/02_ddr_protocol/command_overview.md` linked its "Visual Example" diagram to a file under the author's user folder on their own computer. The image was never committed, so GitHub showed a broken image and the link exposed the local folder layout. The text below the link described a diagram nobody could see.
- **A wrong label name.** `CONTRIBUTING.md` and the "Task labels" table in `START_HERE.md` send newcomers to issues labelled `good-first-issue`. The repository's label is `good first issue`, GitHub's default name, so a search for the hyphenated name finds nothing.
- **A wrong claim in #39's record.** It says the netlists under `build/` and `baseline/` keep their local paths. For `baseline/phase1/` that is false: [#28](https://github.com/Pactolus-Microelectronics/ddr4-udimm/pull/28) rewrote the user folder in both netlists to `USERNAME` on 2026-06-01.
- **Two things to record, not change.** The four `build/p2` to `build/p5` netlists hold 18 path lines. The `v1.1.0` release files `omi_v1.net` and `omi_v1.netlist.xml` hold five each.

## What Changed

| File | Change |
|---|---|
| `docs/02_ddr_protocol/command_overview.md` | The "Visual Example" section's broken image, its lead sentence and its four bullets are replaced by a dated correction and four bullets that state the sequence in words, linking to the tRRD and tFAW entries in `timing_parameters.md`. The heading is kept. |
| `CONTRIBUTING.md` | `good-first-issue` becomes `good first issue`, the label's actual name. |
| `START_HERE.md` | The same change in the "Task labels" table. |
| `docs/implementations/2026-11-17-export-local-paths.md` | A dated correction is appended: `baseline/` was already redacted in #28, `build/p2` to `p5` keep their paths, and the `v1.1.0` release files keep theirs. |
| `docs/implementations/2026-11-17-local-path-followups.md` | New. This record. |

## Implementation Approach

The sweep was `git grep -n -I -i -E` over `origin/main` (the only branch) for the maintainer's user-folder path, the names of the local folders it contains, and `AppData`. The tag and release files were checked with `git show v1.1.0:<file> | grep -c` and the release's asset digests.

Each fix follows the June records' rule: correct in place with a dated note rather than silently. The diagram section keeps its heading so existing links to it still work. Its new bullets say only what the old bullets said, and they point to where the repository already explains tRRD and tFAW, so no new timing claim is introduced.

## Design Decisions

- **No replacement image.** The local file is a timing figure of the kind found in DRAM vendor datasheets, and nothing records where it came from. Committing it could republish someone else's figure without a source. A diagram drawn for this project, with every timing value sourced, is possible later work.
- **Fix the document, not the label.** `good first issue` is GitHub's default label name, so renaming the label would make the repository less standard. The only issue that uses it is #3, now closed.
- **`build/p2` to `p5` keep their paths.** They are the evidence four phase records cite for the ERC 135 to 85 to 0 steps, and they are not edited. GitHub code search indexes only the default branch ([About GitHub Code Search](https://docs.github.com/en/search-github/github-code-search/about-github-code-search), read 2026-11-17), so redacting them on `main` would hide the lines from search. The same lines would stay in the `v1.1.0` tag, the history and every fork.
- **The release files are not replaced.** The `v1.1.0` erratum states that all six attached files are unchanged and keep their SHA-256 digests. Those digests match the tagged `exports/` files, whose hashes `MANIFEST.txt` records. Replacing two files would break both.
- **The rest of the `START_HERE.md` label table is left for now.** Its other four labels (`review-needed`, `test-needed`, `docs`, `tooling`) do not exist in the repository. Creating them, or rewriting the table to list the labels that do exist, is a separate decision.
- **No history rewrite.** It would break the `v1.1.0` tag, the commit `MANIFEST.txt` names, and the forks.

## Verification

1. `git grep -n -E '\]\([A-Za-z]:/' -- docs` finds no Markdown link to a local drive, and `command_overview.md` has no image link left.
2. `git grep -n 'good-first-issue' -- ':!docs/implementations'` prints nothing; the remaining hits are in this record, which quotes the old name. `gh label list` shows `good first issue`.
3. `git grep -c USERNAME -- baseline` finds 3 lines in each `baseline/phase1` netlist: the three standard-library `uri` lines. Their `source` lines are relative paths. Searching `baseline/` for the maintainer's user-folder path finds nothing. This confirms the correction to #39's record.
4. On the pull request's rendered `command_overview.md`, both links jump to the tRRD and tFAW headings in `timing_parameters.md`.
5. `git diff --stat origin/main` lists only the five files in the table above. No design file or export changed.

## Related Docs

- [Keep local paths out of the exports](2026-11-17-export-local-paths.md), the #39 record this corrects
- [Timing parameters](../02_ddr_protocol/timing_parameters.md)
- [Contributing](../../CONTRIBUTING.md)
