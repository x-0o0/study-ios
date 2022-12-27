## 개요

> **중요**
> iOS16 이상

지도앱처럼 flexible height를 갖는 Bottom sheet 뷰 만들기

> **참고**
> [유튜브](https://www.youtube.com/watch?v=Y-D12Dc7HBk) 영상 보면 됩니다.

### `View` 에 `.bottomSheet` 함수 추가하기
```swift
import SwiftUI

extension View {
    // MARK: ref - https://www.youtube.com/watch?v=Y-D12Dc7HBk
    @ViewBuilder
    func bottomSheet<Content: View>(
        presentationDetents: Set<PresentationDetent>,
        isPresented: Binding<Bool>,
        dragIndicator: Visibility = .visible,
        sheetCornerRadius: CGFloat?,
        largestUndimmedIdentifier: UISheetPresentationController.Detent.Identifier = .large,
        isTranparentBackground: Bool = false,
        interactiveDisabled: Bool = true,
        @ViewBuilder content: @escaping () -> Content,
        onDismiss: @escaping () -> ()
    ) -> some View {
        self
            .sheet(isPresented: isPresented) {
                onDismiss()
            } content: {
                content()
                    .presentationDetents(presentationDetents)
                    .presentationDragIndicator(dragIndicator)
                    .interactiveDismissDisabled(interactiveDisabled)
                    .onAppear {
                        // Presented VC 찾기
                        guard let windows = UIApplication.shared.connectedScenes.first as? UIWindowScene else {
                            return
                        }
                        
                        if let controller = windows.windows.first?.rootViewController?.presentedViewController,
                           let sheet = controller.presentationController as? UISheetPresentationController {
                            // 투명한 배경
                            if isTranparentBackground {
                                controller.view.backgroundColor = .clear
                            }
                            
                            // 지도 위 버튼에 대한 틴트가 hidden form으로 적용되는 걸 방지하기
                            controller.presentingViewController?.view.tintAdjustmentMode = .normal
                            
                            sheet.largestUndimmedDetentIdentifier = largestUndimmedIdentifier
                            sheet.preferredCornerRadius = sheetCornerRadius
                        } else {
                            print("MapBottomSheet를 위한 presented vc 가 없습니다.")
                        }
                    }
            }

    }
}

```

### 사용하기

```swift
MapView()
    .bottomSheet(
        presentationDetents: [.medium, .large, .height(70)],
        isPresented: .constant(true),
        sheetCornerRadius: 20,
        isTranparentBackground: true
    ) {
        BottomSheetView()
            .background { // blur 배경
                Rectangle()
                    .fill(.ultraThinMaterial)
                    .ignoresSafeArea()
            }
            // MARK: 스유에서 하나의 뷰컨은 하나의 시트만 띄울 수 있음 -> showAnotherSheet 상태프로퍼티 추가
            // But there's a work around
            // 간단히 모든 시트와 full screen cover 뷰들을 bottom sheet에 넣ㅇ을 것
            // bottom sheet은 새 뷰컨이기 때문에 또다른 시트를 보여줄 수 있음
            .sheet(isPresented: $showsAnotherSheet) {
                Text("설정화면")
            }
    } onDismiss: { }
```

