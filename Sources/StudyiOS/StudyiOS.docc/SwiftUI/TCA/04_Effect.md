# Part 4. Effect

## 왜 공부하는가?
어제 Store 를 공부하면서 Reducer를 공부했고, Reducer를 공부하면서 Effect를 잘 써야 `ReducerProtocol` 의 메소드인 `reduce(into:inout:)` 을 올바르게 잘 구현할 수 있다는 생각을 했다.
그래서 이번엔 Effect를 공부하고자 한다.

## 복습

`Action` -> `Reducer` -> `State` (when, the effect is `.none`)

기본적으로 Reducer는 Action 에 따라 적절하게 State를 변형하는 BlackBox 역할을 한다. 이렇게 State를 직접 변경하고 끝나버리는 경우 Effect는 `.none` 이다.

만약에 전달받은 Action이 API 요청을 보내고 응답에 따라 State를 처리해야하는 거라면 위의 흐름은 아래와 같이 변한다.

`Action` -> `Reducer` -> `Effect` -> `API request` - - -> `API response` -> `Action` -> `Reducer` -> `State` (when, the first effect has API task, the second effect is `.none`)

위의 흐름에 따르면 `Reducer` 는 외부와 통신 하는 작업을 수행하는 `Effect` 를 반환하고, 작업 결과에 따라 적절한 `Action`를 다시 `Reducer` 로 전달한다.

이 흐름에 쓰이는 `Effect`들을 본격적으로 알아보자.


## TCA 공식 문서 - 튜토리얼 정독하기

TCA 의 Essentials 을 모아둔 튜토리얼 부터 정독했다.

[튜토리얼 | 사이드이펙트](https://pointfreeco.github.io/swift-composable-architecture/main/tutorials/composablearchitecture/02-addingsideeffects) 를 보면 다음과 같은 Effect들을 소개한다.

## Network 요청

버튼을 탭했을 때 API 통신을 하는 액션이 있고 아래와 같이 코드를 짰다고 해보자.

```swift
case .factButtonTapped:
      state.fact = nil
      state.isLoading = true

      let (data, _) = try await URLSession.shared
        .data(from: URL(string: "http://numbersapi.com/\(state.count)")!)
      // 🛑 'async' call in a function that does not support concurrency
      // 🛑 Errors thrown from here are not handled

      state.fact = String(decoding: data, as: UTF8.self)
      state.isLoading = false

      return .none
```
코드만 봐서는 특별히 문제가 없어보이지만, `reduce()` 는 `async` 함수가 아니기 때문에 `try await` 같은 concurrency를 지원하지 않는다. 따라서 컴파일에러가 발생한다.

이 경우 `EffectTask` 를 생성하여 처리해야한다. `run(priority:operation:catch:fileID:line:)` 가 가장 우선적인 생성 방법이다.

```swift
case .factButtonTapped:
      state.fact = nil
      state.isLoading = true
      return .run { [count = state.count] send in
        // ✅ Do async work in here, and send actions back into the system.
        let (data, _) = try await URLSession.shared
          .data(from: URL(string: "http://numbersapi.com/\(count)")!)
        let fact = String(decoding: data, as: UTF8.self)
        await send(.factResponse(fact))
      }
```
원하는 응답 값을 받았으면 이를 `TaskResult` 를 파라미터로 하는 Action 안에 할당하고 `await send(_:)` 를 호출한다.

## 타이머

이번엔 스케쥴 타이머같은 것을 사용할 때의 Effect 를 알아보자.

`Task.sleep(for: .seconds(1))` 를 사용하여 1초마다 state를 변경하고 싶은 경우 아래와 같이 코드를 작성한다.

```swift
case .toggleTimerButtonTapped:
      state.isTimerRunning.toggle()
      return .run { send in
        while true {
          try await Task.sleep(for: .seconds(1))
        }
      }
```
그리고

```swift
case .timerTick:
      state.count += 1
      state.fact = nil
      return .none
```

타이머를 취소하고 싶다면

```swift
enum CancelID { case timer }

// ....

case .toggleTimerButtonTapped:
      state.isTimerRunning.toggle()
      if state.isTimerRunning {
        return .run { send in
          while true {
            try await Task.sleep(for: .seconds(1))
            await send(.timerTick)
          }
        }
        .cancellable(id: CancelID.timer)
      } else {
        return .cancel(id: CancelID.timer)
      }
```
