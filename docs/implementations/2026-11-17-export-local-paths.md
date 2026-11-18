# Keep local paths out of the exports

Date: 2026-11-17

## Problem / Motivation

The committed netlists, `exports/omi_v1.netlist.xml` and `exports/omi_v1.net`, contained absolute paths from the machine that made them, in five places each:

- the schematic `source`, with the full project path under the maintainer's user folder;
- the library `uri` of the project's own symbol library;
- the library `uri` of three KiCad standard libraries (`Device`, `Memory_EEPROM`, `Memory_RAM`). KiCad was installed per user, so this path also contains the user name.

That publishes the maintainer's Windows user name and folder layout, and makes every export differ by machine. The library tables already write the project library as `${KIPRJMOD}/omi.kicad_sym`, a path relative to the project. The absolute paths come from `kicad-cli`, which expands path variables when it writes a netlist, so the fix belongs in the export script, not in the tables.

## What Changed

| File | Change |
|---|---|
| `tools/export/export.ps1` | After the netlist export, rewrites the project directory to `${KIPRJMOD}` and KiCad's symbol directory to `${KICAD9_SYMBOL_DIR}` in both netlists. Adds a gate that fails the run if a text artifact still contains the repository path or the user-profile path. The manifest gains a "Local paths" line. |
| `tools/export/README.md` | New "No local paths" section describing the rewrite and the gate. |
| `exports/erc.json`, `exports/omi_v1.netlist.xml`, `exports/omi_v1.net`, `exports/omi_v1.schematic.pdf`, `exports/MANIFEST.txt` | Regenerated with KiCad 9.0.8 by the changed script. `exports/omi_v1.bom.csv` came out byte-identical and is unchanged. |
| `docs/implementations/2026-11-17-export-local-paths.md` | New. This record. |

## Implementation Approach

The rewrite runs straight after the two netlist exports.

1. **Work out the two directories.** The project directory is the folder that holds the root schematic. KiCad's symbol directory is `share\kicad\symbols` beside the `kicad-cli` binary in use.
2. **Replace them as literal strings.** Each directory is replaced with KiCad's own name for it: `${KIPRJMOD}`, and `${KICAD9_SYMBOL_DIR}`, KiCad 9's variable for that folder. The `.net` format escapes each backslash as two, so its search strings are doubled; the XML is not escaped.
3. **Leave everything else alone.** No regular expressions are used. Files are read and written as UTF-8 without a byte-order mark, so line endings and every other byte are unchanged.

The gate runs after the PDF export and before the manifest. It searches `erc.json`, both netlists and the BOM for the repository root and for `%USERPROFILE%`, in plain and doubled-backslash form, ignoring case. A hit stops the script with a non-zero exit and no manifest.

The PDF is not covered by the gate, because its text streams are compressed. It was checked by hand for this change instead; see Verification.

## Design Decisions

- **Post-process rather than reconfigure KiCad.** In KiCad 9.0.8, `kicad-cli sch export netlist` takes only an output file and a format (`--help`), so it has no option to keep path variables unexpanded. Changing the library tables would have no effect, because they are already relative.
- **KiCad's own variable names, not a placeholder of our own.** `${KIPRJMOD}` and `${KICAD9_SYMBOL_DIR}` mean something to anyone who reads KiCad files, and they match how the library tables write paths.
- **A gate as well as a fix.** The first run of the fixed script, which rewrote only the project directory, failed this gate on the standard-library paths. They had been in the committed netlists all along without being noticed. A rewrite on its own can be incomplete; the gate makes an incomplete one fail loudly.
- **The old evidence keeps its paths.** Netlists under `build/` and `baseline/` record the June work and are evidence, so they are not edited. The paths also remain in the git history. This change stops new exports from carrying them; it does not remove what is already published.

## Verification

1. `powershell.exe -ExecutionPolicy Bypass -File tools\export\export.ps1` with KiCad 9.0.8 exits 0. It prints the ERC gate (0 violations) and the new local-path gate as passed.
2. The regenerated files differ from the previous export only where they should:
   - **Netlists:** only in the `source`, `date` and five `uri` lines.
   - **`erc.json`:** only in `date`.
   - **PDF:** only in 9 bytes, all in its creation date. Searching the PDF's raw bytes and its decompressed streams finds no local path.
   - **BOM:** byte-identical.
3. Every SHA-256 in `exports/MANIFEST.txt` matches its file (`sha256sum exports/<file>`). The manifest names `ccc862c`, the commit that changed the script; the regenerated files are committed in the commit after it.
4. Gate test: the first run, which rewrote only the project directory, stopped with `LOCAL-PATH GATE FAILED: omi_v1.netlist.xml contains a local path` and wrote no manifest.

## Related Docs

- [Export script usage](../../tools/export/README.md)
- [Phase 6 record](2026-06-02-phase6-exports.md), which introduced the pipeline

## Correction (2026-11-17, later)

The Design Decisions above say the netlists under `build/` and `baseline/` keep their paths. That holds for `build/p2` to `build/p5`: 18 lines across four files, kept as June evidence. It does not hold for `baseline/phase1/`. Both netlists there were rewritten in [#28](https://github.com/Pactolus-Microelectronics/ddr4-udimm/pull/28) on 2026-06-01, so the user folder reads `USERNAME`; the original lines are in history at `ca4925a`. The `v1.1.0` release files `omi_v1.net` and `omi_v1.netlist.xml` also carry five path lines each. They stay as released, because the release's SHA-256 digests match the tagged `exports/` files, whose hashes `MANIFEST.txt` records. See [2026-11-17-local-path-followups.md](2026-11-17-local-path-followups.md).
