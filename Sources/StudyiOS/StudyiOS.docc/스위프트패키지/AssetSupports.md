# 애셋 제공하기

애셋을 제공할 패키지 이름은 `MyPackage` 라고 하겠습니다.

```
Sources
 L MyPackage
    L Assets.xcassets
    L ColorSet.swift
```

`MyPackage` 는 위와 같은 경로를 갖고 `ColorSet.swift` 파일에는 다음과 같이 `Assets` 의 색상을 사용하는 퍼블릭 인터페이스가 있습니다.</br>
`Assets` 의 색상에 접근하기 위해서 `Bundle.main` 에 접근해야합니다.

```swift
public struct ColorSet {
    public static let green = UIColor(
        named: "ColorSet.green",
        in: Bundle.main, // 패키지 안에서는 Bundle.main 에 접근하여 애셋을 사용할 수 있었을 겁니다.
        compatibleWith: nil
    ) ?? UIColor.green
}
```

## 문제점

이 상태에서는 분명 패키지 내에서는 애셋 접근에 이상이 없었으며 Test 코드를 돌려도 문제는 없었습니다.</br>
하지만 배포하고 나서 SwiftUI 를 쓰고 있는 다른 스위프트 패키지에서 위 패키지를 디펜던시에 추가하고 나면 SwiftUI 프리뷰에서 다음과 같은 에러가 발생했습니다.

`Fatal error: unable to find bundle named MyPackage_MyPackage: file MyPackage/resource_bundle_accessor.swift, line 27`

이를 해결하기 위해 여러 시도를 했고 그 결과 해결책은 다음과 같았습니다.

> **업데이트** `resources` 에 추가하면 자동으로 스위프트 패키지에서 `Bundle.module` 이라는 internal static property 를 제공해줌.

## 시작!

1. `target` 수정

```swift
// Package.swift

.target(
    name: "MyPackage",
    path: "Sources/MyPackage",  // (옵셔널) 타겟 경로를 명시합니다.
    resources: [
        .process("Assets.xcassets") // 애셋카탈로그를 resources에 추가합니다.
    ]
),
```

2. 새 Bundle 추가하기

애셋카탈로그와 같은 레이어에 `Bundle.MyPackage.swift` 라는 extension 파일을 추가하고 다음 소스코드를 복사 붙여넣기 합니다.

이때 `myModule` 과 `MyPackage_MyPackage` 를 적절하게 바꿔줘야 합니다.

```swift
import Foundation

private class CurrentBundleFinder { }

extension Foundation.Bundle {
    // 패키지용 번들 이름을 설정합니다.
    static var myModule: Bundle = {  
        let bundleName = "MyPackage_MyPackage"
        let candidates = [
            Bundle.main.resourceURL,
            Bundle(for: CurrentBundleFinder.self).resourceURL,
            Bundle.main.bundleURL,
            Bundle(for: CurrentBundleFinder.self).resourceURL?.deletingLastPathComponent().deletingLastPathComponent(),
        ]
        for candidate in candidates {
            let bundlePath = candidate?.appendingPathComponent(bundleName + ".bundle")
            if let bundle = bundlePath.flatMap(Bundle.init(url:)) {
                return bundle
            }
        }
        fatalError("unable to find bundle named \(bundleName)")
    }()
}
```

3. Bundle.main -> Bundle.myModule 로 수정하기

```swift
public struct ColorSet {
    public static let green = UIColor(
        named: "ColorSet.green",
        in: Bundle.myModule, // 👈
        compatibleWith: nil
    ) ?? UIColor.green
}
```

4. 배포하기
