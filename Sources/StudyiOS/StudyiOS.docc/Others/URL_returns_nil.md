# URL 이 nil 을 리턴하는 경우

올바른 string 값을 가지고 URL을 생성했음에도 불구하고 `nil` 을 리턴했다면, 아래의 케이스에 해당하는지 확인해보자.
- string에 스페이스가 있진 않은가?
- 한글이 들어가 있진 않은가?

해결방법: 인코딩한 값으로 URL을 생성하면 된다.

```swift
if let encodedString = urlStr.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed), let url = URL(string: encodedString) {
    // Handle `url` here
}
```
