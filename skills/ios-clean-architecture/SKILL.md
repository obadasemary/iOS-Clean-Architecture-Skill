---
name: ios-clean-architecture
description: Modular Clean Architecture template for iOS apps using SwiftUI, SPM, and Swift 6.2. Scaffolds layered packages with protocol-first boundaries, `@Observable` view models, Builder-based DI, and Swift Testing doubles.
---

# iOS Clean Architecture (SwiftUI + SPM, Swift 6.2)

A modular Clean Architecture template for iOS apps built with Swift 6.2 and SwiftUI. Outer layers depend on inner layers, never the reverse. Data flows unidirectionally from user interactions through ViewModels → UseCases → Repositories → NetworkServices.

## When To Use This Skill

Use this skill when the user is:

- Scaffolding a new SwiftUI feature module and needs layered structure
- Splitting a monolithic iOS app into multiple SPM packages
- Migrating from `ObservableObject` / `@Published` to `@Observable`
- Enabling Swift 6.2 default actor isolation (`.defaultIsolation(MainActor.self)` per target) to reduce explicit `@MainActor` annotations
- Wiring dependencies through a service-locator DI container
- Adding Swift Testing doubles for an isolated view-model / use-case / repository

Do **not** use this skill for: UIKit-first apps, Combine-heavy pipelines the user wants to preserve, or projects that have explicitly rejected multi-module SPM layouts.

## Dependency Direction

```
View  →  ViewModel  →  UseCase  →  Repository  →  NetworkService
                                        ↓
                                    Endpoints
```

Outer layers may import inner layers. Never the reverse. Views do not import Repositories. ViewModels do not call `URLSession`. Cross-layer calls route through a UseCase.

## Module Layout

Each responsibility lives in its own SPM package:

```
Packages/
  NetworkService/          # URLSession wrapper, request execution
  Endpoints/               # Endpoint definitions per API
  Repositories/            # Repository protocols + default implementations
  UseCases/                # UseCase protocols + default implementations
  DependencyContainer/     # Service-locator registration + resolution
  Features/
    CharacterList/         # View + ViewModel + Builder for one feature
    CharacterDetails/
  SharedUI/                # Reusable SwiftUI components
  PreviewSupport/          # Preview fixtures and fakes
App.xcworkspace            # App target + local packages
```

Cross-module dependencies are always on **protocols**, never concrete types.

## Core Patterns

### Endpoint

```swift
import Foundation

// Endpoints/Sources/Endpoints/CharacterEndpoint.swift
public struct CharacterEndpoint: Sendable {
    public let path: String
    public let query: [URLQueryItem]

    public static func list(page: Int) -> Self {
        .init(path: "/character", query: [.init(name: "page", value: "\(page)")])
    }
}
```

### Repository

```swift
// Repositories/Sources/Repositories/CharacterRepository.swift
public protocol CharacterRepository: Sendable {
    func fetchCharacters(page: Int) async throws -> [Character]
}

public struct DefaultCharacterRepository: CharacterRepository {
    private let network: NetworkService
    public init(network: NetworkService) { self.network = network }

    public func fetchCharacters(page: Int) async throws -> [Character] {
        try await network.request(CharacterEndpoint.list(page: page))
    }
}
```

### UseCase

```swift
// UseCases/Sources/UseCases/FetchCharactersUseCase.swift
public protocol FetchCharactersUseCase: Sendable {
    func execute(page: Int) async throws -> [Character]
}

public struct DefaultFetchCharactersUseCase: FetchCharactersUseCase {
    private let repository: CharacterRepository
    public init(repository: CharacterRepository) { self.repository = repository }

    public func execute(page: Int) async throws -> [Character] {
        try await repository.fetchCharacters(page: page)
    }
}
```

### @Observable ViewModel

`@Observable` replaces `ObservableObject` / `@Published`. Swift 6.2 (SE-0466) lets you opt an entire module into main-actor isolation by adding `.defaultIsolation(MainActor.self)` to each target in `Package.swift`:

```swift
// Package.swift (per-target opt-in)
.target(
    name: "CharacterList",
    swiftSettings: [.defaultIsolation(MainActor.self)]
)
```

With that setting, all types in the target are implicitly `@MainActor` — no manual annotation needed. **Without it, types default to nonisolated** and you must add `@MainActor` explicitly. Prefer explicit `@MainActor` on ViewModels unless you are certain every target in the module has `.defaultIsolation(MainActor.self)` enabled.

```swift
// Features/CharacterList/Sources/CharacterList/CharacterListViewModel.swift
@Observable
@MainActor
public final class CharacterListViewModel {
    public private(set) var characters: [Character] = []
    public private(set) var isLoading = false
    public private(set) var error: Error?

    private let fetchCharacters: FetchCharactersUseCase

    public init(fetchCharacters: FetchCharactersUseCase) {
        self.fetchCharacters = fetchCharacters
    }

    public func load(page: Int = 1) async {
        isLoading = true
        defer { isLoading = false }
        do {
            characters = try await fetchCharacters.execute(page: page)
        } catch {
            self.error = error
        }
    }
}
```

### Builder

Feature instantiation is mandatory through a Builder. View sites stay free of construction logic.

```swift
// Features/CharacterList/Sources/CharacterList/CharacterListBuilder.swift
public struct CharacterListBuilder {
    private let container: DIContainer
    public init(container: DIContainer) { self.container = container }

    public func build() -> CharacterListView {
        let viewModel = CharacterListViewModel(
            fetchCharacters: container.resolve(FetchCharactersUseCase.self)
        )
        return CharacterListView(viewModel: viewModel)
    }
}
```

### DIContainer (service locator)

```swift
// DependencyContainer/Sources/DependencyContainer/DIContainer.swift
public final class DIContainer: @unchecked Sendable {
    private let lock = NSLock()
    private var factories: [ObjectIdentifier: @Sendable () -> Any] = [:]

    public init() {}

    public func register<T>(_ type: T.Type, factory: @escaping @Sendable () -> T) {
        lock.withLock { factories[ObjectIdentifier(type)] = factory }
    }

    public func resolve<T>(_ type: T.Type) -> T {
        let factory = lock.withLock { factories[ObjectIdentifier(type)] }
        guard let factory, let value = factory() as? T else {
            fatalError("Dependency \(type) not registered")
        }
        return value
    }
}
```

Register at app launch (all registrations happen before any async work begins):

```swift
let container = DIContainer()
let network = DefaultNetworkService()
container.register(CharacterRepository.self) {
    DefaultCharacterRepository(network: network)
}
container.register(FetchCharactersUseCase.self) {
    DefaultFetchCharactersUseCase(repository: container.resolve(CharacterRepository.self))
}
```

## Testing With Swift Testing

Use `swift-testing`, not XCTest. Test doubles conform to protocols and live in the same package's test target.

```swift
import Testing
@testable import CharacterList

struct CharacterListViewModelTests {

    @Test
    func loadPopulatesCharacters() async {
        let useCase = FakeFetchCharactersUseCase(result: .success([.preview]))
        let viewModel = CharacterListViewModel(fetchCharacters: useCase)

        await viewModel.load()

        #expect(viewModel.characters == [.preview])
        #expect(viewModel.isLoading == false)
        #expect(viewModel.error == nil)
    }
}

final class FakeFetchCharactersUseCase: FetchCharactersUseCase, @unchecked Sendable {
    let result: Result<[Character], Error>
    init(result: Result<[Character], Error>) { self.result = result }
    func execute(page: Int) async throws -> [Character] { try result.get() }
}
```

Test double naming convention:
- `Mock*` — programmable expectations (matches arguments, returns canned replies)
- `Fake*` — lightweight stand-in with in-memory behavior
- `Spy*` — records calls for later assertion (common for routers / coordinators)

## Critical Rules (DO / DON'T)

**DO**
- Add `.defaultIsolation(MainActor.self)` to each SwiftUI feature target in `Package.swift` to get implicit `@MainActor` isolation — then omit the explicit annotation.
- Add explicit `@MainActor` to ViewModels in targets that have **not** enabled `.defaultIsolation(MainActor.self)`.
- Make cross-module types `Sendable` when they cross concurrency boundaries.
- Depend on protocols across modules.
- Route every network call through a UseCase.
- Use `.xcworkspace` when the app target consumes local SPM packages.

**DON'T**
- Assume `@MainActor` is implicit without first confirming `.defaultIsolation(MainActor.self)` is set for that target.
- Force-unwrap outside Builder internals.
- Let a View import a Repository or NetworkService directly.
- Let a ViewModel call `URLSession` directly.
- Import concrete types across modules when a protocol exists.
- Use `.xcodeproj` alone once you have local SPM packages.

## Migration Notes: ObservableObject → @Observable

- Remove `: ObservableObject`, add `@Observable` to the class declaration.
- Delete every `@Published` — `@Observable` tracks stored properties automatically.
- In views, replace `@StateObject` / `@ObservedObject` with `@State` / `@Bindable`.
- Remove explicit `@MainActor` on the class only if the target has `.defaultIsolation(MainActor.self)` enabled; otherwise keep it.
- Update tests that relied on Combine publishers — read properties directly after `await`.

## Checklist For A New Feature Module

1. Create the SPM package under `Packages/Features/<FeatureName>/`.
2. Add `View`, `ViewModel` (`@Observable`), and `Builder` files.
3. Depend on UseCase protocols only — no concrete UseCase imports.
4. Register any new UseCase / Repository in the DIContainer.
5. Add Swift Testing target with `Fake*` / `Mock*` / `Spy*` doubles.
6. Wire the Builder into the parent navigation flow.
7. Run `swift test` per package before integrating.
