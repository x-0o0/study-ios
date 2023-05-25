# SwiftUI 모델생성

애플은 `ObservableObject` 도 *Model* 폴더에 포함시킨다... (Fruta 샘플앱 참고)

암튼 애플은 `AuthenticationServices` 를 어떻게 쓰는지 알아보자.

```swift
import AuthenticationServices
```

## ObservableObject

```swift
class Model: ObservableObject { ... }
```
아예 모델이라고 네이밍 했다. 뷰모델로 생각을 하지 않는 것인가. 최근 스유를 공부하다보면 스유가 MVVM이 적합해보이지만 실제로 MVVM 은 아니라는 글들을 많이 접하고 있다.

오직 View + State ? View + Model 이 둘만의 구조인 것처럼 설명하는 글들을 보았는데, 대충본거라 이 문장은 유심있게 볼 필요는 없다. 사실 나도 뭔 말인지 모르니까 ㅋ

## @Published Account

```swift
@Published var account: Account?
```

`Account` 타입 변수를 `Published` 속성으로 선언하고 있다. `Account` 내부를 살펴보자.

```swift
struct Account {
  ...
  // Used for calculating free smoothie redemption
}
```
내부에는 특별한건 없었다. 그냥 주문기록, 사용한 포인트, 적립한 포인트 이런 기록들 뿐이었다. 그럼 다시 돌아와서 다음 줄을 살펴보자.

## UserCredential

### hasAccount

```swift
var hasAccount: Bool {
    #if targetEnvironment(simulator)
    return true
    #else
    return userCredential != nil && account != nil
    #endif
}
```
현재 계정 유무를 확인하는 read-only 프로퍼티가 있다.

시뮬이면 일단 `true` 로, 실기기면 `userCredential`를 바라본다. `userCredential`은 뭘까.

### userCredential

```swift
let defaults = UserDefaults(suiteName: "group.example.fruta")

private var userCredential: String? {
    get { defaults?.string(forKey: "UserCredential") }
    set { defaults?.setValue(newValue, forKey: "UserCredential") }
}
```

`defaults` 에 `"UserCredential"` 이라는 키로 읽고 쓰고 있는 값이다. `UserDefaults(suiteName:)` 이라는게 눈에 들어왔다.

### UserDefaults.init(suiteName:)

> **`init(suiteName:)`**
> 특정 데이터 베이스 이름을 위한 디폴트와 함께 유저 디폴트 객체를 생성한다.</br>
> 파라미터에 `nil`을 넣으면 `standard` 를 사용한다. 앱의 메인 번들 ID 나 글로벌 도메인을 쓰지 않도록 주의한다.(앱이 값을 기록할 수 없음) </br>
> 1) 환경설정이나 앱간의 데이터 공유를 위한 앱 공간을 개발할 때 이 메소드를 사용한다. </br>
> 2) extension 과 관련 앱 사이의 환경설정 및 데이터를 공유하기 위해 앱 extension을 개발할때도 사용한다. </br>
> 링크  👉 https://developer.apple.com/documentation/foundation/userdefaults/1409957-init

앱과 extension 간의 UserDefaults 공유 예시

```swift
extension UserDefaults {
    // 방법1: 새로운 UserDefaults
    static var shared: UserDefaults {
        UserDefaults(suiteName: "group.example.service")!
    }
    
    // 방법2: 기존 standard 를 새 UserDefaults로 결합시키기 -> 위젯에서 적용 안됨. standard는 앱과 위젯이 서로 다른 저장소이기 때문.
    static var shared: UserDefaults {
        let combined = UserDefaults.standard
        combined.addSuite(named: "group.example.service")
        return combined
    }
    
    // 방법3-1: 기존 앱 standard를 위젯까지 접근할 수 있도록 새 UserDefaults 업데이트 하기.
    static var shared: UserDefaults {
        UserDefaults(suiteName: "group.example.service")!
    }
}

// 방법3-1: 기존 앱 standard를 위젯까지 접근할 수 있도록 새 UserDefaults 업데이트 하기.
UserDefaults.standard.dictionaryRepresentation().forEach {
    UserDefaults.shared.set($0, forKey: $1)
}
```

위에서 애플이 `userCredential` 를 읽고 쓸 때, `suiteName` 을 쓰는 이유는 </br>
multi platform + App clipe + widget 형태의 앱프로젝트이기 때문에 그들간의 저장소 공유를 하기 위해서라는 것을 알 수 있다.

## init

자 그러면 다시 `Model` 코드를 살펴보자.

```swift
init() {
    // userCredential 존재여부확인
    guard let user = userCredential else { return }
    
    // 애플 ID 제공자 객체 생성
    let provider = ASAuthorizationAppleIDProvider()
    
    // 제공자 통해서 userCredential 에 맞는 credential 상태 가져오기
    provider.getCredentialState(forUserID: user) { state, error in
        // 인증 되었거나 전송되었으면 계정 생성
        if state == .authorized || state == .transferred {
            DispatchQueue.main.async {
                self.createAccount()
            }
        }
    }
    }
}
```

### userCredential

`userCredential` 은 유저 아이디 임을 알 수 있다.

### ASAuthorizationAppleIDProvider

> **`ASAuthorizationAppleIDProvider`**
> 애플 아이디 기반으로 사용자를 인증하기 위한 요청을 생성하는 메카니즘.

아래 예시 코드 처럼 `ASAuthorizationAppleIDRequest` 타입의 request 객체를 생성할 때 이 제공자를 사용합니다.

```swift
let provider = ASAuthorizationAppleIDProvider()
let request = provider.createRequest()
let controller = ASAuthorizationController(authorizationRequests: [request])
```

성공한 경우, 컨트롤러 delegate는 `ASAuthorization` 객체를 전달 받고, 이 안에는 `ASAuthorizationAppleIDCredential` 타입의 credential 이 들어가 있습니다.</br>
이 credential은 불투명한 user ID를 갖고 있습니다 (불투명이란 여기서 변환을 거친 애플 ID 값을 의미한다고 보면 된다).</br>
이 ID는 추후에 사용자의 credential 상태를 확인하기 위한 용도로 쓸 수 있습니다.</br>

```swift
let user = authorization.credential.user
provider.getCredentialState(forUserID: user) { state, error in
    // Check for error and examine the state.
}
```

즉, `provider.getCredentialState(forUserID:)` 를 쓰기전 한번 인증 요청에 성공하여 `credential`를 갖고 있어야한다.

그럼 인증은 어디서 하고 있을까...? 일단은 다음으로 넘어가자.

## `authorizeUser(_:)`

```swift
func authorizeUser(_ result: Result<ASAuthorization, Error>) {
    guard case .success(let authorization) = result, let credential = authorization.credential as? ASAuthorizationAppleIDCredential else {
        if case .failure(let error) = result {
            print("Authentication error: \(error.localizedDescription)")
        }
        return
    }
    DispatchQueue.main.async {
        self.userCredential = credential.user
        self.createAccount()
    }
}
```

인증결과가 성공이고 `ASAuthorization` 객체에서 애플 ID credential를 얻을 수 있는 경우</br>
`userCredential` 를 저장하고 `createAccount()` 함수를 호출한다.</br>

실패거나, credential을 얻지 못한 경우 즉각 리턴한다.
    
## createAccount

```swift
func createAccount() {
    guard account == nil else { return }
    account = Account()
}
```

`createAccount()` 에 특별한건 없다. `account` 값이 `nil` 인 경우 `Account` 객체를 생성해서 할당해주는 작업만 한다.
