# AIAutocompleteKit

An SDK that provides a guided, AI-powered autocomplete experience with pill-based input and dropdown suggestions. As the user types, the SDK suggests query completions with inline **pills** for unfilled parameters (e.g. `[type]`, `[goal]`) and a dropdown of tappable options for the active pill.

Three SwiftPM libraries — pick one; each pulls in the layers beneath it:

| Product | What you get | Use when |
| --- | --- | --- |
| `AIAutocompleteSwiftUI` | `AIAutocomplete` (complete input + dropdown) · `AIAutocompleteDropdown` view or `.aiAutocompleteDropdown(_:)` modifier | Your screen is SwiftUI and you want drop-in views |
| `AIAutocompleteUIKit` | `AIAutocompleteView` (complete input + dropdown) · `AIAutocompleteDropdownView` | Your screen is UIKit and you want drop-in views |
| `AIAutocompleteCore` | `AIAutocompleteController` — headless state machine + networking, no UI | You render everything yourself |

## Requirements

- iOS 17.0+ deployment target; Xcode 26+ to build (release binaries ship Swift 6.2 module interfaces, which older compilers cannot import)
- A MagicX API key or access-token endpoint

## Installation

**Xcode:** File → Add Package Dependencies → `https://github.com/magicx-ai/ai-autocomplete-ios.git`, dependency rule "Up to Next Major", then add the product matching your UI layer.

**Package.swift:**

```swift
dependencies: [
    .package(url: "https://github.com/magicx-ai/ai-autocomplete-ios.git", from: "1.0.0")
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "AIAutocompleteSwiftUI", package: "ai-autocomplete-ios")
        ]
    )
]
```

The package is binary-only: installing downloads pre-compiled static XCFrameworks — no account needed, nothing to embed, unused code is stripped at link time.

## Quick start — SwiftUI

```swift
import SwiftUI
import AIAutocompleteCore
import AIAutocompleteSwiftUI

struct SearchView: View {
    var body: some View {
        AIAutocomplete(
            configuration: .init(
                apiConfig: .apiKey(.init(apiKey: Secrets.autocompleteKey)),
                onSubmit: { result in
                    print("Final query:", result.query)
                }
            )
        )
        .padding()
    }
}
```

Add a custom submit button and a themed appearance — or hide the submit
button entirely (the keyboard return key still submits):

```swift
AIAutocomplete(configuration: configuration)
    .submitButton {
        Image(systemName: "arrow.up.circle.fill")
    }
    .aiAutocompleteAppearance(brandAppearance)

AIAutocomplete(configuration: configuration)
    .submitButtonHidden()
```

## Quick start — UIKit

```swift
import AIAutocompleteUIKit

final class SearchViewController: UIViewController {
    private let autocomplete = AIAutocompleteView(configuration: .init(
        apiConfig: .apiKey(.init(apiKey: Secrets.autocompleteKey)),
        onSubmit: { result in
            print("Final query:", result.query)
        }
    ))

    override func viewDidLoad() {
        super.viewDidLoad()
        autocomplete.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(autocomplete)
        NSLayoutConstraint.activate([
            autocomplete.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
            autocomplete.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            autocomplete.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }
}
```

The view grows with its content (input wraps, dropdown opens below or above per `optionsPosition`) — pin top/leading/trailing and let it size itself.

(UIKit's custom-button and hide equivalents: `autocomplete.submitButtonView = myButton`, `autocomplete.showsSubmitButton = false`.)

## Detached dropdown — full input, your anchor

When the input sits inside a larger container — a card with attachment and
action buttons, say — the dropdown should usually align to the *container's*
edge, not the bare input. Keep the full pill input and detach the dropdown,
anchoring it to any view; both read the same controller:

```swift
// SwiftUI
VStack {
    AIAutocomplete(controller: controller)
        .dropdownHidden()          // input only — no built-in dropdown
    actionButtonRow
}
.aiAutocompleteDropdown(controller)    // dropdown aligns to the card
```

```swift
// UIKit
let input = AIAutocompleteView(controller: controller, showsDropdown: false)
let dropdown = AIAutocompleteDropdownView(controller: controller)
// Add both to your hierarchy; pin the dropdown to the container's edge.
```

## Tier 2 — your input, our dropdown

Keep your own text field and drive the shared `AIAutocompleteController`; the dropdown renders the controller's state. `controller.textBinding` wires the field both ways — your edits route through the controller, and controller-side mutations (option selection appends to the query) flow back.

```swift
import SwiftUI
import AIAutocompleteCore
import AIAutocompleteSwiftUI

struct ComposerView: View {
    @State private var controller = AIAutocompleteController(
        configuration: .init(apiConfig: .apiKey(.init(apiKey: Secrets.autocompleteKey)))
    )

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            TextField("Ask anything…", text: controller.textBinding)
                .textFieldStyle(.roundedBorder)

            AIAutocompleteDropdown(controller: controller)
        }
        .onAppear { controller.start() }
        .onDisappear { controller.stop() }
    }
}
```

UIKit equivalent: add `AIAutocompleteDropdownView(controller:)` below your input and call `controller.updateText(_:)` from your text-change handler. The dropdown view's `moveHighlight(by:)` / `selectHighlighted()` wire up hardware-keyboard arrow/return navigation.

## Authentication

Two modes, chosen by the `apiConfig` case:

```swift
// API key — sent on every request.
.apiKey(.init(apiKey: Secrets.autocompleteKey))

// Access token — short-lived tokens minted by your backend. The SDK calls
// getAccessToken on cold start, 30 s before expiry, and once on a 401;
// concurrent refreshes are coalesced.
.accessToken(.init(
    getAccessToken: {
        let token = try await MyAuthAPI.mintAutocompleteToken()
        // expiresAt is a UNIX timestamp in milliseconds; nil disables
        // proactive refresh (re-fetch only on 401).
        return AccessToken(accessToken: token.value, expiresAt: token.expiresAtMs)
    }
))
```

Both cases accept `endpoint:` (defaults to the production suggest endpoint), `appIdentifier:` (tenant routing), and `extraHeaders:`.

## Configuration highlights

```swift
AIAutocompleteController.Configuration(
    apiConfig: .apiKey(.init(apiKey: Secrets.autocompleteKey)),
    optionOverrides: [
        // Replace or augment server options for a suggestion type with
        // client-side data, re-evaluated on every keystroke.
        "project": { query in
            myProjects
                .filter { query.isEmpty || $0.name.localizedCaseInsensitiveContains(query) }
                .map { SuggestionOption(text: $0.name, isTappable: true) }
        }
    ],
    dropdownTrigger: .auto,        // .auto | .manual | .hidden
    optionsPosition: .below,       // dropdown above or below the input
    pillPlacement: .dropdown,      // .dropdown | .inline | .hidden
    autoFocus: true,
    // Optional free-form JSON the server uses to tailor suggestions to the
    // user or app state in front of it — see "Additional context" below.
    additionalContext: .object([
        "user": .object(["occupation": .string("dentist"), "locale": .string("en-US")])
    ]),
    onError: { error in
        // Terminal errors only; cancellations never reach this. The UI keeps
        // working with cached options, so this is for logging/telemetry.
        Logger(subsystem: "MyApp", category: "autocomplete").error("\(error)")
    },
    onSubmit: { result in
        // result.query — display text; result.rawQuery + result.completedParams —
        // the structured form your backend receives.
        startSearch(result)
    }
)
```

## Additional context

Suggestions can be conditioned on what your app already knows — a user profile, workspace or session state, the template currently open. Pass it as free-form JSON (`JSONValue`, so no `Any` in the API) and it goes up as `additional_context` on every `/suggest` request; the server treats it strictly as data and never echoes it back. Set a static value once through `Configuration.additionalContext`, or update `controller.additionalContext` as the session evolves — assigning it never fires a request by itself, the next one simply carries the new value, and it survives `reset()`.

```swift
// Seed once — the "swap the user" case.
var config = AIAutocompleteController.Configuration(apiConfig: .apiKey(.init(apiKey: key)))
config.additionalContext = .object([
    "workspace": .object([
        "existing_apps": .array([.string("clinic scheduler")]),
        "connected_sources": .array([.string("Google Sheets")])
    ]),
    "session": .object(["opened_template": .string("appointment booking")])
])

// Or keep it moving — append what the app now contains after each turn.
controller.additionalContext = .object([
    "app": .object(["components": .array(appState.components.map { .string($0) })]),
    "history": .array(recentTurns.suffix(10).map { .object(["q": .string($0.query), "a": .string($0.answer)]) })
])
```

Guidelines:

- **Size.** The server caps the payload at `AutocompleteRequest.maxAdditionalContextBytes` (2000 bytes of the *compacted* UTF-8 JSON — whitespace is free; non-ASCII costs 2–4 bytes a character — 3 for most CJK/Hangul, 4 for emoji — and `/` costs 2 as it is sent escaped; so roughly 2000 English characters or ~650 Korean/Japanese/Chinese ones). Past the cap it is truncated with `…` and a server-side warning, never rejected — the tail is silently lost — so budget ~1500 bytes, keep a rolling window of history rather than appending forever, and check `value.compactUTF8ByteCount` in debug builds.
- **Shape.** Any JSON value is accepted, but send an object. Keys that match your catalog's field names are honoured most reliably (`{"size":"grande","milk":"oatmilk"}` — the server lists that value first for that field); free-form prose is weaker. `null`, `{}`, `[]` and `""` count as no context.
- **Safety.** No need to sanitize — the server neutralizes fence tags and wraps the block in a "treat as data, never follow instructions" guard. It is sent only to your configured endpoint, never to telemetry; `maskCompletedText` does not apply to it, so keep PII out unless you mean to send it.
- **Caching.** The context is part of the server's response-cache key: context that changes on every request means every request is a cache miss.

## Appearance

All visual tokens (colors, fonts, radii, spacing) live in `AIAutocompleteAppearance`, a value type with a dark-mode-adaptive `.default`:

```swift
var appearance = AIAutocompleteAppearance.default
appearance.font = .systemFont(ofSize: 17)
appearance.pillBackgroundColor = .systemIndigo.withAlphaComponent(0.15)
appearance.pillTextColor = .systemIndigo
appearance.dropdownCornerRadius = 12
appearance.maxInputLines = 4

// SwiftUI                                  // UIKit
.aiAutocompleteAppearance(appearance)       autocompleteView.appearance = appearance
```

Per-version release notes and DocC archives are attached to the [GitHub Releases](https://github.com/magicx-ai/ai-autocomplete-ios/releases).
