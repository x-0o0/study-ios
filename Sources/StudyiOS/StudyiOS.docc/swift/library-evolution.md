# 라이브러리 에볼루션 (Library Evolution)

## 개요

Swift 5.1 이상부터 다음 두가지 기능이 제공됩니다.
1. 모듈 안정성 (Module Stability)
   - 서로 다른 버전의 컴파일러에서 빌드된 스위프트 모듈들이 하나의 앱에서 함께 쓰일 수 있게 해주는 기능
2. 라이브러리 에볼루션 지원 (Library Evolution Support, 이하 "에볼루션")
   - 바이너리 프레임워크 개발자들이 바이너리가 이전 버전과 호환가능하게 유지하면서 프레임워크의 API 에 추가적인 변화를 만들도록 허용
  
모듈 안정성 기능은 라이브러리 에볼루션 지원 기능을 필요로 합니다.

배포를 위해 바이너리 프레임워크를 빌드하는 경우 일반적으로 이 두가지 기능을 활성화하게 될 것입니다.
> **NOTE**
>
> 바이너리 프레임워크는 동적(dynamic) 라이브러리라고 이해하면 다음 내용들을 이해하기 수월합니다.
> 스위프트 패키지는 기본적으로 정적(static) 라이브러리 입니다.

## 라이브러리 에볼루션 지원 활성화 시키는 경우
기본적으로 에볼루션 기능은 꺼져 있습니다.
빌드와 배포를 항상 하는 프레임워크의 경우 (예: 스위프트 패키지, 또는 앱 내부의 바이너리 프레임워크) 에볼루션 기능을 쓰지 말아야합니다.

에볼루션 기능은 오직 클라이언트 와는 별도로 프레임워크가 빌드되고 업데이트 되는 경우에만 사용해야합니다.
예를 들어 옛 버전의 프레임워크로 빌드한 앱이 다시 컴파일 하지 않고 새버전의 프레임워크에서 돌아갈 수 있는 경우가 이에 해당합니다.

에볼루션 기능을 켜는 것은 프레임워크의 퍼포먼스 형식을 바꾸고 소스 비호환 언어 변화를 도입합니다.

에볼루션 없이 빌드한 프레임워크는 바이너리 호환성을 보장해주지 않기 때문에 에볼루션 기능을 키게 되면 바이너리가 호환되지 않는 변경이 됩니다.

> **INFO**
>
> 예전에 다음과 같은 의존성을 가진 프로젝트에서 SDK 내 수정된 인터페이스에 대한 signature를 못 찾아 크래시가 난적이 있다.
>
> 앱 --의존--> Sendbird UIKit --의존---> Chat SDK
>
> 이때 Sendbird UIKit은 구버전을 그대로 사용하고, Chat SDK 만 업데이트 했는데,
> Sendbird UIKit은 구버전의 Chat SDK의 interface를 사용하고 있었지만, 새 버전의 Chat SDK 에서 해당 인터페이스가 수정되면서 시그니처가 사라져었다.

## 라이브러리 에볼루션 지원 활성화 하기
### Xcode
Xcode 를 사용해서 개발하고 있다면 프레임워크 타겟에서 `BUILD_LIBRARY_FOR_DISTRIBUTION` 빌드 세팅을 설정합니다. 이 설정은 모듈 안정성과 라이브러리 에볼루션 기능 둘 다 켜줍니다.
> **IMPORTANT**
> 
> 반드시 Debug, Release 모드 둘 다 이 설정을 사용해야합니다.

> **NOTE**
> [Binary frameworks in Swift | WWDC19](https://developer.apple.com/wwdc19/416) 영상 참고

### 컴파일러에서 직접 호출
`swiftc` 를 직접 사용하고 있다면, 커맨드 라인이나 빌드 시스템 둘 중 한 곳에서 `-enable-library-evolution` 과 `-emit-module-interface` 플래그를 전달하면 됩니다.
```bash
$ swiftc Tack.swift Barn.swift Hay.swift \
  -module-name Horse \
  -emit-module -emit-library -emit-module-interace \
  -enable-library-evolution
```
위의 명령어를 돌리면, `Horse.swiftinterface` 라는 이름의 모듈 인터페이스 파일과 `libHorse.dylib`(macOS) 또는 `libHorse.so`(리눅스) 공유 라이브러리가 생성됩니다.
> **INFO**
>
> `dylib` 는 동적라이브러리(Dynamic Library) 를 의미합니다.

