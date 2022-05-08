
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





