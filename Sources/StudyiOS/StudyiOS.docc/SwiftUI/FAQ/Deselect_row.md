# Row 선택 해지

## 코드 결론 미리보기

리스트아이템에 적용

```swift
// Row.swift

var body: some View {
    { ... }
        .buttonStyle(.plain)
}
```

또는 리스트에 적용

```swift
List {
    ForEach(...) {
        Row()
            .buttonStyle(.plain)
    }
}
```


## 개요

```swift
List {
    ForEach(...) {
        Row()
            .buttonStyle(.plain) // 2
    }
}
.listStyle(.plain)
.buttonStyle(.plain) // 1
```

### 1번 경우

`ForEach` 에 적용하는 경우 de-select가 좀 늦게 동작하는 현상이 있었다. 예를 들어 swipe action을 하는 경우 selected 상태가 유지 되어있다가 터치를 떼는 순간 deselect 된다.

### 2번 경우

기가막히게 잘 된다. 경이롭다. 그냥 묻지도 말고 따지지도 말고 Row 뷰에 바로 적용하는 것을 추천한다.
