# Part 9. Dependency

## 개요

> Note: [Offical Documentations | pointfreeco](https://pointfreeco.github.io/swift-dependencies/main/documentation/dependencies) and [GitHub Repo | pointfreeco](https://github.com/pointfreeco/swift-dependencies)

```swift
@Dependency(\.continuousClock) var clock  // Controllable way to sleep a task
@Dependency(\.date.now) var now           // Controllable way to ask for current date
@Dependency(\.mainQueue) var mainQueue    // Controllable scheduling on main queue
@Dependency(\.uuid) var uuid              // Controllable UUID creation
```

```swift
func addButtonTapped() async throws {
    try await self.clock.sleep(for: .seconds(1))  // 👈 Don't use 'Task.sleep'
    self.items.append(
        Item(
            id: self.uuid(),  // 👈 Don't use 'UUID()'
            name: "",
            createdAt: self.now  // 👈 Don't use 'Date()'
        )
    )
}
```

## Dependencies 등록하기

1. `DependencyKey` 준수하는 타입 생성
2. `liveValue` 구현: 앱이 돌아갈 때 사용되는 값으로 실제 외부 서버로의 네트워크 요청을 생성하는데 적합
3. `DependencyValues` 의 `extension` 에 dependency를 위한 computed property 추가

```swift
private enum APIClientKey: DependencyKey {
    static let liveValue = APIClient.live
}

extension DependencyValues {
    var apiClient: APIClient {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}
```

### 사용
```swift
final class TodosModel: ObservableObject {
  @Dependency(\.apiClient) var apiClient
  // ...
}
```
Previews 에서는 live dependency를 자동으로 사용하게 된다.
mock 데이터를 쓰고 싶으면 아래와 같이 주입하면 된다.

```swift
@MainActor
func testFetchUser() async {
    let model = withDependencies {
        $0.apiClient.fetchTodos = { _ in Todo(id: 1, title: "Get milk") }
    } operation: {
        TodosModel()
    }

    await store.loadButtonTapped()
    XCTAssertEqual(
        model.todos,
        [Todo(id: 1, title: "Get milk")]
    )
}
```

늘 `DependencyKey` 를 준수하는 새로운 타입을 만들 필요는 없다. 이미 가지고 있는 타입에 대해 dependency를 등록하고자 한다면 그냥 그 타입에 채택하면 된다.
```swift
extension APIClient: DependencyKey {
    static let liveValue = APIClient.live
}

extension DependencyValues {
    var apiClient: APIClient {
        get { self[APIClient.self] }
        set { self[APIClient.self] = newValue }
    }
}
```
That can save a little bit of boilerplate.
