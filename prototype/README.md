# TI list row: horizontal overflow fix (prototype)

A self-contained HTML prototype of the Test Instance list rows in LambdaTest Test Manager (the right pane of the Test Run detail page). It shows the current layout next to the proposed one so you can see the horizontal scroll disappear at both target screen sizes with worst-case data.

Version 3 applies the 28 Sep 2026 review, which compared version 2 with the live Test Instance listing in a test run: the live page's horizontal geometry, invisible buttons for assignee and status, an Unassigned option, no count badge on the bug button, and two first-line bug action modes. Version 2 restyled the page to the `design-context` repo. Everything else in the brief still holds; what the review changed is listed under "Changes from the 28 Sep 2026 review".

## How to open

- Live: https://rajat-lt.github.io/ti-list-overflow-prototype/
- Locally: double-click `prototype/index.html`. One file, no build step. Icons load from the Lucide CDN, so the machine needs internet access the first time.

## The control bar (top of the page)

Primary toggles

| Control | Options | Default | What it changes |
|---|---|---|---|
| View | Current, Proposed | Proposed | Current replicates today's row: natural-width columns, no `min-width: 0` and no `overflow-wrap: anywhere` on the title block, list scrolls horizontally. Proposed is the fixed layout. |
| Screen width | 1280, 1512 | 1280 | Sets the outer wrapper to exactly that width. Nothing depends on the real browser window. |
| Folder panel | Open, Closed | Open | Closed removes the folder pane and its 12 px left margin (312 px at 1512, 264 px at 1280); the list gets that width. |

Decision toggles (the proposed spec is the default; the alternative exists so you can compare)

| Control | Options | Default | What it changes |
|---|---|---|---|
| Status label | Icon + label, Icon only | Icon + label | Icon only shrinks the status column from 130 px to 48 px and moves the label into a tooltip. |
| Assignee avatar | Hidden, Shown | Hidden | Shown adds a 20 px initials avatar in front of the name, inside the same 150 px column. Unassigned rows show a user icon in that slot only when avatars are shown. |
| Bug actions | Visible on hover first line, Always visible first line | Visible on hover, first line | Visible on hover: the split button is the first action item in each row, before assignee. Its 60 px column and 8 px gutter are always reserved, and the button appears on row hover and on keyboard focus within the row. Always visible: the split button sits between status and execute, where the live page has it. |

Utility

- **Pin hover state**: forces every row into its hover state so the bug actions are visible for screenshots.
- **Overflow check**: live PASS or FAIL pill. Hover it for the full list of failures.

Everything re-renders immediately. Nothing is persisted; reload to reset.

## Layout

| Screen | Rail | Folder pane left margin | Folder pane | List left margin | List content | List right margin |
|---|---|---|---|---|---|---|
| 1512 | 56 | 12 | 300 | 24 | **1096** | 24 |
| 1280 | 56 | 12 | 252 | 24 | **912** | 24 |

With the folder pane closed the list content is 1408 px at 1512 and 1176 px at 1280.

The 1512 values are the live page's. Measured against the review screenshot (2000 px wide, so 1.32275 image px per CSS px), the folder pane's right edge sits at 367.8 px and the list container runs from 392.7 to 1487.4 px, matching 56 + 12 + 300 = 368, 368 + 24 = 392 and 1512 - 24 = 1488.

The 1280 folder pane is the 1512 pane scaled by the width to the right of the fixed 56 px rail: 300 x (1280 - 56) / (1512 - 56) = 252.2, rounded to 252, a multiple of 4. The margins are spacing, so they stay 12 and 24. This also reproduces the brief's section 3 row for 1280 exactly: a 960 px right area (912 + 48) and 912 px of list content. Scaling by the whole screen width instead (300 x 1280 / 1512 = 254) would give 910. Every number is in `LAYOUT` at the top of the script.

`design-context` specifies a 320 px folder pane (`patterns/test-entity-listing.md` section 6, and the side-pane scale in `guidelines/README.md` section 2). This prototype uses the live values by decision, and `design-context` is deliberately not updated.

## What the check measures

After every render and on window resize:

1. The list container: `scrollWidth <= clientWidth`.
2. Every row: `scrollWidth <= clientWidth`.
3. In the Proposed view, every first-line right-group column matches its spec within 1 px: assignee 150, status 130 (48 in icon-only mode), execute 32, and bug 60, which is a first-line column in both bug modes.

The pill reads `PASS`, or `FAIL: row 3` with the first failing row. The Current view fails at both widths, folder pane open or closed (row 3 carries a 120+ character unbroken URL). The Proposed view passes in all 32 combinations of the primary and decision toggles. Each render also logs one console table row: screen width, folder panel state, list content width, and the title block width on row 1.

Verified for this version in Chromium at 1280 and 1512 px: the layout table above to the pixel; 32 of 32 Proposed combinations PASS; no assignee or status button overflows its column in any combination; in hover mode the bug column is first, hidden at rest, shown on hover, on keyboard focus within the row and with Pin hover state, and revealing it changes no row height and moves no column; picking Unassigned from the menu by mouse and by keyboard updates the row and returns focus to the button; no count badges; every Lucide icon renders; no console errors.

## Interactions in the prototype

- Assignee: an invisible button (no border or fill until hover). Click opens a menu with Unassigned first, a divider, then five names; the current value is checked and focused. Picking one updates the row. Truncated names show the full name in a tooltip on hover and on keyboard focus.
- Status: an invisible button in both label modes. Click opens all statuses (built-in and custom), each with its icon; picking one updates the button and the leading icon.
- Execute: green primary when the row has a configuration. Without one it takes the inactive treatment (grey, still focusable) with the tooltip `Add a configuration to run this.`; clicking it repeats that tooltip and does nothing else.
- Bug actions: the split button (bug icon button plus a caret button opening Raise bug, Link issue, View issues), placed per the Bug actions toggle. No count badge.
- `No Configuration added` and every action show a toast (top right, 4 seconds) instead of a real flow.
- Checking a row's checkbox marks it selected (grey fill). The header checkbox selects all and shows the indeterminate state for a partial selection.

## What the rows demonstrate

The 8 rows are the brief's, in the brief's order. Title line counts measured in this version with the folder pane open; both bug modes give the same counts because the bug column is always reserved.

| Row | Title | At 1280 px | At 1512 px | Assignee |
|---|---|---|---|---|
| 1 | Login functionality sentence, 103 characters | 2 lines | 2 lines | 41-character name, ellipsized, full name in the tooltip |
| 2 | Licence sentence, with two tags | 2 lines | 1 line | Short name |
| 3 | 120+ character unbroken URL | 6 lines, clamped to 2 | 4 lines, clamped to 2 | Unassigned |
| 4 | `wdwdqwDEWQD` | 1 line | 1 line | Unassigned |
| 5 | 166-character sentence, with two tags | 3 lines, clamped to 2 | 2 lines | Short name |
| 6 | Blocked payment flow sentence | 1 line | 1 line | Short name |
| 7 | `Skip legacy export` | 1 line | 1 line | Unassigned |
| 8 | In progress row | 1 line | 1 line | Single-word name |

So 1280 shows one-line, two-line and three-line titles (the three-line one clamped to two lines plus an ellipsis, as the brief requires), 1512 shows one-line, two-line and clamped titles, and both widths show short and ellipsized assignee names, with avatars hidden or shown. The `titleLines()` helper on `window.__ti` reports the natural line count behind the clamp.

## Changes from the 28 Sep 2026 review

Each one supersedes the brief where the two differ.

1. **Geometry from the live page** (see "Layout"). Supersedes the brief's section 3 split of a 24 px page margin plus a 288 px folder panel with a 12 px right margin. At 1512 the total before the list area is the same 368 px, so the 1096 px list content is unchanged; at 1280 the list content is now 912 px, the brief's own number.
2. **No count badge on the bug button.** Supersedes the linked-issue badge in brief section 4, item 9. The `linkedIssues` data field went with it.
3. **The Unassigned user icon follows the avatar toggle.** Hidden avatars: `Unassigned` alone. Shown avatars: a user icon in the avatar's 20 px slot, so Unassigned lines up with the names.
4. **Unassigned is an option in the assignee menu**, first, above a divider.
5. **Assignee and status are invisible buttons** (LTButton `variant="invisible"` with a trailing caret) in both views and both status-label modes, as on the live page.
6. **Bug actions: two first-line modes.** "Hover, second line" is gone. "Visible on hover, first line" (new, default) puts the split button first in the right group with its 60 px plus 8 px always reserved; "Always visible, first line" is unchanged. Supersedes the second-line placement in brief section 4, item 9, and the section 11 rule against first-line bug actions in the default Proposed view.

## Also aligned to the live page

Where the brief and `design-context` say nothing, version 3 follows what the review screenshot shows.

- The list header row sits on `canvas.subtle` (`#f6f8fa`), measured; rows stay white.
- Selection uses `bg.neutralSubtle` (`#E7EBEF`): the live folder pane's selected row measures `#e7ebee`. Selected list rows use the same fill, since the guidelines ask for one selection treatment everywhere.
- Tag outlines use `accent.emphasis` (`#0969DA`): the live outline measures a saturated accent blue, where version 2 used the much lighter `accentMuted`.
- In the Current view, `Select Assignee` is default text colour, as on the live page (version 2 had it muted).
- The folder pane's root row reads `All Test Instances`.
- The configuration kind reads `virtual` and `real` in lowercase, as on the live page and in the brief's section 7.
- Rows 6 and 8 reuse the brief's two configurations (iPhone 15 Pro, Galaxy Z Fold4). The brief only says "configuration present" for them, and version 2 had invented other devices.

## Live page vs this prototype

Seen in the review screenshot and deliberately not copied, because the brief or `design-context` decides otherwise. Listed for the next review.

- **Carets**: solid triangles on the live page; the brief specifies Lucide `chevron-down`.
- **Status glyph size**: 16 px on the live page (about 15 px measured); the brief specifies 20 px.
- **Not Started glyph**: a blue circle with a centre dot on the live page, which matches `mock-data.md`'s description of the Status component's `Not Started-2`. `mock-data.md` says to use `Not Started` (dotted outline) and asks the design-system owner what `Not Started-2` is for.
- **Meta line**: a commit-style icon before the version, round dot separators, the configuration kind truncated to `virt...`, and Android and Chrome brand marks. The brief's meta format and its Lucide-only rule keep `·` separators and Lucide stand-ins, and nothing in the brief truncates the kind.
- **Tags**: a long tag name is truncated in the middle, and the fourth live row shows two tags plus a `+1` count. The brief's data never triggers either.
- **Execute**: no play button on any visible live row. The brief keeps it.
- **Folder pane**: the live page has no add-folder button, and it adds a `Shared Incoming` section with its own count. The pattern's add-folder button is kept, and the brief needs no folder tree. The selected folder's orange bar measures about `#ed5e00`, which is not in `tokens.json`, so it is still not drawn.

## Where the brief and design-context disagree

Recorded per the house rule: the brief (and now the review) wins, and every conflict is listed so it can be settled once.

1. **`35 Instances`.** `guidelines/README.md` section 8 says a label that starts with a number keeps the rest lowercase. The brief's string is kept; the live page uses the same capitalisation.
2. **Tags beneath the title.** `test-entity-listing.md` renders tags inline after the title; the brief and the live page put them on their own line beneath it.
3. **Row spacing.** The pattern's row is `padding: 16px 12px` with 12 px gaps; the brief's is 16 px horizontal, 12 px vertical, 8 px gutters between the right-side columns. Brief kept; the left group (checkbox, status icon, title) uses the pattern's 12 px because the brief does not specify it.
4. **Status icon size.** The Figma Status component, the pattern and the live page are 16 px; the brief's leading icon and button icon are 20 px. Brief kept; menu items use 16 px.
5. **Folder pane width.** The pattern confirms 320 px; this prototype uses the live page's 300 px at 1512 and the scaled 252 px at 1280, by decision, with `design-context` unchanged.
6. **Execute without a configuration.** The brief calls it disabled; `guidelines/ltbutton.md` and `lticonbutton.md` say never disable, use inactive. It is drawn with the inactive tokens and stays focusable, which is also what makes the brief's tooltip reachable from the keyboard.
7. **Row overflow menu.** The pattern gives every row a trailing `...` menu (slot 6). The brief's description of today's row and the live page have none, so none is drawn.
8. **Status glyphs.** `mock-data.md` says instantiate the Status component and never redraw it; the brief allows Lucide only. The glyphs are composed from Lucide to the component's written descriptions, and swap out in one object when the assets exist.
9. **Selection treatment.** The guidelines want an orange left bar plus a grey fill; the orange has no token, so selected rows and the selected folder use the fill only.
10. **Font stack.** The brief names SF Pro Text; the theme forbids naming a font and ships the system stack. Both resolve to the same face on macOS, so the brief's stack is kept.
11. **Assignee and status buttons.** The pattern names `LTSelect` for the row's inline value pickers; the review specifies LTButton `variant="invisible"`. The review's is drawn.

## Where this prototype departs from the brief, and why

Each of these is deliberate and reversible in one line.

1. **Statuses follow mock-data.md.** The brief's placeholder set (Not Started, In Progress, Passed, Failed, Blocked, Skipped) is replaced by the 12-value execution vocabulary, so `In Progress` renders as `Running`. `Blocked` is not one of the 12 built-ins, so row 6 ships it as a custom status; if it is confirmed built-in, move it into `STATUSES`.
2. **Assignee names come from the mock-data cast** (Darth Vader, Gabbar Singh, Oppenheimer) in place of the brief's names, because mock-data.md forbids inventing people. Row 1 keeps the brief's 41-character worst case, which extends a cast name.
3. **OS, browser and device marks are Lucide stand-ins** (`layers`, `globe`, `smartphone`); `icons.md` treats brand marks as in-house assets, which this file does not have.
4. **Row 3's URL carries an extra `&sig=` token.** The brief describes the URL as a 120+ character unbroken string, but browsers may break a line after `/` and `-`, so the URL as written has a longest unbreakable run of about 72 characters (its query string). That fits at 1512 px and would hide the failure the Current view exists to show. A 64-character signature token appended to the query string makes the run about 140 characters (1044 px of minimum width), genuinely unbroken, so the Current view overflows at both widths whether the folder pane is open or closed. The meta line is also prevented from dictating row width (`contain: inline-size`), so the Current view fails because of the title, as the brief describes.
5. **Folder rows.** The brief allows a grey placeholder; the pane instead shows the pattern's anatomy so the page reads as the real one. `mock-data.md` has no folder names, so the six names are the scope halves of its test-run names (Login and signup flows, Checkout revamp, Payments regression, Cross-browser sanity, Smoke, Release 8.4 regression) with counts that sum to the 35 instances. The pane is static and inert.

## Design-context gaps this prototype surfaced

Things `design-context` could not answer, worth fixing there rather than here. The folder pane width is deliberately not one of them.

- `icons.md`: the approved icon table is TO FILL for Passed, Failed, Running, More actions and Search, and has no entries for OS, browser or device marks, the caret, or the version icon. This prototype had to pick Lucide names by intent.
- `guidelines/README.md` section 1: the orange selection-bar colour has no token (the live page measures about `#ed5e00`, and no orange of that value is in `tokens.json`), so no prototype can draw the selection treatment the hard rule requires.
- `data/tokens.json`: `btn.invisible.bg` is `#FFFFFF`, not transparent, so an invisible button on a tinted surface (a hovered or selected row) would show a white box. This prototype keeps it transparent at rest and uses the token's hover and active fills.
- `mock-data.md`: no folder names for the Test Manager folder pane; no custom-status examples, although custom statuses exist in the product (the brief names two); `Blocked` is used in the product but is outside the 12-value vocabulary; no mobile device or OS-version values for instance configurations (the brief's Galaxy Z Fold4, iPhone 15 Pro, Android 13.0, iOS 17.4 are used as-is). Its open question on `Not Started-2` now has evidence: the live TI listing draws not-started with that glyph.
- `patterns/test-entity-listing.md`: the Test Instances row differs from the brief and the review in tags placement, spacing, status icon size, the `...` menu, the pickers' component, and the `35 Instances` count label (items 1 to 4, 7 and 11 above), and its folder pane shows an add-folder button the live test-run page does not have. Once the overflow fix is accepted, the pattern's slot 5 should carry the fixed columns (assignee 150, status 130, execute 32, bug 60) and whichever bug mode ships.
- `COMPONENTS.md`: `LTTestStatusLabel.status` has no value list; the mock-data vocabulary was assumed.

## Open items

- The exact built-in status set, icons and colours are placeholders pending confirmation (now the 12 from mock-data.md, with Lucide glyphs standing in for the Figma Status component).
- The proposed default is status icon + label (130 px). Icon-only is available as a toggle for comparison only; with a shared icon for all custom statuses, icon-only makes custom statuses unreadable without hovering.
- Assignee avatar defaults to hidden per the current decision; the toggle exists so the PM can compare scanability with it shown.
- Which bug actions mode ships. Visible on hover is the default here; it has no touch fallback by design (desktop-only scope).
- Whether custom status names and user display names have a product-level length cap is unknown. The fixed columns plus tooltips handle any length regardless.
- Whether `Blocked` is a built-in or a custom status; see departure 1.
- Whether the live page's missing execute button is intended; the brief keeps it.
- Which way each conflict in "Where the brief and design-context disagree" should be settled, and which items in "Live page vs this prototype" to adopt.
