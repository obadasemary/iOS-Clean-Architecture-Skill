# iOS Clean Architecture Agent Skill

An open-source Agent Skill that scaffolds and enforces a modular Clean Architecture for iOS apps built with **SwiftUI** and **Swift Package Manager** on **Swift 6.2** with default main-actor isolation.

## Quick Start

Install using [npx skills](https://skills.sh):

```bash
npx skills add Obadasemary/ios-clean-architecture-skill
```

Then open your iOS project and ask:

> Use the ios-clean-architecture skill to scaffold a new feature module with View, ViewModel, UseCase, Repository, and DI wiring.

The skill will set up layered SPM packages, protocol-first cross-module contracts, `@Observable` view models, Builder-based feature instantiation, and Swift Testing doubles — following the patterns documented in `skills/ios-clean-architecture/SKILL.md`.

## What The Skill Covers

- **Swift 6.2 default actor isolation** — main-actor applies automatically; no manual `@MainActor` sprinkled across the codebase
- **SwiftUI + `@Observable`** — modern state management in place of `ObservableObject` / `@Published`
- **Layered SPM packages** — NetworkService, Endpoints, Repositories, UseCases, DependencyContainer, feature view packages, shared UI, preview utilities
- **Protocol-first cross-module boundaries** — modules depend on protocols, never on concrete types from other modules
- **Service-locator DIContainer** — centralized dependency registration and resolution
- **Builder pattern** — mandatory feature instantiation through dedicated builders, keeping view sites free of construction logic
- **Swift Testing** — `MockNetworkService`, `FakeFeedUseCase`, `SpyRouter`-style doubles for isolated component verification

## Dependency Direction

```
View  →  ViewModel  →  UseCase  →  Repository  →  NetworkService
```

Outer layers depend on inner layers, never the reverse. Views never touch repositories directly, view models never touch `URLSession` directly — every cross-layer call routes through a use case.

## Critical Constraints The Skill Enforces

- Do **not** write `@MainActor` manually in Swift 6.2 targets with default isolation enabled.
- Avoid force unwrapping except within Builder internals.
- Never allow ViewModels to call `URLSession` or Repositories directly — route through UseCases.
- Use `.xcworkspace`, not `.xcodeproj` alone.

## Installation Options

### Option A: npx skills (recommended)

Works with Claude Code, Cursor, Copilot, and [40+ other agents](https://skills.sh):

```bash
npx skills add Obadasemary/ios-clean-architecture-skill
```

Install globally (available across all projects):

```bash
npx skills add Obadasemary/ios-clean-architecture-skill -g
```

Install to a specific agent only:

```bash
npx skills add Obadasemary/ios-clean-architecture-skill -a claude-code
```

### Option B: Claude Code Plugin

1. Add the marketplace:

```bash
/plugin marketplace add Obadasemary/ios-clean-architecture-skill
```

2. Install the plugin:

```bash
/plugin install ios-clean-architecture@ios-clean-architecture
```

To enable for everyone in a repository, add to your project's Claude Code configuration:

```json
{
  "enabledPlugins": {
    "ios-clean-architecture@ios-clean-architecture": true
  },
  "extraKnownMarketplaces": {
    "ios-clean-architecture": {
      "source": {
        "source": "github",
        "repo": "Obadasemary/ios-clean-architecture-skill"
      }
    }
  }
}
```

### Option C: Manual Install

1. Clone this repository.
2. Copy or symlink `skills/ios-clean-architecture/` into your agent's skills directory.
3. Ask the agent to use the `ios-clean-architecture` skill.

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
