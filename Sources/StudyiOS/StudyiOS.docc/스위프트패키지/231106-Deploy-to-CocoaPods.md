# 스위프트 패키지를 코코아팟에 배포하기

스위프트 패키지를 코코아팟(CocoaPods) 의존성 관리도구(Dependency Manager) 에 배포하는 과정

## 1. 배경 및 목적

Xcode 에서 개발한 Framework 를 코코아팟에 배포하고 XCFramework 를 추출해서 스위프트 패키지 매니저에도 배포는 수차례 해보았지만
스위프트 패키지 매니저에 배포하던 스위프트 패키지를 코코아팟에 배포하는 것은 경험이 없었다.

CocoaPods의 매니페스트(Manifest) 설정을 담당하는 Podspec 를 가볍게 수정하는 것으로 스위프트 패키지를 코코아팟에 배포할 수 있다.

이 문서에서는 스위프트 패키지에 `.podspec` 파일을 생성하고 코코아팟 트렁크(Trunk) 에 푸시하여 배포하는 과정을 다뤄보겠습니다.

## 2. 진행 방법

### 2.1. Podspec 생성 및 수정

> **중요**
>
> `pod` 커맨드라인 도구 설치가 사전에 필요합니다.

```bash
pod spec create 패키지이름
```

`spec create` 는 `.podspec` 파일을 생성하는 명령어 입니다.sup>[1](#footnote_1)</sup>

그럼 폴더 구조가 다음과 같습니다.

```
MyPackage
|--- Package.swift
|--- 패키지이름.podspec
|--- Sources
     |--- MyPackage
```

`.podspec` 을 열고 아래와 같이 수정 합니다. 여기서 가장 중요한 것은 `spec.source_files` 입니다. Sources 하위 폴더 중 팟으로 제공하고자 하는 소스의 폴더를 명시해줘야 합니다.

```podspec
Pod::Spec.new do |spec|
  spec.name         = "라이브러리이름"
  spec.version      = "1.0.0"
  spec.summary      = "한 줄 설명"

  spec.description  = <<-DESC
    라이브러리에 대한 자세한 소개 문구가 들어갑니다.
                   DESC

  spec.license      = { :type => "MIT", :file => "LICENSE" }

  spec.author             = { "Jaesung Lee" => "이메일주소" }

  spec.ios.deployment_target = "16.0"
  spec.swift_version = "5.9"

  spec.source       = {
    :git => "https://github.com/jaesung-0o0/package-library.git",
    :tag => "#{spec.version}"
  }

  spec.source_files  = "Sources/폴더이름/**/*"

end
```

| 항목 | 설명 | 예시 |
| --- | --- | --- |
| `name` | | |
| `version` | | |
| `summary` | | |
| `description` | | |
| `license` | | |
| `author` | | |
| `ios.deployment_target` | | |
| `swift_version` | | |
| `source` | | |
| `source_files` | | |

## 4. 참고 문헌

<a name="footnote_1">1</a>: [https://guides.cocoapods.org/making/specs-and-specs-repo](https://guides.cocoapods.org/making/specs-and-specs-repo.html)

<a name="footnote_2">2</a>: `**` 는 모든 하위폴더를 의미, `*` 모든 파일을 의미
