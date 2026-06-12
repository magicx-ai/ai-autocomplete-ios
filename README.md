# AIAutocompleteKit

AI-powered autocomplete for iOS. As the user types, the SDK suggests query completions with inline **pills** for unfilled parameters (e.g. `[type]`, `[goal]`) and a dropdown of tappable options for the active pill — the same server contract and behavior as the MagicX React SDK, adapted to native iOS idioms.

The SDK ships as three SwiftPM libraries (pick one — each pulls in the layers beneath it):

| Product | What you get | Use when |
| --- | --- | --- |
| `AIAutocompleteSwiftUI` | `AIAutocomplete` view + modifiers | Your screen is SwiftUI |
| `AIAutocompleteUIKit` | `AIAutocompleteView` (input + dropdown), `AIAutocompleteDropdownView` | Your screen is UIKit |
| `AIAutocompleteCore` | Headless `AIAutocompleteController` (state machine + networking, no UI) | You render everything yourself |

## Requirements

- iOS 17.0+
- Xcode 16+ (the binaries ship as library-evolution XCFrameworks built from public `.swiftinterface`)
- A MagicX API key or access-token endpoint (the SDK is gated by API credentials, not by package access)

## Installation

The SDK is distributed as a public, binary-only SwiftPM package — pre-compiled XCFrameworks plus a consumer manifest. No account or auth is needed to install it.

**Xcode:** File → Add Package Dependencies → `https://github.com/magicx-ai/ai-autocomplete-ios.git`, dependency rule "Up to Next Major" from `1.0.0`, then add the one product matching your UI layer.

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

The frameworks are static — nothing to embed, and unused code is stripped from your app at link time.

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

Keep your own text field and drive the shared `AIAutocompleteController`; the dropdown view renders the controller's state. Forward text changes with `updateText(_:)` and read `controller.text` back when the controller mutates it (option selection appends to the query).

```swift
import SwiftUI
import AIAutocompleteCore
import AIAutocompleteSwiftUI

struct ComposerView: View {
    @State private var controller = AIAutocompleteController(
        configuration: .init(apiConfig: .apiKey(.init(apiKey: Secrets.autocompleteKey)))
    )
    @State private var text = ""

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            TextField("Ask anything…", text: $text)
                .textFieldStyle(.roundedBorder)
                .onChange(of: text) { _, newValue in controller.updateText(newValue) }
                .onChange(of: controller.text) { _, newValue in text = newValue }

            AIAutocompleteDropdown(controller: controller)
        }
        .onAppear { controller.start() }
        .onDisappear { controller.stop() }
    }
}
```

UIKit equivalent: create `AIAutocompleteDropdownView(controller:)`, add it below your input, and call `controller.updateText(_:)` from your text-change handler. `moveHighlight(by:)` / `selectHighlighted()` wire up hardware-keyboard arrow/return navigation.

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
        return AccessToken(accessToken: token.value, expiresAt: token.expiresAt)
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

## Versioning

Strict [semver](https://semver.org) from `1.0.0`; pre-releases use `1.0.0-rc.1` tags. Update by bumping your version constraint — each tag is immediately resolvable. Per-version release notes, `.xcframework.zip` artifacts, SHA256 checksums, and DocC archives are attached to the [GitHub Releases](https://github.com/magicx-ai/ai-autocomplete-ios/releases).
