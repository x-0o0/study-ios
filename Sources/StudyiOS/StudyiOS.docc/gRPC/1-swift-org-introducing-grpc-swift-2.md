---
날짜: 25.08.03
주제: Swift.org 에 소개된 grpc-swift-2 를 읽고 정리한 것
링크: https://www.swift.org/blog/grpc-swift-2
---

# Swift.org - Introducing gRPC Swift 2

gRPC는 클라이언트와 서버간에 서로 다른 언어를 사용해도 프로토콜 버퍼 (protobufs)를 통해 API 와 메세지를 정의하여 자유롭게 주고받을 수 있고,
서비스 계약에 사용되는 `.proto`는 바이너리 시리얼라이징을 통해 JSON 과 같은 다른 표준들보다 더 작고 가볍다.

async/await 같은 오늘날의 Swift API 를 대응할 수 있도록 grpc-swift-2 가 새롭게 소개되었다. (25년 2월)
이전에는 2018년에 [SwiftNIO](https://github.com/apple/swift-nio) 의 이벤트 기반 Concurrency 모델을 사용하여 grpc-swift 를 제공해왔다.

## 주요 특징
- 현대적, 유연함, 쉬운 API, _관용적 코드(Idiomatic code)_[^1]
- 리눅스, 애플 플랫폼 지원
- SwiftNIO 위에 빌드된 고성능 HTTP/2 를 포함하는 Pluggable Transport[^2], 테스트에 훌륭한 in-process transport[^3]
- 클라이언트 사이드의 로딩 밸런싱 (부하 분산), 플러그인 방식의 Name Resolution[^4] 매커니즘, 자동 재시도와 같은 똑똑한 클라이언트 기능들
- 인증, 로깅, 메트릭(성능/상태 수치 데이터) 수집과 같은 서비스간 공통 로직을 구현할 수 있게 하는 유연한 인터센터 레이어(요청/응답 흐름 중간에 끼어드는 것을 의미)

## Hello, swift.org!

`.proto` 파일 작성방법

```protobuf
syntax = "proto3";

service GreetingService {
  // 개인 맞춤 환영 메세지 반환
  rpc SayHello(SayHelloRequest) returns (SayHelloResponse);
}

message SayHelloRequest {
  // 환영할 사람의 이름
  string name = 1;
}

message SayHelloResponse {
  // 개인 맞춤 환영 메세지
  string message = 1;
}
```
gRPC는 다음을 생성하도록 설정됩니다.
- 비지니스 로직을 구현할 수 있는 서비스 코드
- 서비스에 대한 요청을 만들 수 있는 클라이언트 코드

메세지에 대한 코드는 SwiftProtobug 에 의해 생성되고, gRPC 코드와 함께 사용됨.

## 서비스 코드
생성된 코드는 Swift 프로토콜을 포함하며 서비스 정의에 있는 `rpc` 키워드 당 하나의 메서드를 가짐.
이 서비스 프로토콜을 구현하여 비지니스 로직을 구현할 수 있음. 아래 예시의 `SimpleServiceProtocol` 참고.
더 유연하게 사용하고 싶다면, `ServiceProtocol` 이나 `StreamingServiceProtocol` 를 사용하면 됨. (단, 간결함을 포기해야함)

서비스를 시작하려면, transport를 사용하도록 설정된 서버와 서비스 인스턴스 생성 필요.

```swift
import GRPCCore
import GRPCNIOTransportHTTP2
```
```swift
struct Greeter: GreetingService.SimpleServiceProtocol {
  func sayHello(
    request: SayHelloRequest,
    context: ServerContext
  ) async throws -> SayHelloResponse {
    return SayHelloResponse.with {
      $0.message = "Hello, \(request.name)!"
    }
  }
}
```
```swift
@main
struct GreeterServer {
  static func main() async throws {
    // HTTP/2 기반의 SwiftNIO를 사용하여 128.0.0.1:8080 를 바라보는 Plaintext 서버 생성
    let server = GRPCServer(
      transport: .http2NIOPosix(
        address: .ipv4(host: "127.0.0.1", port: 8080),
        transportSecurity: .plaintext
      ),
      services: [Greeter()]
    )

    // 중단되지 않게 무기한으로 서비스 시작
    try await server.serve()
  }
}
```

## 클라이언트 코드

gRPC 는 관용적 클라이언트를 생성함. (관용적 의미는 아래 주석 [^1] 참고)
이를 사용하려면, raw한 클라이언트를 생성하고, 서비스에 특화되어 있는 클라이언트(gRPC가 생성했던)로 감쌀 것.
type-safe 방식을 제공하기 때문에 서비스에 붙이기 쉽다.

```swift
import GRPCCore
import GRPCNIOTransportHTTP2

@main
struct SayHello {
  static func main() async throws {
    // HTTP/2 기반 SwiftNIO 를 사용하여
    // 128.0.0.1:8080 를 바라보는 서비스에 연결하는 Plaintext 클라이언트 생성
    try await withGRPCClient(
      transport: .http2NIOPosix(
        target: .dns(host: "127.0.0.1", port: 8080),
        transportSecurity: .plaintext
      )
    ) { client in
      let greeter = GreetingService.Client(wrapping: client)
      let greeting = try await greeter.sayHello(.with { $0.name = "swift.org" })
      print(greeting.message)
    }
  }
}
```

## 패키지 생태계 (Ecosystem)

gRPC Swift2 는 여러 패키지들의 컬렉션으로 배포 되어 있어서 본인에게 가장 알맞는 구성요소를 선택할 수 있다.
이러한 기능들은 아래 패키지들에서 제공됩니다.
- grpc/grpc-swift-2 는 런타임 abstration과 타입들을 제공합니다.
- grpc/grpc-swift-nio-transport 는 HTTP/2를 사용하여 클라이언트와 서버를 구현하며 SwiftNIO 위에서 동작합니다.
- grpc/grpc-swift-protobuf 는 `.proto` 파일에 정의되어 서비스를 위한 코드 생성하는 `SwiftProtobuf` 를 통합시킵니다.
- grpc/grpc-swift-extras 는 아래의 일반적인 gRPC 확장 기능들을 제공합니다.
  - 리플렉션(Reflection) 및 헬스(Health 서비스)
  - Swift Service Lifecycle과의 통합
  - OpenTelemetry 를 사용해 RPC를 추척할 수 있는 인터셉터

## 다음 단계

[튜토리얼](https://swiftpackageindex.com/grpc/grpc-swift-2/documentation)과 [grpc/grpc-swift-2](https://github.com/grpc/grpc-swift-2) 에 있는 예제 프로젝트 확인


## Footnote
[^1]: SW에서 관용적 코드(Idiomatic code)이란 특정 프로그래밍 언어에서의 자연스러운 표현 방식으로 작성된 코드를 의미. ([출처](https://www.quora.com/What-do-you-mean-by-idiomatic-code-in-computer-programming))
[^2]: Pluggable Transport 는 특정 프로토콜/기술을 사용하여 네트워크 통신 시스템에서 필요에 따라 다른 프로토콜/기술로 쉽게 교체할 수 있도록 설계된 구성요소를 의미. 마치 플러그 꽂듯이 필요에 따라 기능을 추가/변경할 수 있는 전송 매커니즘. [출처](https://www.pluggabletransports.info/about/#:~:text=Pluggable%20Transports%20have%20been%20created%20to%20help,Pluggable%20Transports%20are%20needed%20and%20what%20they)
[^3]: In-process transport 는 외부 시스템이나 네트워크 프로토콜 없이도 동일 프로세스/스레드와 같이 하나의 프로그램 내 서로 다른 부분에서 통신이나 데이터 교환이 발생하는 매커니즘을 말합니다. 효율성, 속도적 측면, 특히 테스트 같은 시나리오에 주로 사용됩니다. [참고](https://www.bwplotka.dev/2025/go-grpc-inprocess-iter/)
[^4]: Name Resolution (이름 해석) 이란 도메인 이름이나 서비스 이름을 실제 네트워크 주소(IP 등)으로 바꾸는 것. 예를 들자면, `example.com` -> `192.168.1.10`. Pluggable Name Resolution은 특정 Name Resolution에 종속되지 않고 환경에 따라 쉽게 교체가 가능하여 마이크로서비스 구조에서 유연한 확장성을 가질 수 있도록 함. (by GPT)
