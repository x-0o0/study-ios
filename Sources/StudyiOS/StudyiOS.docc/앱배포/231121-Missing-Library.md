# 라이브러리 에러와 임베드

정적 라이브러리와 동적 라이브러리에 대해 알아보고 라이브러리 에러 분석하고 해결하기

## 1. 서론

사이드 프로젝트 앱 심사 중 다음과 같은 크래시 로그 파일을 전달 받았습니다.

> *crashlog-3BD80720-2A0C-4403-BB56-6EFC6063243F.txt*
>
> ```
> "termination" : {"code":1,"flags":518,"namespace":"DYLD","indicator":"Library missing","details":["(terminated at launch; ignore backtrace)"],"reasons":["Library not loaded: @rpath\/KuringSDK.framework\/KuringSDK"
> ```

앱 크래시의 원인은 `"termination"` 항목에서 확인할 수 있고, 보면 `DYLD`, `Library missing`, `Library not loaded` 이러한 용어들이 나옵니다. 용어를 몰라도 대충 KuringSDK 프레임워크가 로딩 되지 않아서 크래시가 났구나 하고 알 수는 있습니다. 
이 문서에서는 동적라이브러리, 정적 라이브러리에 대한 중요한 내용들을 다뤄보면서 위의 크래시 로그가 정확히 무슨 의미인지, 그리고 어떻게 해결하는 지를 알 수 있습니다.

## 2. 알아보기

### 2.1. 앱 패키지

앱 타겟을 빌드하면 결과물로 다음과 같은 패키지 폴더가 생성됩니다.

<img width="369" alt="스크린샷 2023-11-21 오후 3 23 22" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/0f6ea399-37e6-4473-9338-650dbd61a038">

여기서 실행파일은 실제 앱 구동에 사용됩니다.

### 2.2. 동적 라이브러리 그리고 정적 라이브러리

라이브러리의 코드가 앱의 소스 코드에 합쳐지는 것을 **"링크(Link)"** 라고 합니다. 링크 되는 방식에 따라 라이브러리는 **정적 라이브러리(Static Library)** 그리고 **동적 라이브러리(Dynamic Library)** 로 구분됩니다.

**정적 라이브러리**는 앱 타겟 빌드 시, **스태틱 링커**(Static Linker)에 의해 실행파일에 라이브러리의 코드가 앱의 소스코드와 합쳐지게 됩니다. 

<img width="339" alt="스크린샷 2023-11-21 오후 3 23 54" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/fbd4feec-4dcf-47e4-8ef1-53c3b9623b95">

**동적 라이브러리**는 앱 타겟 빌드 시, 실행파일에 라이브러리의 코드에 대한 참조만 포함되고 앱 구동 중에 라이브러리 코드가 필요한 시점이 오면 **다이나믹 링커**(Dynamic Linker, 또는 다이나믹 로더 Dynamic Loader)가 이 참조를 가지고 라이브러리를 로딩하게 됩니다.

<img width="357" alt="스크린샷 2023-11-21 오후 3 24 14" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/9189f7a7-5e27-4b69-b7cf-b99d7265a2e0">

| 종류 | 사용하는 링커 | 링크 방법 |
| --- | --- | --- |
| 정적 라이브러리 | Static Linker | 빌드 시, 앱 실행파일에 필요한 라이브러리 코드를 합침. |
| 동적 라이브러리 | Dynamic Linker/Loader (DYLD) | 빌드 시, 앱 실행파일에 라이브러리의 참조를 포함시킴.</br> 런타임에 필요한 시점에 참조를 통해 라이브러리를 로딩. |

### 2.2. DYLD

DYLD 는 Dynamic Loader 를 의미합니다. 다이나믹 링커(Dynamic Linker) 라고도 합니다. 보통 라이브러리 관련해서 dy 로 시작하는 약어는 일반적으로 Dynamic (동적) 을 의미하고 좀 더 구체적으로는 다이나믹 라이브러리에 대한 것을 의미합니다. 예를 들어, dylib 는 Dynamic Library 를 의미합니다.

동적 라이브러리의 경우, 앱 타겟 빌드 결과물의 실행파일에는 라이브러리에 대한 **참조** 가 들어있어서, 앱을 실행하면 이 참조를 가지고 필요한 라이브러리의 코드를 로딩하게 됩니다. 이 **로딩 과정을 진행하는 주체**가 바로 **DYLD** 입니다.

### 2.3. 라이브러리와 임베드

임베드(Emded) 옵션은 라이브러리를 Frameworks 폴더에 포함시킬지 말지에 대한 여부를 나타내는 옵션입니다.

정적 라이브러리의 경우, 이미 필요한 라이브러리 코드를 앱 실행파일이 전부 갖고 있기 때문에 굳이 Frameworks 폴더에 라이브러리를 중복으로 들고 있을 필요가 없습니다. 따라서 임베드 옵션을 `Do Not Embed` 로 설정할 수 있습니다. 다만 번들의 경우 실행파일에 합쳐지는 것이 아니므로 정적 라이브러리가 번들과 같이 앱 구동 중 로딩이 필요한 요소를 갖고 있다면 `Embed` 로 설정해야 합니다.

<img width="284" alt="스크린샷 2023-11-21 오후 3 24 44" src="https://github.com/jaesung-0o0/study-ios/assets/53814741/eb4a1057-ee12-488c-9e4e-a0a1b69c6964">

동적 라이브러리의 경우, 앱 실행파일이 참조만 갖고 있기 때문에 이 참조를 가지고 다이나믹 로더가 라이브러리를 로딩할 수 있도록 Frameworks 폴더 안에 라이브러리가 있어야 합니다. 따라서 임베드 옵션을 `Embed & Sign` 혹은 `Embed Without Signing` 으로 설정해줘야 합니다.

## 3. 결론

서론에서 보았던 `Library not loaded` 에러는 문제가 되었던 라이브러리가 동적 라이브러리 였고 **"Do Not Embed"** 로 옵션이 설정되어 있었기 때문에 발생한 에러였습니다.
빌드 결과물의 실행파일에는 참조만 있고 앱 실행시 **다이나믹 로더(DYLD)** 가 이 참조를 가지고 라이브러리를 로드해야 하는데 임베드 옵션이 "Do Not Embed" 이었기 때문에, 로드할 라이브러리가 Frameworks 폴더에 없어서 크래시가 난 상황입니다.
그러므로 임베드 옵션을 **"Embed~~~"** 로 변경해주면 크래시 에러를 해결할 수 있습니다.

## 4. 참고문헌

[Overview of Dynamic Libraries, developer.apple.com | Apple, Inc.](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
