# Firefox 155 Upstream Compatibility Design

## Goal

Bring Lucid Foxgate into compatibility with Firefox 155.0.1 by manually porting only verified framework and bug fixes from `yiiyahui/Neptune-Firefox`, while preserving Lucid Foxgate's glass composition, layout geometry, accessibility behavior, and performance corrections.

## Baseline

- Lucid Foxgate branch baseline: `2fd3de2`.
- Upstream merge base: `f2c8fbf45bdf10e3fc5815adb44d8ff47024e1cb`.
- Audited upstream head: `f27b0db3039ad3871db4c18cbe8b0a2d79055c9c`.
- Installed target browser: Firefox 155.0.1.
- GitHub ancestry reports Lucid Foxgate as 11 commits ahead and 52 commits behind. This count must not be treated as 52 missing behaviors because Lucid already carries consolidated and manually ported upstream work.

## Integration Strategy

Use a curated compatibility patch instead of merging or cherry-picking the upstream history. Each change must be checked against the installed Firefox 155.0.1 source and applied to Lucid's existing files without replacing the surrounding visual system.

Upstream changes are inputs for compatibility analysis, not authoritative replacements for Lucid files. When upstream styling conflicts with Lucid styling, preserve Lucid unless the installed Firefox source proves the existing selector or variable is obsolete.

## Required Changes

### 1. URLbar height interface

Replace every active Lucid declaration and consumption of `--urlbar-min-height` with Firefox 155's `--urlbar-height`. Update the complete dependency chain together so toolbar, URLbar, sidebar, controls, and transparent-startpage calculations remain internally consistent.

Expected affected files include:

- `chrome/neptune/theme/global.css`
- `chrome/neptune/theme/toolbox.css`
- `chrome/neptune/theme/urlbar.css`
- `chrome/neptune/theme/icons.css`
- `chrome/neptune/theme/sidebar.css`
- `chrome/neptune/optionals/macos_controls.css`
- `chrome/neptune/optionals/macos_tahoe_theme_toolbar.css`
- `chrome/neptune/optionals/windows_native_controls.css`
- `chrome/neptune/optionals/Toolbox_transparent_startpage.css`

No compatibility alias should remain after all consumers are migrated.

### 2. Transparent-startpage vertical sidebar

In `chrome/neptune/optionals/Toolbox_transparent_startpage.css`:

- Replace selectors that target the removed `#sidebar-main` ID with the current `#sidebar-container` element.
- Replace both uses of the removed `--browser-stack-z-index-rdm-toolbar` token with the final upstream result `z-index: 4`.
- Preserve Lucid's clear vertical rail and do not add full-height sidebar blur.

### 3. Download notification animation contract

Update only these two existing assets using the current upstream Neptune artwork:

- `chrome/neptune/assets/icons/notification-start-animation.svg`: 860 by 20 pixels.
- `chrome/neptune/assets/icons/notification-finish-animation.svg`: 800 by 20 pixels.

Do not import the unrelated `downloads.svg` redesign.

### 4. Obsolete reload/stop animation override

Remove the dead `#stop-reload-button[animate]` override from `chrome/neptune/theme/icons.css`. Delete only the two assets that become unreferenced because of that removal:

- `chrome/neptune/assets/icons/reload-to-stop.svg`
- `chrome/neptune/assets/icons/stop-to-reload.svg`

Do not alter Firefox's native reload and stop button behavior.

### 5. Nova New Tab compatibility

In `chrome/neptune/firefox/startpage.css`:

- Remove the blanket cursor override applied to every New Tab descendant.
- Apply `cursor: default` only to Nova top-site descendants where upstream currently requires it.
- Preserve Lucid's modal material, blur, border, and accessibility fallbacks.
- Remove the fixed `40em` width and `25vh` top offset from the edit-top-site modal so Firefox 155 controls responsive geometry.
- Add Firefox 155-compatible `.icon-clear-input` button sizing tokens without changing Lucid's input appearance.

### 6. Verified stale framework variables

Migrate only variables confirmed absent from Firefox 155.0.1:

- `--border-color-card` to `--card-border-color`.
- Replace dependencies on removed `--input-bgcolor` with the current input-background interface while preserving Lucid's `--nept-input-bgcolor` value.
- Replace obsolete popup geometry `--arrowpanel-*` variables with their current `--panel-*` equivalents while preserving Lucid's numeric spacing, radius, colors, Mica/non-Mica composition, and accessibility fallbacks.

Do not rename retained `--arrowpanel-*` color/material variables unless Firefox 155 source confirms that the specific token is absent.

### 7. Tahoe ineffective shadow declarations

Remove active Tahoe declarations that reference undefined `--shadow-inner-left`, `--shadow-inner-top`, `--shadow-inner-right`, or `--shadow-inner-bottom` tokens. Do not replace them with a new effect unless removal visibly regresses Lucid's established toolbar material.

### 8. Findbar and browser content grid

Treat findbar placement as a compatibility check, not an automatic upstream copy:

- Confirm the current `order: -1` declaration is ineffective under Firefox 155's grid-based `.browserContainer`.
- Remove that ineffective declaration.
- Add `min-height: unset !important` to retain Lucid's compact findbar sizing.
- Do not import upstream's custom full `.browserContainer` grid template unless a reproducible test proves it is necessary to preserve Lucid's current visible findbar placement.

## Preserved Lucid Behavior

- `chrome/neptune/optionals/liquid_glass.css` remains the final active import in `chrome/userChrome.css`.
- Existing Mica and non-Mica popup surfaces, URLbar drop-down glass, rounded geometry, and hover contrast remain intact.
- Horizontal tab bounds remain 48 to 225 pixels, including pinned, grouped, container, split-view, overflow, drag, keyboard-focus, and reduced-motion behavior.
- Reduced-transparency and forced-colors fallbacks remain intact.
- New Tab wallpaper and Lucid service-card assets remain intact.
- Video and Picture-in-Picture surfaces retain `backdrop-filter: none` and must not gain full-surface filters, persistent `will-change`, or animated blur.
- Vertical tab rails remain clear of unnecessary full-height filters.

## Explicit Exclusions

- No upstream branch merge or broad cherry-pick.
- No wholesale replacement of `userChrome.css`, `userContent.css`, `global.css`, `popups.css`, `urlbar.css`, `tabsbar.css`, `root.css`, `startpage.css`, or transparent-startpage CSS.
- No upstream palette, wallpaper, tab geometry, download-icon shape, AI/IP/VPN artwork, or general visual retuning.
- No new JavaScript, WebExtension manifest, runtime background process, or dependency.
- No edits to the user's active Firefox profile until the branch passes review and the user approves deployment.

## Verification

The implementation must provide fresh evidence for all of the following:

1. Repository search finds no active `--urlbar-min-height`, removed sidebar IDs/tokens, dead reload animation selector, deleted asset references, `--border-color-card`, or dependencies on removed `--input-bgcolor`.
2. Any remaining `--arrowpanel-*` variable is individually verified as current in Firefox 155 or intentionally defined by Lucid; obsolete geometry aliases are absent.
3. The start and finish notification sprites report 860×20 and 800×20 dimensions.
4. All CSS imports and local asset references exist with exact path casing and no import cycles.
5. Firefox-aware Stylelint reports zero errors across all CSS sources.
6. `git diff --check` succeeds.
7. The implementation report maps every changed line to a required section of this design.
8. An independent task reviewer finds no open Critical or Important defect.
9. A final whole-branch reviewer confirms no deviation from Lucid's preserved-behavior and exclusion lists.

Live Browser Toolbox checks should cover light and dark sites, New Tab, URLbar results, app menus, horizontal and vertical tabs, compact findbar, downloads animation, reduced motion, reduced transparency, and forced colors when the environment permits. If live inspection is unavailable, that limitation must be reported rather than inferred away.

## Delivery

Implementation occurs only on `codex/firefox-155-sync`. No push, release, merge into `main`, or active-profile deployment is authorized by this design.
