# TI list row: horizontal overflow fix (prototype)

A self-contained HTML prototype of the Test Instance list rows in LambdaTest Test Manager (the right pane of the Test Run detail page). It shows the current layout next to the proposed one so you can see the horizontal scroll disappear at both target screen sizes with worst-case data.

## How to open

- Live: https://rajat-lt.github.io/ti-list-overflow-prototype/
- Locally: double-click `prototype/index.html`. One file, no build step. Icons load from the Lucide CDN, so the machine needs internet access the first time.

## The control bar (top of the page)

Primary toggles

| Control | Options | Default | What it changes |
|---|---|---|---|
| View | Current, Proposed | Proposed | Current replicates today's row: natural-width columns, no `min-width: 0` and no `overflow-wrap: anywhere` on the title block, list scrolls horizontally. Proposed is the fixed layout from the brief. |
| Screen width | 1280, 1512 | 1280 | Sets the outer wrapper to exactly that width. Nothing depends on the real browser window. |
| Folder panel | Open, Closed | Open | Closed removes the 288 px folder panel; the list gets that width. |

Decision toggles (the proposed spec is the default; the alternative exists so you can compare)

| Control | Options | Default | What it changes |
|---|---|---|---|
| Status label | Icon + label, Icon only | Icon + label | Icon only shrinks the status column from 130 px to 48 px and moves the label into a tooltip. |
| Assignee avatar | Hidden, Shown | Hidden | Shown adds a 20 px initials avatar in front of the name, inside the same 150 px column. |
| Bug actions | Hover second line, Always visible first line | Hover, second line | First line adds the split button back as a 60 px first-line column. |

Utility

- **Pin hover state**: forces every row into its hover state so the bug actions are visible for screenshots.
- **Overflow check**: live PASS or FAIL pill. Hover it for the full list of failures.

Everything re-renders immediately. Nothing is persisted; reload to reset.

## What the check measures

After every render and on window resize:

1. The list container: `scrollWidth <= clientWidth`.
2. Every row: `scrollWidth <= clientWidth`.
3. In the Proposed view, every first-line right-group column matches its spec within 1 px: assignee 150, status 130 (48 in icon-only mode), execute 32, bug 60 when it is a first-line column.

The pill reads `PASS`, or `FAIL: row 3` with the first failing row. The Current view is expected to fail at both widths with the folder panel open (row 3 carries a 120+ character unbroken URL). The Proposed view passes in every combination of the toggles. Each render also logs one console table row: screen width, folder panel state, list content width, and the title block width on row 1.

## Interactions in the prototype

- Assignee: click opens a stub list of five names; picking one updates the row. Truncated names show the full name in a tooltip.
- Status: click opens all statuses (built-in and custom); picking one updates the dropdown and the leading icon.
- Execute: green when the row has a configuration; grey and inert when it does not, with the tooltip `Add a configuration to run this.` The inert button uses `aria-disabled` rather than `disabled` so keyboard users can still reach its tooltip.
- Bug actions: the split button (bug icon button plus a caret button opening Raise bug, Link issue, View issues) appears on row hover and on keyboard focus within the row, at the right end of the meta line. A count badge shows linked issues.
- `No Configuration added` and every action show a toast instead of a real flow.
- Checking a row's checkbox marks it selected (grey fill).

## Data and vocabulary

- Row data is one array (`ROWS`) at the top of the script in `index.html`; the status model is one object (`STATUSES`) next to it. Edit values there, not the markup.
- Statuses are the 12 execution statuses from `design-context/mock-data.md`, which apply to test instances. People are the mock-data cast. Tokens come from `design-context/data/tokens.json` (light scheme).

## Where this prototype departs from the brief, and why

Each of these is deliberate and reversible in one line.

1. **List content width at 1280 is 864 px, not 912.** The brief's table lists 912 for 1280, but the spacers it specifies (56 sidebar + 24 page margin + 288 folder panel + 24 + 24 right-area margins) leave 864 in a 1280 px wrapper; 912 only results if the right-area margins are dropped at that size. The prototype builds the geometry from the spacers, which is the stricter test, and passes at 864. Constants are in `LAYOUT` if a different assumption is confirmed.
2. **Statuses follow mock-data.md.** The brief's placeholder set (Not Started, In Progress, Passed, Failed, Blocked, Skipped) is replaced by the 12-value execution vocabulary, so `In Progress` renders as `Running`. `Blocked` is not one of the 12 built-ins, so row 6 ships it as a custom status; if it is confirmed built-in, move it into `STATUSES`.
3. **Assignee names come from the mock-data cast** (Darth Vader, Gabbar Singh, Oppenheimer) in place of the brief's names, because mock-data.md forbids inventing people. Row 1 keeps the brief's 41-character worst case, which extends a cast name.
4. **Status icons are Lucide stand-ins.** mock-data.md points at the Figma Status component for status glyphs; the brief allows Lucide only. The Lucide picks follow the glyph descriptions (green circled tick, red circled cross, and so on). OS, browser and device marks are also Lucide stand-ins (`layers`, `globe`, `smartphone`); icons.md treats brand marks as in-house assets, which this file does not have.
5. **Rows with a one-line title and no tags reserve 61 px of title-block height in the hover mode.** The hover-revealed split button sits on the meta line under the execute column; with a one-line title the meta line overlaps the 32 px execute button vertically, so the block reserves room. Rows with two-line titles or a tag line are not affected. The reservation is constant, so hovering never shifts layout. Not applied in the first-line bug mode.
6. **Small house-style alignments from guidelines/README.md**: the count reads `35 instances` (a label starting with a number keeps the rest lowercase), meta text uses the 12/18 body-small style, `border.default` and `danger.fg` use the tokens.json values (`#d1d9e0`, `#d1242f`) rather than the brief's section 9.
7. **Selected rows use the theme's `bg.selected` fill only.** The guidelines call for an orange left bar as well, but its token is still marked TO FILL there, so no colour was invented.
8. **Row 3's URL carries an extra `&sig=` token.** The brief describes the URL as a 120+ character unbroken string, but browsers may break a line after `/` and `-`, so the URL as written has a longest unbreakable run of about 72 characters (its query string). That fits at 1512 px and would hide the failure the Current view exists to show. A 64-character signature token appended to the query string makes the run about 140 characters (1044 px of minimum width), genuinely unbroken, so the Current view overflows at both widths whether the folder panel is open or closed. The meta line is also prevented from dictating row width (`contain: inline-size`), so the Current view fails because of the title, as the brief describes.

## Open items

- The exact built-in status set, icons and colours are placeholders pending confirmation (now the 12 from mock-data.md, with Lucide glyphs standing in for the Figma Status component).
- The proposed default is status icon + label (130 px). Icon-only is available as a toggle for comparison only; with a shared icon for all custom statuses, icon-only makes custom statuses unreadable without hovering.
- Assignee avatar defaults to hidden per the current decision; the toggle exists so the PM can compare scanability with it shown.
- Hover-revealed bug actions have no touch fallback by design (desktop-only scope).
- Whether custom status names and user display names have a product-level length cap is unknown. The fixed columns plus tooltips handle any length regardless.
- The 1280 px list content width (864 vs 912) needs one confirmation; see departure 1.
- Whether `Blocked` is a built-in or a custom status; see departure 2.
