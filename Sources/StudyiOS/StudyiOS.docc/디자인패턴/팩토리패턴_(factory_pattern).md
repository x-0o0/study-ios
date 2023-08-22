# 팩토리 패턴 (Factory pattern)

자체 구조 내에서 객체 생성 로직이 돌도록 하기 위한 패턴

## 구성

다음의 2가지 종류로 구성된다.

<img width="550" alt="Screen Shot 2022-05-12 at 7 53 27 AM" src="https://user-images.githubusercontent.com/53814741/167960332-8e947b34-8206-4850-988e-955c548bdfa0.png">

1. 공장(Factory): 객체를 생성하는 역할
2. 생산품(Product): 생성된 객체 

이 패턴은 다양한 형태로 존재한다. 간단한 팩토리를 쓰는 경우, abstract 팩토리를 쓰는 경우 등등...
암튼 목표는 하나! 자체 구조 내에서 객체 생성 로직이 돌도록 하기 위함.

## 언제 써야 하나?

객체가 쓰이는 곳에서 직접 객체를 생성하는 것이 아니라 별도의 객체 생성 로직을 분리 시키고 싶은 경우 팩토리 패턴을 씁니다.

다양한 형태의 서브 클래스 들이나, 같은 프로토콜을 준수하는 여러가지의 객체들 등 서로 관련된 어떤 생산품들의 그룹을 가질 때 유용합니다.

예를 들어, 네트워크 응답을 체크하고 이것을 구체적인 하위타입 모델로 전환시키는데에 팩토리를 사용할 수 있습니다.

또한 팩토리는 하나의 생산품 타입을 가질 때 유용하기도 합니다. 하지만 이 경우에는 생산품을 생성하기 위해 제공되어야하는 정보나 디펜던시를 필요로 합니다.

예를 들어, **채용 프로세스 답변 메일** 을 생성하는데에 팩토리 패턴을 쓸 수 있습니다. 팩토리는 지원자의 채용 단계 상황에 따라 적절한 회사메일을 생성할 수 있습니다.

## Simple Factory

```swift
struct JobApplicant {
  let name: String
  let email: String
  var status: Status
  
  enum Status {
    case new, interview, hired, rejected
  }
}

struct Email {
  let subject: String
  let body: String
  let recipientEmail: String
  let senderEmail: String
}
```
이메일 제목과 내용은 `JobApplicant`의 `status` 에 따라 달라질 것입니다.

```swift
// 팩토리를 만듭니다.
struct EmailFactory {
  // 이 값은 이메일 팩토리의 생성자에서 세팅됩니다.
  let senderEmail: String
  
  // `JobApplicant` 가지고 `Email` 생성하는 함수
  func createEmail(to recipient: JobApplicant) -> Email {
    let subject: String
    let body: String
    
    switch recipient.status {
    case .new:
      subject = "너의 지원서를 받았다"
      body = "지원해줘서 고맙수다. 며칠이내로 연락줄게."
    case .interview:
      subject = "면접 볼래?"
      body = "이력서 잘 봤고, 면접 보면 좋을 것 같은데? 답장 좀"
    case .hired:
      subject = "합격했수다"
      body = "축하축하! 이제 우리 회사의 노예가 되어보자꾸나!"
    }
    
    return Email(
      subject: subject,
      body: body,
      recipientEmail: recipient.email,
      senderEmail: senderEmail
    )
  }
}
```

**사용법**

```swift
var jaesung = JobApplicant(
  name: "재성",
  email: "jaesung@kingwang.jjang"
  status: .new
)

let factory = EmailFactory(senderEmail: "join.us@ku-ring.com")
print(factory.createEmail(to: jaesung))

jaesung.status = .interview
print(factory.createEmail(to: jaesung))

jaesung.status = .hired
print(factory.createEmail(to: jaesung))
```

이렇게 status 에 따라서 회사 이메일을 생성한다.

## 주의해야할 점

만약 생성하려는 객체가 매우 단순하다면 팩토리 없이 그냥 직접 사용하는 곳(e.g., 뷰컨트롤러)에서 생성로직을 짜도 된다. 

만약에 생성하려는 객체가 생성하기까지 여러 단계가 존재한다면, 그때는 빌더 패턴을 쓰는게 좋습니다.

즉, 객체를 만들때 늘 팩토리가 필요한 건 아니라는 것입니다.

## 이 외의 사용예시

```swift
class ViewController: UIViewController {
  let factory = AnnotationFactory()
  
  private func addAnnotations() {
    for business in businesses {
      // 비즈니스 별 지도 뷰모델 생성
      guard let viewModel = factory.createBusinessMapViewModel(for: business) else { continue }
      mapView.addAnnotation(viewModel)
    }
  }
}
```

## 요점

- 팩토리의 목표는 자체 구조 안에서 객체의 생성 로직을 고립시키기 위함이다. 
  즉, 외부에서(쓰는 곳에서) 매번 상황에 따른 객체를 생성하는 로직을 짜는 것이 아니라 한 곳에서 관리. 그리고 그것이 팩토리
- 팩토리는 만약 관련된 생산품 그룹을 갖고 있거나, 추가 정보가 들어오기 전(콜백, 유저입력)까지 객체를 생성하지 못하는 경우 매우 유용하다.
- 팩토리 함수는 추상화 계층을 추가하여 코드 중복을 줄입니다.




