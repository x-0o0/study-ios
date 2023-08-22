# AppDelegate 에서 UIViewController 인스턴스로 띄우기

```swift
self.window = UIWindow(frame: UIScreen.main.bounds)
let viewController = MyAppViewController.init()
let navigationController = UINavigationController.init(rootViewController: viewController) // if you use navigation controller
self.window?.rootViewController = navigationController
self.window?.makeKeyAndVisible()
```
