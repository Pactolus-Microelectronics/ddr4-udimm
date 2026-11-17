# Say what the schematic is, and remove the backup zips

Date: 2026-11-17

## Problem / Motivation

The README, `START_HERE.md` and `CHARTER.md` called the schematic stage "complete". The schematic does pass ERC with 0 violations, and every placed part has a footprint. But passing ERC shows only that the connections are consistent, not that the design is finished. Three things a working DDR4 module needs are not in it:

- **Decoupling:** the BOM (`exports/omi_v1.bom.csv`) lists no capacitors.
- **Termination:** the only resistors are R1 to R8, the 240 Ω ZQ resistors. Nothing terminates the command, address and clock nets, and the Stage 7.2 docs say no termination networks were placed.
- **The SPD device:** `U_SPD0` uses the `Memory_EEPROM:AT24CS01-MAHM` symbol, which its own library description calls a "1Kb (128x8)" EEPROM. It has no manufacturer part number, and no SPD contents exist (Stage 9, not started).

A reader who stopped at "complete" would take the design as more finished than it is. The project rule is that a status word says at what level, and links to the evidence.

Separately, 15 KiCad automatic backup zips (644 KB) were committed under `design/power/omi_v1_power/omi_v1_power-backups/`. No file in the repository refers to them, and git already keeps every committed state of the design. (A backup may also hold a state between two commits. Removing the zips does not lose those either: they stay in history.)

## What Changed

| File | Change |
|---|---|
| `README.md` | Project Status now reads "an ERC-clean connectivity schematic", with an "Open items" list sourced to the BOM and a dated correction of the earlier "complete". Four other "complete" or "completed schematic" phrases were reworded to match. |
| `START_HERE.md` | The introduction and the Stage 7 row say "connectivity captured" and name the open items. Two "completed schematic" phrases were reworded. |
| `CHARTER.md` | The current-status note says "connectivity schematic" and names the open items. |
| `design/power/omi_v1_power/omi_v1_power-backups/*.zip` | Removed: 15 files. |
| `.gitignore` | Ignores `*-backups/`, so KiCad's automatic backups are not committed again. |
| `docs/implementations/2026-11-17-status-wording-and-backups.md` | New. This record. |

## Implementation Approach

The status wording is the same everywhere: "ERC-clean connectivity schematic; decoupling, termination and the SPD device are open items; no layout, no board". The README carries the detail and the evidence, and the other two documents point to it.

The README's correction is added next to the new wording rather than replacing the old wording silently, the same approach the June records' corrections take. "Out of scope and do not exist" became "do not exist yet", because the project now aims to take this design to a measured board.

The zips were removed with `git rm`, so they stay in history: `git log --all -- 'design/power/omi_v1_power/omi_v1_power-backups/*'` lists the commits that added them, and any of them can be restored from those commits.

## Design Decisions

- **Name the open items, and source each one.** "Not finished" alone gives a reader nothing to check. Each item points to something they can open: the BOM, the Stage 7.2 docs, the symbol's own description.
- **No number without a source.** The SPD entry states the EEPROM's size as its library description gives it. It does not compare that with the DDR4 SPD size, because no document in this repository states that size yet. Choosing the SPD device is open work, and that comparison belongs with it.
- **Stage docs and June records are left alone.** The Stage 7 documents and the phase records are a history of the work. The correction belongs on the pages that state the project's current status.
- **Ignore the backup folder** rather than turning KiCad's backups off. That is a per-user KiCad setting, which the repository cannot control.

## Verification

1. `git grep -n -i complete -- README.md START_HERE.md CHARTER.md` finds no remaining claim that the schematic is complete. Every remaining "complete" refers to something else, such as documentation or the charter's goals.
2. `git ls-files '*.zip'` prints nothing. `git check-ignore -v design/power/omi_v1_power/omi_v1_power-backups/x.zip` reports the new `.gitignore` rule.
3. The design is unchanged: `kicad-cli sch erc --severity-all --exit-code-violations` (KiCad 9.0.8) still finds 0 violations and exits 0. No file under `exports/` changed.
4. Each open item can be checked against `exports/omi_v1.bom.csv` (no capacitors; R1 to R8 are the only resistors, all 240 Ω) and against the `U_SPD0` symbol in `design/power/omi_v1_power/spd-and-configuration.kicad_sch` (description "I2C Serial EEPROM, 1Kb (128x8)").

## Related Docs

- [README → Project Status](../../README.md#project-status)
- [Stage 7.2 CA/CLK final](../07_schematic_capture/stage-7-2-ca-clk-final.md)
- [Stage 7.4 SPD final](../07_schematic_capture/stage-7-4-spd-final.md)
