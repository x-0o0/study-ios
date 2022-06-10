# What's new in SwiftUI

## 개요

이번 스유에 새롭게 등장한 내용들을 요약하자면 아래와 같다.

- ShapeStyle Extensions
- Control styles
- Charts
- MenuBarExtra
- Layout
- Table
- NavigationStack & NavigationSplitView & Window
- ResizableSheets 😍
- Text Transitions
- Toolbar
- Sharing & Transferable 😍

😍 은 내가 필요로 했던 것을 표시했다.

## Swift Charts

state-driven chart.

### 바

그냥 `List` 처럼 쓰면 된다.

```swift
Chat(tasks) { task in
    BarMark(
        x: .value(..., ..., unit: ...)
        y: .value(..., ...)
    )
}
```

<img width="139" alt="Screen Shot 2022-06-10 at 10 50 05 AM" src="https://user-images.githubusercontent.com/53814741/172974052-8faa3239-fcf5-45c6-9b9b-833a07d88cd1.png">


```swift
Chat(tasks) { task in
    LineMark(
        x: .value("Date", $0.date, unit: .day),
        y: .value("Tasks Remaining", $0.remainingCount)
    )
    .foregroundStyle(by: .value("Category", $0.category))
    .symbol(by: .value("Category", $0.category))  //
    .annotation(position: .leading) {   // 선 시작점(leading)에 아이콘 생성
        Text("\(task.category.emoji)")
    }
}
```

**foregroundStyle(by:)**

category에 따라 그룹 지은 모습

<img width="142" alt="Screen Shot 2022-06-10 at 10 50 45 AM" src="https://user-images.githubusercontent.com/53814741/172974128-775a7df9-9686-4be1-a686-9919ab058760.png">

**symbol(by:)**

라인에 점표시들이 추가됨.

<img width="138" alt="Screen Shot 2022-06-10 at 10 51 42 AM" src="https://user-images.githubusercontent.com/53814741/172974256-51e3f09e-acb7-4f66-aab7-60b96a0bafb2.png">

**annotation(position:content:)

<img width="140" alt="Screen Shot 2022-06-10 at 10 52 37 AM" src="https://user-images.githubusercontent.com/53814741/172974338-41dc8511-cbb9-4a54-80b1-baf05ede817e.png">

### RuleMark

```swift
Chart {
    ForEach(partyTasksRemaining) { task in
        LineMark(...)
    }
    
    RuleMark(y: .value("Value", 5))
        .foregroundStyle(.red)
        .lineStyle(StrokeStyle(lineWidth: 2.0, dash: [4, 5]))
        .annotation(position: .top, alignment: .trailing) {
            VStack(alignment: .trailing) {
                Text("Today's Goal")
                Text("Status: ✔︎")
            }
            .font(.caption)
            .foregroundColor(.gray)
            .padding(.trailing, 2)
        }
}
```

<img width="139" alt="Screen Shot 2022-06-10 at 10 52 57 AM" src="https://user-images.githubusercontent.com/53814741/172974364-0f1d3c91-662d-4201-a5ab-f1f1b54e03d2.png">

### 관련 세션

- Hello Swift Charts
- Swift Charts: Raised the bar

## 네비게이션, 윈도우 기술

<img width="741" alt="Screen Shot 2022-06-10 at 10 57 22 AM" src="https://user-images.githubusercontent.com/53814741/172974820-17be0540-ce0b-4df0-b302-3121933022ef.png">

### Stacks

<img width="264" alt="Screen Shot 2022-06-10 at 10 59 57 AM" src="https://user-images.githubusercontent.com/53814741/172975163-3118c67b-467e-40b1-a014-131d52151259.png">

```swift
NavigationStack { // push & pop 스타일 네비게이션 지원
  List(foodItems) { item in
    NavigationLink {
      FoodDetailView(item: item)
    } label: {
      Label(item.title, image: item.iconName)
    }
  }
  .navigationTitle("Party food")
  .navigationDestination(for: FoodItem.self) { item in
    FoodDetailView(item: item)
  }
}
```

#### `navigationDestination(for:content)`

네비게이션의 상태를 관리

#### `NavigationLink(value:content:)`

```swift
NavigationLink(value: item) { // value는 destination 을 나타냄
  Label(item.title, image: item.iconName)
}
```

네비게이션 링크를 탭하면, destination을 찾기 위해 알맞는 `value` 타입을 찾아서 stack에 푸시한다.

#### `NavigationStack(path:)`

```swift
@State private var selectedFoodItems: [FoodItem] = []

var body: some View {
  NavigationStack(path: $selectedFoodItems) { 
    ... 
    .navigationDestination(for: FoodItem.self) { item in
      FoodDetailView(item: item, path: $selectedFoodItems) // 새 destination 으로 갈때마다 selectedFoodItems 가 업데이트
    }
  }
}
```

```swift
// FoodDetailView.body

Button("첫 번째 아이템으로 되돌아가기") {
  // 첫번째 아이템을 제외하고 전부 제거
  selectedFoodItems.removeSubrange(1...)
}
```

### NavigationSplitView

<img width="304" alt="Screen Shot 2022-06-10 at 11 13 22 AM" src="https://user-images.githubusercontent.com/53814741/172976662-0b0f6c17-8af2-4515-bdfa-7562582704f8.png">

```swift
@State private var selectedTask: PartyTask?
```

```swift
NavigationSplitView { // 2열, 3열 레이아웃을 선언할 수 있음
  List(PartyTask.allCases, selection: $selectedTask) { // 사이드바 영역
    NavigationLink(value: $0) {
      TaskLabel(task: $0)
    }
  } detail: { // 화면 우측 디테일 영역
    switch selctedTask {
    case .food: FoodOverview()
    case .music: MusicOverview()
    // ...
    }
  }
}
```

`NavigationSplitView` 와 `NavigationStack` 을 같이 써도 좋다. 아래 스샷의 detail 뷰는 `NavigationStack` 를 쓰고 있다.

<img width="410" alt="Screen Shot 2022-06-10 at 11 16 19 AM" src="https://user-images.githubusercontent.com/53814741/172976975-224cd075-c0e7-49cd-a7f6-8a30681a7b1a.png">

### 네비게이션 관련 세션

- The SwiftUI cookbook for navigation

### Scene 

```swift
WindowGroup("Party Planner") {
  TaskViewer()
}

// New
Window("Party Budget", id: "budget") {
  BudgetView()
}
.keyboardShortcut("0")
.defaultPosition(.topLeading)
.defaultSize(width: 220, height: 250)
```

<img width="174" alt="Screen Shot 2022-06-10 at 11 19 50 AM" src="https://user-images.githubusercontent.com/53814741/172977322-2c68e436-f8f1-483d-9e13-8191faf985b8.png">

<img width="325" alt="Screen Shot 2022-06-10 at 11 20 18 AM" src="https://user-images.githubusercontent.com/53814741/172977362-175c5b79-9b2d-425d-8374-b649433b397d.png">

#### `toolbar` and `\.openWindow`

```swift
// DetailView
@Environment(\.openWindow) var openWindow
```

```swift
// DetailView
.toolbar {
  Button {
    openWindow(id: "budget") // Window(_:id:content:)의 id를 찾는다
  } label: {
    Imㅁge(systemName: "dollarsign")
  }
}
```

#### 리사이징 가능한 시트

<img width="251" alt="Screen Shot 2022-06-10 at 11 27 06 AM" src="https://user-images.githubusercontent.com/53814741/172978068-94079bd6-4bc2-4056-8d1b-771ddfd27f1c.png">

```swift
// Resizable sheets

.sheet(isPresented: $showBudget) {
  BudgetView()
    .presentationDetents([.height(250), .medium])
    .presentationDragIndicator(.visible)
}
```

#### MenuBarExtra

<img width="277" alt="Screen Shot 2022-06-10 at 11 28 31 AM" src="https://user-images.githubusercontent.com/53814741/172978232-fde05dd4-bc90-48ce-b942-0a2b20823496.png">

```swift
// App.body
MenuBarExtra("Bulletin Board", systemImage: "quote.bubble") {
  BulletinBoardView()
}
.menuBarExtraStyle(.window)
```

## 컨트롤 고급편

## 공유하기

## 그래픽과 레이아웃
