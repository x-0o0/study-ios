# TextEditor 투명한 백그라운드

## 해결방법

```swift
TextEditor(text: $text)
    .scrollContentBackground(.hidden) // 1
    .background { ... } // 2
```
1. 기존에 TextEditor 가 갖고 있는 배경을 hidden 처리
2. 새 백그라운드 세팅
