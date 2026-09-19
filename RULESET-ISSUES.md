# Suspected Fantasy Grounds PF2 ruleset / library bugs

Things found while building this explorer that look like bugs in **Fantasy Grounds
itself** rather than in a character sheet. Recorded here so they aren't rediscovered,
and so the Data Health checks that work around them have something to point at.

Read from source and from real exports. **None of it has been reproduced by running
Fantasy Grounds** — there's no FG installation here — so every "crashes" or "never
applies" below is inferred from the code, not observed in the app. Confidence is
stated per item.

Ruleset examined: `PFRPG2/` — `<root version="3.0" release="21.0">`, announcement
string *"Pathfinder RPG Second Edition ruleset (2026-06-22) (PF2 Release 21)"*.

---

## 1. Ability-score automation is unimplemented, and reaching it raises a Lua error

**Confidence: high on the code shape, inferred on the crash.**

`scripts/manager_automation_pfrpg2.lua:104`

```lua
function processAbilityAutomation(nodeChar, nodeEntry, sData)
	GlobalDebug.consoleObjects("AutomationManagerPFRPG2.processAbilityAutomation.  nodeChar, nodeEntry, sData = ", nodeChar, nodeEntry, sData);


end
```

The body is empty, and `onInit` (`:2070`–`:2118`) registers every other handler with
`AutomationManager.setCustomAutomationHandler(...)` but never registers this one.

The dispatcher at `:1942` then does:

```lua
if StringManager.contains(DataCommon.abilities, sAutomationName) then
	return processAbilityAutomation["ability"](nodeChar, nodeEntry, sData);
```

`processAbilityAutomation` is a **function**, and it is being indexed with the string
key `"ability"`. In Lua that is `attempt to index a function value`, so an automation
string whose first word is an ability name — `strength`, `dexterity`, `constitution`,
`intelligence`, `wisdom`, `charisma` (`scripts/data_common.lua:16`) — should error
rather than silently do nothing. Two commented-out copies of the same expression sit
just below it at `:1946` and `:2018`, so this looks like an unfinished refactor.

**Consequence.** Nothing can automate an ability score. Apex items are the obvious
casualty: *Crown of Intellect* says *"increase your Intelligence modifier by 1 or
increase it to +4, whichever would give you a higher value"*, and the item ships with
a `<howtouse>` telling the player to CTRL+mouse-wheel the field on the Main tab by
hand. That manual step is invisible to everything else and easy to forget on a
re-import.

**Worked around by** `checkApexAbility` in `char_explorer.html`, which reconciles
invested apex items against the hand-entered `miscmod` / `bonusmodifier`.

---

## 2. Stock library effects use `SKILL:` for Perception, where only `PERC:` resolves

**Confidence: high.** This is the one with real evidence on both sides.

A Perception roll and a skill roll read **different effect tags**.
`scripts/manager_action_skill.lua:191`:

```lua
local aSKILLEffectsUntyped, aSKILLEffectBonuses, aSKILLEffectPenalties, nEffectCount =
	EffectManager.getBonusData(rSource, bIsPerception and "PERC" or "SKILL", tSrcEffData);
```

`PERC` and `SKILL` are separate namespaces, chosen by *which roll is being made* —
not by what the effect's target text says. So a clause reading
`SKILL: -2 perception, status` is never consulted on a Perception check. It shows on
the sheet and does nothing. `scripts/manager_action_init.lua:151` makes the same
split for initiative rolled with Perception.

Three effects in one export carry exactly that form, and all three sit on entries
marked `locked=1` with a Paizo `source`, i.e. supplied by the library modules rather
than typed by the player:

| Entry | Source | Stored label |
|---|---|---|
| Bon Mot | Player Core 2 | `Bon Mot; SKILL: -2 perception, status; SAVE: -2 will, status` |
| Bon Mot | Player Core 2 | `Bon Mot; SKILL: -3 perception, status; SAVE: -3 will, status` |
| Honeyed Words | Player Core | `to Discern Lie; SKILL: +4 circumstance, perception` |

The `SAVE: -2 will` half of Bon Mot works. The Perception half does not — so the
feat silently delivers half its effect.

What makes this look like a library bug rather than a convention is that **other
stock entries get it right**. Every sheet checked carries
`PERC: +4 circumstance, lie` and `PERC: +4 circumstance, create a diversion` from the
library, and two sheets carry `PERC: -1 status` / `PERC: -2 status`. So the correct
form is in use elsewhere in the same data set.

Correct spellings would be `PERC: -2 status`, `PERC: -3 status`, and
`PERC: +4 circumstance`.

Two similar-looking constructs that are **not** bugs and should not be "fixed":

- `[SKILL perception expert]` — proficiency automation, a different syntax entirely
  (no colon), handled by `processSkillProfAutomation`. It sets a rank.
- `INIT: +2 circumstance, perception` on Battlefield Surveyor — resolves against the
  `INIT` tag, where `perception` is a filter word meaning "when initiative is rolled
  using Perception". Correct as written.

**Surfaced by** `checkPerceptionEffectKey` in `char_explorer.html`.

---

## 3. Pathfinder 1e class data still shipping in a PF2-only ruleset

**Confidence: high that it's dead; low that it matters.** Cosmetic, listed only so
nobody mistakes it for a usable PF2 progression table.

`scripts/data_common.lua:1038`–`~1094` carries a per-class table of Pathfinder
*First* Edition statistics:

```lua
["barbarian"] = {
	hd = "d12", bab = "fast", fort = "good", ref = "bad", will = "bad", skillranks = 4,
	skills = "Acrobatics (Dex), Climb (Str), Craft (Int), Handle Animal (Cha), ...",
},
```

`bab`, `skillranks`, saves as good/bad, and PF1e skills (Appraise, Linguistics,
Use Magic Device, Knowledge (arcana)) have no meaning in the Remaster. Harmless
if unused — but anything reading this table for PF2 class progression would be
wrong, so don't.

---

## Not a ruleset bug, recorded to avoid a re-diagnosis

`exports/` has shown spell names holding a literal `\n` (backslash-n, two
characters) and, previously, a spliced-in `Tevok.xml`. That is export or hand-edit
residue in the character file, **not** a ruleset defect. `checkNameArtifacts` in
`char_explorer.html` reports it, because the parsers strip it and nothing else on the
page would otherwise mention it.
