# 네비게이션: 글로벌 라우팅

## 1. 개요
 
링크드인에서 네비게이션을 Environment 로 관리하는 것에 대한 코드를 보았는데 참고하기 좋은 코드인 것 같다.

## 2. 글로벌 라우팅

### 2.1. 라우트 관리를 위한 NavigationAction 정의

```swift
struct NavigationAction {
    let action: (Route) -> ()
    func callAsFunction(_ route: Route) {
        action(route)
    }
}
```

### 2.2 네비게이션을 위한 Envionment

```swift
struct NavigationEnvironmentKey: EnvironmentKey {
    static var defaultValue: NavigationAction = NavigationAction(action: { _ in })
}

extension EnvironmentValues {
    var navigate: (NavigationAction) {
        get { self[NavigateEnvironmentKey.self] }
        set { self[NavigateEnvironmentKey.self] = newValue }
    }
}
```

```swift
struct ContentView: View {
    @Environment(\.navigate) private var navigate

    var body: some View {
        VStack {
            Button("로그인") {
                navigate(.login)
            }
            Button("제품 상세") {
                navigate(.detail(Product(name: "쿠키")))
            }
        }
    }
}
```

### 2.3. 네비게이션 동작을 위한 onNavigation 함수 정의

```swift
extension View {
    func onNavigate(_ action: @escaping NavigateAction.Action) -> some View {
        self.environment(\.navigate, NavigateAction(action: action))
    }
}
```

### 2.4. 사용 예시

```swift
struct ContentViewContainer: View {
    @State private var routes: [Route] = []

    var body: some View {
        NavigationStack(path: $routes) {
            ContentView()
                .navigationDestination(for: Route.self) { route in
                    switch route {
                    case .home:
                        HomeView()
                    case .login:
                        LoginView()
                    case .detail(let product):
                        DetailView(product: product)
                    }
                }
        }
        .onNavigation { route in
            routes.append(route)
        }
    }
}
```

## 3. 결론

스택 방식의 네비게이션에는 아주 좋은 참고 코드가 될 수 있다고 생각한다. 하지만 트리방식에는 적용하기엔 적합하지 않으므로 트리방식에는 어떻게 사용할 수 있을지도 고민해보면 좋을 것 같다.

## 4. 출처

[Global Routing in SwiftUI - Mohammad Azam | Linkedin](https://www.linkedin.com/posts/mohammad-azam-5537993_swiftui-iosdevelopment-iosdev-activity-7130749881278742530-3gKp?utm_source=share&utm_medium=member_desktop)
