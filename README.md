Recipes App
==

## Description

- An app that showcases many recipes together with their detail information
- Contain lots of good practices for structuring iOS app and handle dependencies
- Support iOS 9+
- Use Xcode 9.3, Swift 4.0
- Ideas are from my experience and learnings from colleagues, community. Pull requests are more than welcome ❤️

<div align="center">
<img src="Screenshots/Home.png" height="500" />
<img src="Screenshots/Detail.png" height="500" />
</div>

### Minimum supported iOS version

- As of early 2018, the latest version is iOS 11
- See measurements about iOS adoption on [AppStore](https://developer.apple.com/support/app-store/)
- Apple suggests supporting 2 latest versions
- Supporting iOS 9 is not a pain for this app, it's good to have since there's quite a few percentage of users according to the chart below.
- Ideally, to determine when to drop support old iOS versions, we need analytics about how users use our app.

<div align="center">
<img src="Screenshots/AppStore.png" height="500" />
</div>


### Project

- Create new project, select both Unit Tests and UI Tests as we should write tests early.
- We can organise files by feature/module or role/types. Each has its pros and cons
- By role/type:
  - Pros: 
    - Less thinking about where to put files.
    - Easier to apply scripts or filter
  - Cons:
    - Hard to correlate if we want to find files related to same feature
    - Take time to reorganise files if we want to make them into reusable components
- By feature/module
  - Pros:
    - Make things modular
    - Encourage composition
  - Cons:
    - May be messy when many files of different types stay close

### Modular

- It may depend on projects, but I usually group things by feature/module. It answers the question "What does this app do?" instead of "What is this file?".

  - Source: contains source code files, splitted into features
    - Features: contain main features in the app
      - Home: the home screen, showing a list of recipes and open search
      - List: show a list of recipes, know how to reload and show empty view
      - Search: handle search and debouncing
      - Detail: show detail information
    - Library
      - Flow: contains FlowController to manage flows
      - Adapater: Generic data source for `UICollectionView`
      - Extension: convenient extensions for common operations
      - Model: The model in the app, parsed from json
  - Resource: contains plist, resource, Storyboard files

![](Screenshots/Project.png)

### Code convention

- I agree with most of the style guides in https://github.com/raywenderlich/swift-style-guide and https://github.com/github/swift-style-guide
- Use consistent and easy-to-reason indentation [Indenting Swift code](https://github.com/onmyway133/blog/issues/93)
- Prefer camelCase [Using camelCase for abbreviations](https://github.com/onmyway133/blog/issues/147)

### Documentation

- Good code explains itself.
- I find it useful to add documentations to classes and methods to briefly explain what they do
- Press `Cmd+Alt+/` to generate documentation with `///`


### MARK

- Using of `MARK` can be helpful to separate sections of code. It also groups functions nicely in Navigation Bar

![](Screenshots/MARK.png)

- You can also use `extension` groups related properties, methods
- For `UIViewController` we possibly have

```
// MARK: - Init
// MARK: - View life cycle
// MARK: - Setup
// MARK: - Action
// MARK: - Data
```

### Git

- Use `.gitignore` file from https://www.gitignore.io/api/swift
- There are both pros and cons in [checking in dependencies](https://guides.cocoapods.org/using/using-cocoapods#should-i-check-the-pods-directory-into-source-control) files (CocoaPods and Carthage). Here I choose to ignore them to not clutter the code base.

> Whether or not you check in the Pods directory, the Podfile and Podfile.lock should always be kept under version control.

### Launch Screen

- Prefer using `LaunchScreen.storyboard`
- Add launch image to `Asset Catalog`, open `LaunchScreen.storyboard`, add `UIImageView` and pin it to the egdes of `UIView`
- Should not pin to Safe Area as we want image to be full screen. Also unselect any margins in Auto Layout constraints
- Set `UIImageView` 's `contentMode` as `Aspect Fill`

![](Screenshots/LaunchScreen.png)

### App Icon

- Make sure image has no transparent pixels, otherwise it results in black background. Read [
Human Interface Guidelines - App Icon](https://developer.apple.com/ios/human-interface-guidelines/icons-and-images/app-icon/)

> Keep the background simple and avoid transparency. Make sure your icon is opaque, and don’t clutter the background. Give it a simple background so it doesn’t overpower other app icons nearby. You don’t need to fill the entire icon with content.

- Preferrably use image with size greater than 1024x1024
- Use [IconGenerator](https://github.com/onmyway133/IconGenerator) to generate `AppIcon.appiconset`

![](Screenshots/AppIcon.png)

### API

- Register for account in http://food2fork.com/about/api
- I use [1Password](https://1password.com/) to generate and store my passwords.
- I use [Insomnia](https://github.com/getinsomnia/insomnia) to test and analyze API response.

![](Screenshots/Insomnia.png)

### SwiftLint

- Regardless of what platform we develop, it's good to have a linter to enfore consistent conventions
- I use [SwiftLint](https://github.com/realm/SwiftLint)
- Add `pod 'SwiftLint', '~> 0.25'` to `Podfile`. It's also a good practice to specify [version](https://guides.cocoapods.org/syntax/podfile.html#pod) of the dependencies
- Add `.swiftlint.yml`. A sample can be found [here](https://github.com/realm/SwiftLint/blob/master/.swiftlint.yml)
- Add new Run Script Phrase to execute `swiftlint` after compiling

![](Screenshots/SwiftLint.png)

### Model

- It may sound boring but clients are just a prettier way to represent the API response. The model is perhaps the most basic thing and we use it a lot in the app. It plays such an important role and there can be some obvious bugs related to malformed models and assumption about how a model should be parsed.
- We should have test for every model of the app. Ideally we need automate testing of models from API response in case the model is changed from the backend.
- From Swift 4, we can conform our model to [Codable](https://developer.apple.com/documentation/swift/codable) to easily serialise to and from json.
- Model should be immutable

```swift
struct Recipe: Codable {
  let publisher: String
  let url: URL
  let sourceUrl: String
  let id: String
  let title: String
  let imageUrl: String
  let socialRank: Double
  let publisherUrl: URL

  enum CodingKeys: String, CodingKey {
    case publisher
    case url = "f2f_url"
    case sourceUrl = "source_url"
    case id = "recipe_id"
    case title
    case imageUrl = "image_url"
    case socialRank = "social_rank"
    case publisherUrl = "publisher_url"
  }
}
```

- We can use some test frameworks if you like fancy syntax or Rspec style. Some 3rd party test frameworks may have issues. I find `XCTest` good enough.

```swift
import XCTest
@testable import Recipes

class RecipesTests: XCTestCase {
  func testParsing() throws {
    let json: [String: Any] = [
      "publisher": "Two Peas and Their Pod",
      "f2f_url": "http://food2fork.com/view/975e33",
      "title": "No-Bake Chocolate Peanut Butter Pretzel Cookies",
      "source_url": "http://www.twopeasandtheirpod.com/no-bake-chocolate-peanut-butter-pretzel-cookies/",
      "recipe_id": "975e33",
      "image_url": "http://static.food2fork.com/NoBakeChocolatePeanutButterPretzelCookies44147.jpg",
      "social_rank": 99.99999999999974,
      "publisher_url": "http://www.twopeasandtheirpod.com"
    ]

    let data = try JSONSerialization.data(withJSONObject: json, options: [])
    let decoder = JSONDecoder()
    let recipe = try decoder.decode(Recipe.self, from: data)

    XCTAssertEqual(recipe.title, "No-Bake Chocolate Peanut Butter Pretzel Cookies")
    XCTAssertEqual(recipe.id, "975e33")
    XCTAssertEqual(recipe.url, URL(string: "http://food2fork.com/view/975e33")!)
  }
}

```

### FlowController

- FlowController is used to manage many `UIViewController` related to a common feature.
- Read [FlowController and Coordinator](https://github.com/onmyway133/blog/issues/106)
- There are `AppFlowController` that manages changing `rootViewController`. For now it start `RecipeFlowController`

```swift
window = UIWindow(frame: UIScreen.main.bounds)
window?.rootViewController = appFlowController
window?.makeKeyAndVisible()
appFlowController.start()
```

- `RecipeFlowController` manages, in fact it is, `UINavigationController`, that handles pushing `HomeViewController, RecipesDetailViewController, SafariViewController`

```swift
final class RecipeFlowController: UINavigationController {
  /// Start the flow
  func start() {
    let service = RecipesService(networking: NetworkService())
    let controller = HomeViewController(recipesService: service)
    viewControllers = [controller]
    controller.select = { [weak self] recipe in
      self?.startDetail(recipe: recipe)
    }
  }

  private func startDetail(recipe: Recipe) {}
  private func startWeb(url: URL) {}
}
```

- The `UIViewController` can use `delegate` or `closure` to notify `FlowController` about changes or next screen in the flow. For `delegate` there may need for check when there are 2 instances of the same classes. Here we use `closure` for simplicity.

### Auto Layout

- I try to use Auto Layout as much as possible to make adaptive UI.
- We can use libraries like [Anchors](https://github.com/onmyway133/Anchors) to do declarative and fast Auto Layout.
- However in this app, we can just use `NSLayoutAnchor` since it is from iOS 9.
- Inspired by [Constraint](https://github.com/hyperoslo/Sugar/blob/master/Sources/iOS/Constraint.swift)

```swift
extension NSLayoutConstraint {
  static func activate(_ constraints: [NSLayoutConstraint]) {
    constraints.forEach {
      ($0.firstItem as? UIView)?.translatesAutoresizingMaskIntoConstraints = false
      $0.isActive = true
    }
  }
}
```

### Architecture

- I'm a fan of exploring [architectures](https://github.com/onmyway133/fantastic-ios-architecture)
- All architectures, patterns define roles for each objects, and how to connect them.
- Remember these principles
  - Encapsuate what varies
  - Favor composition over inheritance
  - Program to interface, not to implementation

### Massive View Controller

- There is no massive view controller, it's just us writing bad code. However there are ways to [slim](http://khanlou.com/2014/09/8-patterns-to-help-you-destroy-massive-view-controller/) it down.
- In this app. I use
  - Service: injected into view controller to perform a single task
  - Generic View: move view and controls declaration to View layer
  - Child view controller: Compose child view controllers to build more features

### Access Control

- From [Access Control](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html)

> Access control restricts access to parts of your code from code in other source files and modules. This feature enables you to hide the implementation details of your code, and to specify a preferred interface through which that code can be accessed and used.

- Everything should be `private` and `final` by default. This also helps the [compiler](https://developer.apple.com/swift/blog/?id=27).
- When seeing a public property, we need to search for it across project before doing further with it. If the property is used only within a `class`, making it `private` means we don't need to care if it breaks elsewhere.
- Declare as many `final` as possible

```swift
final class HomeViewController: UIViewController {}
```

- Declare properties as `private` or at least `private(set)`

```swift
final class RecipeDetailView: UIView {
  private let scrollableView = ScrollableView()
  private(set) lazy var imageView: UIImageView = self.makeImageView()
}
```

### Lazy properties

- For properties that can be accessed at a later time, we can declare them as `lazy`
- We can use `closure` for fast construction

```swift
final class RecipeCell: UICollectionViewCell {
  private(set) lazy var containerView: UIView = {
    let view = UIView()
    view.clipsToBounds = true
    view.layer.cornerRadius = 5
    view.backgroundColor = Color.main.withAlphaComponent(0.4)

    return view
  }()
}
```

- We can use `make` functions if we plan to reuse the same function for multiple properties. 

```swift
final class RecipeDetailView: UIView {
  private(set) lazy var imageView: UIImageView = self.makeImageView()

  private func makeImageView() -> UIImageView {
    let imageView = UIImageView()
    imageView.contentMode = .scaleAspectFill
    imageView.clipsToBounds = true
    return imageView
  }
}
```

- Read [Strive for Fluent Usage](https://swift.org/documentation/api-design-guidelines/)

> Begin names of factory methods with “make”, e.g. x.makeIterator().


## Credit

- Launch image is from http://desertrosemediapa.com/
- App icon is from https://www.flaticon.com/free-icon/rice_168559

## Licence

This project is released under the MIT license. See LICENSE.md.
