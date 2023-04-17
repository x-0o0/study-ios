> **공식문서**: https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/

you can use actors to safely access mutable state.

느리거나 버그가 있는 코드에 Concurrency를 적용한다고 빨라지거나 코드가 올바르게 동작하거나 하진 않는다. 오히려 디버깅 하기가 어려워질 것이다.

적어도 컴파일 때 문제점을 발견해줄 수 있도록 도와줄 것이다.

**노트** : 이전에 동시성 코드를 작성한적이 있다면 아마 쓰레드를 가지고 동작하도록 하곤 했을 것이다. 스위프트에서 Concurrency 모델은 쓰레드 가장 상단에서 설계되었다. 스위프트에서는 어떤 비동기 함수가 첫번째 함수가 멈춘동안 돌아갈 수 있도록 다른 비동기 함수가 돌고 있는 쓰레드를 버릴 수 있다. 비동기 함수가 재개되면 스위프트는 이 비동기 함수가 어느 쓰레드에서 돌지 보장해주지 않는다.

---

다음은 사진을 다운받는 비동기 코드다.

```swift
listPhotos(inGallery: "Summer Vacation") { photoNames in
    let sortedNames = photoNames.sorted()
    let name = sortedNames[0]
    downloadPhoto(named: name) { photo in
        show(photo)
    }
}
```

간단해보이지만 이 예시는 핸들러를 연속으로 사용하고 있다. 이렇게 연속으로 클로져를 쓰는 경우 코드가 복잡해져서 다루기 어렵다.

### 비동기 함수 정의 및 호출하기

throws 처럼 파라미터 다음에 async를 쓰면 된다.

```swift
func listPhotos(inGallery name: String) async -> [String] {
    let result = // 비동기 네트워킹 코드 ...
    return result
}
```

만약 비동기랑 throwing을 같이 쓰는 경우, `async throws` 로 쓰면 된다

비동기 함수를 호출하면 함수가 리턴할때까지 실행이 멈춘다.

멈출 가능성이 있는 모든 비동기 함수 호출 앞에 `await` 를 쓴다(마치 `try` 처럼)

다른 비동기 함수를 호출하기 전까지는 비동기 함수 내부 코드실행은 중지되지 않는다(중지는 절대 암시되거나 선수치지 않는다)

예시로 다음 코드는 갤러리에서 사진 이름을 가져와서 그에 해당하는 사진을 다운로드 받아서 보여주는 코드다.

```swift
let photoNames = await listPhotos(inGallery: "Summer Vacation")
let sortedNames = photoNames.sorted()
let name = sortedNames[0]
let photo = await downloadPhoto(named: name)
show(photo)
```

`listPhotos(inGallery:)` 와 `downloadPhoto(named:)` 둘 다 네트워크 요청을 만들어 이에 대한 완료까지 시간이 걸리는 비동기 코드다.

함수 선언시 `->` 전에 `async`를 써서 사진이 준비될 때까지 기다리는 동안 앱은 다른 동작을 계속 할 수 있도록 한다.

위의 동시성 코드를 이해해보자면,

1. 첫번째 `await`를 마주하면 `listPhotos(inGallery:)` 가 값을 리턴할때까지 실행을 중지한다.
2. 코드가 중지되는 동안 같은 프로그램 내의 다른 동시성 코드들은 계속 돌아간다. 예를 들어 백그라운드에서 새 사진갤러리 리스트를 계속 업데이트 작업이 돌아갈 수 있다. 이 코드는 다음 `await`를 만나거나 실행이 완료될 때까지 계속 돌아간다. 
3. `listPhotos(inGallery:)` 가 리턴하면 실행이 재개되고 `photoNames` 에 값이 할당된다.
4. 그 다음 줄의 `sortedName`과 `name`은 평범한 동기성 코드이다. `await`로 표시된 것이 없기 때문에 멈출 가능성이 있는 지점이 없다.
5. `downloadPhoto(named:)` 에서 다시 한번 동작이 멈추고 리턴할때까지 기다린다. 그동안 다른 동시성 코드들은 돌아갈 수 있다.
6. `downloadPhoto(named:)`가 값을 리턴하면 다시 실행이 재개되고 `photo`에 값이 할당되어 show(_:) 에 전달되면서 마지막 줄 코드가 실행된다.

멈출 가능성이 있는 코드에 전부 await를 표시하는 것을 yielding the thread 라고도 부른다. 무대 뒷편에서 스위프트는 현재 쓰레드에서 코드가 중지되면 다른 코드를 동일 쓰레드에서 재개 시키기 때문이다.

`await`는 실행이 멈출 수 있어야 하는 곳에서 쓰일 수 있기 때문에, 다음과 같은 상황에서만 프로그램은 비동기 함수를 호출할 수 있다.

- 비동기 함수 또는 프로퍼티 내부
- `@main` 이 표시된 struct, class, enum의 `main` static 함수 내부
- [구조화 되지 않은](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency#Unstructured-Concurrency) 하위 업무 코드

멈출 가능성이 있는 지점들 사이에서 코드들은 다른 동시성 코드의 방해를 받을 일 없이 시퀀셜하게 돌아간다.

```swift
let firstPhoto = await listPhotos(inGallery: "Summer Vacation")[0]
add(firstPhoto, toGallery: "Road Trip")
// At this point, firstPhoto is temporarily in both galleries.
remove(firstPhoto, fromGallery: "Summer Vacation")
```

위 예시 코드에서 add(*:toGallery:) 와 remove(*:fromGallery:) 사이에서 다른 코드가 실행될 일은 없다. (동기적 코드)

```swift
let firstPhoto = await downloadPhoto(named: photoNames[0])
let secondPhoto = await downloadPhoto(named: photoNames[1])
let thirdPhoto = await downloadPhoto(named: photoNames[2])

let photos = [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

위 코드는 순차적으로 실행되는 코드이다. 하지만 차례대로 사진을 가져와 보여주는 것은 실제 상황에서 비효율적이다.

나란히 여러 비동기 코드들을 실행시키고 싶다면, `async let`을 붙이고 이를 사용할때마다 앞에 await을 붙이면 된다.

```swift
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])

let photos = await [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

이는 사진 다운로드를 순차적으로 실행하지 않고 나란히 실행하며 시스템 여유가 있다면 동시에 실행될 수도 있다.

함수 결과값을 기다리기 위해 멈추지 않기 때문에 다운로드 앞에 await 붙이지 않는다.

“다운로드1 시작, 다운로드2 시작, … 결과 기다려~”
