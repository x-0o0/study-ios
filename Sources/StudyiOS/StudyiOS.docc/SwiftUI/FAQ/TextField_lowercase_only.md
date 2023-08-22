# TextField 텍스트 전부 소문자로 입력받기

Lowercased Textfield

## 해결방법

```swift
TextField("placeholder", text: $text)
    .textInputAutocapitalization(.never) // 1
    .textCase(.lowercase) // 2
```
1. 자동으로 대문자로 입력되는 것을 해지
2. 기본 입력 `.lowercase` 로 세팅

> **이슈** 이렇게 이중으로 처리해도 대문자 입력이 허용된다. 시뮬레이터에서만의 문제인지 실제 디바이스도 그런지는 확인해봐야할 듯
