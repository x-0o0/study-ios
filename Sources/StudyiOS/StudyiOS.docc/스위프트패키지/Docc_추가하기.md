# DocC 추가하기

## 개요

### 1. Docc 폴더 추가

Product > Build Documentation

### 2. Docc 플러그인 추가

```swift
// Package.swift
dependencies: [
  .package(url: "https://github.com/apple/swift-docc-plugin", from: "1.0.0"),
],
```

### 3. shell script 생성

```shell
# build_docc.sh

# Setup constants
PROJECT_NAME="KuringCommons" # 스킴이름 넣으면 됨
HOSTING_BASE_PATH="kuring-ios-commons" # 레포 이름 넣으면 됨
DOCS_DIR="docs" # docs 폴더에 저장해야 GitHub Pages에 띄울 수 있음

echo "🧹 빌드를 준비합니다..."
rm -rf "./${DOCS_DIR}"

mkdir -p "./${DOCS_DIR}"

echo "🔨 DOCC 문서 빌드를 시작합니다..."
xcodebuild docbuild -scheme "${PROJECT_NAME}" \
    -destination generic/platform=iOS \
    OTHER_DOCC_FLAGS="--output-path ${DOCS_DIR} --transform-for-static-hosting --hosting-base-path ${HOSTING_BASE_PATH}"

echo "🎉 성공적으로 완료하였습니다!"

```

### 4. 스크립트 빌드

```bash
$ chmod +x build_docs.sh
$ ./build_docs.sh
```

docs 폴더에 파일들 잘 들어갔는지 확인

### 5. 깃헙에 올리고 깃헙페이지 확인

<img width="781" alt="Screen Shot 2022-12-27 at 11 55 48 PM" src="https://user-images.githubusercontent.com/53814741/209683896-ecde4d98-0f25-415a-b042-154809c90e41.png">

`https://{유저이름_또는_Org이름}.github.io/{레포이름}/documentation/{PROJECT_NAME_소문자}`

에서 확인가능



