# 캐싱하기

## 메모리 캐시
`NSCache` 를 통해 메모리 캐시를 사용할 수 있다.
- 시스템에서 자체적으로 제공
- App 을 끄면 저장 내용이 사라짐
- 처리속도 빠름
- 저장공간 작음. 캐시에서 제거되어도 되는 객체는 `NSDiscardableContent` 프로토콜을 conform 하면 됨

## 디스크 캐시
캐시에 저장할 데이터를 기기 안에 아카이브 하는 방식. 앱을 껐다가 켜도 데이터가 계속 남음.
앱을 삭제해도 캐시가 남도록 하려면 `FileManager` 를 사용하여 파일 경로에 저장하면 된다. 
앱 삭제시 데이터도 제거하고 싶다면 `UserDefaults` 를 사용.