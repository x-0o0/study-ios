# SwiftLint via Homebrew

1. 터미널 열기
2. 프로젝트 폴더로 이동
3. 터미널에 설치 명령어 실행
```bash
$ brew install swiftlint
```
4. 루트폴더에 `EmptyFile` 생성 후 `.swiftlint.yml`로 명명
5. 프로젝트 `Build Phase`에 `Run script` 추가
```sh
export PATH="$PATH:/opt/homebrew/bin" # m1 을 위한 코드라인
if which swiftlint > /dev/null; then
  swiftlint --fix # --fix는 자동 수정 옵션
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```
6. `Run Script` 순서를 `Compile Source` 앞으로 가져오기
