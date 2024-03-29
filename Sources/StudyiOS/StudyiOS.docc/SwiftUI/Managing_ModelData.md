# 모델 데이터 관리하기

> 링크: [managing-model-data-in-your-app](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app)

> 잡담: 어떨 때는 주로 제목, 소제목에는 Model data, 본문에는 Data model 이라는 용어를 쓰는데 둘의 차이를 잘 모르겠어서
> 어떤 상황에 어떤 명칭을 써야하는지가 궁금했는데, 마침 [그 내용이 개요 부분에 있었다](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app#:~:text=model%20data%20(that%20is%2C%20an%20instance%20of%20a%20data%20model)).
>
> "데이터 모델의 인스턴스가 모델 데이터" (그러면... `let modelData = DataModel()` 인건가......🫤)

## 개요

데이터 모델은 데이터와 뷰 간의 **분리** 를 제공합니다. 
이러한 분리는 
- 모듈화와 testablility를 향상시키고
- 앱 동작 방식에 대한 추리를 쉽게 하도록 도와줍니다.

> 잡담: 예전에 사내 스유 스터디 할 때, 회사 동료분이 뷰와 데이터 로직간의 분리에 대해 고민을 하셨던게 기억이 났다. 
> 그 때 이후로 나도, 뷰와 데이터 로직간의 분리를 많이 고민했고 데이터 모델을 어떻게 활용하고 뷰에서 이를 어떻게 접근하는지가 키포인트라고 생각이 든다.
>
> 요즘 도메인에 따른 캡슐화에 많이 신경을 쓰고 있어서 이 문서를 꼼꼼히 봐야할 중요성을 다시 한번 느꼈다.

iOS 17, macOS 14, tvOS 17, watchOS 10부터 ``Observation`` 매크로를 사용할 수 있다.

### 옵저빙 가능한 모델 데이터 만들기

변화를 시각화 하고 싶은 데이터가 있다면 그것의 데이터모델에 ``Observable`` 매크로를 적용해야한다.
이 매크로는 컴파일 타임에 데이터모델에 옵저빙을 지원을 위한 코드를 생성한다. (Observable 프로토콜을 채택하게 함)
```swift
@Observable class Book: Identifiable {
    var title = "책 제목"
    var author = Author()
    var isAvailable = true
}
```
옵저빙은 참조타입, 값타입 둘다 사용이 가능하다.
> 잡답: "값타입" 이라고 표기한거면, enum 도 된다는 건데, 한번 사용해봐야겠다.

> Important: `Observable` 프로토콜 스스로 옵저빙 기능을 추가해주진 않으므로 해당 프로토콜을 데이터 모델에 적용하지 말고
> 항상 `Observable` 매크롤 사용할 것. by 애플

### 뷰에서 모델 데이터 옵저빙하기

스유에서, `body` 가 데이터 모델 객체의 프로퍼티를 읽는다면, 해당 뷰는 해당 데이터 모델에 디펜던시를 형성합니다. 
만약 읽어오는 데이터 모델 내 프로퍼티가 없으면 디펜던시를 형성할 시도조차 안합니다. 
스유는 오직 `body` 가 읽고 있는 프로퍼티의 변화만 감지합니다. 즉, 읽지 않는 프로퍼티는 값이 바뀌어도 뷰에 아무런 영향이 없어 업데이트가 발생하지 않습니다.
```swift
struct BookView: View {
    var book: Book

    var body: some View {
        Text(book.title) // title 값 변화에만 뷰 업뎃하고 `author`, `isAvailable` 변화에는 반응 없음
    }
}
```
그리고 뷰가 데이터모델 객체를 소유하지 않더라도, `body` 에서 읽기만하면 디펜던시 트랙킹을 합니다. (예: 전역 변수, 싱글톤)
```swift
var globalBook: Book = Book()

struct BookView: View {
    var body: some View {
        Text(globalBook.title)
    }
}
```
그리고 computed property 가 옵저빙 가능한 프로퍼티를 사용하면 해당 computed 프로퍼티도 트래킹할 수 있도록 지원합니다.
```swift
@Observable clas Library {
    var books: [Book] = [Book(), Book(), Book()]

    var availableBooksCount: int {
        books.filter(\.isAvailable).count // 오호... 고차함수에 key path...
    }
}
```

To be continued...
