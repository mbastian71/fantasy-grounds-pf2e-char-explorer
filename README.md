# Fantasy Grounds PF2e Character Explorer

A single-file, browser-based viewer for **Fantasy Grounds** character exports,
built specifically for the **Pathfinder Second Edition _Remastered_** ruleset.
Drop (or auto-load) the `.xml` file that Fantasy Grounds produces and it renders a
clean, searchable character sheet — then goes further, cross-checking the numbers
and flagging things that look off.

Everything runs locally in your browser. Nothing is uploaded anywhere.

## What it does

- **Displays everything in the XML.** The full character is laid out in readable
  sections: ability scores, defenses (AC / saves / HP), active effects, weapons,
  skills, exploration & other activities, proficiencies, spell statistics and spell
  lists, class features, feats, languages, senses, hero points, currency, the full
  equipment breakdown (worn / carried / by container / not carried, with invested-item
  tracking), notes, and buffs & consumables.
- **Inline rules links.** Spell, feat, and item names are linked to their full rules
  text — hover or tap a dotted-underlined name to read the whole thing without leaving
  the sheet.
- **Advanced search & filtering.** A global search box scans every tab at once, and
  each tab (spells, feats, equipment, …) has its own filter bar with structured
  filters — by trait, save, rank, duration, gear type, and free text — so you can
  answer "which of my spells need a Reflex save?" in a couple of clicks.
- **Limited analysis — a "Data Health" report.** After rendering, it audits the sheet
  and reports issues it can detect, including:
  - **Missing FG automation** — worn/passive items whose bonuses aren't tracked as
    effects, with a suggested automation string to paste back into Fantasy Grounds.
  - **Skill total mismatches** and **hardcoded misc bonuses** not linked to any effect.
  - **Non-stacking item bonuses** (same skill, or among worn items per their text).
  - **Active effects from stowed items** and **effect-name typos** that won't match.
  - **HP** and **armor check penalty** sanity checks.
  - **Signature spell** coverage/duplication for spontaneous casters.

  It won't catch everything — it's a helper, not a rules engine — but it surfaces the
  common bookkeeping mistakes that quietly break a sheet.
- **View or export.** Read it in the browser, flip to raw Markdown, or download the
  sheet as `.md` / the (optionally edited) `.xml`.

## Usage

Open `char_explorer.html` and load a character one of these ways:

- **Drag and drop** a Fantasy Grounds `.xml` export onto the page (several at once
  is fine), or click **Browse file…**.
- **From an `exports/` folder** — put your exports in a folder named `exports/` next
  to the HTML file:
  - Served over http (e.g. `python -m http.server`, then open `http://localhost:8000/char_explorer.html`),
    the tool auto-discovers valid characters in `exports/` and opens the first one.
  - Opened directly as a `file://` double-click, click **Load from exports folder…**
    and pick the folder.

To export from Fantasy Grounds: open the **Characters** list, right-click your
character and choose **Export** (or use `/exportchar` in chat).

> The `exports/` folder is git-ignored — your character files stay local and are
> never committed.

## Tip: signature spells (spontaneous casters)

Fantasy Grounds has **no marker for signature spells**, so this tool infers them from
how you list your spells — and how you list them decides whether you get a clean
overview or a scattered mess.

**Do this** (the standard FG workaround): keep a **heightened copy of each signature
spell at every castable rank**. For example, a signature *Fireball* known at rank 3
gets listed once at rank 3, 4, 5, 6, 7 … up to your top slot. You may optionally
prefix each copy with a marker like `(S)` — `(S) Fireball` — but the marker is
purely cosmetic; detection is marker-agnostic.

The explorer recognises that contiguous run of copies, **collapses it into a single
row with a rank-range badge** (e.g. `Fireball  [ranks 3–7]`), and the **Data Health**
check warns you if a rank is missing or if two different signatures collide at the
same rank.

**Avoid these** — they defeat detection and leave your signatures as unrelated rows:

- **Baking the rank into the name** — `Summon Animal Grad 3`, `Heal Grad 5`. Because
  each name differs, the copies can't be grouped. Use the *same* spell name on every
  copy and let the rank (slot level) do the work.
- **A single entry with a trailing rank suffix** — `Shock to the System (S7)`,
  `Ferrous Form (S8)`. One entry can't show coverage across ranks; the tool only
  merges an actual per-rank run, and a trailing `(S7)` isn't stripped the way a
  leading `(S)` is.
- **A lone placeholder entry** named e.g. `Signature Spells`. It's just a note — the
  tool has nothing to group.

In short: **one copy per rank, contiguous ranks, identical spell name** → the nicest
overview.

## Caveats — please read

- **This is purely vibe-coded.** It was thrown together for personal use, without
  much in the way of tests or rigor. Expect rough edges.
- **Pathfinder 2e Remastered only.** It targets the Pathfinder Second Edition
  *Remastered* ruleset and assumes that data shape. Other systems and pre-Remaster
  content are not supported.
- **Tested against a narrow slice of characters.** It was built around the party in
  my current campaign — Paizo's *Strength of Thousands* Adventure Path — so classes,
  archetypes, and options outside that group are likely to render incorrectly or be
  missing entirely.

Contributions, fixes, and bug reports for other classes/content are welcome.
