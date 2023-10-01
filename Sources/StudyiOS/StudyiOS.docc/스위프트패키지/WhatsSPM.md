
## 모듈이 뭔데? (What's a module?)

```swift
import Alamofire
```

import 뒤에 있는게 모듈.

<img width="422" alt="Screenshot 2023-09-10 at 4 15 58 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/109e0a79-5a9a-4e97-8ca7-5de9cb8a43c8">

**프로그램**은 하나 이상의 **모듈**로 구성
**모듈**은 하나이상의 `.swift` 파일로 구성


## Swift Package

### 기본 구성
<img width="427" alt="Screenshot 2023-09-10 at 4 15 19 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/1a3c3d68-fb1c-4d3c-a93d-d25efafb923b">

스위프트 패키지는 소스와 매니페스트로 구성
- 매니페스트: Package.swift
  - PackageDescription 의존성을 사용하여 패키지의 상세정보를 명시
- 소스: Sources 폴더에 위치

### 타겟

<img width="368" alt="Screenshot 2023-09-10 at 4 17 11 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/7fb1162e-565b-47ab-b1f0-29dc4571639e">

- 타겟: 패키지에는 여러 타겟을 추가할 수 있음.
  - 타겟은 프로덕트를 명시하고, 의존성을 하나 이상 정의할 수 있음
 
- 타겟은 라이브러리를 빌드할 수도, 실행파일(executable)을 빌드할 수도 있음.
<img width="215" alt="Screenshot 2023-09-10 at 4 18 40 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/17ee3fde-c9e7-4b78-a530-7de124a9a366">

- 즉 하나의 패키지는 라이브러리와 실행파일을 타겟으로 가집니다.
<img width="215" alt="Screenshot 2023-09-10 at 4 19 26 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/0ea178a2-e1ee-437d-8c48-47fcb7b8b143">

- **라이브러리** 는 다른 스위프트 코드에서 불러올 수 있는 모듈을 갖고 있습니다.
<img width="215" alt="Screenshot 2023-09-10 at 4 20 40 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/768cedb5-5f46-4dc0-9eb3-081b158bbd31">

- **실행파일(an executable)** 은 운영체제에 의해 돌아가는 프로그램입니다. (such as `.exe` in windows)
<img width="215" alt="Screenshot 2023-09-10 at 4 20 50 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/93944322-feb8-40ca-b684-738b7c171ba3">

- 각 타겟에는 의존성을 하나이상 정의할 수 있습니다.
  - 의존성을 정의할 때는 다음의 2가지 정보가 필요합니다.
    - 패키지 소스의 URL
    - 버전 정보

<img width="215" alt="Screenshot 2023-09-10 at 4 22 08 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/ab9d5da5-e4a3-4787-a390-9e5448280b6a">

### 의존성 그래프

- SPM 의 역할은
  - 프로젝트를 위한 모든 의존성들을 자동으로 빌드하고 다운로드 해서 Coordination cost를 줄이는 것입니다.
  - 이는 다음과 같은 재귀적인 과정을 거칩니다.
  - 하나의 의존성은 스스로를 위한 의존성을 가질 수 있고 하위 의존성들 또한 스스로를 위한 의존성을 가지므로서 의존성 그래프를 형성하게 됩니다.
 
<img width="237" alt="Screenshot 2023-09-10 at 4 24 37 PM" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/43e20a46-639d-49bb-ae32-8cf4d12163c9">

- SPM은 의존성 그래프 전체를 충족시키기 위해 필요한 모든 것들을 다운 받고 빌드합니다.
- 다운로드된 소스들은 `.build/checkouts` 경로에 위치합니다.
- `swift` 빌드 명령이 실행되면 SPM 은 모든 의존성을 다운로드 받고, 전부 컴파일(빌드) 한 뒤에 의존성들을 패키지 모듈과 연결시킵니다.
- 이는 `import` 구문을 통해 각각의 의존성 모듈의 퍼블릭 멤버들에 접근할 수 있게 해줍니다.
