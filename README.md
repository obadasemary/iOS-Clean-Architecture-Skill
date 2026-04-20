# iOS Clean Architecture Agent Skill

An open-source Agent Skill that scaffolds and enforces a modular **Clean Architecture** for iOS apps built with **SwiftUI** and **Swift Package Manager** on **Swift 6.2** with default main-actor isolation.

## Quick Start

Add `extraKnownMarketplaces` and `enabledPlugins` to your project's `.claude.json` (create it at the repo root if it doesn't exist):

```json
{
  "extraKnownMarketplaces": {
    "ios-clean-architecture": {
      "source": {
        "source": "github",
        "repo": "Obadasemary/ios-clean-architecture-skill"
      }
    }
  },
  "enabledPlugins": {
    "ios-clean-architecture@ios-clean-architecture": true
  }
}
```

Then open your iOS project and ask:

> Use the `ios-clean-architecture` skill to scaffold a new feature module with View, ViewModel, UseCase, Repository, Builder, and DI wiring.

The skill sets up layered SPM packages, protocol-first cross-module contracts, `@Observable` view models, Builder-based feature instantiation, and Swift Testing doubles — following the patterns documented in [`skills/ios-clean-architecture/SKILL.md`](skills/ios-clean-architecture/SKILL.md).

## Example Prompts

- "Scaffold a new `CharacterList` feature module following the `ios-clean-architecture` skill."
- "Migrate `ProfileViewModel` from `ObservableObject` / `@Published` to `@Observable` per the skill's rules."
- "Add a `FetchOrdersUseCase` and register it in the DIContainer."
- "Write Swift Testing doubles (`FakeOrdersRepository`, `SpyRouter`) for the new feature."
- "Audit this file for violations of the skill's DO/DON'T checklist."

## What The Skill Covers

- **Swift 6.2 default actor isolation** — main-actor applies automatically; no manual `@MainActor` sprinkled across the codebase
- **SwiftUI + `@Observable`** — modern state management in place of `ObservableObject` / `@Published`
- **Layered SPM packages** — NetworkService, Endpoints, Repositories, UseCases, DependencyContainer, feature view packages, shared UI, preview utilities
- **Protocol-first cross-module boundaries** — modules depend on protocols, never on concrete types from other modules
- **Service-locator DIContainer** — centralized dependency registration and resolution
- **Builder pattern** — mandatory feature instantiation through dedicated builders, keeping view sites free of construction logic
- **Swift Testing** — `Mock*` / `Fake*` / `Spy*` doubles for isolated component verification

## Dependency Direction

```
View  →  ViewModel  →  UseCase  →  Repository  →  NetworkService
                                        ↓
                                    Endpoints
```

Outer layers depend on inner layers, never the reverse. Views never touch repositories directly; view models never touch `URLSession` directly — every cross-layer call routes through a UseCase.

## Critical Constraints The Skill Enforces

- Do **not** write `@MainActor` manually in Swift 6.2 targets with default isolation enabled.
- Avoid force unwrapping except within Builder internals.
- Never allow ViewModels to call `URLSession` or Repositories directly — route through UseCases.
- Use `.xcworkspace`, not `.xcodeproj` alone.
- Cross-module dependencies are always on protocols.

Full DO / DON'T checklist lives in the [SKILL.md](skills/ios-clean-architecture/SKILL.md#critical-rules-do--dont).

## Installation Options

### Option A: `.claude.json` (recommended)

Add the plugin directly to your project's `.claude.json` at the repo root. This is the most reliable method and works regardless of the state of other registered marketplaces.

```json
{
  "extraKnownMarketplaces": {
    "ios-clean-architecture": {
      "source": {
        "source": "github",
        "repo": "Obadasemary/ios-clean-architecture-skill"
      }
    }
  },
  "enabledPlugins": {
    "ios-clean-architecture@ios-clean-architecture": true
  }
}
```

Commit this file to share the skill across your entire team automatically.

### Option B: CLI install

```bash
/plugin marketplace add Obadasemary/ios-clean-architecture-skill
/plugin install ios-clean-architecture@ios-clean-architecture
```

> **Known issue:** `/plugin install` loads all registered marketplaces before resolving the target plugin. If Anthropic's `claude-plugins-official` marketplace has schema validation errors (a known upstream bug), this command will fail with an unrelated error. Use Option A or Option C in that case.

### Option C: Manual Install

1. Clone this repository.
2. Copy or symlink `skills/ios-clean-architecture/` into your Claude skills directory.
3. Ask Claude to use the `ios-clean-architecture` skill.

Useful docs: [Claude Code Agent Skills](https://code.claude.com/en/skills)

## Repository Layout

```text
.claude-plugin/
  marketplace.json       # one-plugin marketplace catalog
  plugin.json            # plugin manifest pointing at ./skills/ios-clean-architecture
skills/
  ios-clean-architecture/
    SKILL.md             # the skill content (Swift 6.2, Clean Architecture patterns)
README.md
LICENSE
.gitignore
```

## Who This Is For

- iOS / SwiftUI developers adopting Swift 6.2 concurrency with default main-actor isolation
- teams standardizing a modular SPM layout across multiple feature modules
- engineers migrating away from `ObservableObject` / `@Published` to `@Observable`
- anyone who wants a reusable Agent Skill to keep Clean Architecture discipline across a codebase

## Contributing

Issues and PRs are welcome — especially real-world examples of the patterns applied in production apps, or clarifications to the constraints when Swift / SwiftUI evolve.

## License

MIT. See [LICENSE](LICENSE).

## Author

Created by [Abdelrahman Mohamed (@Obadasemary)](https://github.com/Obadasemary).
