# Part 10. SwitchStore & CaseLet

## 이슈

parent reducer의 state 를 enum으로 선언했을 경우,

```swift
struct ClubhouseReducer: ReducerProtocol {
    enum State: Equatable {
        case auth(AuthReducer.State)
        case room(RoomReducer.State)
        
        public init() {
            self = .auth(.init())
        }
    }
}
```

일반적인 `WithViewStore` 를 사용하면 `scope` 시에 아래와 같은 에러를 마주한다.

```swift
WithViewStore(self.store, observe: { $0 }) { viewStore in
    ZStack {
        Color("sand.light")
            .ignoresSafeArea()
         
        AuthView(
            store: store.scope(
                state: ClubhouseReducer.State.auth,
                action: ClubhouseReducer.Action.auth
            )
        )
    }
}
```

> Warning: Failed to produce diagnostic for expression; please submit a bug report (https://swift.org/contributing/#reporting-bugs) and include the project.

state가 struct 가 아닌 enum 이기 때문에 이에 맞는 `ViewStore`를 사용해야한다. 이때는 `SwitchStore` 를 사용하면 된다. 그리고 state 를 `switch` 구문을 사용해서 case 에 따라 `CaseLet` 을 사용하여 store를 하위 뷰에 주입한다.

```swift
SwitchStore(self.store) { state in
    switch state {
    case .auth:
        CaseLet(state: /ClubhouseReducer.State.auth, action: ClubhouseReducer.Action.auth) { store in
            AuthView(store: store)
        }
    case .room:
        CaseLet(state: /ClubhouseReducer.State.room, action: ClubhouseReducer.Action.room) { store in
            ContentView()
        }
    }
}
```

## 결론

state 가 enum 일 때는 `SwitchStore` 와 `CaseLet` 를 써야한다.

