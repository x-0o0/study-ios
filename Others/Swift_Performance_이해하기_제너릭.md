```swift
struct Pair {
    var first: Drawable
    var second: Drawable
    
    init(_ f: Drawable, _ s: Drawable) {
        first = f
        second = s
    }
}

let pairOfLines = Pair(Line(), Line())
let pairOfPoint = Pair(Point(), Point())
```

### Protocol

다이나믹 폴리모피즘(다형성) (Dynamic Polymorphism)

다이나믹 디스패치 방식을 사용하지만 클래스의 vtable이 아닌 프로토콜 전용의 witness table(PWT)를 사용.

```swift
protocol Drawable {
    func draw()
}

struct Point: Drawable {
    func draw() { ... }
}

struct Line: Drawable {
    func draw() { ... }
}
```

| PointDrawable | LineDrawable |
| ------------- | ------------ |
| draw:         | draw:        |
| ...           | ...          |




### Generic

스태틱 폴리모피즘 (Static polymorphism)

즉, 런타임동안 타입이 바뀔 수 없다.

```swift
struct Pair<T: Drawable> {
    var first: T
    var second: T
    
    init(_ f: T, _ s: T) {
        first = f
        second = s
    }
}

var pair = Pair(Line(), Line())
pair.first = Point // ❌
```

| Pair |
| ---- |
| ==Line== |
| x1: 0.0 |
| y1: 0.0 |
| x2: 0.0 |
| y2: 3.0 |
| ==Line== |
| x1: 1.0 |
| y1: 2.0 |
| x2: 0.0 |
| y2: 3.0 |

