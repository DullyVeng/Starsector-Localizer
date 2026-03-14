---
name: starsector-localizer
description: Localize Starsector mods into Simplified Chinese with safe, file-aware handling for CSV, JSON, .faction, descriptions, and Java/JAR-backed UI text. Use when Codex is asked to translate or review Starsector mod text, preserve game-safe UTF-8 without BOM formatting, inspect mod data folders, patch hardcoded tooltip strings, compile updated classes to the correct Java target, and repack JARs without repeated trial-and-error.
---

# Starsector Localizer

Localize Starsector mods conservatively. Prioritize game-safe text assets first, preserve structure exactly, and treat Java or JAR-backed text as high risk.

Load [references/localization-workflow.md](references/localization-workflow.md) when performing actual localization work or terminology review.

## Workflow

1. Inspect the target mod under `mods/<mod-name>` and list localizable files in `data/`.
2. Process files in this order unless the user asks otherwise:
   - `mod_info.json`
   - `data/hulls/ship_data.csv`
   - `data/weapons/weapon_data.csv`
   - `data/hullmods/hull_mods.csv`
   - `data/world/factions/*.faction`
   - `data/shipsystems/ship_systems.csv`
   - `data/strings/descriptions.csv`
3. Preserve delimiters, quoting, placeholders, and schema exactly.
4. Save every edited text file as UTF-8 without BOM.
5. Verify that placeholders like `%s`, `%d`, `$name`, and CSV quotes remain intact.

## File Rules

- For `.csv`, translate text cells only. Keep comma structure and quote closure valid.
- For `.json` and `.faction`, translate only user-facing strings. Do not change keys, arrays, braces, IDs, or numeric values.
- For long lore text in `descriptions.csv`, prefer fluent Chinese but keep placeholders and formatting markers unchanged.
- For scripted UI text in config or strings files, translate only confirmed display strings.

## Hardcoded Text Boundary

Treat `.java` and `.jar` text as dangerous by default.

- Read Java source before choosing a workflow.
- If the mod ships only a compiled jar and no usable source, decompile only the target classes first instead of unpacking or rewriting the whole mod.
- Prefer a pure command-line decompiler path. Avoid GUI-first tools that block automation unless no other option is available.
- A reliable fallback is CFR via `java -jar <cfr.jar> <mod-jar> --outputdir <dir> --silent true`.
- If the mod already ships compiled classes in `jars/*.jar`, assume edited `.java` files will not take effect until the relevant `.class` files are rebuilt and written back into the jar.
- If the source contains modern Java syntax such as `->`, `var`, `yield`, or newer `switch` forms, do not use a Janino recompilation path.
- In that case, prefer normal `javac` compilation to the runtime version actually used by the game, then update the existing jar in place.
- Before touching anything under `jars/`, require a backup copy with a `.original` suffix.
- Never save Java source with BOM.

## Java/JAR Workflow

When tooltip text or hullmod text is hardcoded:

1. Confirm whether the target class exists in both source and `jars/*.jar`.
2. If source is missing, decompile only the needed classes to a writable temp directory and treat that output as temporary source.
3. Translate the source strings first.
4. Scan the target Java files for every user-facing string before compiling, not just the first obvious tooltip block.
5. Specifically inspect `addPrimaryDescription`, `addPostDescriptionSection`, `addEmptySysModText`, `addEmptyMiscModText`, `add*SysModText`, `add*MiscModText`, `getFlavorText`, `getUnapplicableReason`, and any `addSectionHeading`, `addTitle`, or `addPara` calls.
6. If the source came from a decompiler, expect minor compile fixes before translation can build:
   - cast reflective `MethodHandle.invoke(...)` results back to `String`, `Class[]`, or the expected type
   - keep synthetic inner classes such as `$1.class` and helper inner classes when recompiling
   - prefer editing the smallest affected class set instead of trying to rebuild the entire decompiled mod
7. Compile only the changed classes unless a wider rebuild is clearly required.
8. Match the compile target to the runtime supported by the game, not to the host machine's newest JDK.
9. Verify class major version before updating the jar.
10. Replace only the changed class entries inside the jar.
11. Keep the `.original` backup until the user has tested the mod in game.

## Tooltip Sweep Rules

When localizing a hullmod or shipsystem Java file, do a full tooltip sweep before claiming completion.

- Search for remaining English in all display strings after each edit pass.
- Treat "system modification" and "miscellaneous modification" sections as mandatory review areas, because many mods split text across helper methods there.
- Translate both section body text and per-entry titles, not just bullet summaries.
- Check for broken bullet prefixes or mojibake such as `??`, `鈥?`, or other damaged punctuation in `setBulletedListMode` and replace them with safe ASCII like `- `.
- Prefer one final pass that searches for user-facing English in the edited files before compiling and repacking.

Load [references/localization-workflow.md](references/localization-workflow.md) for the concrete compile/repack checklist and version checks.

## Translation Standards

- Use established Starsector terminology consistently.
- Favor concise, game-appropriate Simplified Chinese over literal translation.
- Preserve line breaks, escape sequences, and token order unless the file format clearly allows changes.
- If a translation decision is ambiguous, surface the source string and explain the tradeoff briefly.

## Response Pattern

When asked to localize a mod:

1. Summarize which files will be edited.
2. Call out any hardcoded-text risks early.
3. Apply translations incrementally instead of mixing safe asset edits with risky code edits in one step.
4. Report what was translated, what was intentionally skipped, and any encoding or compile-risk constraints.
5. If Java/JAR files were touched, report the compile target, jar backup path, and whether jar replacement succeeded.

## Reference Use

- Read [references/localization-workflow.md](references/localization-workflow.md) for terminology, file search patterns, and the full SOP.
- Prefer loading only that reference file rather than duplicating its tables into the main skill.
