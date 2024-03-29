# 아이콘 이미지 색상 바꾸기

## 결론 미리보기

> **스택오버플로우 왈** "For setting the color to single-color images(like icons and symbols) with the foregroundColor modifier, you should make sure that image renderingMode is set to template"

```swift
Image(named: "MyIcon")
  .renderingMode(.template)
  .foregroundColor(.green)
```

`renderingMode(_:)` 를 사용하여 아이콘 이미지의 색상을 바꿀 수 있다.

## 개요

png 로 다운로드 받은 인스타그램 로고를 스유에 띄워보려고 했다.

```swift
Image(named: "instagram.logo")
  .foregroundColor(.green)
```

응 실패.

싱글 컬러 이미지는 `renderingMode(_:)` 로 렌더링 모드를 `.template` 으로 변경해야지 원하는 색상을 적용할 수 있다.

이렇게...🌟

```swift
Image(named: "instagram.logo")
  .renderingMode(.template)
  .foregroundColor(.green)
```
