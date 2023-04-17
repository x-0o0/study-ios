태스크를 사용하면 프로그램을 고립된 동기성 조각으로 나눌 수 있습니다. 각각의 태스크는 다른 태스크로 부터 고립되어있습니다.(독립적이라는 의미인듯) 이는 동시에 여러 태스크가 돌아갈 때 각각의 태스크를 안전케 합니다. 하지만 간혹 태스크간에 정보를 공유해줘야할 때도 있습니다. 액터는 태스크간에 인전하게 정보를 공유할 수 있도록 해줍니다.

클래스 처럼 액터도 참조타입입니다. 하지만 클래스와는 달리 액터는 상태 변화 접근시 한번에 오직 하나의 태스크만 허용합니다. 이는 하나의 액터와 동시에 상호작용하는 태스크들로부터 코드를 안전하게 합니다. 예를 들어, 온도를 기록하는 액터가 있다고 가정해보겠습니다.

```swift
actor TemperatureLogger {
    let label: String
    var measurements: [Int]
    private(set) var max: Int

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }
}
```

액터 선언시 actor 키워드를 사용하면 됩니다. TemparatureLogger 액터에는 다른 코드에서 접근 가능한 프로퍼티들이 선언되어있습니다. max 만 제한을 걸어 액터내부에서만 값을 업데이트 할 수 있습니다.

클래스와 구조체와 동일한 생성자를 액터에서도 사용할 수 있습니다. 액터의 프로퍼티나 메소드에 접근할때는 잠재적 중지 지점에 await를 표시해야합니다.

```swift
let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
print(await logger.max)
// Prints "25"
```

이 예시에서 logger.max 에 접근하는 건 중지 가능지점입니다. 액터는 동시에 오직 하나의 태스크만 상태변화가 가능하기 때문에 이미 logger와 상호작용 중인 태스크가 있으면 이 코드는 접근을 기다리는 동안 중지되기 때문입니다.

이와 대조적으로 액터 내 코드는 액터의 프로퍼티에 접근할때 await을 쓰지 않습니다. 새 온도값으로 TemperatureLogger를 업데이트하는 메소드를 예시로 들어보겠습니다.

```swift
extension TemperatureLogger {
    func update(with measurement: Int) {
        measurements.append(measurement)
        if measurement > max {
            max = measurement
        }
    }
}
```

update(with:) 는 이미 액터에서 도는 메소드이기 때문에, max와 같은 프로퍼티에 접근할 때, await을 쓰지 않습니다. 또한 이 메소드 왜 액터가 한번에 오직 하나의 태스크만 상태변화에 접근하도록 하는지 알려주고 있습니다. 몇몇의 액터의 상태에 대한 일시적인 업데이트들은 불변성을 깨뜨리곤 합니다. TemperatureLogger 액터는 계속 온도값들리스트와 max 값을 트래킹합니다. 그리고 새 측정값이 있으면 max 값을 업데이트 합니다. 업데이트 도중, 새 측정값을 리스트에 추가는 했지만 아직 max 값을 업데이트 하기 전인 경우, temperature logger는 일시적으로 모순된 상태가 됩니다. 여러 태스크가 동시에 하나의 인스턴스와 상호작용하는 것을 막으면 다음과 같은 일련의 사건을 방지할 수 있습니다.

1. update(with:) 을 호출하면 가장 먼저 measurements 값이 업데이트 됩니다
2. 코드가 max 값을 업데이트 하기 전, 외부에서 max 값과 measurements 값을 읽습니다
3. 코드가 max 값을 업데이트하고 종료됩ㄴ

이때 2번 상황에서, update(with:) 호출도중 데이터가 일시적으로 유효하지 않을때 액터가 *인터리빙* 되기 때문에, 외부 코드는 잘못된 정보를 읽게 됩니다. 

<aside>
<img src="/icons/info-alternate_blue.svg" alt="/icons/info-alternate_blue.svg" width="40px" /> **인터리빙(메모리 인터리빙):** 주기억장치의 접근 속도를 빠르게 하는 기법으로 인접한 메모리 위치를 서로 다른 메모리뱅크에 둠으로써 “**동시에 여러곳을 접근**“할 수 있게 함.

</aside>

스위프트의 액터는 상태에 대한 수행작업을 한번에 하나씩만 허용하고 있고 await로 표시된 중지 가능지점에서만 코드가 방해받을 수 있기 때문에 이러한 문제점을 방지할 수 있습니다.
