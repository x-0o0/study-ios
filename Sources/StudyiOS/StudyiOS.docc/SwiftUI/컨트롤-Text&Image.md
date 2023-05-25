
# 뷰 생성하기

```swift
struct WelcomeView: View {
  var body: some View {
    Text("환영합니다.")
  }
}
```

## Preview 생성하기

```swift
struct WelcomeView_Previews: PreviewProvider {
  static var previews: some View {
    WelcomeView()
  }
}
```

## Text (텍스트)

```swift
Text("환영합니다.")
```

`UIKit.UILabel` 같은 거라고 생각하면 됩니다.

### Modifiers

`Text` 를 추가했다면 다음 스텝은 모양과 색 등 스타일 요소를 변경하는 것입니다.

modifier 은 뷰의 복사본을 생성하는 함수 입니다.

![image](https://user-images.githubusercontent.com/53814741/167291781-f1fc5c8d-363f-4b1c-bdc2-bd3697f1c695.png)

```swift
Text("환영합니다.")
  .font(.system(size: 60).bold()) // 폰트 관련 요소는 한번에 적용가능
  .foregroundColor(.red)
  .multilineTextAlignment(.center)
  .lineLimit(1)
```

![image](https://user-images.githubusercontent.com/53814741/167292074-51660e92-e776-4e42-b272-e21faf4608ac.png)

![image](https://user-images.githubusercontent.com/53814741/167292078-2065f05f-76f0-41ff-9843-15641fa4fe0a.png)

Command + click 해서 뷰 인스펙터를 띄울 수 있습니다. 프리뷰에서 뷰요소를 클릭하여 Xcode 우측에 인스펙터를 띄울 수도 있습니다(Option + Command + 4).

<details>
  
  ![image](https://user-images.githubusercontent.com/53814741/167292170-a8a488bd-cc26-4bc2-b6fd-284a8318de42.png)

  ![image](https://user-images.githubusercontent.com/53814741/167292206-9905c473-0c86-43ba-b04d-ad7b89521f40.png)

</details>

**modifiers 가 효율적인가?**

모든 modifier들은 새 뷰를 리턴하기 때문에 효율성에 대해 의구심이 생길 것입니다. SwiftUI는 modifier를 호출할 때마다 새 뷰에 기존 뷰를 임베드 합니다.

마치 러시아 인형처럼 생각할 수 있습니다.

겉으로 봤을 때는 리소스 낭비처럼 보일 것입니다. 하지만 SwiftUI는 그렇게 멍청하지 않습니다 호호. 실제 뷰 렌더링을 위해 효율적인 데이터 구조로 관리합니다.
그러니 돈 워리하고 마음껏 쓰세요.

**modifier 순서**

modifier 호출 순서는 과연 중요할까요~? 정답은 "그렇다" 입니다. 보통은 크게 상관이 없지만 중요한 경우가 있습니다.

```swift
Text("hihi")
  .background(Color.red)
  .padding()
  
Text("hihi")
  .padding()
  .background(Color.red)
```

이 둘은 아래 이미지 처럼 모습이 다릅니다.

![image](https://user-images.githubusercontent.com/53814741/167292605-c7afe055-2b90-4eb2-a06a-b4bd0aa57691.png)

### 자주 쓰는 `modifier` 들

```swift
Text("Welcome to Kuchi")
  .font(.system(size: 30))
  .bold()
  .foregroundColor(.red)
  .lineLimit(2)
  .multilineTextAlignment(.center)
```

## Image (이미지)

```swift
var body: some View {
  Image(systemName: "table")
  
  Image("local.image")
}
```

### 사이즈 변경

**`resizable` modifier**
- Parameter
  - `.tile`
  - `.stretch` (default with no insets)

`resizable`을 사용하지 않으면 이미지는 본래의 사이즈를 유지합니다. 

때문에 크기 변경하는 modifier를 적용해도 이미지를 갖는 뷰의 크기만 변경할 뿐 이미지 자체의 사이즈를 변경하진 않습니다.

```swift
Image(systemName: "table")
  .frame(width: 30, height: 30)
```
<img width="194" alt="Screen Shot 2022-05-16 at 9 18 54 AM" src="https://user-images.githubusercontent.com/53814741/168500833-4cbda43c-b28f-4556-be30-a5f15dcb17cd.png">

```swift
Image(systemName: "table")
  .resizable()
  .frame(width: 30, height: 30)
```

<img width="77" alt="Screen Shot 2022-05-16 at 9 31 08 AM" src="https://user-images.githubusercontent.com/53814741/168501423-f4f4b7fc-d1ca-446e-8956-df8cc7d5ea6a.png">

그외에 필요한 `modifier` 은 `.frame` 이후에 호출하면 됨다.

```swift
.cornerRadius(30 / 2) // 불필요
.overlay(Circle().stroke(Color.gray, lineWidth: 1))
.background(Color(white: 0.9))
.clipShape(Circle())
.foregroundColor(.red)
```

<img width="682" alt="Screen Shot 2022-05-16 at 9 38 05 AM" src="https://user-images.githubusercontent.com/53814741/168501808-072b8655-e7ef-4f29-a5d0-f7f1a48a9ce0.png">

```swift
Image("some.image")
  .resizable()
  // 이미지를 최대크기로. 본래의 비율로 뷰에 "볼 수 있는 영역 내에" 최대 크기로 채워짐
  .scaledToFit()
  // 비율을 설정하고, contentMode를 설정할 수 있다. 화면 밖을 벗어나는 것과 상관 없이 할 수 있는 최대 크기로 커짐
  .aspectRatio(contentMode: .fill)
  // safe area 를 무시하고 확장. 파라미터로 `.top, .bottom, .leading, .trailing, .vertical, .horizontal` 사용 가능
  .edgesIgnoringSafeArea(.all)
  // 색상의 saturation 설정
  .saturation(0.5)
  // 블러효과
  .blur(radius: 5)
  // 투명도 설정
  .opacity(0.88)
```

<img width="710" alt="Screen Shot 2022-05-16 at 9 45 29 AM" src="https://user-images.githubusercontent.com/53814741/168502219-2c7973ab-8c5c-4a5a-b01a-7477d2c7534b.png">

## Stack views

```swift
HStack {
  Image(..)
    ...
  
  Text(...)
    ...
}
```

```swift
ZStack(alignment: .bottom) { ... }

VStack(spacing: 8) { ... }

HStack { }
```

### 글자 분리

```swift
VStack {
  Text("Welcome to")
    .font(.system(size: 30))
    .bold()
  Text("Kuchi")
    .font(.largeTitle) // 애플 권장
    .bold()
}
.foregroundColor(.red)
.multilineTextAlignment(.leading)
.lineLimit(2)
```

이렇게 하면 각각의 Text 내에서 `multilineTextAlignment` 를 적용하고 동시에 `Vstack` 은 가운데 정렬이라 
텍스트 정렬이 안된것으로 보일 수 있다. `multilineTextAlignment` 대신에 `Vstack(alignment:)` 를 사용해보면 원하는 결과를 얻을 수 있다.

```swift
VStack(alignment: .leading) { 
  ...
}
...
```

### 마크다운

```swift
Text("**Welcome**")
```
근데 아래의 경우는 안되었음

```swift
var string = "Welcome to Sendbird"
// Sendbird -> **Sendbird** 로 변환
print(string) // "Welcome to **Sendbird**"

Text(string) // 마크다운 적용 안되고 그냥 string 출력결과와 동일한 문장을 보여줌
```

> **추가설명** `AttributedString` 도 동일 적용가능

## Label

```swift
Label("Welcome", systemImage: "hand.wave")

Label {
  VStack { ... }
} icon: {
  Image(...)
}
```

<img width="259" alt="Screen Shot 2022-05-16 at 10 10 46 AM" src="https://user-images.githubusercontent.com/53814741/168503658-991c14da-d719-4516-af2a-38b0ac44bea2.png">

### 커스텀 스타일

```swift
protocol LabelStyle {
  func makeBody(configuration: Self.Configuration) -> Self.Body
}
```

`LabelStyle` 프로토콜을 채택하여 새 커스텀 스타일을 만들 수 있다.

```swift
import SwiftUI

struct HorizontallyAlignedLabelStyle: LabelStyle {
  func makeBody(configuration: Configuration) -> some View {
    HStack {
      configuration.icon
      configuration.title
    }
  }
}
```

**사용**

```swift
Label(...)
  .labelStyle(HorizontallyAlignedLabelStyle())
```

## 요약
- `Text`, `Image` 는 각각 텍스트, 이미지를 보여주고 설정하는데에 사용
- `Label` 로 텍스트랑 이미지를 한번에 보여줄 수 있다.
- modifier 들로 뷰의 모습을 변경할 수 있다. 보통은 순서를 신경쓰지 않아도 되지만 순서에 따라 결과가 달라질 수도 있는 걸 잊지말아야한다.
- `VStack, HStack, ZStack` 으로 뷰들의 레이아웃을 설정할 수 있다.
