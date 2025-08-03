---
날짜: 25.08.03
주제: gRPC Swft Tutorials 문서 보면서 첫 단계 시작해보기
링크: https://swiftpackageindex.com/grpc/grpc-swift-2/2.1.0/tutorials/grpccore/hello-world
---

## 사전 작업
### protoc 설치 
[참고](https://swiftpackageindex.com/grpc/grpc-swift-protobuf/2.0.0/documentation/grpcprotobuf/installing-protoc)

```bash
$ brew install protobuf

# starts download...
Emacs Lisp files have been installed to:
  /opt/homebrew/share/emacs/site-lisp/protobuf
```

### 프로젝트 다운 받기
https://github.com/grpc/grpc-swift-2 레포지토리 다운로드(또는 클론) 후 `Examples/hello-world` 경로로 이동할 것.

> **주의:** 문서에는 grpc-swift 레포의 2.0.0 브랜치를 클론하라고 되어있지만 grpc-swift-2의 main 브랜치를 클론해야함

```bash
$ cd grpc-swift-2-main/
$ cd Examples/hello-world/
```

## Hello, Word!

### gRPC 앱 실행하기

예제 프로그램을 실행해보자~

아래의 SWift 명령어 앞에 모두 `PROTOC_PATH=$(which protoc)` 가 붙어 있는데, 이는 빌드 시스템에 `protoc` 위치를 알려주어 stub 코드를 자동 생성할 수 있게 하기 위함.

1. [서버시작] 터미널에서 `PROTOC_PATH=$(which protoc) swift run hello-world serve` 명령어를 실행시켜 서버를 시작합니다. 기본적으로 31415 포트를 바라봅니다.

> **참고**: 정말 그냥 which protoc 이라고 그대로 입력하면 됨.

명령어를 실행하면 아래와 같이 뜬다.
```bash
Build of product 'hello-world' complete! (155.87s)
Greeter listening on [ipv4]127.0.0.1:31415 # 31415 포트 바라보는 중!
```

2. [클라이언트 생성] 다른 터미널을 열고 `PROTOC_PATH=$(which protoc) swift run hello-world greet` 를 실행하여 클라이언트 생성하고 시작했던 서버에 연결하여 요청을 보내고 응답을 프린트 합니다.

```bash
# 새 터미널
$ cd grpc-swift-2-main/Examples/hello-world/
$ PROTOC_PATH=$(which protoc) swift run hello-world greet
```

```bash
# 명령어 실행 결과
Build of product 'hello-world' complete! (8.54s)
Hello, stranger
```

3. gRPC Swift 를 사용하여 클라이언트-서버를 구동시켜보았습니다. 이제 구동중인 프로세스를 종료시켜도 됩니다.

```
control˄ + C
```
