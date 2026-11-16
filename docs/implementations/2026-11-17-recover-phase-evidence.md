# Recover the phase 3 and 5 evidence, and correct three records

Date: 2026-11-17

## Problem / Motivation

The implementation records for phases 3, 4 and 5 (June 2026) each list `build/` evidence as added that the repository does not contain:

- **Phase 3** lists `build/p3/erc_*.json` as "per-step ERC" evidence for the ERC 135 to 85 table, but only the baseline and final reports were committed.
- **Phase 4** lists a "pad-match report" that was never saved anywhere.
- **Phase 5** lists `build/p5/*` as per-step ERC and netlist evidence, a symbol re-verify and a ledger, but only the baseline, final ERC and final netlist were committed.

A reader following those records to the evidence would not find it. That breaks the project rule that a claim links to the artifact that backs it.

The missing files were not lost. They stayed untracked in the old local checkout and were kept, checked byte for byte, when that checkout was retired on 2026-11-17.

## What Changed

| File | Change |
|---|---|
| `build/p3/erc_step01_alert_nc.json` to `build/p3/erc_step08_vref_merge.json` (8 files) | New. Recovered per-step ERC reports for phase 3 steps 1 to 8. |
| `build/p3/erc_step09_rm_jpwr.json` | New. A recovered intermediate report at step 9 (89 violations) that the phase 3 table does not list. |
| `build/p5/erc_flag1.json`, `erc_nc_ties.json`, `erc_reverted.json`, `erc_test_dram0.json`, `erc_test_libonly.json`, `erc_test_powout.json`, `erc_vddq.json`, `erc_zq.json`, `erc_zq2.json` | New. Recovered intermediate ERC reports from phase 5. |
| `docs/implementations/2026-06-01-phase3-integration.md` | Appended a dated correction: what was committed, what was recovered, and the reproduction result. |
| `docs/implementations/2026-06-01-phase4-footprints.md` | Appended a dated correction: the pad-match report does not exist; the result it stood for, re-run. |
| `docs/implementations/2026-06-01-phase5-erc.md` | Appended a dated correction: what was recovered, what is held back, and where the re-verify and ledger are. |
| `docs/implementations/2026-11-17-recover-phase-evidence.md` | New. This record. |

## Implementation Approach

1. Every untracked file under the old checkout's `build/` was compared against what each record claims. No tracked file names any of them individually; the records claim them by glob.
2. Only files a record claims were added. The phase 3 records claim ERC reports, not per-step netlists, so the step netlists were not added. Helper scripts, PR-body drafts and scratch files stay out: no record lists them as added, and the scripts' own headers call them uncommitted scratch.
3. Files that duplicate tracked ones were not added. They were compared field by field after parsing, ignoring `date`:
   - `erc_step00_baseline.json` is byte-identical to `build/p3/erc_baseline.json`.
   - `erc_step09_rm_jpwr_block.json` matches `build/p3/erc_final.json`.
   - `build/p5/erc_preflight.json` is byte-identical to `build/p5/erc_baseline_85.json`.
   - `build/p5/erc_flags.json` matches `build/p5/erc_final.json`.
4. The nine phase 5 intermediate netlists were held back. Like the tracked netlists, they embed the absolute library path of the machine that made them. Adding them would add new permanent copies of the path that a separate correction is about to remove. They were not edited either, because an altered evidence file is not evidence.
5. The old records were not rewritten. Each gets a dated "Correction (2026-11-17)" section at its end. Nothing under `validation/evidence/` was touched; that is the only directory covered by the evidence hash chain.

## Design Decisions

- **Recover rather than regenerate.** The phase 3 steps can be regenerated from their commits, and were (see Verification), but the recovered files are the reports the June work actually produced. Committing them, and recording that a re-run reproduces them, is a stronger record than either alone. The phase 5 intermediates cannot be regenerated at all, since they were never committed states.
- **Correct by appending, not by editing the June text.** The original wording stays visible next to its correction, the same approach the archive notice took for "schematic complete".
- **Hold the netlists, do not redact them.** See step 4 above.

## Verification

1. **Phase 3 reproduces.** At each of the ten commits named in the phase 3 table (`bcff461` to `6473a5d`), checked out in a temporary worktree, `kicad-cli sch erc --severity-all --format json` (KiCad 9.0.8) gave 135, 134, 133, 132, 125, 118, 100, 89, 88 and 85 violations. That is the table exactly. Every step's report matched the recovered or tracked file in every field except `date`, apart from step 8. There, one `same_local_global_label` violation names a different one of the eight equivalent `VREF` global labels; the count and the per-sheet totals are the same.
2. **Phase 5 recovered files.** Violation counts: `erc_test_powout.json` 210, which is the spike the record describes in section 5a. `erc_reverted.json` 85, `erc_test_libonly.json` 86, `erc_test_dram0.json` 76, `erc_nc_ties.json` 13, `erc_zq.json` 13, `erc_zq2.json` 5, `erc_vddq.json` 4, `erc_flag1.json` 3. All report `kicad_version` 9.0.8 and a `date` on 2026-06-01.
3. **Generators, re-run at `89c3d42`.**
   - `python tools/edge_symbol/gen_edge_symbol.py --verify` printed `VERIFY: PASS`, with 288/288 pins matched.
   - `python tools/edge_footprint/gen_edge_footprint.py --verify` printed `VERIFY OK: 288 pads, numbers 1..288 == CSV pins, faces 144 F.Cu / 144 B.Cu`. The footprint it regenerates first was identical to the committed file apart from line endings.
4. **The committed files are the recovered files.** The recovered copies have Windows (CRLF) line endings. Git stores them with LF, as it stores every other tracked file under `build/` (`git ls-files --eol build` shows `i/lf`). For each added `build/` file, the archive copy with its carriage returns removed (`tr -d '\r' < <copy> | git hash-object --no-filters --stdin`) equals the blob in this commit. Nothing else differs.

## Related Docs

- [Phase 3 record](2026-06-01-phase3-integration.md), [phase 4 record](2026-06-01-phase4-footprints.md), [phase 5 record](2026-06-01-phase5-erc.md)
- [Implementation-record template](_TEMPLATE.md)
