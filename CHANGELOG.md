# Changelog

All notable changes to AIAutocompleteKit are documented in this file. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[Semantic Versioning](https://semver.org).

Sections are written by the release workflow from its "What changed" input
(Actions → Release → Run workflow) — see the Releases section of CLAUDE.md.

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
