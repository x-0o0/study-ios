# 어댑터 패턴

호환되지 않는 타입끼리 같이 동작활 수 있도록 해주는 패턴.

<img width="549" alt="Screen Shot 2022-05-19 at 8 18 30 PM" src="https://user-images.githubusercontent.com/53814741/169281409-878293fa-d355-4a9c-be78-6047165fd332.png">

어댑터는 4가지 구성요소를 갖는다
- 어댑터를 사용하는 객체: 뉴 프로토콜에 의존한다.
- 뉴 프로토콜: 사용하려는 프로토콜
- 레거시 객체: 뉴 프로토콜이 생성되기 전까지는 존재할 수 없고 뉴프로토콜을 준수하기 위해 직접 수정할 수 없는 객체
- 어댑터: 뉴 프로토콜을 준수하기 위해 생성됨. 레거시 객체로 호출을 전달한다.

### 재밌고 이해 잘 되는 예시

최신 아이폰은 해드폰잭이 없지 않습니까~? 그래서 헤드폰잭을 라이트닝 커넥터로 변환해주는 어댑터가 필요합니다.

즉 서로 다른 두 요소를 연결해주는 것. 그게 바로 어댑터 입니다.

## 레거시

```swift
class GoogleAuthenticator {
    func login(email: String, password: String, completion: @escaping (GoogleUser?, Error?) -> Void) {
        let token = "some-token"
        let user = GoogleUser(
            email: email,
            password: password,
            token: token
        )
        completion(user, nil)
    }
}
```

`GoogleAuthenticator` 는 수정될 수 없는 서드파티 클래스다. 즉, 레거시 객체다.

```swift
struct GoogleUser {
    var email: String
    var password: String
    var token: String
}
```

## 뉴프로토콜

```swift
protocol AuthenticationService {
    func login(
        email: String, 
        password: String, 
        success: @escaping (User, Token) -> Void,
        failure: @escaping (Error?) -> Void
    )
}
```

뉴 프로토콜이다. 이메일, 암호를 필요로 한다. `GoogleAuthenticator` 대신 이 프로토콜을 직접 사용한다.

```swift
struct User {
    let email: String
    let password: String
}

struct Token {
    let value: String
}
```

## 어댑터

```swift
class GoogleAuthenticatorAdapter: AuthenticationService {
    var authenticator = GoogleAuthenticator()
    
    func login(
        email: String,
        password: String,
        success: @escaping (User, Token) -> Void,
        failure: @escaping (Error?) -> Void
    ) {
        authenticator.login(email: email, password: password) { googleUser, error in 
            guard let googleUser = googleUser else {
                failure(error)
                return
            }
    // 5
          let user = User(email: googleUser.email,
                          password: googleUser.password)
          let token = Token(value: googleUser.token)
          success(user, token)
     }
}
```

