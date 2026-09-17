# Firefox 155 Upstream Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Port verified Firefox 155 compatibility fixes from Neptune into Lucid Foxgate without importing upstream visual changes or weakening Lucid's accessibility and performance safeguards.

**Architecture:** Apply three sequential, independently reviewed patches: first migrate shared Firefox framework interfaces, then adapt Firefox 155 DOM/layout changes, then update animation assets and remove dead overrides. Preserve Lucid's existing values and surfaces; upstream code is evidence for selector and token compatibility, not a replacement for Lucid files.

**Tech Stack:** Firefox 155 userChrome/userContent CSS, SVG sprite assets, PowerShell, ripgrep, Git, Node.js filesystem audit, Stylelint.

**Spec:** `docs/superpowers/specs/2026-09-11-firefox-155-upstream-sync-design.md`

## Global Constraints

- Target Firefox 155.0.1.
- Make surgical edits only in existing theme files and the four explicitly named existing SVG assets; do not add a manifest, JavaScript, runtime service, or dependency.
- Do not merge or cherry-pick the upstream branch. Use upstream commits only as source material for individually verified patches.
- Preserve Lucid's glass surfaces, Mica/non-Mica popup composition, 48–225px tabs, semantic tab/container colors, accessibility fallbacks, wallpapers, and import order.
- `chrome/neptune/optionals/liquid_glass.css` must remain the final active import in `chrome/userChrome.css`.
- Existing `backdrop-filter: none` safeguards in `chrome/neptune/firefox/root.css` must remain unchanged.
- Do not add full-height sidebar blur, content/Picture-in-Picture blur, animated blur, or persistent `will-change`.
- Do not modify the active Firefox profile, push, publish a release, or merge into `main`.
- Every implementation task must be committed and pass an independent spec-and-quality review before the next task begins.

---

### Task 1: Migrate Firefox 155 framework variables

**Files:**
- Modify: `chrome/neptune/share/shared.css`
- Modify: `chrome/neptune/firefox/aboutpage.css`
- Modify: `chrome/neptune/optionals/Toolbox_transparent_startpage.css`
- Modify: `chrome/neptune/optionals/macos_controls.css`
- Modify: `chrome/neptune/optionals/macos_tahoe_theme_toolbar.css`
- Modify: `chrome/neptune/optionals/windows_native_controls.css`
- Modify: `chrome/neptune/theme/buttons.css`
- Modify: `chrome/neptune/theme/global.css`
- Modify: `chrome/neptune/theme/icons.css`
- Modify: `chrome/neptune/theme/popups.css`
- Modify: `chrome/neptune/theme/sidebar.css`
- Modify: `chrome/neptune/theme/tabsbar.css`
- Modify: `chrome/neptune/theme/toolbox.css`
- Modify: `chrome/neptune/theme/urlbar.css`

**Interfaces:**
- Consumes: Firefox 155's `--urlbar-height`, `--card-border-color`, `--input-text-background-color`, and `--panel-*` interfaces.
- Produces: One internally consistent Lucid token surface with no active consumers of the removed Firefox names.

- [ ] **Step 1: Record the failing compatibility baseline**

Run:

```powershell
rg -n --glob '*.css' -- '--urlbar-min-height|--border-color-card|--input-bgcolor|--arrowpanel-(menuitem|padding|shadow|header|border-radius|background|border-color|color)' chrome
```

Expected: matches in the files listed above, proving the compatibility check fails before the patch.

- [ ] **Step 2: Migrate URLbar, card, and input variables atomically**

Use `apply_patch` to make these exact interface changes everywhere they are actively declared or consumed:

```text
--urlbar-min-height         → --urlbar-height
--border-color-card         → --card-border-color
--input-bgcolor             → --input-text-background-color
```

Retain the existing Lucid values. In `global.css`, define `--input-text-background-color: var(--nept-input-bgcolor) !important` directly and make `--background-color-box` consume it. Do not create aliases under the removed names.

- [ ] **Step 3: Migrate popup geometry and material interfaces**

Use `apply_patch` for these exact mappings, retaining Lucid's current values and `!important` choices:

```text
--arrowpanel-menuitem-margin-inline          → --panel-menuitem-margin-inline
--arrowpanel-menuitem-padding-inline         → --panel-menuitem-padding-inline
--arrowpanel-menuitem-padding-block          → --panel-menuitem-padding-block
--arrowpanel-padding                         → --panel-padding
--arrowpanel-shadow-margin                   → --panel-box-shadow-margin
--arrowpanel-menuitem-border-radius          → --panel-menuitem-border-radius
--arrowpanel-border-radius                   → --panel-border-radius
--arrowpanel-header-back-icon-full-width     → --panel-header-back-icon-full-width
--arrowpanel-header-min-height               → --panel-header-min-height
--arrowpanel-header-back-icon-padding        → --panel-header-back-icon-padding
--arrowpanel-background                      → --panel-background-color
--arrowpanel-border-color                    → --panel-border-color
--arrowpanel-color                           → --panel-text-color
```

Also migrate existing `--panel-shadow-margin` declarations to `--panel-box-shadow-margin`: Firefox 155 consumes only the latter. Preserve the existing `22px` root/feature-callout and `0px` macOS popup values, respecting the already-current Windows overrides.

Do not alter unrelated `--panel-*` definitions, popup colors, opacity, backdrop filters, borders, radii, or shadows.

- [ ] **Step 4: Remove ineffective Tahoe shadow references**

In `macos_tahoe_theme_toolbar.css`, remove only the undefined `--shadow-inner-left`, `--shadow-inner-top`, `--shadow-inner-right`, and `--shadow-inner-bottom` components from the five affected `box-shadow` declarations. Preserve each declaration's defined `--opt-tahoe-theme-toolbar-item-shadow` component and surrounding selectors.

- [ ] **Step 5: Verify Task 1**

Run:

```powershell
$removed = rg -n --glob '*.css' -- '--urlbar-min-height|--border-color-card|--input-bgcolor|--arrowpanel-(menuitem|padding|shadow|header|border-radius|background|border-color|color)|--panel-shadow-margin|--shadow-inner-(left|top|right|bottom)' chrome
if ($LASTEXITCODE -eq 0) { $removed; exit 1 }
if ($LASTEXITCODE -gt 1) { exit $LASTEXITCODE }
rg -n --glob '*.css' -- '--urlbar-height|--card-border-color|--input-text-background-color|--panel-menuitem|--panel-padding|--panel-box-shadow-margin|--panel-border-radius|--panel-background-color|--panel-border-color|--panel-text-color' chrome
git diff --check
```

Expected: the removed-name search returns no matches; current interfaces are present; `git diff --check` exits zero.

- [ ] **Step 6: Self-review and commit**

Confirm every changed line implements Steps 2–4 and no Lucid material value changed. Commit:

```powershell
git add chrome
git commit -m "fix: migrate Firefox 155 theme variables"
```

---

### Task 2: Adapt Firefox 155 DOM and layout behavior

**Files:**
- Modify: `chrome/neptune/optionals/Toolbox_transparent_startpage.css`
- Modify: `chrome/neptune/firefox/startpage.css`
- Modify: `chrome/neptune/theme/toolbox.css`

**Interfaces:**
- Consumes: Task 1's `--urlbar-height` and `--panel-*` migrations.
- Produces: Current sidebar/New Tab selectors and compact findbar behavior without importing upstream visual composition.

- [ ] **Step 1: Record the failing DOM/layout baseline**

Run:

```powershell
rg -n -- '#sidebar-main|--browser-stack-z-index-rdm-toolbar|cursor:\s*auto|width:\s*40em|top:\s*25vh|order:\s*-1' chrome/neptune/optionals/Toolbox_transparent_startpage.css chrome/neptune/firefox/startpage.css chrome/neptune/theme/toolbox.css
```

Expected: matches for every obsolete construct named by the task.

- [ ] **Step 2: Update transparent-startpage sidebar targeting**

In `Toolbox_transparent_startpage.css`, use `apply_patch` to replace every selector occurrence of `#sidebar-main` with `#sidebar-container`. Replace the two obsolete z-index declarations with exactly:

```css
z-index: 4;
```

Do not add `backdrop-filter`, `filter`, `will-change`, or new sidebar surface colors.

- [ ] **Step 3: Update Nova cursor and edit-dialog rules**

In `startpage.css`, use `apply_patch` to:

- Remove the universal New Tab `* { cursor: auto !important; }` block.
- Add `cursor: default !important` only inside `.nova-enabled .top-sites-list *`.
- Remove only `width: 40em !important` and `top: 25vh !important` from the `.edit-topsites-wrapper` modal rule.
- Replace the obsolete `.topsite-form .top-site-outer` spacing override with Firefox 155's clear-button sizing rule:

```css
> .topsite-form .form-wrapper .field .icon-clear-input {
  --button-size-icon-small: var(--size-item-small);
  --button-min-height: var(--size-item-small);
  --button-border-radius: var(--border-radius-medium);
}
```

Match the file's tab indentation. Preserve the surrounding `.modal` glass and accessibility blocks unchanged.

- [ ] **Step 4: Remove the ineffective flex-order assumption**

In `toolbox.css`, add `min-height: unset !important` to `findbar` and remove only `order: -1`. Do not add or replace `.browserContainer` grid templates in this task.

- [ ] **Step 5: Verify Task 2**

Run:

```powershell
$removed = rg -n -- '#sidebar-main|--browser-stack-z-index-rdm-toolbar|cursor:\s*auto|width:\s*40em|top:\s*25vh|order:\s*-1' chrome/neptune/optionals/Toolbox_transparent_startpage.css chrome/neptune/firefox/startpage.css chrome/neptune/theme/toolbox.css
if ($LASTEXITCODE -eq 0) { $removed; exit 1 }
if ($LASTEXITCODE -gt 1) { exit $LASTEXITCODE }
rg -n -F -- '#sidebar-container' chrome/neptune/optionals/Toolbox_transparent_startpage.css
rg -n -F -- 'z-index: 4;' chrome/neptune/optionals/Toolbox_transparent_startpage.css
rg -n -F -- '.icon-clear-input' chrome/neptune/firefox/startpage.css
rg -n -F -- 'min-height: unset !important' chrome/neptune/theme/toolbox.css
$addedFilters = git diff --unified=0 -- chrome/neptune/optionals/Toolbox_transparent_startpage.css | Select-String -Pattern '^\+.*(?:backdrop-filter|filter:|will-change)'
if ($addedFilters) { $addedFilters; exit 1 }
git diff --check
```

Expected: obsolete constructs are absent; current selectors/rules are present; no full-height filter was introduced; `git diff --check` exits zero.

- [ ] **Step 6: Self-review and commit**

Confirm New Tab material, forced-colors, reduced-transparency, wallpapers, and sidebar transparency remain unchanged. Commit:

```powershell
git add chrome/neptune/optionals/Toolbox_transparent_startpage.css chrome/neptune/firefox/startpage.css chrome/neptune/theme/toolbox.css
git commit -m "fix: adapt Firefox 155 chrome structure"
```

---

### Task 3: Update Firefox 155 animation assets and remove dead overrides

**Files:**
- Modify: `chrome/neptune/assets/icons/notification-start-animation.svg`
- Modify: `chrome/neptune/assets/icons/notification-finish-animation.svg`
- Modify: `chrome/neptune/theme/icons.css`
- Delete: `chrome/neptune/assets/icons/reload-to-stop.svg`
- Delete: `chrome/neptune/assets/icons/stop-to-reload.svg`

**Interfaces:**
- Consumes: Neptune commit `f27b0db3039ad3871db4c18cbe8b0a2d79055c9c`, whose notification sprites match Firefox 155's frame widths.
- Produces: Correct 43-frame start and 40-frame finish sprites with no dead custom reload/stop animation path.

- [ ] **Step 1: Record the failing asset baseline**

Run:

```powershell
Select-String -Path chrome/neptune/assets/icons/notification-start-animation.svg -Pattern '<svg[^>]+width="340"[^>]+height="20"'
Select-String -Path chrome/neptune/assets/icons/notification-finish-animation.svg -Pattern '<svg[^>]+width="540"[^>]+height="20"'
rg -n -- '#stop-reload-button\[animate\]|reload-to-stop\.svg|stop-to-reload\.svg' chrome
```

Expected: both old sprite dimensions and the dead override/assets are present.

- [ ] **Step 2: Replace only the two notification sprites**

Fetch the audited upstream commit without adding a permanent remote:

```powershell
git fetch https://github.com/yiiyahui/Neptune-Firefox.git f27b0db3039ad3871db4c18cbe8b0a2d79055c9c
git restore --source=FETCH_HEAD -- chrome/neptune/assets/icons/notification-start-animation.svg chrome/neptune/assets/icons/notification-finish-animation.svg
```

Do not restore any other upstream asset or CSS file.

- [ ] **Step 3: Remove the dead reload/stop override and assets**

Use `apply_patch` to remove only the `#stop-reload-button[animate] .toolbarbutton-animatable-box > .toolbarbutton-animatable-image` rule from `icons.css`. Delete only `reload-to-stop.svg` and `stop-to-reload.svg`; Git retains full recovery history.

- [ ] **Step 4: Verify Task 3**

Run:

```powershell
Select-String -Path chrome/neptune/assets/icons/notification-start-animation.svg -Pattern '<svg[^>]+width="860"[^>]+height="20"'
Select-String -Path chrome/neptune/assets/icons/notification-finish-animation.svg -Pattern '<svg[^>]+width="800"[^>]+height="20"'
$dead = rg -n -- '#stop-reload-button\[animate\]|reload-to-stop\.svg|stop-to-reload\.svg' chrome
if ($LASTEXITCODE -eq 0) { $dead; exit 1 }
if ($LASTEXITCODE -gt 1) { exit $LASTEXITCODE }
git diff --check
```

Expected: new sprite dimensions match; the dead selector and asset references are absent; `git diff --check` exits zero.

- [ ] **Step 5: Run the complete structural and CSS validation**

Use a temporary Firefox-aware Stylelint configuration outside the repository that enables `color-no-invalid-hex` and disables generic-parser false positives for Firefox-only at-rules, media queries, properties, and pseudo selectors. Run Stylelint across `chrome/**/*.css` and require zero problems.

Run a temporary Node.js `fs.existsSync()` audit outside the repository that walks all 28 CSS files and verifies every active local import and asset URL, exact path casing, reachability from `userChrome.css`/`userContent.css`, and import-cycle absence.

Also run:

```powershell
git diff --check
git status --short
```

Expected: zero Stylelint problems; no missing path, casing mismatch, or import cycle; only planned files are changed.

- [ ] **Step 6: Self-review and commit**

Confirm `downloads.svg`, Lucid wallpapers, glass CSS, and video/Picture-in-Picture filter safeguards are unchanged. Commit:

```powershell
git add chrome/neptune/assets/icons chrome/neptune/theme/icons.css
git commit -m "fix: update Firefox 155 animation assets"
```

---

### Task 4: Final integration verification

**Files:**
- Verify: all files changed by Tasks 1–3
- Verify: `chrome/userChrome.css`
- Verify: `chrome/neptune/optionals/liquid_glass.css`
- Verify: `chrome/neptune/firefox/root.css`

**Interfaces:**
- Consumes: the three independently reviewed task commits.
- Produces: a release candidate that is ready for user-directed deployment or branch integration, with no automatic profile copy, push, release, or main-branch merge.

- [ ] **Step 1: Run the complete compatibility search**

Run the removed-name, asset-dimension, import/asset graph, Stylelint, and `git diff --check` validations from Tasks 1–3 against the integrated branch.

- [ ] **Step 2: Verify preserved Lucid invariants**

Run exact searches confirming:

```powershell
Get-Content chrome/userChrome.css | Select-Object -Last 8
rg -n -F -- '--tab-max-width: 225px' chrome/neptune/theme/tabsbar.css
rg -n -- 'backdrop-filter:\s*none' chrome/neptune/firefox/root.css
rg -n -- 'prefers-reduced-motion|prefers-reduced-transparency|forced-colors' chrome/neptune/optionals/liquid_glass.css chrome/neptune/theme/popups.css chrome/neptune/theme/urlbar.css chrome/neptune/theme/tabsbar.css chrome/neptune/firefox/startpage.css
```

Expected: `liquid_glass.css` remains the final active userChrome import; tab maximum remains 225px; root content/Picture-in-Picture filter suppression remains; accessibility media queries remain present.

- [ ] **Step 3: Inspect branch scope**

Run:

```powershell
git log --oneline 2fd3de2..HEAD
git diff --stat 2fd3de2..HEAD
git diff --name-status 2fd3de2..HEAD
```

Expected: only the specification, plan, explicitly required theme files, two updated notification sprites, and two deleted obsolete sprites appear.

- [ ] **Step 4: Request final whole-branch review**

Generate a full review package from `2fd3de2` to `HEAD`. The final reviewer must compare the diff against the specification and this plan, triage any deferred minor findings, and explicitly verify both required changes and preserved-behavior exclusions.

- [ ] **Step 5: Report the release candidate**

Report commits, fresh verification output, reviewer verdict, all rulings, and any live Browser Toolbox limitation. Do not claim runtime visual verification unless it was directly performed.
