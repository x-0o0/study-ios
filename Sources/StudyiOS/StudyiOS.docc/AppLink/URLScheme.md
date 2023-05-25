# URLScheme
> **⚠️주의**
> 
> URL 스킴은 앱 공격으로 악용될 수 있음을 유의해야합니다. 따라서 모든 URL 파라미터들이 *유효한지 확인*하고 옳지 못한 URL 들은 *걸러내어야* 합니다. 
> 사용자 데이터를 위협하지 않도록 실행 가능한 동작들을 *제한*해야합니다.
> 예를 들어, 다른앱이 컨텐츠를 제거하거나 유저에 대한 민감한 정보에 접근하지 않도록 해야합니다.
> URL을 다루는 코드를 테스트 할 때는, 잘못된 형식의 URL들에 대한 테스트 케이스도 반드시 추가할 것을 권장합니다.

커스텀 URL 스킴을 지원하기 위해서 아래의 사항들을 준수해야합니다.

1. 앱의 URL 들을 위한 포맷을 정의합니다.
2. 스킴을 등록하여 시스템이 앱으로 적절한 URL을 전달할 수 있도록 합니다.
3. 앱이 전달받은 URL들을 처리할 수 있도록 합니다.

URL 들은 반드시 커스텀 스킴 이름으로 시작해야 합니다. 
앱이 지원하는 옵션에 대한 파라미터들을 추가합니다. 예를 들어, 사진첩 앱이라면 URL 포맷을 정의할 때 보여주고자 하는 사진 앨범의 이름이나 인덱스를 포함시킬 것입니다. 대충 이런 모습이 되겠죠?

```
myphotoapp:albumname?name="albumname"
myphotoapp:albumname?index=1
```

클라이언트 단에서는 `UIApplication` 의 `open(_:options:completionHandler:)` 함수를 호출하여 스킴기반 URL에 대한 처리를 할 수 있습니다.
클라이언트는 앱이 URL을 열 때 알려달라고 시스템에 요청할 수 있습니다.

```swift
let url = URL(string: "myphotoapp:Vacation?index=1")

UIApplication.shared.open(url!) { (result) in
    if result {
       // URL 이 성공적으로 전달되었습니다.
    }
}
```

## URL 스킴 등록하기

Xcode > 프로젝트 세팅 > Info 탭 에서 스킴을 등록할 수 있습니다.

다음 지시사항에 따라 앱이 지원하는 모든 URL 스킴들을 선언할 수 있게 URL 타입 섹션을 업뎃해주세요.

- URL 스킴 영역에 URL에 사용될 prefix를 적어줍니다.
- 앱의 역할을 고릅니다. 정의한 URL 스킴에 대한 편집자 역할인지, 아니면 앱이 채택했지만 정의하지 않은 스킴에 대한 뷰어 역할인지 (뭔말인지 모르겠음)
- 앱의 ID를 적어줍니다.

<img width="679" alt="Screen Shot 2023-01-04 at 12 18 12 AM" src="https://user-images.githubusercontent.com/53814741/210386335-c7ed7486-903c-4e0f-bcab-3ae73a4e264c.png">

앱 ID 는 같은 스킴 지원을 정의한 다른 앱들로 부터 본인의 앱을 구별하는데에 쓰입니다.
유니크함을 확실히 하고 싶다면 회사 도메인과 앱네임이 통합된 도메인 주소를 반대로 적어주세요. (예: com.companyname.appname)
도메인 주소를 반대로 적는 것이 가장 좋은 방법이긴 하나 다른 앱들이 동일한 스킴을 등록하고 관련 링크를 다루는 것을 막을 순 없습니다. 
당신의 웹사이트와 유일하게 연관된 링크들을 정의하고 싶다면 커스텀 URL스킴말고 유니버셜 링크를 사용해주세요.

> **메모**
>
> 만약 여러 앱이 같은 스킴을 등록하고 있다면, 시스템은 어떤 앱을 타켓으로 할지 정의하지 않습니다. 
> 타켓 앱을 변경하거나 공유시트에 뜨는 앱들의 순서를 변경하거나 하는 매커니즘은 없습니다.

이미 시스템에 예약된 스킴도 있으니 주의하세요. 이미 등록되어 있는 스킴은 [여기에서](https://developer.apple.com/library/archive/featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007899) 확인할 수 있습니다.

## 수신한 URL 처리하기

다른 앱에서 당신의 커스텀 스킴을 포함한 URL을 열게되면, 시스템은 필요한경우 당신의 앱을 실행하여 포어그라운드로 가져옵니다 (즉, 화면에 앱을 띄움).
시스템은 당신의 앱의 app delegate 에 정의된 `application(_:open:options:)` 함수를 호출하여 URL 를 당신의 앱으로 전달합니다.
URL에 담긴 내용을 파싱하고 적절한 액션을 취할 수 있도록 메소드를 정의해줘야 합니다.
URL이 올바르게 파싱되었는지 확인하기 위해 `NSURLComponents` API 를 사용하여 컴포넌트들의 압축을 풀어주세요.
어떤 앱을 열지와 같은 URL에 대한 추가적인 정보를 시스템이 제공하는 옵션 딕셔너리로부터 가져오도록 합니다.

```swift
func application(_ application: UIApplication,
                 open url: URL,
                 options: [UIApplicationOpenURLOptionsKey : Any] = [:] ) -> Bool {

    // 누가 URL을 보냈는지 알아내기
    let sendingAppID = options[.sourceApplication]
    print("source application = \(sendingAppID ?? "Unknown")")

    // URL 처리.
    guard let components = NSURLComponents(url: url, resolvingAgainstBaseURL: true),
        let albumPath = components.path,
        let params = components.queryItems else {
            print("Invalid URL or album path missing")
            return false
    }

    if let photoIndex = params.first(where: { $0.name == "index" })?.value {
        print("albumPath = \(albumPath)")
        print("photoIndex = \(photoIndex)")
        return true
    } else {
        print("Photo index missing")
        return false
    }
}
```

또한 시스템은 앱이 지원하는 커스텀 파일 타입을 열때도 `application(_:open:options:)` 메소드를 사용합니다.

만약에 앱이 AppDelegate 방식 말고 Scene 방식을 채택하고 있다면 아래 두 방식으로 URL을 전달합니다
- 앱이 돌아가는 중이 아니라면, 앱을 시작시키고 난 후 `scene(_:willConnectTo:options:)` 델리게이트 메소드로 URL를 전달.
- 앱이 실행 중이거나 메모리에 있으면, `scene(_:openURLContexts:)` 델리게이트 메소드로 URL를 전달.

```swift
func scene(_ scene: UIScene, 
           willConnectTo session: UISceneSession, 
           options connectionOptions: UIScene.ConnectionOptions) {

    // 누가 URL을 보냈는지 알아내기
    if let urlContext = connectionOptions.urlContexts.first {

        let sendingAppID = urlContext.options.sourceApplication
        let url = urlContext.url
        print("source application = \(sendingAppID ?? "Unknown")")
        print("url = \(url)")

        // 위의 App delegate 에서의 예시와 유사한 방식으로 URL을 처리합니다.
    }

    /*
     *
     */
}
```


## 참고문서

> **애플 문서** 
> [앱을 위한 커스텀 URL스킴 정의하기](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app)

> **블로그 - 김종권의 iOS 앱개발 알아가기**
> [딥링크 (URL Scheme, Universal Link](https://ios-development.tistory.com/207)
