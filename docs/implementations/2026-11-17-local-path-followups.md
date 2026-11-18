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
- **The release files are not replaced.** The `v1.1.0` erratum already says that the attached netlists record absolute file paths, and that "the attached files are left exactly as released, so their hashes still match the attached `MANIFEST.txt`". Those digests also match the tagged `exports/` files. Replacing two files would break the manifest and contradict the erratum.
- **The rest of the `START_HERE.md` label table is left for now.** Its other four labels (`review-needed`, `test-needed`, `docs`, `tooling`) do not exist in the repository. Creating them, or rewriting the table to list the labels that do exist, is a separate decision.
- **No history rewrite.** It would break the `v1.1.0` tag, the commit `MANIFEST.txt` names, and the forks.

## How the two keep-as-is decisions were reached

Two questions came out of the sweep. Should the four `build/` netlists be redacted on `main`? Should the two `v1.1.0` release netlists be replaced? The maintainer decided on 2026-11-18 to keep both as they are. These are the steps that led there. Every command ran against `origin/main` at `baa44f5` unless it names another ref.

### Question 1: the four `build/` netlists

1. **Find and count the lines.** `grep -c` for the user-folder path gives 5 lines in `build/p2/netlist_after.xml`, 4 in `build/p3/netlist_final.xml`, 4 in `build/p4/netlist_fp.xml` and 5 in `build/p5/netlist_final.xml`, 18 in all. In each file, one is the netlist `source` (the June checkout's absolute path) and the rest are library `uri` lines (the per-user KiCad install and the project library).
2. **Say what they expose.** The lines hold the Windows account name and the local folder layout. They hold no credentials, keys or tokens.
3. **Find what depends on the files.** `git grep -l -E 'build/p[2-5]' -- docs/implementations/2026-06-0*` lists four records, phases 2, 3, 4 and 5. Each cites its `build/` folder as the evidence for its step of the ERC 135 to 85 to 0 work. The files are evidence, and the project does not edit evidence.
4. **Find where else the same lines live.** `git ls-tree -r --name-only v1.1.0` shows all four files in the release tag. `git log --diff-filter=A` shows they were added on 2026-06-01, in `ee4457b`, `8beb3b7`, `d03e9ad` and `a8b982d`. `gh api .../forks` lists five forks, four of them owned by other people. Whatever happens on `main`, those copies keep the lines.
5. **Work out what a change on `main` could reach.** GitHub code search indexes only the default branch, and skips files over 350 KiB ([About GitHub Code Search](https://docs.github.com/en/search-github/github-code-search/about-github-code-search), read 2026-11-17). The four files are 123,813 to 153,484 bytes, so they are searchable. Redacting them would therefore change only two things: what `main` shows, and what code search finds. It would not remove the lines from anywhere they already are.
6. **Check for a precedent.** `git log -- baseline/phase1/` shows that [#28](https://github.com/Pactolus-Microelectronics/ddr4-udimm/pull/28) redacted the two baseline netlists in place on 2026-06-01, so it has been done once. That happened before the evidence rule. Doing it again now would need an explicit waiver of the rule.
7. **Weigh it.** On one side, a search hit on `main` would go away. On the other, the evidence the four records cite would change after the fact. The same lines would stay in the tag, the history and the forks. The waiver would also set a precedent for editing evidence.
8. **Decide and record it.** The decision is to keep the files and say so in writing: the correction appended to [#39's record](2026-11-17-export-local-paths.md) states that `build/p2` to `p5` keep their paths, and why.

### Question 2: the two `v1.1.0` release netlists

1. **Count the lines in the tagged files.** `git show v1.1.0:exports/omi_v1.net | grep -c` gives 5, and the same for `omi_v1.netlist.xml` gives 5. The exports on `main` have none, because #39 rewrote them.
2. **Confirm the release files are the tagged files.** `gh api .../releases/tags/v1.1.0` gives each asset's SHA-256. `omi_v1.net` is `f1f1dc1b…` and `omi_v1.netlist.xml` is `82a9698c…`, the same as `sha256sum` of the tagged files.
3. **Find what depends on those digests.** `git show v1.1.0:exports/MANIFEST.txt` records both hashes, on lines 21 and 22. The manifest is also attached to the release. Anyone checking a download compares it with those hashes.
4. **Read what has already been said publicly.** The erratum posted on the release on 2026-11-17 already discloses the paths, and says the attached files "are left exactly as released, so their hashes still match the attached `MANIFEST.txt`".
5. **Work out what replacing them would reach.** New uploads would no longer match the manifest, and would contradict the erratum. The tag would still hold the original files, so GitHub's source archive for `v1.1.0` would still contain the lines.
6. **Decide and record it.** The decision is to keep the release files. The erratum covers the release page, and the correction in #39's record covers the repository.

## Verification

1. `git grep -n -E '\]\([A-Za-z]:/' -- docs` finds no Markdown link to a local drive, and `command_overview.md` has no image link left.
2. `git grep -n 'good-first-issue' -- ':!docs/implementations'` prints nothing; the remaining hits are in this record, which quotes the old name. `gh label list` shows `good first issue`.
3. `git grep -c USERNAME -- baseline` finds 3 lines in each `baseline/phase1` netlist: the three standard-library `uri` lines. Their `source` lines are relative paths. Searching `baseline/` for the maintainer's user-folder path finds nothing. This confirms the correction to #39's record.
4. On the pull request's rendered `command_overview.md`, both links jump to the tRRD and tFAW headings in `timing_parameters.md`.
5. `git diff --stat origin/main` lists only the five files in the table above. No design file or export changed.
6. The numbers in "How the two keep-as-is decisions were reached" can be re-run with the commands it names. Each count and hash there was checked on 2026-11-18 against the values in this record.

## Related Docs

- [Keep local paths out of the exports](2026-11-17-export-local-paths.md), the #39 record this corrects
- [Timing parameters](../02_ddr_protocol/timing_parameters.md)
- [Contributing](../../CONTRIBUTING.md)
