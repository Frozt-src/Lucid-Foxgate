<h1 align="center"><strong>Lucid Foxgate</strong></h1>

<p align="center">A restrained, adaptive glass interface for Firefox, evolved from Neptune Firefox.</p>

Lucid Foxgate refines Firefox with quiet depth, rounded geometry, adaptive color, and short compositor-friendly transitions. The design favors legibility and responsiveness over exaggerated effects: the glass layer is mostly static CSS, with no injected JavaScript or persistent animated filters.

<img src="info/preview.png" alt="Lucid Foxgate preview" width="800px">

## Recommended experience

- **Windows 11 Mica:** Recommended for the best native popup composition. Lucid Foxgate also includes a visually consistent non-Mica fallback.
- **[Adaptive Tab Bar Colour](https://addons.mozilla.org/firefox/addon/adaptive-tab-bar-colour/):** Recommended for Safari-like toolbar tinting that follows the active website. The theme consumes Firefox's standard `--lwt-accent-color` variable, so the extension remains optional.
- **Current Firefox release:** Custom Firefox CSS can change between browser releases, so keep Firefox and the theme together when updating.

## Installation

1. Open `about:support` in Firefox and select **Open Folder** beside **Profile Folder**.
2. Close Firefox.
3. Copy this repository's `chrome` folder into the Firefox profile folder. Replace the previous theme files when upgrading.
4. Open `about:config` and set:
   - `toolkit.legacyUserProfileCustomizations.stylesheets` to `true`.
   - `svg.context-properties.content.enabled` to `true`.
   - `widget.non-native-theme.use-theme-accent` to `true`.
   - On Windows 11, optionally set `widget.windows.uwp-system-colors.highlight-accent` to `true` for closer Mica accent integration.
5. Restart Firefox.

If Adaptive Tab Bar Colour is installed, set its Theme Builder color controls to `0%` so Lucid Foxgate can supply the glass and contrast layers without competing tints.

## Configuration

The active optional modules are declared near the top of `chrome/userChrome.css` and `chrome/userContent.css`.

- Lucid Foxgate ships with macOS-style traffic-light controls enabled. To use native Windows controls, comment out `macos_controls.css` and enable `windows_native_controls.css` instead.
- `macos_tahoe_theme_toolbar.css` provides the compact floating toolbar treatment.
- `liquid_glass.css` provides the restrained tab and address-field glass states.
- The transparent New Tab toolbox requires both `Toolbox_transparent_startpage.css` in `userChrome.css` and `Startpage_for_transparent_toolbox.css` in `userContent.css`.
- New Tab and Private Browsing wallpapers can be changed in `chrome/userContent.css`.

## Changelog

### Unreleased - Firefox 157 Nova candidate - 2026-10-06

- Retained the completed Firefox 155/156 compatibility work and adapted Firefox 157's separate URLbar results popover to Lucid's existing glass surfaces and solid accessibility fallbacks.
- Restored Nova tab-group pale colors, current shadow/customization tokens, and autocomplete corner radius.
- Corrected reduced-motion selector specificity for horizontal and vertical tabs.
- Removed Nova's decorative accent rings from the open address field and suggestion rows while retaining selection feedback and forced-colors styling.
- Passed the complete static import/asset audit, Stylelint, independent branch review, and 35 targeted Firefox 157.0.1 browser checks in a disposable profile.
- This remains an untagged candidate. Visible remote-page blur, forced-colors rendering, sensitive form panels, and decoded-video/compositor performance still require acceptance. See the [verification report](docs/compatibility/2026-10-06-firefox-157-nova.md) for evidence and limits.

### Lucid Foxgate 1.3b - 2026-08-30

- Updated renamed Firefox 154 tab, toolbar-button, and URLbar design tokens while preserving Lucid Foxgate's established dimensions and color values.
- Updated URL-result action menus and native autocomplete selectors for Firefox 154's current accessibility state and popup structure.
- Ported current split-view clipping and overflowing-tab label behavior without changing the theme's compact tab geometry.
- Added restrained `scale3d` menu motion, multiselect breathing feedback, and reduced-motion fallbacks.
- Added Nova New Tab panel-list glass styling with reduced-transparency and forced-colors fallbacks.
- Corrected two malformed media-player shadow colors that could cause Firefox to discard the affected filter declarations.
- Verified the complete CSS import and asset graph, checked path casing and import cycles, and passed a Firefox-aware Stylelint validation across all theme stylesheets.

### Lucid Foxgate - 2026-08-28

- Added a low-overhead Liquid Glass layer for the address field, horizontal tabs, vertical tabs, and pinned tabs.
- Added short state transitions with reduced-motion handling and disabled motion during tab dragging or toolbar customization.
- Added Mica-aware Windows popup composition plus a matching non-Mica fallback with unified corner radius, border, spacing, shadow geometry, and subtle popup blur while preserving solid reduced-transparency and forced-colors fallbacks.
- Rebuilt the URLbar results popup as one continuous rounded, blurred surface over both websites and internal Firefox pages.
- Corrected URLbar suggestion/history overlap, hover contrast, result spacing, and the overly prominent Windows accent frame.
- Reduced tab and address-field highlights for a quieter, more premium appearance.
- Added a Safari-like separation between the address bar, browser frame, and horizontal tab row.
- Capped horizontal tabs at a Firefox-like `225px` and hid empty or single-tab rails without suppressing pinned tabs.
- Added glass styling for the vertical tabs interface while keeping invisible rails free of unnecessary filters.
- Preserved semantic Firefox Container and tab-group colors across selected, hover, pinned, and drag states.
- Expanded keyboard focus, reduced-motion, reduced-transparency, and forced-colors coverage throughout the browser chrome.
- Scoped Firefox content styling to internal pages so theme rules do not leak into ordinary websites.
- Updated New Tab, Reader View, PDF, dialog, sidebar, button, and media-player styling.
- Added updated wallpapers, service-card icons, trust-state icons, translation assets, and tab-note assets.

## Roadmap

- Complete the remaining Firefox 157 Nova visual, forced-colors, form-panel, and video/compositor acceptance checks documented in the [candidate report](docs/compatibility/2026-10-06-firefox-157-nova.md).

## Credits

Lucid Foxgate is a personal evolution of the Neptune Firefox theme. The project remains available under the included MIT License.
