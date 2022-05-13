Fruta 앱 하단에 아래와 같은 뷰가 들어간다.

```swift
import AuthenticationServices

struct OrderPlacedView: View {
    @EnvironmentObject private var model: Model
    
    var presentingBottomBanner: Bool { !model.hasAccount }
    
    var body: some View {
        if presentingBottomBanner {
            bottomBanner
        }
    }
}
```

## SignInWithAppleButton

```swift
var bottomBanner: some View {
    VStack {
        if !model.hasAccount { // 1. 계정이 없으면
            SignInWithAppleButton(.signUp, onRequest: { _ in }, onCompletion: model.authorizeUser)
                .frame(minWidth: 100, maxWidth: 400)
                .padding(.horizontal, 20)
                .frame(height: 45)
        } else {  // 2. 계정이 있으면
            ...
        }
    }
}
```
(`body` 의 외에 `some View` 프로퍼티를 생성하지말고 새로운 뷰 struct를 만들 것을 권장한다는 글을 애플 문서에서 본 것 같은데...)

`SignInWithAppleButton` 을 사용하는 부분을 보면 `.signUp` 을 사용한다. `.signIn` 이 아닌 `.signUp` 을 쓰면 뭐가 다를까. 
찾아봤는데 그냥 버튼 문구 차이 밖에 없다...

끝.
