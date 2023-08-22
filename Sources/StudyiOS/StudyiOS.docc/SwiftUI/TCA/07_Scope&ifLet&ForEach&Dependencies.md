# Part 7. Scope & ifLet & ForEach & Dependencies

## Scope

```swift
struct TabAReducer: ReducerProtocol {
    struct State { }
    
    enum Action { }
    
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        return .none
    }
}
```

```swift
struct AppReducer: ReducerProtocol {
    struct State {
        var tabA: TabAReducer.State
        var tabB: TabBReducer.State
    }
    
    enum Action {
        case tabA(TabAReducer.Action)
        case tabB(TabBReducer.Action)
    }
    
    var body: some ReducerProtocol<State, Action> {
        Scope(state: \.tabA, action: /Action.tabA) {
            TabAReducer()
        }
        Scope(state: \.tabB, action: /Action.tabB) {
            TabBReducer()
        }
    }
}
```
## ifLet
```swift
struct FeatureReducer: ReducerProtocol {
    struct State { }
    
    enum Action { }
    
    let date: () -> Date
    
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        return .none
    }
}
```

```swift
struct IfLetParentReducer: ReducerProtocol {
    struct State {
        // 👉🏼 옵셔널
        var feature: FeatureReducer.State?
    }
    
    enum Action {
        case feature(FeatureReducer.Action)
    }
    
    let date: () -> Date
    
    var body: some ReducerProtocol<State, Action> {
        
        Reduce { state, action in
            // Parent logic
            return .none
        }
        // 👀 state 가 `enum` 이면 ``ifCaseLet`` 사용
        .ifLet(\.feature, action: /Action.feature) {
            FeatureReducer(date: self.date)
        }
    }
}
```
## ForEach

```swift
struct RowReducer: ReducerProtocol {
    struct State: Identifiable {
        var id: String { UUID().uuidString }
    }
    
    enum Action { }
    
    let date: () -> Date
    
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        return .none
    }
}
```

```swift
struct ListReducer: ReducerProtocol {
    struct State {
        var rows: IdentifiedArrayOf<RowReducer.State>
    }
    
    enum Action {
        case row(id: RowReducer.State.ID, action: RowReducer.Action)
    }
    
    let date: () -> Date
    
    var body: some ReducerProtocol<State, Action> {
        Reduce { state, action in
            return .none
        }
        .forEach(\.rows, action: /Action.row) {
            RowReducer(date: self.date)
        }
    }
}
```
## Dependencies

```swift
struct APIFeatureReducer: ReducerProtocol {
    let apiClient: APIClient
    let date: () -> Date
    
    struct State { }
    
    enum Action { }
    
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        return .none
    }
}
```
### Pre-defined Dependencies
TCA에서 기본적으로 지원하는 `Dependency` key path 들이 있다. (e.g., `@Dependency(\.date)`)

```swift
struct APIFeatureReducer: ReducerProtocol {
    /// `let date: () -> Date`
    @Dependency(\.date) var date
}
```

### API Client
```swift
struct APIClient {
    static let live = APIClient()
}
```
1. Key 정의
```swift
private enum APIClientKey: DependencyKey {
    static let liveValue = APIClient.live
}
```
2. Value 정의
```swift
extension DependencyValues {
    var apiClient: APIClient {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}
```
3. 사용
```swift
struct APIFeatureReducer: ReducerProtocol {
    /// `let apiClient: APIClient`
    @Dependency(\.apiClient) var apiClient
}
```
