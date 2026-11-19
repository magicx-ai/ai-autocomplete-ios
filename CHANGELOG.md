# Changelog

All notable changes to AIAutocompleteKit are documented in this file. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[Semantic Versioning](https://semver.org).

Sections are written by the release workflow from its "What changed" input
(Actions → Release → Run workflow) — see the Releases section of CLAUDE.md.

## [Unreleased]

## [2.0.0] - 2026-11-19

### Breaking changes
- `optionOverrides` entries are asked asynchronously: `OptionOverride` is now `@Sendable (String, Suggestion) async throws -> [SuggestionOption]`. A pill of an overridden parameter is asked the moment it becomes active (and when a completed param of it is re-edited), then again on the typing debounce with each new phrase; the server's options for it are never shown or promoted, and the server is not asked while the override owns the pill. The answer replaces the pill's options and is listed as-is for the phrase it answered; an empty answer for a typed phrase hands that phrase to the server once. A thrown error counts as an empty answer. Mirrors JS #227.
- Use suggest schema 1.0 and rename parameter identifiers to `parameter`, identified values to `text`, and source suggestion identifiers to `suggestionParameter`.
- Replace `formatType` with the server's optional `SuggestionType` descriptor.
- `AIAutocompleteAppearance.optionLeadingInset` is now `CGFloat?`, defaulting to `nil`: the dropdown's option text then starts on the same line as the input's typed text, following `inputInsets.left`. An explicit value pins the old fixed inset.
- The calendar opens only for a parameter whose `type` the server names as `date` or `date_range`; parameter names and date-like options are no longer read. `FormatType.resolve(parameter:declared:options:)` becomes `FormatType.resolve(declared:)`. An identified date opens a calendar when its `input` item carries a date `type`.

### Changed
- The location search typed in the input never starts with a joining word: leading "in", "at", "to", "off", "on", "from", "the" and similar are dropped ("dropping off in" searches nothing, "to the beach" searches "beach"), and words the server recognized as another parameter (a date span such as "27th") are skipped, so only text after them is searched. A span recognized as the asked location is still searched. Mirrors JS #472.
- A hardware-keyboard Tab in the input picks the first tappable option of the open dropdown, as a tap would (after ending an open word, which still wins). Calendars, multi-select lists and a closed dropdown are unchanged. Headless consumers call the new `pickFirstOption()`. Mirrors JS #468.
- The empty box shows a hint built from the placeholder's lead word and the first three starting options, one at a time, every 2.6 seconds ("Find a table" → "Find a date spot" → "Find trending restaurants"), in light gray text with the option's icon at 70%. Words the lines share stay put (sliding if they must move); the words that change fade. It holds on the first option when `animationsEnabled` is false and needs at least two starting options; otherwise the plain placeholder shows. New `AIAutocompleteAppearance.startingHintColor` sets its color. Mirrors the JS SDK.
- While a location question is on screen, what the user types in the input is searched in the location field, cut to the place name: the last word ("going to paris" → "paris"), two words around a place-name start or end word ("to san fran" → "san fran", "miami beach"), three when both frame it ("new york city"). The phrase is the text after the last answer, so a `/suggest` response mid-word no longer clears the field; a phrase under 2 characters leaves the field alone; a closed panel searches nothing; a question arriving after the text was typed searches it. Typing in the field itself takes over while it has focus. Mirrors the JS SDK.
- The multi-select and time-picker Done buttons are 44pt tall with 12pt corners and 15pt semibold text, a soft blue-gray (`#e4e8f0`) with dark navy text (`#2a3150`) on light; dark keeps the confirm grey and black text. New `AIAutocompleteAppearance.doneBackgroundColor`, `donePressedBackgroundColor` and `doneTextColor` set them; the ticked checkbox stays on the confirm colors, so a custom `confirm*` no longer recolors Done. Mirrors the JS SDK.
- The multi-select Done button is as wide as the options above it (the widest row's checkbox, icon, text and count), at least 120pt and at most the row, so a short list gets a compact button. The time picker's Done stays full width. Mirrors the JS SDK.
- A picked date without a server-declared `date_format` commits the way a person would write it, as on the web: its weekday ("Tuesday") within the next week, "March 23" later this year, and "March 23 2027" in another year (was always "March 23 2026"). Each end of a range keeps the month and day. Re-editing reads all three back; a weekday reads as its date in the coming week. `DateGrid.displayText(for:format:now:)` and `date(fromDisplayText:format:now:)` gain a `now`, and `DateGrid.absoluteText(for:format:now:)` is new.
- The dropdown fades in and out as one piece, as on the web: in over 320ms after a 40ms delay, out over 280ms, on cubic-bezier(0.25, 0.1, 0.25, 1). No unfold, slide or scale. Skipped when `animationsEnabled` is off (including Reduce Motion).
- `dropdownBorderWidth` defaults to 0: the floating card draws no stroke. Set it to 0.2 to keep the old hairline.
- The dropdown's pill bar shows only the current question, in sentence case, then a thin divider and "skip". The other questions come up through skip.
- Everything inside the options box is 30% smaller, as on the web: options at 9.8pt (UIFontMetrics rounds it to 10pt by default), the option picture, padding, corner radii and spacing, the skip label and divider, inline option icons (which now follow the option font), and the product cards and list (`productListMaxHeight` 224, `productListImageSize` 30.8, `productCardWidth` 81.2, plus the card and list-row padding, gaps, badge and "Best match" label). The text box, input pills and pickers keep their size.
- The skip label never draws smaller than 9pt, whatever the text size, so it stays legible in the smaller options box.
- The brand footer's badge sits as far from the card's trailing edge as from its bottom (the trailing inset drops from 14pt to 10pt), as on the web.
- Flatter default chrome: `inputCornerRadius` and `dropdownCornerRadius` default to 6 (were 12), and `dropdownShadow` defaults to `nil` (no shadow). Set `dropdownShadow = .default` and the radii to 12 to keep the old look.
- The dropdown footer sits 4pt further from the last option.
- The built-in submit button is grey (`submitButtonIdleBackgroundColor` / `submitButtonIdleForegroundColor`) until the query is ready (`controller.isReady`), then turns to the accent color. Set the idle colors to the ready ones to keep the always-dark button.
- The words the SDK draws itself — skip, the submit button's VoiceOver label, "Smart Suggestion", Done, the time label, and the "and" joining multi_select picks — follow `language` in the web SDK's 18 languages, and are English when `language` is `nil`. Behaviour change: these previously followed the device language.

### Added
- A location question picks its top place when a space follows typed words that are exactly that place's name, or what its brackets hold ("sfo" for "San Francisco International Airport (SFO)"), ignoring capitals and accents, as a tap would, the caret left after the place. Only once the search for those words has answered, and never while the location field itself has focus. Mirrors JS.
- The starting-state placeholder types itself into the empty input one character at a time (33ms each, a short hold after a word), and a new phrase fades the old one out first, as on the web. `controller.placeholder` is unchanged. Skipped when `animationsEnabled` is off (including Reduce Motion).
- `AIAutocompleteController.optionQuery`, `isSearchingOptions` and `optionSearch` (`OptionSearch`): the phrase an override is asked with and whether its answer is still pending. A pending answer slower than a couple of frames shows the dropdown's loading skeleton.
- `Configuration.productImageFallback` (`.image`, the default, or `.initial`), the JS SDK's `productImageFallback`: `.initial` shows the title's first character (`Product.initial`) on a product tile whose picture is missing or fails to load, in the new `AIAutocompleteAppearance.productInitialFont`.
- `Configuration.productsLayout` (`.row`, the default, or `.list`), the JS SDK's `productsLayout`: `.list` stacks the product strip's cards as full-width rows — a thumbnail on the left, vendor, title and price on the right — scrolling vertically inside the new `AIAutocompleteAppearance.productListMaxHeight` (320). New tokens `productListImageSize`, `productListBadgeBackgroundColor` and `productListBadgeTextColor` style the rows and their "Sold out" banner.
- The product strip's state, in Core: `controller.products` holds a `Product` card for each item the `/suggest` response reports (`custom_fields.items.matched`), and a non-empty strip holds the dropdown open. New `Configuration.showProducts` (default `true`), `mapProducts`, `onProductSelect` and `products` (`ProductsConfiguration`: a custom search run on the `/suggest` cadence, with an optional `limit`), plus `selectProduct(_:)`, `refreshProducts()` and `Product(matchedItem:language:)`. While the strip has cards, the slow typing tick still sends a request when the text matches some options but not exactly, so the cards follow what is typed. With funnel telemetry on, `impression` lists the cards' `product_ids` and `selectProduct(_:)` sends `product_click`. `AIAutocompleteDropdownView` draws them as a labelled, horizontally scrolling row of cards between the options and the footer: an image tile (Shopify-hosted pictures load as 360px thumbnails), vendor, a two-line title, and the price with the struck price beside it. A sold-out card (`available == false`) shows a "Sold out" badge over a washed-out picture. A tap calls `onProductSelect`. The label ("Best match") and badge follow `language`. New `AIAutocompleteAppearance` tokens `productCardWidth`, `productCardCornerRadius`, `productCardHighlightBackgroundColor`, `productImageCornerRadius`, `productImageBackgroundColor`, `productTitleColor`, `productPriceColor`, `productVendorColor`, `productTitleFont`, `productPriceFont`, `productVendorFont`, `productSectionLabelFont`, `productBadgeBackgroundColor` and `productBadgeTextColor`, plus `Product.imageSources`. The option list gives up the strip's height when the keyboard leaves little room, so the whole card stays above it.
- `MatchedItem` decodes the catalog's `compare_at_price`, `variant_id` and `available` (`compareAtPrice`, `variantId`, `available`). `productLink` opens a handle link on the chosen variant (`/products/{handle}?variant=…`), and `displayCompareAtPrice(language:)` formats the struck price only when the item is actually marked down.
- The dropdown draws the location search field (`LocationFieldView`) for a location pill: a search field and place rows, with new `AIAutocompleteAppearance` tokens `locationFieldBackgroundColor`, `locationFieldCornerRadius`, `locationFont` and `locationHighlightedRowBackgroundColor`, and `AIAutocompleteDropdownView.locationFocusReturnTarget` for Tier 2 inputs.
- A location parameter sent without options is answered by searching the server's location route (`…/locations/autocomplete`, or `APIKeyConfig`/`AccessTokenConfig.locationEndpoint`): `FormatType.location`, `controller.locationView`, `setLocationQuery(_:)`, `moveLocationHighlight(by:)`, `commitLocation(at:)`, `setLocationFieldFocused(_:)`, and `Configuration.locationSearch` (default `true`). A picked place carries `metadata["aiaPlace"]`. ⚠️ `FormatType` gains a case.
- Suggest requests send the device's IANA time zone as `meta.timezone` (`TimeZone.current.identifier`). An identified span's carried dates (`iso_date`, `iso_date_end`) decode onto `InputItem.isoDate` / `isoDateEnd` and `IdentifiedParamState.isoDate` / `isoDateEnd`; a date span's calendar opens on its carried date, so a relative phrase such as "this Saturday" opens with its day marked.
- An identified `input` token with `state: "need_confirmation"` (`InputItem.State.needConfirmation`) is a value the user may still be typing: it gets no pill and is not echoed as `identified_params`. `submit()` and `prepareSubmit()` add it to `identifiedParams` when the text has not changed since that response; any edit drops it. `recently_suggested` entries now carry the tappable option texts shown for each question (`RecentlySuggested.options`).
- `AIAutocompleteAppearance.lastOptionTrailingIcon` / `lastOptionTrailingIconColor`: an SF Symbol drawn at the far right of each suggestion's last option row (default `nil`, no marker).
- Picker Done buttons (multi-select and time picker) are full-width buttons (see the Done entry under Changed), and a ticked multi-select checkbox is a solid fill with a contrasting tick. New `AIAutocompleteAppearance.confirmBackgroundColor`, `confirmPressedBackgroundColor` and `confirmTextColor` color the checkbox (near-black grey with white on light, soft grey with black on dark). The brand footer is not shown while a picker's Done button is (multi-select, time picker).
- The house typing-and-chip motion in the UIKit input and dropdown: the option matching what is being typed is highlighted while the other rows dim, a new chip's fill fades in, option rows leave and enter with a short stagger, the input's height change eases, and the dropdown fades and scales slightly on close. All of it is skipped when `animationsEnabled` is off (including Reduce Motion). New appearance token `optionMatchBackgroundColor`.
- `AIAutocompleteController.optionFilterQuery`: the text being typed that the dropdown's options are matched against, for custom dropdowns that draw their own live-match cue.
- In a dropdown with pictures, an icon row's SF Symbol draws larger (1.5× the option font, following Dynamic Type) in the picture column; `AIAutocompleteAppearance.optionTileIconSize` overrides it.
- Option pictures and icons are all-or-nothing per list, as on the web: if any tappable option has neither (with the icon and picture switches applied), no row draws one.
- An option picture's placeholder fill drops once it loads, and a tall picture keeps its own width with rounded corners (`AIAutocompleteAppearance.optionPictureCornerRadius`, default 6).
- Option pictures scale to fit their tile instead of being cropped.
- Shopify-hosted WebP option pictures load at `width=120` too, letting Shopify pick AVIF or WebP.
- Option pictures hosted on Shopify's CDN load as small thumbnails (`width=120&format=pjpg` for PNGs, via the new `ShopifyImage`), falling back to the original once; `SuggestionOption.imageSources` lists both.
- In a dropdown where some options have pictures, rows without one keep the picture column (icon centred, or empty indent) and the tile's height, so every option's text starts at the same place.
- `Configuration.showOptionImages` and `showOptionCounts` (both default `true`) switch off option pictures and item counts; with pictures off, an option's icon draws instead.
- `AIAutocompleteAppearance.optionCountUnit` (`SuggestionOption.ItemCountUnit`) names the word after the count, singular and plural, or leaves the number alone.
- Option rows draw the option's picture as a rounded thumbnail in place of its icon, and its item count at the trailing edge (also read by VoiceOver). New appearance tokens `optionImageSize`, `optionImageCornerRadius`, `optionImageSpacing`, `optionImagePlaceholderColor`, `optionCountColor`, `optionCountOpacity`.
- `CompletedParamState.imageURL`: the picture of the option that answered the param, so a custom renderer can show it on the chip.
- Options carry the server's optional `image_url` and `item_count`: `SuggestionOption.imageURL` / `itemCount`, with `imageSource` (only an `http(s)` URL with a host is loaded), `hasImage` and `itemCountLabel` ("12 items").
- An option row's loading skeleton keeps that row's icon column, so its bar starts where the text will and nothing shifts when the options return.
- A completed chip draws the SF Symbol of the option that answered it inside its highlight capsule (`showChipIcons`).
- Option icons carry onto the param they answer: `SuggestionOption.iconSvg` (the server's `icon_svg` markup, carried but not drawn by the bundled views), `CompletedParamState.icon` / `iconSvg`, and `Configuration.showOptionIcons` / `showChipIcons` (both default `true`).
- The input's leading inset is applied with a floor (currently `5pt`), the tightest inset the dropdown's option text can still line up with. A host that set `inputInsets.left` below it now gets the floor.
- The dropdown's loading skeleton after an answer mirrors the options the user just answered — same row count and widths — instead of the next pill's. New `AIAutocompleteController.dropdownOptions`, `dropdownSuggestion` and `drawsSkeletonMirror` (with `skeletonOptions` / `skeletonSuggestion`) for custom dropdowns.
- Widget-local light, dark, and auto mode through `AIAutocompleteAppearance`, including live UIKit and SwiftUI updates without changing the host window.
- `AIAutocompleteAppearance.writtenTextFontWeight` sets the weight of typed text and the placeholder without restyling completed params (`nil`, the default, keeps `font` unchanged).
- Per-option parameters and canonical catalog values through selection, exact-match promotion, requests, submit, and re-edit.
- Server-declared date formats and a native two-endpoint date-range picker.
- `Suggestion.leadIn`: a date or date range picker types the server's lead-in ("from", "until") as plain text before the picked date, while the completed param keeps only the date. Not typed when the text already ends with those words or the date starts with them.
- A time wheel for parameters the server declares as `type.kind == "time"`: `FormatType.time`, `controller.timeView`, `setTime(minutes:)` / `stepTime(by:)` / `resetTime()` / `commitTime()`. A committed time reads `7:30 PM` and carries `metadata["aiaTime"] = "19:30"`. `FormatType.isPicker` asks "are this pill's options hidden?"; `isCalendar` is now true only for dates and date ranges.
- multi_select questions in Core (`type.kind == "multi_select"`): a row tap toggles a pick (`toggleMultiSelectOption(_:)`, `controller.multiSelectView.picked`) and sends nothing. `commitMultiSelect()` answers with every pick in one request, joined in plain words ("Red, Blue and Green") as one chip, with one `completed_params` entry per value (`CompletedParamState.group`). With no picks it skips the question. Group chips never open for re-edit; `removeGroupedParam(atCaret:)` deletes one whole.
- multi_select rows answer on a tap outside the checkbox: a tap on or just past the checkbox (10pt beyond it, at least 44pt from the row's leading edge) still ticks the row and waits for Done, and a tap anywhere else on the row answers with that option plus anything already ticked. VoiceOver, Switch Control and the hardware keyboard keep toggling. New `AIAutocompleteController.answerMultiSelect(with:)`.
- A multi_select list shows at most three rows and scrolls past that, whatever `maxVisibleOptionRows` says, so Done stays in view.
- multi_select in UIKit: option rows lead with an 18pt checkbox (soft accent fill when picked, read as selected by VoiceOver), and a trailing capsule Done button sits under the options. Return commits the picks, and Backspace removes a group chip in one press.
- A "Smart Suggestion" header, with a twinkling purple sparkle, over traditional autocomplete options in the dropdown, in the web SDK's 18 languages.
- Traditional autocomplete in Core, as in the web SDK: requests send `accepts_traditional_autocomplete`, a `traditional_autocomplete` suggestion answers its response alone with every option shown, it is never skipped, tapped, promoted or listed in `recently_suggested`, and a pick replaces the typed fragment with a chip named after the option's parameter.
- Submitting (`submit()`, `submitAndReset()`, keyboard return, the built-in submit button) turns a typed phrase into a completed param when it equals exactly one traditional autocomplete candidate. New `prepareSubmit()` does the same for a submit result you build yourself. Typing, space and Tab are unchanged.
- Every suggest request names the parameters your `optionOverrides` supply options for in `client_option_parameters` (`[]` when there are none), so the server skips client-supplied parameters you don't serve.
- `Configuration.commitOnWordEnd` (default `false`): a word the user is still typing becomes a chip — exact match or server-identified param — only once a space, Return/submit, or hardware-keyboard Tab ends it. `endTypedWord()` ends the word from headless key handling.
- Putting the keyboard away (a tap on the page, or the keyboard's hide key) now leaves the open dropdown on screen; the next tap outside the input and dropdown closes it. A tap on a button or another field outside still closes it straight away. New `AIAutocompleteController.isDropdownHeld` and `setDropdownHeld(_:)`. Behaviour change: `AIAutocompleteView.blur()` likewise leaves an open dropdown up; call `controller.setDropdownHeld(false)` too to close it at once.

### Fixed
- A date written as a day alone ("on the 1st", "the 23rd") now reads as the next date on that day, today included, so its calendar opens with that day marked. A day past 31 or a bare number still reads as nothing. Mirrors JS #469.
- With `optionsPosition: .above`, the option rows cascade in from the bottom row (the one against the input) instead of the top, as on the web.
- A product card's "Sold out" badge is uppercased by `language`'s casing rules rather than the device's, so Turkish reads "TÜKENDİ". A sold-out card's desaturated picture keeps its orientation exactly once.
- A catalog-grounded response with any applied or dropped filter failed to decode: schema 1.0 names each filter's parameter `parameter`, and `ExpressedFilter` still required the pre-1.0 `param`. It now reads `parameter`, still accepting `param`; `ExpressedFilter.param` and `init(param:value:op:)` are deprecated in favour of `parameter`.
- Draw the date picker in the UIKit `AIAutocompleteView`: a date parameter showed only the pill bar, with no calendar to answer it.
- Send a re-edited answer back to the server when the replacement is typed to an exact match, as a tapped replacement already did, so the suggestions that follow reflect the new answer.
- Make `.focused(_:)` on the SwiftUI `AIAutocomplete` track the field: the `@FocusState` now reads `true` while the input is focused, and clearing it dismisses the keyboard.
- Fetch suggestions when the query is replaced by a different text of the same length, such as a paste or select-all.
- A picked option joins the typed text the way the server's optional `completes_word` flag says (`Suggestion.completesWord`): `true` replaces the last typed word, `false` appends the option without removing overlapping text, and absent keeps the overlap removal as before.
- Preserve UIKit’s Dynamic Type font rounding when resolving the stock input and option fonts at a requested content size category.
- `language` on `Configuration` and `AIAutocompleteController`: a BCP 47 tag sent as `meta.language` on every suggest request, and as `language` on every telemetry event, instead of the device language. Assigning the controller property never fires a request by itself, and it survives `reset()`.
- Support the server's `meta.phrase_exact_match` switch (`AutocompleteResponse.Meta.phraseExactMatch`). When it is on, a typed phrase that completes an option starting at the word the last response cut becomes a chip. For example, "Pantalones de mujer" typed past suggestions for "Pantalones" now becomes a chip instead of leaving the search stuck on the "Pantalones" result.
- `AIAutocompleteController.requestId`: the `meta.request_id` of the latest applied `/suggest` request, so you can join your own analytics events to the suggestions that were on screen. `nil` before the first response and after `reset()`.
- `currency` on `Configuration` and `AIAutocompleteController` (`CurrencyOptions`: ISO 4217 code, conversion rate, optional country), sent as `meta.currency`, `meta.currency_rate` and `meta.country` when the code is non-empty and the rate is finite and positive. Assigning the controller property never fires a request by itself, and it survives `reset()`.
- `Configuration.productId`: what the dropdown offers before anything is typed is read from a cacheable `…/starting-state/{productId}/{language}` URL with no credential (`{language}` is the lowercased primary subtag of the request language, e.g. `es` for `es-MX`, except that Traditional Chinese — `zh-TW`, `zh-HK`, `zh-MO` or a `zh-Hant` script, any case, `-` or `_` — reads `zh-TW`; a language without a 2–3 letter one reads `…/starting-state/{productId}`), falling back to a suggest request. The default and dev endpoints read it through their CDN hosts (`cdn.ai-autocomplete.com`, `dev-cdn.ai-autocomplete.com`).

## [1.8.0] - 2026-08-20

### Personalization
- New `additionalContext` on `Configuration` and `AIAutocompleteController`: free-form JSON (`JSONValue`) sent as `additional_context` with every suggest request, so suggestions bend toward what your app knows — a user profile, workspace state, the app being edited. Set it once in the configuration, or update the controller property as the session evolves; updates never fire a request of their own and survive `reset()`
- Keys travel to the server verbatim; keys matching your catalog's field names are honoured most reliably
- New `JSONValue.compactUTF8ByteCount` and `AutocompleteRequest.maxAdditionalContextBytes` (2000): size payloads against the server's cap, measured exactly as the request encoder puts them on the wire — the server truncates silently past the cap, never rejects
- Pairs with server-side generated starting states: a product whose saved starting state has no options gets model-generated — and therefore personalized — options on the very first screen, with no client change

### Fixes
- `customPlaceholder` now works: it overrides the server-suggested ghost text in an empty input, and `""` hides the ghost entirely. Display-only — selecting an option from an empty input still builds the query from the server placeholder, so requests are unchanged

## [1.7.0] - 2026-08-17

### Skip suggestions
- A new **skip** button at the trailing edge of the dropdown's pill bar lets users dismiss the active parameter without answering it. Skipped params are echoed to the server so it stops re-suggesting them.
- `AIAutocompleteController.skipActivePill()` — headless skip API for custom UIs, alongside a `skippedParams` property.
- `Configuration.showSkipButton` (default `true`) — opt out of the built-in button (consulted for `.dropdown` pill placement).
- New `AIAutocompleteAppearance.skipButtonTextColor` token for the button's resting ink.
- The button hides during re-edit, while the input is empty, and when there's no real pill to skip; it's disabled during the brief selection animation.

### Fixes
- Dropdown parameter pills no longer play a brief "swipe to reveal" wipe when they change (most visible right after tapping skip).

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
