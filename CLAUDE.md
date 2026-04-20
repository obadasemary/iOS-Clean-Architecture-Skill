# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code Agent Skill** repository — not an iOS app. It packages architectural guidance for iOS projects as a reusable plugin that Claude Code can install and invoke. There is no build system, no compiled code, and no test runner.

The repository structure:

```
.claude-plugin/
  plugin.json          # Plugin manifest (name, version, skills pointer)
  marketplace.json     # One-plugin marketplace catalog
skills/
  ios-clean-architecture/
    SKILL.md           # The skill content Claude reads and applies
```

## Making Changes

The only file that affects the skill's behavior is `skills/ios-clean-architecture/SKILL.md`. Edits there change what Claude does when the skill is invoked in an iOS project.

`plugin.json` controls the plugin name, version, and description shown in the marketplace. Bump `version` when publishing a new release.

`marketplace.json` mirrors the plugin metadata for catalog discovery — keep it in sync with `plugin.json` after version bumps.

## Installing the Skill

```bash
/plugin marketplace add Obadasemary/ios-clean-architecture-skill
/plugin install ios-clean-architecture@ios-clean-architecture
```

To enable for all contributors in an iOS project, add to that project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "ios-clean-architecture@ios-clean-architecture": true
  },
  "extraKnownMarketplaces": {
    "ios-clean-architecture": {
      "source": { "source": "github", "repo": "Obadasemary/ios-clean-architecture-skill" }
    }
  }
}
```

## Architecture Guidance Encoded in the Skill

The skill enforces a unidirectional dependency chain:

```
View → ViewModel → UseCase → Repository → NetworkService
```

Key patterns the skill scaffolds and enforces:

- **Swift 6.2 default actor isolation** — main-actor applies automatically; never write `@MainActor` manually in targets with default isolation enabled
- **`@Observable` ViewModels** — replaces `ObservableObject` / `@Published`
- **Protocol-first cross-module contracts** — modules depend on protocols, never on concrete types from other modules
- **Builder pattern** — all feature instantiation goes through a dedicated Builder; view sites contain no construction logic
- **Service-locator DIContainer** — centralized registration and resolution
- **Swift Testing** (not XCTest) — test doubles follow `MockNetworkService`, `FakeFeedUseCase`, `SpyRouter` naming

Hard constraints the skill enforces:
- ViewModels must never call `URLSession` or Repositories directly — every cross-layer call routes through a UseCase
- Force unwrapping is only acceptable inside Builder internals
- Open the `.xcworkspace`, not `.xcodeproj`
