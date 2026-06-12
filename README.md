# AIAutocompleteKit

AI-powered autocomplete for iOS. As the user types, the SDK suggests query completions with inline **pills** for unfilled parameters (e.g. `[type]`, `[goal]`) and a dropdown of tappable options for the active pill.

Three SwiftPM libraries — pick one; each pulls in the layers beneath it:

| Product | What you get | Use when |
| --- | --- | --- |
| `AIAutocompleteSwiftUI` | `AIAutocomplete` view + modifiers | Your screen is SwiftUI |
| `AIAutocompleteUIKit` | `AIAutocompleteView` (input + dropdown), `AIAutocompleteDropdownView` | Your screen is UIKit |
| `AIAutocompleteCore` | Headless `AIAutocompleteController` (state machine + networking, no UI) | You render everything yourself |

## Requirements

- iOS 17.0+, Xcode 16+
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

The keyboard's return key renders as a search key and runs the same submit-and-reset sequence as the submit button; pasted line breaks are flattened to spaces, so the query stays a single line.

Add a custom submit button and a themed appearance:

```swift
AIAutocomplete(configuration: configuration)
    .submitButton {
        Image(systemName: "arrow.up.circle.fill")
    }
    .aiAutocompleteAppearance(brandAppearance)
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
    pillPlacement: .inline,        // .inline | .dropdown | .hidden
    autoFocus: true,
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
