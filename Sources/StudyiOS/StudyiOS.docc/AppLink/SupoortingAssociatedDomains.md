# 연결도메인 지원하기 (Supporting Associated Domains)
앱과 웹사이트를 연결하여 기본 앱과 브라우저 경험 둘다 제공합니다.

- - - -
## Overview
Associated Domain(이하 ‘연결도메인’)은 앱과 도메인 간의 보안 연결을 시켜주어 크레덴셜을 공유하거나 브라우저에서 앱의 기능을 제공할 수 있습니다. 
예를 들어, 온라인 쇼핑몰은  자체 웹사이트를 동반하고 사용자 경험을 강화시키는 앱을 제공하려고 할 것 입니다.
공유된 웹 크레덴셜과 유니버셜 링크, 그리고 핸드오프 전부 연결도메인을 사용합니다. 연결도메인은 유니버셜 링크의 밑받침이 됩니다. 앱을 다운받지 않은 사용자들은 네이티브 앱 대신에 웹브라우저를 통해 동일한 정보를 받을 수 있습니다.
	- 유니버셜 링크: 앱이  웹사이트의 일부 또는 전체를 대체하는 컨텐츠를 표시할 수 있도록 해주는 기능
앱과 웹사이트를 연결 시키기 위해, 웹사이트에는 연결 도메인 파일이, 앱에는 적합한 인타이틀먼트(entitlement)이 필요할 것입니다.
웹사이트의 `apple-app-site-association` 파일 안의 앱은 반드시 [연결도메인 인타이틀먼트(Associated Domains Entitlement)](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)와 매칭되어야 합니다.

## 웹사이트에 연결도메인 파일 추가하기
사용자가 앱을 설치할 때, 시스템은 연결도메인 파일을 다운로드 하려고 하며 인타이틀먼트에 있는 도메인을 인증합니다.

> **노트**  
> 만약 여러분의 사이트가 여러 개의 서브도메인 (예: example.com, www.example.com, support.example.com) 을 사용한다면, 각각의 서브도메인은 연결도메인 인타이틀먼트에 있는 개별의 엔트리가 필요합니다. 그리고 각 서브도메인은 반드시 개별의  `apple-app-site-association` 파일을 전달해야합니다.  

웹사이트에 연결도메인 파일을 추가하기 위해서는 `apple-app-site-assocation`이라는 파일을 생성하도록 합니다.(확장자 없이) 그런 다음 여 도메인 상에서 지원하고자 하는 서비스를 위해 파일의 JSON 를 업데이트 합니다. 유니버셜 링크를 위해 `applinks` 서비스 내에 도메인을 위한 앱 ID들을 확실하게 목록화 하도록 합니다.

아래의 JSON 는 간단한 연결 파일의 내용을 보여줍니다.

```json
{
  "applinks": {
      "details": [
           {
             "appIDs": [ "ABCDE12345.com.example.app", "ABCDE12345.com.example.app2" ],
             "components": [
               {
                  "#": "no_universal_links",
                  "exclude": true,
                  "comment": "Matches any URL whose fragment equals no_universal_links and instructs the system not to open it as a universal link"
               },
               {
                  "/": "/buy/*",
                  "comment": "Matches any URL whose path starts with /buy/"
               },
               {
                  "/": "/help/website/*",
                  "exclude": true,
                  "comment": "Matches any URL whose path starts with /help/website/ and instructs the system not to open it as a universal link"
               }
               {
                  "/": "/help/*",
                  "?": { "articleNumber": "????" },
                  "comment": "Matches any URL whose path starts with /help/ and which has a query item with name 'articleNumber' and a value of exactly 4 characters"
               }
             ]
           }
       ]
   },
   "webcredentials": {
      "apps": [ "ABCDE12345.com.example.app" ]
   }
}
```

`appID`와 `apps` 이 두가지 키들은 웹사이트에서 서비스 타입들과 함께 사용가능한 앱의 ID를 명시합니다. 그 키들에 아래의 포맷을 값으로 사용하세요.

```
<Application Identifer Prefix>.<Bundle Idetifier>
```

`details`딕셔너리는 오직 `applinks` 서비스 타입에만 적용됩니다. 다른 서비스 타입들은 해당 딕셔너리를 사용하지 않습니다. `components` 키는 URL 컴포넌트와 매칭하는 패턴들을 제공하는 딕셔너리들의 배열입니다.
연결 파일을 구성하고 나면, 이 파일을 사이트의 `.well-known` 경로에 위치하도록 합니다. 파일의 URL은 반드시 아래의 포맷과 매칭되어야 합니다.

```
htttps://<fully qualified domain>/.well-known.apple-app-site-association
```

반드시 유효한 인증서와 경로 변경 없이  `https://` 를 사용하여 파일을 호스팅 해야합니다.

## 앱에 연결도메인 인타이틀먼트 추가하기
앱의 인타이틀먼트(entitlement)를 설정하기 위해, Xcode에서 해당 프로젝트를 열고 **Signing & Capablities** 탭으로 이동합니다. 그런 다음 **Associated Domains** 캐퍼빌리티를 추가합니다. 이 단계는 앱에 **Associated Domains Entitlement**를 , 앱ID에는 연결도메인 기능을 추가합니다.

인타이틀먼트에 도메인을 추가하기 위해, 먼저 도메인 테이블 아래에 있는 **Add (+)** 버튼을 클릭하여 도메인 플레이스홀더(placeholder) 를 추가하도록 합니다. 그리고 그 자리에 앱이 지원할 서비스에 대한 적합한 prefix와 사이트 도메인을 넣어주도록 합니다.
예)
**Associated Domains**
<details>
```
Domains
____________________
applinks:example.com
____________________
webcredentials:example.com
```
</details>

앱이 크레덴셜을 어디와 공유할 것인지에 대한 도메인들을 추가하도록 합니다. 연결도메인의 모든 서브 도메인들을 전부 매칭시키기 위해 `*.` prefix로 와일드 카드를 특정 도메인 앞에 명시해주세요.

각각의 명시되는 도메인들은 아래의 포맷을 따라야 합니다.
```
<service>:<fully qulified domain>
```

macOS 11, iOS 14에서 시작한다면, 앱은 더이상 `apple-app-site-association` 파일을 웹서버에 직접적으로 요청하지 않습니다. 대신에, 연결도메인을 위해 만들어진 Apple이 관리하는 Content Delivery Network (CDN)에 요청을 하게 됩니다.

앱을 개발하는 동안 웹서버가 공공 인터넷을 통해 접근할 수 없다면 Alternate 모드 기능을 사용하여 CDN로 우회하고 프라이빗 도메인을 직접적으로 연결할 수 있습니다.

Alternate 모드는 연결도메인 인타이틀먼트에 아래의 쿼리 문자열을 추가하여 할성화할 수 있습니다.
```
<service>:<fully qualified domain>?mode=<alternate mode>
```

- Alternate 모드에 대한 추가적인 정보가 필요하다면, [연결도메인 인타이틀먼트 (Assoicated Domains Entitlement)](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)를 참고하세요.
- 유니버셜 링크에 대한 추가적인 정보가 필요하다면, [앱과 웹사이트가 컨텐츠를 연결하도록 하기](bear://x-callback-url/open-note?id=62C28C40-C0AB-4625-AA83-F4E1891B149B-9693-000012FC6533C230) 를 참고하세요.
- 핸드오프에 대한 추가적인 정보가 필요하다면, [핸드오프 프로그래밍 가이드 (Handoff Programming Guide)](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html#//apple_ref/doc/uid/TP40014338)를 참고하세요.
