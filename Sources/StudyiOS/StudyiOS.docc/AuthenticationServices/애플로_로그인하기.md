# 애플로 로그인하기

## develop.apple.com

Certificates, Identifier & Profiles 메뉴로 이동

### Identifier
1. **Idenitifiers** 에서 애플로로그인 기능을 추가하고자 하는 Idenitifier 선택
2. `Sign in with Apple` 옵션 켜기

### Key
1. **Keys** 에서 관련 키 `Edit` 혹은 새로운 키 생성에서 `Sign in with Apple` 옵션 켜기
2. `Configure` 버튼 눌러서 **Configure Key** 창으로 이동
3. **Primary App ID** 에 아까 수정한 **Identifier** 선택
4. 다음으로 넘어가기
<img width="759" alt="Screen Shot 2022-05-08 at 2 26 21 AM" src="https://user-images.githubusercontent.com/53814741/167265176-77d7fae8-01da-480f-b5f3-d685acf6036c.png">
5. 일단 저장 및 다운로드

## Xcode

```swift
import AuthenticationServices
import SwiftUI

struct AppleUser: Codable {
  let userId: String
  let firstName: String
  let lastName: String
  let email: String
  
  init?(credentials: ASAuthorizationAppleIDCredential) {
    guard let firstName = credentials.fullName?.givenName,
          let lastName = credentials.fullName?.familyName,
          let email = credentials.email
    else { return nil }
    
    self.userId = credentials.user
    self.firstName = firstName
    self.lastName = lastName
    self.email = email
  }
}

struct ContentView: View {
  @Environment(\.colorScheme) var colorScheme
  
  var body: some View {
    SignInWithAppleButton(
      .signIn,
      onRequest: configure,
      onCompletion: handle
    ) // 풀스크린 버튼으로 뜰거라서 사이즈 조절 필요
    .signInWithAppleButtonStyle(
      colorScheme = .dark ? .white : .black
    )
    .frame(height: 45)
    .padding()
  }
  
  func configure(_ request: ASAuthorizationAppleIDRequest) {
    request.requestedScopes = [.fullName, .email]
    request.nonce = ""
  }
  
  func handle(_ authResult: Result<ASAuthorization, Error>) {
    switch authResult {
    case .success(let auth):
      switch auth.credential {
      case let appleIdCredentials as ASAuthorizationAppleIDCredential:
        // appleIdCredentials.email
        // appleIdCredentials.user
        if let appleUser = AppleUser(credentials: appleIdCredentials),
           let appleUserData = try? JSONEncoder().encode(appleUser) {
          UserDefaults.standard.setValue(appleUserData, forKey: YOUR_KEY)
          print("성공적으로 애플유저를 저장했습니다.")
        } else {
          // email, full name, user 등의 몇가지 필드가 없을 경우 (보통 재로그인 하는 경우 user만 존재함)
          guard let appleUserData = UserDefaults.standard.data(forKey: appleIdCredentials.user),
                let appleUser = try? JSONDecoder().decode(AppleUser.self, from: appleUserData)
          else { return }
        }
      default:
      }
    case .failure(let error):
    }
  }
}
```


