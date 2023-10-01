# 메멘토 패턴

객체를 저장하고 복구할 수 있도록 하는 패턴.

## 들어가기 앞서 메멘토란?

The object that you keep to remember 무시기. 즉 기억하기 위한 물건, 증표

## 구성 

> 다음 내용은 위키백과를 참고한 내용입니다.

세 가지로 구성: 오리지네이터(Orignator), 케어테이커(Caretaker), 메멘토(Memento)

<img width="712" alt="Screen Shot 2022-04-28 at 7 40 10 PM" src="https://user-images.githubusercontent.com/53814741/165735183-74aa531d-67b3-4706-bef0-f8b829386355.png">

### 오리지네이터

- 저장되거나 복구할 객체
- State를 갖는 객체

다음과 같은 책임을 갖는다.
- 메멘토에 내부 State를 저장
- 메멘토로부터 이전 State 복구

### 메멘토

저장된 State를 나타냄

### 케어테이커 (Caretaker)

오리지네이터에게 저장을 요청하고 응답으로 메멘토를 받는다.
이 메멘토를 보존해야하는 책임이 있고
나중에 오리지네이터에 다시 돌려주어 상태를 복구 시킨다.

1. 먼저 오리지네이터에게 메멘토 객체를 요청. (내부State 저장)
2. 그 뒤 일렬의 명령을 수행
3. 원래 상태로 되돌리기 위해 메멘토 객체를 오리지네이터에 리턴 (이전 State 복구)

## 장점

잘 디자인 된 패턴은 캡슐화된 형태를 갖기 때문에 외부에서 접근이 어려우나 
메멘토 패턴을 쓰면 캡슐화를 훼손하지 않고 오리지네이터의 State를 변경하고 복구할 수 있다.


## 언제 써야 하는가?

어떤 객체 상태를 저장했다가 복구해야하는 경우에 적합.

e.g., 게임을 저장하는 시스템 - 오리지네이터가 게임상태(레벨, 체력 등), 메멘토가 저장된 데이터, 케어테이커는 게임시스템

<details>
  
  **케어테이커와 메멘토**
  
  ![image](https://user-images.githubusercontent.com/53814741/165740262-d2d12aaa-02f1-4691-9a70-a6020b0d73c8.png)

  **오리지네이터**
  
  ![image](https://user-images.githubusercontent.com/53814741/165740394-6a147187-e06b-4b03-96a5-19dd2002c7b3.png)
</details>

메멘토를 배열로 보관할 수 있음. 이전 상태들을 단계별로 저장. IDE에서 뒤로가기, 앞으로가기 등에 쓰기 좋음.

> 쿠링에서 읽은 공지에 대한처리, 앱 실행 시각에 대한 처리 등 저장하는 로직에 알게모르게 쓰고 있었음. 좀 더 메멘토 패턴에 맞춰 잘 디자인화 하면 좋을 것 같음.

## 시퀀스 다이어그램
<img width="562" alt="Screen Shot 2022-04-28 at 7 34 50 PM" src="https://user-images.githubusercontent.com/53814741/165734308-c2f53ea4-70a1-4b99-adcd-3293119e9851.png">

## 예시

### iOS

iOS 에서는 보통 오리지네이터의 상태를 메멘토로 인코딩 하기 위해 `Encoder` 를 사용하고
메멘토를 디코딩하여 오리지네이터로 되돌리기 위해 `Decoder` 를 사용한다.

이는 오리지네이터에 대해 인코딩 / 디코딩 로직을 재사용할 수 있도록 한다.
예를 들어 `JSONEncoder` 와 `JSONDecoder` 는 JSON 데이터 와 객체 사이에서 인코딩과 디코딩을 해준다.
(🧐무슨말인지 사실 모름)

**오리지네이터**

```swift
import Foundation
// MARK: - Originator
public class Game: Codable {

  // 상태 클래스
  public class State: Codable {
    // 남은 시도 
    public var attemptsRemaining: Int = 3
    // 레벨
    public var level: Int = 1
    // 점수
    public var attemptsRemaining: Int = 3
    public var score: Int = 0
  }
  
  // 상태 값
  public var state = State()
  
  // 보스몹 잡아서 점수 획득
  public func rackUpMassivePoints() {
    state.score += 9002
  }
  
  // 몬스터가 플레이어를 잡아먹음
  public func monstersEatPlayer() {
    state.attemptsRemaining -= 1
  }
}
```

**메멘토**

```swift
// MARK: - Memento
typealias GameMemento = Data

// Encoder -> 저장
// Decoder -> 복구
```

**케어테이커**

```swift
// MARK: - CareTaker
public class GameSystem {
  // 데이터(메멘토)로부터 게임(오리지네이터)를 복구하는데 쓰임
  private let decoder = JSONDecoder()
  // 게임(오리지네이터)를 데이터(메멘토)로 저장하는데 쓰임
  private let encoder = JSONEncoder()
  // 로컬 데이터 저장소
  private let userDefaults = UserDefaults.standard
  
  // 저장 로직을 캡슐화함. 로컬데이터 저장소에 오리지네이터를 메멘토 객체로 인코딩 후 저장
  public func save(_ game: Game, title: String) throws {
    let data = try encoder.encode(game)
    userDefaults.set(data, forKey: title)
  }

  // 로딩 로직을 캡슐화함. 메멘토 데이터를 오리지네이터로 복구해서 리턴
  public func load(title: String) throws -> Game {
    guard let data = userDefaults.data(forKey: title),
          let game = try? decoder.decode(Game.self, from: data) else {
          // 에러 리턴
          throw Error.gameNotFound
      }
    return game 
  }
  
  // 에러 정의
  public enum Error: String, Swift.Error {
    case gameNotFound
  }
}
```

**사용예시**

```swift
var game = Game()
game.monstersEatPlayer()
game.rackUpMassivePoints()

 // Save Game
let gameSystem = GameSystem()
try gameSystem.save(game, title: "Best Game Ever")

 // New Game
game = Game()
print("New Game Score: \(game.state.score)") // New Game Score: 0

 // Load Game
game = try! gameSystem.load(title: "Best Game Ever")
print("Loaded Game Score: \(game.state.score)") // Loaded Game Score: 9002
```

### 파이썬

**메멘토**

<details>

```python
class Memento(object):
    def __init__(self, state):
        self._state = state
        
    def get_saved_state(self):
        return self._state
```
</details>

**오리지네이터**

<details>

```python
class Originator(object):
    _state = ""
    
    def set(self, state):
        print("Originator: Setting state to", state)
        self._state = state
        
    def save_to_memento(self):
        print("Originator: Saving to Memento.")
        return Memento(self._state)
        
    def restore_from_memento(self, memento):
        self._state = memento.get_saved_state()
        print("Originator: State after restoring from Memento:", self._state)
```

</details>

**케어테이커**

<details>
  
```python
saved_states = []
originator = Originator()
originator.set("State1")

originator.set("State2")
saved_states.append(originator.save_to_memento())

originator.set("State3")
saved_states.append(originator.save_to_memento())

originator.set("State4")
originator.restore_from_memento(saved_states[0])
```

</details>


