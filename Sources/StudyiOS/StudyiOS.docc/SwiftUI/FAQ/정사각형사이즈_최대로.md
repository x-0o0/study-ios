# 정사각형 사이즈 최대로 키우기

Maximize the size of the square

## 결론 미리보기

`.aspectRatio(_:contentMode:)` -> `frame(maxWidth: .infinity)` 호출해주면 된다.


## 개요

아래 사진처럼 정사각형 형태로 뷰를 최대 사이즈로 키우고 싶다면

`.aspectRatio(_:contentMode:)` -> `frame(maxWidth: .infinity)` 호출해주면 된다.

```swift
Color(.tertiarySystemGroupedBackground)
    .aspectRatio(1, contentMode: .fill)
    .frame(maxWidth: .infinity)
```


<img width="260" alt="Screen Shot 2023-02-14 at 1 50 23 AM" src="https://user-images.githubusercontent.com/53814741/218520295-b9c3307d-28ae-4e22-84b6-01cc2483ba57.png">
