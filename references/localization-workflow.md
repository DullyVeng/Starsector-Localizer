# Starsector Localization Workflow

## Core Rules

- Save all text files as UTF-8 without BOM.
- Keep CSV delimiters and quote pairing valid.
- Preserve JSON and `.faction` structure exactly.
- Keep placeholders such as `%s` and `%d` unchanged.

## Recommended File Order

1. `mod_info.json`
2. `data/hulls/ship_data.csv`
3. `data/weapons/weapon_data.csv`
4. `data/hullmods/hull_mods.csv`
5. `data/world/factions/*.faction`
6. `data/shipsystems/ship_systems.csv`
7. `data/strings/descriptions.csv`

## Search Patterns

- Ship names: `data/hulls/ship_data.csv`
- Weapon names: `data/weapons/weapon_data.csv`
- Lore text: `data/strings/descriptions.csv`
- Faction ranks and titles: `data/world/factions/*.faction`
- Hullmod descriptions: `data/hullmods/hull_mods.csv`

## Terminology

### Resources

| English | Chinese |
| --- | --- |
| Flux | 幅能 |
| Hardflux | 硬幅能 |
| CR (Combat Readiness) | 作战准备值 |
| OP (Ordnance Points) | 配置点数 |
| Hullmod | 船体插件 |
| Vent | 排能 |
| Overload | 超载 |

### Ship Classes

| English | Chinese |
| --- | --- |
| Frigate | 护卫舰 |
| Destroyer | 驱逐舰 |
| Cruiser | 巡洋舰 |
| Capital Ship | 战列舰 |
| Battlecruiser | 战列巡洋舰 |
| Carrier | 航空母舰 |
| Tanker | 油轮 |
| Freighter | 货船 |

### Factions

| English | Chinese |
| --- | --- |
| Hegemony | 霸主 |
| Persean League | 英仙座联盟 |
| Tri-Tachyon | 速子科技 |
| Luddic Church | 卢德教会 |
| Luddic Path | 卢德开拓者 |
| Sindrian Diktat | 辛达强权 |

## Hardcoded Text Safety

- Static text in CSV, JSON, and faction files should be localized.
- Java or JAR-backed dynamic text is high risk.
- Inspect Java source before proposing edits to hardcoded text.
- If the mod ships both source files and `jars/*.jar`, treat the jar as the real runtime artifact unless proven otherwise.
- If Java source uses syntax like `->`, `var`, `yield`, or newer `switch` forms, do not rely on Janino recompilation after stripping JAR contents.
- In those cases, localize safe static text first, then use normal `javac` plus jar replacement for hardcoded tooltip text.
- Back up any file under `jars/` before modification by creating a `.original` copy.
- Never save Java source with BOM.

## JAR Localization SOP

Use this exact sequence when a Starsector mod requires source edits plus jar replacement.

1. Locate runtime artifacts.
   - Read `mod_info.json`.
   - If `jars` is declared, inspect those jar files first.
   - Confirm whether the changed class path already exists inside the jar.

2. Decide whether jar replacement is necessary.
   - If the class exists only in source and the mod is known to rely on runtime source compilation, source-only edits may work.
   - If the class exists in the jar, rebuild and update the jar.

3. Back up the jar before any compile or update step.
   - Example pattern: `Copy-Item <jar> <jar>.original -Force`

4. Build a minimal classpath.
   - Include core Starsector jars from `starsector-core/`.
   - Include dependency mod jars such as `LazyLib`, `GraphicsLib`, or other libs referenced by the target source.
   - Include the target mod jar itself so unchanged dependencies resolve from compiled classes.

5. Compile only the changed classes when possible.
   - Prefer `javac --release 17` for Starsector 0.98a unless local evidence shows a different runtime target.
   - Do not compile with the host default target if it produces a newer class version than the game supports.
   - Use `-encoding UTF-8 -implicit:none -sourcepath ''` to avoid accidental recompilation of unrelated source trees.

5a. Before compiling, sweep for untranslated UI strings.
   - Search edited files for user-facing English in:
     - `addPara("...")`
     - `addTitle("...")`
     - `addSectionHeading("...")`
     - `return "..."`
   - Review helper methods that often hide leftover text:
     - `addPrimaryDescription`
     - `addPostDescriptionSection`
     - `addEmptySysModText`
     - `addEmptyMiscModText`
     - `add*SysModText`
     - `add*MiscModText`
     - `getFlavorText`
     - `getUnapplicableReason`
   - Do not stop after translating the first visible block. Many Starsector mods distribute tooltip text across several helper methods.

6. Verify class version before updating the jar.
   - Check that the generated class major version matches the game's runtime.
   - For Java 17, the major version must be `61`.

7. Update the jar in place.
   - Replace only the changed `.class` entries, including nested classes such as `$LocalData.class` when they were regenerated.

8. Clean temporary build output.
   - Remove temporary build folders after the jar update succeeds.

9. Report exact outcomes.
   - State which source files changed.
   - State which jar entries were replaced.
   - State the backup path.
   - State the compile target and any remaining untranslated dynamic text.

## Compile Guardrails

- If the game reports `UnsupportedClassVersionError` and says it recognizes up to class file version `61.0`, recompile with `--release 17`.
- Do not use `--release 7` unless the actual runtime or project proves that requirement.
- If `javac` starts recompiling the whole dependency graph, add `-implicit:none -sourcepath ''` and keep the target mod jar on the classpath.
- If a tooltip prefix renders as `??` or other mojibake, inspect the source for damaged bullet characters and replace them with safe ASCII such as `- `.
- Prefer ASCII bullet prefixes in code-generated UI strings unless the file is already known-good with Unicode.
- After each major translation pass, run one last search for user-facing English in the edited Java files before recompiling.
- Pay special attention to sections equivalent to `系统改装` and `杂项改装`; these commonly contain the largest untranslated helper blocks.

## Verification Checklist

- `mod_info.json` inspected
- target class confirmed inside jar
- `.original` backup created
- changed source saved as UTF-8 without BOM
- full tooltip/helper string sweep completed
- `javac --release 17` succeeded
- generated class major version checked
- jar updated with changed classes only
- temp build directory removed
- user told to test in game

## Working Prompt Template

Use this task framing when operating on a mod:

`Please localize mods/<mod-name> according to this workflow. First inspect data files, then translate names, hullmods, and descriptions in batches, while preserving UTF-8 without BOM and consistent terminology.`
