### `NormalNoiceTypes.json` 을 `NoticeTests` 에서 사용하려는 경우

<img width="341" alt="Screen Shot 2022-04-27 at 3 07 54 PM" src="https://user-images.githubusercontent.com/53814741/165452304-7064cfba-e850-4401-86a1-d2cc207f4d2c.png">

```swift
import XCTest
@testable import KuringSDK

class NoticeTests: XCTestCase {
    func test_fetchNormalNoticeTypes() throws {
        if let jsonURL = Bundle(for: NoticeTests.self).url(forResource: "NormalNoticeTypes", withExtension: "json") {
            let jsonData = try Data(contentsOf: jsonURL)
            let noticeTypes = try JSONDecoder().decode(NormalNoticeTypeResponse.self, from: jsonData)
            XCTAssert(!noticeTypes.noticeTypes.isEmpty)
        } else {
            XCTFail("NormalNoticeTypes.json 파일을 찾을 수 없습니다.")
        }
    }
    ...
}

```
