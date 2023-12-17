# Mergeable Library: 병합 가능한 라이브러리

## 1. 배경 및 목표

Xcode15 부터 기존의 static linker(이하 정적링커)에 더해 새로운 정적 링커가 도입되었습니다. 
이 정적링크를 사용하면, 병합이 가능한지 여부가 명시된 라이브러리 또는 프레임워크를 하나의 병합된 바이너리로 다룰 수 있게 됩니다.
이 병합 가능한 라이브러리를 Mergeable Library (이하 병합 가능 라이브러리 또는 머져블 라이브러리) 라고 합니다.
이 문서에서는 병합 가능한 라이브러리를 어떻게 만들고 사용하는 지, 그리고 어떻게 동작하는 지를 알아보겠습니다.

## 2. Mergeable 라이브러리 사용방법

### 2.1. 자동 병합 - 앱 타겟에 유용
자동병합은 추가된 모든 의존성을 병합

예를 들어, 
앱타겟에 의존성으로 `SwiftUI`, `ForestBuilder`, `ForestUI`, `Forest` 가 추가되어 있으면 시스템 라이브러리를 제외하고 나머지 3개의 Forest 프레임워크가 병합 가능해집니다.
따라서, 빌드하게 되면 아래와 같이 링커가 Forest 프레임워크를 앱 바이너리에 병합시킵니다.

| 병합 전 | Forest 가 앱 바이너리에 병합된 후 |
| --- | --- |
| ![스크린샷 2023-12-12 오전 9 40 20](https://github.com/x-0o0/study-ios/assets/53814741/020d7baf-2336-4e8c-9467-68262a1e6bed) | ![스크린샷 2023-12-12 오전 9 40 49](https://github.com/x-0o0/study-ios/assets/53814741/11db2e80-12b7-4e82-9de4-91ebbfdad7d6) |

자동병합을 설정 방법은 다음과 같습니다.
1. 앱타겟 Build Settings
2. Create Merged Binary (`MERGED_BINARY_TYPE` 설정에 매핑된 옵션)
3. Automatic 으로 설정

이 때 병합 가능한 라이브러리의 export 는 앱에 보관되어 있는데,
아래 이미지와 같이 **`-no_exported_symbols`** 를 사용하면 용량 줄일 수 있습니다.
![스크린샷 2023-12-12 오전 10 09 35](https://github.com/x-0o0/study-ios/assets/53814741/bbb8d19f-3dd1-40a6-bc45-2c93403ec28c)

만약 앱 Extensions 을 위한 진입점이 필요한 경우 아래 이미지와 같이 **Exported Symbols File** 로 관리할 수 있습니다.
![스크린샷 2023-12-12 오전 10 20 34](https://github.com/x-0o0/study-ios/assets/53814741/1e47fb48-d8ed-4d79-91d8-03f17f9a635b)

### 2.2. 수동 병합 - 앱 Extensions 나 테스트타겟이 있을 경우 유용
수동병합은 병합할 라이브러리를 하나하나 직접 병합 가능한 상태로 지정하는 방식으로 일부 의존성이 앱 번들에 남아있어야 할 때 유용합니다.

예를 들어, 아래 이미지처럼 앱타겟과 테스트타겟(또는 Extension) 에 서로 복잡하게 의존성이 연결되어 있다고 가정 해보겠습니다.
![스크린샷 2023-12-12 오전 10 26 44](https://github.com/x-0o0/study-ios/assets/53814741/05139c1d-d0b0-4890-9dd1-d6ecd21c5db2)

이 때 직접 ForestKit 이라는 프레임워크를 생성해서 테스트타겟이 Forest가 아닌 ForestKit에 의존하도록 수정할 수 있습니다. [이미지1]
ForestKit 이 **그룹 라이브러리**역할을 수행하게 되어 앱과 테스트가 의존하는 병합 가능 라이브러리를 포함시킵니다.
따라서 ForestBuilder, ForestUI, Forest 이 ForestKit 으로 병합하게 되고 앱 타겟과 테스트 타겟은 ForestKit 하나만 의존성으로 가지게 됩니다. [이미지2]

| 이미지1 | 이미지2 |
| --- | --- |
| ![스크린샷 2023-12-12 오전 10 29 15](https://github.com/x-0o0/study-ios/assets/53814741/7b75acd9-8bc1-430f-b9fb-d8920d696eeb) | ![스크린샷 2023-12-12 오후 12 36 48](https://github.com/x-0o0/study-ios/assets/53814741/b93bd806-3d31-4d31-ae6d-9fbc4ece137c) |

수동 병합을 설정하는 방법은 다음과 같습니다.
1. 새 타겟으로 Framework (그룹 라이브러리) 추가
2. Build Phasses > Link Binary With Libraries 에 병합할 라이브러리 추가
3. Build Settings > Create Merged Binary 를 Manual 로 설정
4. Framework 타겟별로 Build Mergeable Library 를 Yes 로 빌드 설정
5. 앱타겟으로 가서 Link Binary With Libraries에서 불필요한 프레임워크 제거 및 단계1 에서 추가했던 Framework을 추가

### 2.3. Debug 모드에서의 동작

Debug 모드에서는 반복 개발 지원을 위해 라이브러리들을 병합을 하지 않습니다. 
대신에 빌드시스템이 링커에게 라이브러리를 re-export 하도록 지시합니다.
앱 Extension 이나 테스트 타겟이 동일한 병합된(merged) 바이너리에만 의존하기만 하면 모든 라이브러리 API 에 접근할 수 있습니다.

![스크린샷 2023-12-12 오후 12 50 35](https://github.com/x-0o0/study-ios/assets/53814741/4b64fdeb-121f-4028-9a4f-98375cfa61f7)

그리고 실행 시 병합된 바이너리에서 직접 API 를 가져오는 것이 아닌 다이나믹로더(dyld)가 re-export 된 라이브러리의 참조를 리다이렉트합니다.

### 2.4. 기존의 의존성 처리 방법

다른 실행파일들이 병합된(merged) 프레임워크에 의존하도록 업데이트 해야함[이미지1, 이미지2 참고]. 자동으로 링크할 때도 동일합니다.

| 이미지1 | 이미지2|
| --- | --- |
| ![스크린샷 2023-12-12 오후 1 19 22](https://github.com/x-0o0/study-ios/assets/53814741/0b1516ad-b9fe-4746-b246-50b7b3c2c5c4) | ![스크린샷 2023-12-12 오후 1 19 47](https://github.com/x-0o0/study-ios/assets/53814741/7c98def0-bb2e-4e6e-a050-a1fe9c2403d9) |

Link Binary With Libraries 에 병합된 프레임워크를 추가 하고 기존의 mergeable 프레임워크는 제거해야 합니다. 그렇지 않은 경우 다이나믹 로더(dyld)가 프레임워크 로딩을 할 수 없습니다.

## 3. 결론

Mergeable 라이브러리는 정적 라이브러리와 동적 라이브러리의 장점을 가져와서 생산성 극대화 및 성능과 앱 사이즈 최적화 라는 효과를 가질 수 있게 합니다.
mergeable 라이브러리는 말 그대로 하나로 "병합"이 가능한 라이브러리를 말하며 Xcode15 이상부터 지원되는 새로운 정적 링커(static linker) 를 사용하여 빌드합니다. 
이 때, 새로 도입된 링커는 armv7k 를 지원하지 않기 때문에 watchOS 9 이상의 버전부터 지원 가능합니다.
새로운 정적 링커를 사용하여 mergeable 라이브러리들을 빌드할 경우 Release 모드일 때는 앱 바이너리에 병합되고 
디버그(Debug) 모드 일 때는 앱 타겟이 merged binary 를 바라보고 실제로는 앱 패키지 내에 ReexportedBinaraies 에 병합 가능한(mergeable) 프레임워크들을 보관합니다. [^1]
Mergeable 라이브러리를 적용하는 방법에는 자동병합과 수동병합 2가지가 있습니다. 수동병합은 일부 의존성이 앱 번들에 남아있어야 하는 Extension 과 같은 타겟이 있는 프로젝트에서 유용합니다.
병합이 완료되면, 병합에 사용된 라이브러리(Mergeable 라이브러리)와 메타데이터는 제거되며, 병합하지 않더라도 Mergeable 라이브러리의 메타데이터는 제거됩니다. 
실제 동적라이브러리 보다 2배 크기의 메타데이터를 갖고 있음에도 불구하고 앱크기에 영향을 주지 않는 이유입니다.

## 4. 참고

### 4.1. 참고 문헌
- Meet Mergeable Library | WWDC23, https://developer.apple.com/videos/play/wwdc2023/10268

### 4.2. 각주

[^1]: 2.3. Debug 모드 참고


