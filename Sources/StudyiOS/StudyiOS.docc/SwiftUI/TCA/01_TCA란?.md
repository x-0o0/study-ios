# Part 1. TCA 란?


> Reference: [The Composable Architecture — Visualize Data Flows With a Diagram](https://medium.com/swlh/the-composable-architecture-visualize-data-flows-with-a-diagram-817306831508)

## 버전

Combine 기반이기 때문에 iOS 13이상 지원

## 컨셉

### Composition

- 큰 기능 -> 작은 부분 => 격리된 모듈로 분리
- 모듈들을 다시 합쳐서 결과적으로 큰 기능을 작은 컴포넌트 집합으로 구성하는 방법 제공

### Testing

End-to-End 테스트 가능

### Ergonomics

심플한 API

## 동작원리

### 액션

뷰 -> User interactions 에 의핸 액션 발생 -> 리듀서로 전달

### 리듀서

Action -> Reducer -> State

- 액션에 따른 상태를 반환하는 **함수*
- 간단한 로직처리의 경우 Reducer에서 즉각 State 반환이 가능
- Environment 를 통해서 State 를 반환하기도 함

### Environment

만약 API 통신으로 뷰에 그릴 데이터를 가져오는 경우,</br>
즉, "앱 외부 데이터" 가 필요한 경우</br>
-> 앱은 해당 API 에 dependency를 가짐.
-> Enviroment는 이 depedency를 갖는다.

API 로 부터 가져온 응답이 원하는 응답일 수도 아닐 수도(Side Effect).
이를 액션으로 간주하고 다시 리듀서로 들어감.

### Store

State + Reducer + Environment 를 모두 감싸고 있음.
=> 개발자가 정의한 앱의 모든 기능이 작동하는 공간

### State
The *data* that:
- needs to perform the logic
- renders UI

```swift
extension Feature {
    struct State: Equatable {
        var count = 0
        var numberFactAlert: String?
    }
}
```

### Action
The **action** that can happen
- e.g., user actions, notifications, event sources, etc.

```swift
import ComposableArchitecture

extension Feature {
    enum Action: Equatable {
        case factAlertDismissed
        case decrementButtonTapped
        case incrementButtonTapped
        case numberFactButtonTapped
        case numberFactResponse(TaskResult<String>)
    }
}
```

### Reducer
A **function** that returns **Effect**s
- current state --(how?)--> next state

```swift
struct Feature: ReducerProtocol {
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        switch action {
        case .factAlertDismissed:
            state.numberFactAlert = nil
            return .none
        case .numberFactButtonTapped:
            // action -> Reducer -> API -> Effect -> action 
            return .task { [count = state.count] in 
                await .numberFactResponse(
                    TaskResult { 
                        try await self.numberFact(count)
                    }
                )
            }
        case let .numberFactResponse(.success(fact)):
            state.numberFactAlert = fact
            return .none
            
        case .numberFactResponse(.failure):
            state.numberFactAlert = "Could not load a number fact :("
            return .none
        }
    }
}
```

### Effect
Returned by *reducer*
- e.g., API requests

```swift
case .numberFactButtonTapped:
    // Effect -> action
    return .task { [count = state.count] in 
        await .numberFactResponse(
            TaskResult { 
                // API request
            }
        )
    }
```

### Store
The **runtime** that actually drives features

```swift
struct FeatureView: View {
    // State 초기값과 Reducer 를 필요로 하는 Store
    let store: StoreOf<Feature>
    
    var body: some View {
        WithViewStore(self.store, observe: { $0 }) { viewStore in
            Button("-") { 
                viewStore.send(.decrementButtonTapped)
            }
        }
        .alert(item: viewStore.binding(
            get: { $0.numberFactAlert.map(FactAlert.init(title:)) },  // state
            send: .factAlertDismissed // action
            )
        ) { factAlert in
            Alert(title: Text(factAlert.title))
        }
    }

```

```swift
import ComposableArchitecture

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            FeatureView(
                store: Store(
                    initialState: Feature.State(), 
                    reducer: Feature()
                )
            )
        }
    }
}
```


