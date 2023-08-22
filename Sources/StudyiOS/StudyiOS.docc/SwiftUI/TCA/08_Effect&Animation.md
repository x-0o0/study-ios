# Part 8. Effect & Animation

```swift
struct AuthView: View {
    let store: StoreOf<AuthReducer>
    
    var body: some View {
        WithViewStore(self.store, observe: { $0 }) { viewStore in
            // TODO: 탭 간의 이동에 애니메이션 how?
            TabView(
                selection: viewStore.binding(
                    get: \.currentStep,
                    send: { .moveStep($0) }
                )
            ) {
                AuthHomeView(store: store.scope(...))
                    .tag(AuthReducer.Step.home)
                
                AuthPhoneField(store: store.scope(...))
                    .tag(AuthReducer.Step.phoneNumber)
            }
            .tabViewStyle(.page(indexDisplayMode: .never))
        }
    }
}
```

1. `AuthReducer` 는 `AuthHomeView` 와  `AuthPhoneField`의 reducer를 scope 하고 있고, 본인의 state에 currentStep 이라는 값을 통해 현재 탭의 값을 관리하고 있다.
2. `AuthHomeView` 와 `AuthPhoneField` 는 각각 `go next` 버튼을 가지고 있고, 각 버튼은 `viewStore.send(.tappedNextButton)` 을 호출하여 `currentStep` 값을 변경하고 있다.

## 이슈

당연하지만 `go next` 버튼을 탭하면 애니메이션 없이 탭이 이동된다. 문제는 `viewStore.send(_:)` 하는 코드를 `withAnimation` 으로 감쌀 수 없다. 감싸면 다음과 같은 타입 에러를 마주한다.

> __Error__:
> Conflicting arguments to generic parameter 'Result' ('Void' vs. 'ViewStoreTask')

`withAnimation` 말고 다른 방안을 찾아보던 중 pointfree 개발자가 비슷한 이슈에서 남긴 [답변](https://github.com/pointfreeco/swift-composable-architecture/issues/207#issuecomment-653567993)을 발견했다.

### Pointfree's Answer

"Hi @technicated. The reason you're not seeing an animation is because withAnimation blocks are performed synchronously, but the sorting is happening later. iOS animates sorting by default, but it appears that macOS does not.

If you want the result of an effect to be animated you must animate from state. One way to do so is by adding animation view modifiers to the view hierarchy. In this particular case you can animate the List and then reset this modifier on the list's content so that not all changes in each row animates:"

```swift
List {
    Section(header: Header(store: store)) {
        ...
    }
    .animation(nil) // don't animate every list item
}
.animation(.default) // animate the state of the list, though, including ordering
```

이에 따라 `TabView` 에 `.animation(.default)` 를 적용하였다.

## 결과

원하던대로 잘 돌아간다.
