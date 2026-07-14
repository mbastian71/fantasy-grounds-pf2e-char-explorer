# Fantasy Grounds Character Explorer

A single-file, browser-based viewer for **Fantasy Grounds** character exports. Drop
(or auto-load) the `.xml` file that Fantasy Grounds produces and it renders a clean,
searchable character sheet — abilities, skills, feats, spells, inventory, and
automation — with rules text linked inline (hover or tap a name for the full spell,
feat, or item description).

Everything runs locally in your browser. Nothing is uploaded anywhere.

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
