# Task

## 개요

프로그램에서 비동기적으로 돌아갈 수 있는 작업단위를 Task라고 부른다

async let 은 하위 태스크를 생성한다. 이러한 하위 태스크는 Task Group에 넣을 수 있다.

태스크들은 계층구조 형태로 정렬되어있다. 태스크그룹의 모든 태스크들은 서로 같은 상위 태스크를 갖는다. 태스크와 태스크그룹간의 관계가 명확하기 때문에 이를 구조적 동기성이라고 한다 (structured concurrency)

```swift
await withTaskGroup(of: Data.self) { taskGroup in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        taskGroup.addTask { await downloadPhoto(named: name) }
    }
}
```

## 비구조적 동기성

비구조적 태스크는 상위태스크를 가지지 않는다. 이는 프로그램이 필요로하는 어떤 방법이던 상관없이 완전히 관리에 대한 유연성을 가진다. 하지만 올바르게 사용하는 것에 대한 책임감은 개발자에게 있다.
현재 액터에서 돌아가는 비구조적 태스크를 생성하려면, `[Task.init(priority:operation:)](https://developer.apple.com/documentation/swift/task/3856790-init)` 생성자를 사용해야한다. 
만약 현재 액터 소속이 아닌 비구조적 태스크로 생성하려면 (주로 Detached task 라고 부른다) `[Task.detached(priority:operation:)](https://developer.apple.com/documentation/swift/task/3856786-detached)` 클래스 메소드를 사용해야한다.

```swift
let newPhoto = // ... some photo data ...
let handle = Task {
    return await add(newPhoto, toGalleryNamed: "Spring Adventures")
}
let result = await handle.value
```

## 태스크 취소하기

각각의 태스크는 실행하고나서 적절한 지점에서 취소 되었는지 체크하고, 적절한 지점에서 취소되었으면 이를 Cancellation에게 응답.

하고 있는 작업에 따라 다음의 경우로 나뉘어질 수 있다.

- CancellationError 와 같은 에러 던지기
- 빈 컬렉션 또는 nil 리턴하기
- 부분적으로 완료된 작업 리턴하기

취소를 체크하는 방법은 다음과 같다.

- 태스크가 취소되면 CancellationError를 던지는 Task.checkCancellation()를 호출하기
- Task.isCancelled 값 확인하고 코드상에서 직접 취소 다루기

예를 들어, 갤러리에서 사진을 다운로드하는 작업은 부분적인 다운로드 삭제나 네트워크 연결 해지가 필요할 수 있다.

수동으로 태스크 취소를 하는 경우, Task.cancel() 를 호출
