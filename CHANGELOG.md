# Changelog

All notable changes to AIAutocompleteKit are documented in this file. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[Semantic Versioning](https://semver.org).

Sections are written by the release workflow from its "What changed" input
(Actions → Release → Run workflow) — see the Releases section of CLAUDE.md.

## [1.6.3] - 2026-08-14

### Styling
- Completed-param chips now default to a 6pt corner radius (previously fully rounded capsules)
- New `completedParamCornerRadius` appearance token to tune or restore the old look

### Fixes
- Dropdown positioned above the input no longer intermittently collapses to a single row with scrolling disabled — most visible with multi-line options in multi-column layouts, e.g. when re-editing a completed param
- A dropdown showing less than its full content now always scrolls, so no option can be left unreachable
- Transient height mis-measurements during layout settle now self-correct instead of freezing the dropdown short

## [1.6.1] - 2026-08-14

- Small bug fixes

## [1.6.0] - 2026-08-14

### New stock appearance
- Completed params render as solid high-contrast capsules (black on light, white on dark) with inverse ink and roomier padding
- The active param pill shows a solid hollow outline; queued pills wait borderless behind it
- Options at 14pt with increased spacing; input line spacing increased; brand footer hidden by default
- Every previous default remains one appearance token away — nothing was removed

### Dropdown
- Options wrap to as many lines as they need instead of truncating
- Option spacing configurable via `optionVerticalPadding`

### Styling
- Eight new `AIAutocompleteAppearance` tokens: `optionVerticalPadding`, `optionSelectedFontWeight`, `completedParamTextColor`, `completedParamHorizontalPadding` / `completedParamVerticalPadding`, `completedParamShimmerColor`, `showsInactivePillBorders`, `showBrandFooter`

### SwiftUI
- `.submitButtonHidden()` hides the built-in submit button
- `.dropdownHidden()` detaches the dropdown so it can anchor to your own container via `.aiAutocompleteDropdown`

### Fixes
- Caret is no longer hidden inside the completed-param chip after answering
- Completing a param that wraps the input now scrolls fully into view
- Dropdown option width/spacing renders exactly as measured

## [1.5.0] - 2026-08-12

### Added
- Redesigned input and dropdown to match the design system: plain small-caps parameter pills (no outline by default), 16pt input text and options, and darker option text.
- iOS 26 Liquid Glass dropdown surface — release binaries are now built with Xcode 26, so the glass surface ships in the framework.
- `maxVisibleOptionRows` appearance token (default 5) — caps the dropdown at five option rows and scrolls the rest, rather than expanding to fill the available space (whether it opens below or above the input).
- LLM-identified parameters are recognized in the user's text and rendered as compact chips, like completed parameters.

### Changed
- Suggestions now refetch after every answered parameter. Each selection is sent to the server immediately so the next parameter is conditioned on it — requests are more frequent, and a brief loading state appears where cached options previously showed instantly.

## [1.4.1] - 2026-07-28

- The dropdown brand link now carries utm_source

## [1.4.0] - 2026-07-23

- Suggestion pills redesigned as transparent chips with a dashed outline, with new appearance tokens to customize it: pillBorderColor, pillBorderWidth, and pillBorderStyle (.dashed or .solid); pillBackgroundColor now defaults to clear.
- Completed parameters now render as compact highlighted chips, matching the web SDK. Emphasis is configurable via completedParamEmphasis — chip fill (default), semibold text, both, or neither — with new fill tokens completedParamBackgroundColor and completedParamActiveBackgroundColor (the darker fill shown while re-editing).
- The dropdown now stays open when the active pill genuinely has no options (or only non-tappable hints with showNonTappableOptions off), keeping its pill chip visible; with inline pill placement, an empty dropdown no longer lingers.
- Tapping outside a parameter being re-edited now exits re-edit mode.
- Fixed completed-chip rendering issues: uneven inner padding, spacing between adjacent chips, chips wrapping across lines, and broken padding after backspacing flush against a chip.
- optionOverrides now fully replace server-supplied options in stored state instead of merging with them.

## [1.3.0] - 2026-06-19

- UIKit `AIAutocompleteView` dropdown now floats over surrounding content instead of pushing it in-flow, matching the SwiftUI wrapper and the React SDK.
- The dropdown's "AI Autocomplete" brand footer is now a tappable link (tap + VoiceOver) to ai-autocomplete.com.
- Re-editing a completed parameter now replaces it and filters its options inline as you type, instead of inserting after the pill.
- The caret can no longer land after a trailing inline pill.
- Tapping an inline pill focuses the input, and the selection haptic fires only when the active pill actually changes.

## [1.2.0] - 2026-06-16

- Dropdown polish: refined option sizing, word-width loaders, and scroll   fade
- Add light haptic feedback on pill taps
- Show the dropdown only once   options arrive (no initial loading flash)

## [1.1.0] - 2026-06-12

- Update keyboard return key appearance & behavior
- Update Documentation

## [1.0.0] - 2026-06-12

- Initial iOS SDK Release
