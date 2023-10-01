# 타겟별 리소스 관리 (ttf & .plist)

## 커스텀폰트 (feat. 토스페이스)

<img width="1840" alt="쏘큩" src="https://github.com/ku-ring/ios-app/assets/53814741/8ce44fd7-b40d-478d-809f-a60915a9c478">

쿠링 v2 는 토스 페이스를 사용하기 때문에 커스텀 ttf 파일을 사용해야했다. 재사용 가능한 UI 컴포넌트들도 모듈화 해야했기 때문에 패키지 내 필요한 타겟들에서 토스페이스 ttf 파일을 사용할 수 있어야 한다.

```swift
.target(
    name: "SubscriptionFeature",
    dependencies: [...],
    path: "Sources/Feature/Subscriptions",
    resources: [.process("Resources")]
)
```
resources 의 "Resources" 는 지정한 path 를 기반으로 하는 하위폴더 "Resources" 이다.

### 해결과제

위 방식은 필요한 타겟마다 Resources 폴더를 생성해야하므로 리소스의 중복이 발생할 수 있다.

## .plist

`.plist` 파일에 쿠링 서버의 API 주소나 스킴관련 정보, 그 외에 기본적인 configuration 정보들을 담고자 한다.

xcconfig 는 xcodeproj를 위한(즉, xcodeproj에 제한된) 파일이기 때문에 스위프트 패키지에서 사용할 수 없다. swift-package-manager 레포와 BuildSettingCondition 같은 내용을 쭉 흝어봤지만 xcconfig 를 스위프트 패키지에서 쓸 수 있는 좋은 방안을 찾지 못했다.

그래서 `plist` 를 사용하는 것으로 대체했다.

<img width="639" alt="Screenshot 2023-09-16 at 1 34 12 AM" src="https://github.com/ku-ring/ios-app/assets/53814741/c88d9bd3-0c21-453b-960c-2ce6d4d666e8">

```swift
.target(
    name: "KuringLink",
    dependencies: [...],
    path: "Sources/Network/KuringLink",
    resources: [.process("Resources/KuringLink-Info.plist")]
)
```

다만,
앱프로젝트에서 plist 는 Bundle.main.infoDictionary 로 딕셔너리 값에 접근할 수 있는 반면에,
스위프트 패키지에서 사용할 때는 Bundle.module 에서도 동일하게 접근하려고 했다면.. NOPE...

```swift
let plistURL = Bundle.module.url(forResource: "KuringLink-Info", withExtension: "plist")!
let dict = try! NSDictionary(contentsOf: plistURL, error: ())
let kuringHost = dict["API_HOST"] as? String
```

이렇게 plist URL 을 가져온 다음 `NSDictionary(contentsOf:)` 를 활용하면 된다. `Dictionary` 에서도 처리가 가능한지는 확인 안해봄.

