# Swift Performance 이해하기 (Reference Counting)

## 개요

**String은 struct 이지만 heap에 할당된다**

struct 안에 reference type을 많이 들고 있으면 
- heap allocation
- reference counting
을 많이 사용하게 된다.

```swift
struct Attachment {
    let fileURL: URL
    let uuid: String
    let mimeType: String
    
    init?(fileURL: URL, uuid: String, mimeType: String) {
        guard mimeType.isMimeType else { return nil }
        self.fileURL = fileURL
        self.uuid = uuid
        self.mimeType = mimeType
    }
}
```

이를 아래와 같이 바꿔 heap allocation을 줄일 수 있다.

```swift
struct Attachment {
    let fileURL: URL
    let uuid: UUID  // heap allocation 줄이기
    let mimeType: MimeType  // heap allocation 줄이기
    
    init?(fileURL: URL, uuid: UUID, mimeType: String) {
        guard let mimeType = MimeType(rawValue: mimeType) else { return nil }
        self.fileURL = fileURL
        self.uuid = uuid
        self.mimeType = mimeType
    }
}
```
