# TI list row: horizontal overflow fix (prototype)

A self-contained HTML prototype of the Test Instance list rows in LambdaTest Test Manager (the right pane of the Test Run detail page). It shows the current layout next to the proposed one so you can see the horizontal scroll disappear at both target screen sizes with worst-case data.

Version 2 restyles the page to the `design-context` repo so the rows read as the real TI listing: theme tokens, the component look-alikes the listing pattern names, the mock-data cast and status vocabulary, and a folder pane with the real pane's anatomy. The brief's layout rules, row anatomy, data, controls and checks are unchanged from version 1.

## How to open

- Live: https://rajat-lt.github.io/ti-list-overflow-prototype/
- Locally: double-click `prototype/index.html`. One file, no build step. Icons load from the Lucide CDN, so the machine needs internet access the first time.

## The control bar (top of the page)

Primary toggles

| Control | Options | Default | What it changes |
|---|---|---|---|
| View | Current, Proposed | Proposed | Current replicates today's row: natural-width columns, no `min-width: 0` and no `overflow-wrap: anywhere` on the title block, list scrolls horizontally. Proposed is the fixed layout from the brief. |
| Screen width | 1280, 1512 | 1280 | Sets the outer wrapper to exactly that width. Nothing depends on the real browser window. |
| Folder panel | Open, Closed | Open | Closed removes the 288 px folder pane; the list gets that width. |

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

The pill reads `PASS`, or `FAIL: row 3` with the first failing row. The Current view fails at both widths, folder pane open or closed (row 3 carries a 120+ character unbroken URL). The Proposed view passes in all 32 combinations of the primary and decision toggles. Each render also logs one console table row: screen width, folder panel state, list content width, and the title block width on row 1.

Verified for this version in Chromium at 1280 and 1512 px: 32 of 32 Proposed combinations PASS; no select anchor overflows its column in any combination; hovering a row never changes its height; the hover-revealed split button clears the execute button by at least 4 px on every row; every Lucide icon renders; no console errors.

## Interactions in the prototype

- Assignee: click opens a stub list of five names; picking one updates the row. Truncated names show the full name in a tooltip on hover and on keyboard focus.
- Status: click opens all statuses (built-in and custom), each with its icon; picking one updates the dropdown and the leading icon.
- Execute: green primary when the row has a configuration. Without one it takes the inactive treatment (grey, still focusable) with the tooltip `Add a configuration to run this.`; clicking it repeats that tooltip and does nothing else.
- Bug actions: the split button (bug icon button plus a caret button opening Raise bug, Link issue, View issues) appears on row hover and on keyboard focus within the row, at the right end of the meta line. A count badge shows linked issues.
- `No Configuration added` and every action show a toast (top right, 4 seconds) instead of a real flow.
- Checking a row's checkbox marks it selected (grey fill). The header checkbox selects all and shows the indeterminate state for a partial selection.

## What the rows demonstrate

The 8 rows are the brief's, in the brief's order. Title line counts measured in this version, folder pane open:

| Row | Title | At 1280 px | At 1512 px | Assignee |
|---|---|---|---|---|
| 1 | Login functionality sentence, 103 characters | 2 lines | 2 lines | 41-character name, ellipsized, full name in the tooltip |
| 2 | Licence sentence, with two tags | 2 lines | 1 line | Short name |
| 3 | 120+ character unbroken URL | 3 or more, clamped to 2 with an ellipsis | 3 or more, clamped to 2 | Unassigned |
| 4 | `wdwdqwDEWQD` | 1 line | 1 line | Unassigned |
| 5 | 165-character sentence, with two tags | 3 lines, clamped to 2 | 2 lines | Short name |
| 6 | Blocked payment flow sentence | 1 line | 1 line | Short name |
| 7 | `Skip legacy export` | 1 line | 1 line | Unassigned |
| 8 | In progress row | 1 line | 1 line | Single-word name |

So both widths show one-line, two-line and clamped three-line titles, and both short and ellipsized assignee names. The brief caps titles at two lines, so a three-line title always renders as two lines plus an ellipsis; the `titleLines()` helper on `window.__ti` reports the natural line count behind the clamp.

## What the page is built from

Everything visual comes from `design-context`; the brief's section 9 fallback tokens were not needed.

- **Tokens**: `data/tokens.json`, light scheme. Colours, radii (3, 6, 100 px), the button shadow and the floating/small overlay shadow are copied by their theme path (`fg.muted`, `btn.hoverBg`, `bg.selected`, and so on) into the `:root` block at the top of `index.html`.
- **Type**: the scale in `guidelines/README.md` section 3. Row titles are body/medium-bold (`SUBHEADER_BOLD`, 14/20/600), meta lines and tags are body/small (`SMALL_REGULAR`, 12/18), select anchors and buttons use the 14 px / 500 the Primer button renders. Font stack per the brief; it resolves to SF Pro Text on macOS exactly as the theme stack does.
- **Page structure**: `patterns/test-entity-listing.md`. Rows sit in one bordered container with dividers (delta 2 there); the folder pane carries the pattern's header row, selected root row with its count, folder search and folder rows.
- **Data**: `mock-data.md`. The 12 execution statuses, the cast names, `#TC-` ids, and initials rules for avatars.
- **Icons**: Lucide only, as the brief requires. `icons.md` is still mostly TO FILL, so the picks are named by intent in the `STATUSES` object and the `renderMeta` function.

Component look-alikes, hand-rolled because `@lambdatestincprivate/lt-components` is private:

| Product component | Where | How it is drawn here |
|---|---|---|
| `LTCheckbox` | Row and select-all | 16 px, 3 px radius, `control.borderColor.emphasis` border, accent fill with a white Lucide check; minus for indeterminate |
| `LTTestStatusLabel` | Leading icon, status anchor, status menu | Status component glyphs from `mock-data.md` composed from Lucide: a coloured circle with a white glyph where the description says "circle", a bare coloured glyph otherwise |
| `LTText` | Title, meta | `SUBHEADER_BOLD` and `SMALL_REGULAR` measurements |
| `LTTag` | Tag line | `labelStyle="outline"`, `variant="blueAccent"`, `size="small"`: 20 px pill, accent text, accent-muted border, in a line beneath the title per the brief |
| `LTMetaData` | Meta line | 12/18 muted text with `·` separators |
| `LTSelect` | Assignee and status | The Primer button the anchor renders: `btn.bg`, `btn.border`, 32 px, 500 weight, trailing caret, left-aligned text |
| `LTIconButton` | Execute, bug | `outline` and `primary` variants from the `btn` tokens, `inactive` from `btn.inactive` |
| `LTButtonGroup` | Bug split button | Two attached icon buttons, exactly as the split-button recipe in `guidelines/ltbuttongroup.md` |
| `LTActionMenu` / `LTSelect` overlay | Menus | `canvas.overlay`, floating/small shadow, 256 px, 32 px items, `actionListItem` hover and selected fills |
| `LTAvatar` | Assignee toggle | 20 px initials, never an image |
| `LTCounterLabel` | Folder pane count | 20 px secondary-scheme pill beside its label |
| `LTToast`, `LTTooltip` | Feedback | Dark surface (`neutral.emphasisPlus`), toast top right for 4 seconds |

## Where the brief and design-context disagree

Recorded per the house rule: the brief wins, and every conflict is listed so it can be settled once.

1. **`35 Instances`.** `guidelines/README.md` section 8 says a label that starts with a number keeps the rest lowercase. The brief's string is kept.
2. **Tags beneath the title.** `test-entity-listing.md` renders tags inline after the title; the brief puts them on their own line beneath it. Brief kept.
3. **Row spacing.** The pattern's row is `padding: 16px 12px` with 12 px gaps; the brief's is 16 px horizontal, 12 px vertical, 8 px gutters between the right-side columns. Brief kept; the left group (checkbox, status icon, title) uses the pattern's 12 px because the brief does not specify it.
4. **Status icon size.** The Figma Status component and the pattern are 16 px; the brief's leading icon and anchor icon are 20 px. Brief kept; menu items use 16 px.
5. **Folder pane width.** The pattern confirms 320 px; the brief reserves 288 px (276 plus a 12 px margin). Brief kept, because it is the number the overflow arithmetic depends on.
6. **Execute without a configuration.** The brief calls it disabled; `guidelines/ltbutton.md` and `lticonbutton.md` say never disable, use inactive. It is drawn with the inactive tokens and stays focusable, which is also what makes the brief's tooltip reachable from the keyboard.
7. **Row overflow menu.** The pattern gives every row a trailing `...` menu (slot 6). The brief's description of today's row has none and forbids inventing features, so none is drawn.
8. **Status glyphs.** `mock-data.md` says instantiate the Status component and never redraw it; the brief allows Lucide only. The glyphs are composed from Lucide to the component's written descriptions, and swap out in one object when the assets exist.
9. **Selection treatment.** The guidelines want an orange left bar plus a grey fill; the orange token is still TO FILL, so selected rows and the selected folder use the fill only.
10. **Font stack.** The brief names SF Pro Text; the theme forbids naming a font and ships the system stack. Both resolve to the same face on macOS, so the brief's stack is kept.

## Where this prototype departs from the brief, and why

Each of these is deliberate and reversible in one line.

1. **List content width at 1280 is 864 px, not 912.** The brief's table lists 912 for 1280, but the spacers it specifies (56 sidebar + 24 page margin + 288 folder panel + 24 + 24 right-area margins) leave 864 in a 1280 px wrapper; 912 only results if the right-area margins are dropped at that size. The prototype builds the geometry from the spacers, which is the stricter test, and passes at 864. Constants are in `LAYOUT` if a different assumption is confirmed.
2. **Statuses follow mock-data.md.** The brief's placeholder set (Not Started, In Progress, Passed, Failed, Blocked, Skipped) is replaced by the 12-value execution vocabulary, so `In Progress` renders as `Running`. `Blocked` is not one of the 12 built-ins, so row 6 ships it as a custom status; if it is confirmed built-in, move it into `STATUSES`.
3. **Assignee names come from the mock-data cast** (Darth Vader, Gabbar Singh, Oppenheimer) in place of the brief's names, because mock-data.md forbids inventing people. Row 1 keeps the brief's 41-character worst case, which extends a cast name.
4. **OS, browser and device marks are Lucide stand-ins** (`layers`, `globe`, `smartphone`); `icons.md` treats brand marks as in-house assets, which this file does not have.
5. **Rows with a one-line title and no tags reserve 61 px of title-block height in the hover mode.** The hover-revealed split button sits on the meta line under the execute column; with a one-line title the meta line would overlap the 32 px execute button vertically, so the block reserves room. Rows with two-line titles or a tag line are not affected. The reservation is constant, so hovering never shifts layout. Not applied in the first-line bug mode.
6. **Row 3's URL carries an extra `&sig=` token.** The brief describes the URL as a 120+ character unbroken string, but browsers may break a line after `/` and `-`, so the URL as written has a longest unbreakable run of about 72 characters (its query string). That fits at 1512 px and would hide the failure the Current view exists to show. A 64-character signature token appended to the query string makes the run about 140 characters (1044 px of minimum width), genuinely unbroken, so the Current view overflows at both widths whether the folder panel is open or closed. The meta line is also prevented from dictating row width (`contain: inline-size`), so the Current view fails because of the title, as the brief describes.
7. **Folder rows.** The brief allows a grey placeholder; the pane instead shows the pattern's anatomy so the page reads as the real one. `mock-data.md` has no folder names, so the six names are the scope halves of its test-run names (Login and signup flows, Checkout revamp, Payments regression, Cross-browser sanity, Smoke, Release 8.4 regression) with counts that sum to the 35 instances. The pane is static and inert.

## Design-context gaps this prototype surfaced

Things `design-context` could not answer, worth fixing there rather than here.

- `icons.md`: the approved icon table is TO FILL for Passed, Failed, Running, More actions and Search, and has no entries for OS, browser or device marks. This prototype had to pick Lucide names by intent.
- `guidelines/README.md` section 1: the orange selection-bar colour has no token, so no prototype can draw the selection treatment the hard rule requires.
- `mock-data.md`: no folder names for the Test Manager folder pane; no custom-status examples, although custom statuses exist in the product (the brief names two); `Blocked` is used in the product but is outside the 12-value vocabulary; no mobile device or OS-version values for instance configurations (the brief's Galaxy Z Fold4, iPhone 15 Pro, Android 13.0, iOS 17.4 had to be used as-is).
- `patterns/test-entity-listing.md`: the Test Instances row differs from the brief in tags placement, spacing, status icon size, folder pane width, the `...` menu and the `35 Instances` count label (items 1 to 7 above). Once the overflow fix is accepted, the pattern's slot 5 should carry the fixed columns (assignee 150, status 130, execute 32) and the hover-revealed split button.
- `COMPONENTS.md`: `LTSelect` documents no anchor styling (padding, weight, caret icon), and `LTTestStatusLabel.status` has no value list. Both were assumed from the Primer base and the mock-data vocabulary.

## Open items

- The exact built-in status set, icons and colours are placeholders pending confirmation (now the 12 from mock-data.md, with Lucide glyphs standing in for the Figma Status component).
- The proposed default is status icon + label (130 px). Icon-only is available as a toggle for comparison only; with a shared icon for all custom statuses, icon-only makes custom statuses unreadable without hovering.
- Assignee avatar defaults to hidden per the current decision; the toggle exists so the PM can compare scanability with it shown.
- Hover-revealed bug actions have no touch fallback by design (desktop-only scope).
- Whether custom status names and user display names have a product-level length cap is unknown. The fixed columns plus tooltips handle any length regardless.
- The 1280 px list content width (864 vs 912) needs one confirmation; see departure 1.
- Whether `Blocked` is a built-in or a custom status; see departure 2.
- Which way each conflict in "Where the brief and design-context disagree" should be settled, so the pattern and the brief stop disagreeing.
