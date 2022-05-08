
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

`UIKit.UILabel` 같은 거라고 생각하면 된다.

### Modifiers

`Text` 를 추가했다면 다음 스텝은 모양과 색 등 스타일 요소를 변경하는 것이다.

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

