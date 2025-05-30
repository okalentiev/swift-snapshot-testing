# 📸 SnapshotTesting

[![CI](https://github.com/pointfreeco/swift-snapshot-testing/workflows/CI/badge.svg)](https://actions-badge.atrox.dev/pointfreeco/swift-snapshot-testing/goto)
[![Slack](https://img.shields.io/badge/slack-chat-informational.svg?label=Slack&logo=slack)](http://pointfree.co/slack-invite)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fpointfreeco%2Fswift-snapshot-testing%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/pointfreeco/swift-snapshot-testing)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fpointfreeco%2Fswift-snapshot-testing%2Fbadge%3Ftype%3Dplatforms)](https://swiftpackageindex.com/pointfreeco/swift-snapshot-testing)

Delightful Swift snapshot testing.

## Usage

Once [installed](#installation), _no additional configuration is required_. You can import the
`SnapshotTesting` module and call the `assertSnapshot` function.

``` swift
import SnapshotTesting
import Testing

@MainActor
struct MyViewControllerTests {
  @Test func myViewController() {
    let vc = MyViewController()

    assertSnapshot(of: vc, as: .image)
  }
}
```

When an assertion first runs, a snapshot is automatically recorded to disk and the test will fail,
printing out the file path of any newly-recorded reference.

> ❌ failed - No reference was found on disk. Automatically recorded snapshot: …
>
> open "…/MyAppTests/\_\_Snapshots\_\_/MyViewControllerTests/testMyViewController.png"
>
> Re-run "testMyViewController" to test against the newly-recorded snapshot.

Repeat test runs will load this reference and compare it with the runtime value. If they don't
match, the test will fail and describe the difference. Failures can be inspected from Xcode's Report
Navigator or by inspecting the file URLs of the failure.

You can record a new reference by customizing snapshots inline with the assertion, or using the
`withSnapshotTesting` tool:

```swift
// Record just this one snapshot
assertSnapshot(of: vc, as: .image, record: .all)

// Record all snapshots in a scope:
withSnapshotTesting(record: .all) {
  assertSnapshot(of: vc1, as: .image)
  assertSnapshot(of: vc2, as: .image)
  assertSnapshot(of: vc3, as: .image)
}

// Record all snapshot failures in a Swift Testing suite:
@Suite(.snapshots(record: .failed))
struct FeatureTests {}

// Record all snapshot failures in an 'XCTestCase' subclass:
class FeatureTests: XCTestCase {
  override func invokeTest() {
    withSnapshotTesting(record: .failed) {
      super.invokeTest()
    }
  }
}
```

## Snapshot Anything

While most snapshot testing libraries in the Swift community are limited to `UIImage`s of `UIView`s,
SnapshotTesting can work with _any_ format of _any_ value on _any_ Swift platform!

The `assertSnapshot` function accepts a value and any snapshot strategy that value supports. This
means that a view or view controller can be tested against an image representation _and_ against a
textual representation of its properties and subview hierarchy.

``` swift
assertSnapshot(of: vc, as: .image)
assertSnapshot(of: vc, as: .recursiveDescription)
```

View testing is highly configurable. You can override trait collections (for specific size classes
and content size categories) and generate device-agnostic snapshots, all from a single simulator.

``` swift
assertSnapshot(of: vc, as: .image(on: .iPhoneSe))
assertSnapshot(of: vc, as: .recursiveDescription(on: .iPhoneSe))

assertSnapshot(of: vc, as: .image(on: .iPhoneSe(.landscape)))
assertSnapshot(of: vc, as: .recursiveDescription(on: .iPhoneSe(.landscape)))

assertSnapshot(of: vc, as: .image(on: .iPhoneX))
assertSnapshot(of: vc, as: .recursiveDescription(on: .iPhoneX))

assertSnapshot(of: vc, as: .image(on: .iPadMini(.portrait)))
assertSnapshot(of: vc, as: .recursiveDescription(on: .iPadMini(.portrait)))
```

> **Warning**
> Snapshots must be compared using the exact same simulator that originally took the reference to
> avoid discrepancies between images.

Better yet, SnapshotTesting isn't limited to views and view controllers! There are a number of
available snapshot strategies to choose from.

For example, you can snapshot test URL requests (_e.g._, those that your API client prepares).

``` swift
assertSnapshot(of: urlRequest, as: .raw)
// POST http://localhost:8080/account
// Cookie: pf_session={"userId":"1"}
//
// email=blob%40pointfree.co&name=Blob
```

And you can snapshot test `Encodable` values against their JSON _and_ property list representations.

``` swift
assertSnapshot(of: user, as: .json)
// {
//   "bio" : "Blobbed around the world.",
//   "id" : 1,
//   "name" : "Blobby"
// }

assertSnapshot(of: user, as: .plist)
// <?xml version="1.0" encoding="UTF-8"?>
// <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
//  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
// <plist version="1.0">
// <dict>
//   <key>bio</key>
//   <string>Blobbed around the world.</string>
//   <key>id</key>
//   <integer>1</integer>
//   <key>name</key>
//   <string>Blobby</string>
// </dict>
// </plist>
```

In fact, _any_ value can be snapshot-tested by default using its
[mirror](https://developer.apple.com/documentation/swift/mirror)!

``` swift
assertSnapshot(of: user, as: .dump)
// ▿ User
//   - bio: "Blobbed around the world."
//   - id: 1
//   - name: "Blobby"
```

If your data can be represented as an image, text, or data, you can write a snapshot test for it!

## Documentation

The latest documentation is available
[here](https://swiftpackageindex.com/pointfreeco/swift-snapshot-testing/main/documentation/snapshottesting).

## Installation

### Xcode

> **Warning**
> By default, Xcode will try to add the SnapshotTesting package to your project's main
> application/framework target. Please ensure that SnapshotTesting is added to a _test_ target
> instead, as documented in the last step, below.

 1. From the **File** menu, navigate through **Swift Packages** and select
    **Add Package Dependency…**.
 2. Enter package repository URL: `https://github.com/pointfreeco/swift-snapshot-testing.git`.
 3. Confirm the version and let Xcode resolve the package.
 4. On the final dialog, update SnapshotTesting's **Add to Target** column to a test target that
    will contain snapshot tests (if you have more than one test target, you can later add
    SnapshotTesting to them by manually linking the library in its build phase).

### Swift Package Manager

If you want to use SnapshotTesting in any other project that uses
[SwiftPM](https://swift.org/package-manager/), add the package as a dependency in `Package.swift`:

```swift
dependencies: [
  .package(
    url: "https://github.com/pointfreeco/swift-snapshot-testing.git",
    from: "1.12.0"
  ),
]
```

Next, add `SnapshotTesting` as a dependency of your test target:

```swift
targets: [
  .target(name: "MyApp"),
  .testTarget(
    name: "MyAppTests",
    dependencies: [
      "MyApp",
      .product(name: "SnapshotTesting", package: "swift-snapshot-testing"),
    ]
  )
]
```

## Features

  - [**Dozens of snapshot strategies**][available-strategies]. Snapshot
    testing isn't just for `UIView`s and `CALayer`s. Write snapshots against _any_ value.
  - [**Write your own snapshot strategies**][defining-strategies].
    If you can convert it to an image, string, data, or your own diffable format, you can snapshot
    test it! Build your own snapshot strategies from scratch or transform existing ones.
  - **No configuration required.** Don't fuss with scheme settings and environment variables.
    Snapshots are automatically saved alongside your tests.
  - **More hands-off.** New snapshots are recorded whether `isRecording` mode is `true` or not.
  - **Subclass-free.** Assert from any XCTest case or Quick spec.
  - **Device-agnostic snapshots.** Render views and view controllers for specific devices and trait
    collections from a single simulator.
  - **First-class Xcode support.** Image differences are captured as XCTest attachments. Text
    differences are rendered in inline error messages.
  - **Supports any platform that supports Swift.** Write snapshot tests for iOS, Linux, macOS, and
    tvOS.
  - **SceneKit, SpriteKit, and WebKit support.** Most snapshot testing libraries don't support these
    view subclasses.
  - **`Codable` support**. Snapshot encodable data structures into their JSON and property list
    representations.
  - **Custom diff tool integration**. Configure failure messages to print diff commands for
    [Kaleidoscope](https://kaleidoscope.app) or your diff tool of choice.
    ``` swift
    SnapshotTesting.diffToolCommand = { "ksdiff \($0) \($1)" }
    ```

[available-strategies]: https://swiftpackageindex.com/pointfreeco/swift-snapshot-testing/main/documentation/snapshottesting/snapshotting
[defining-strategies]: https://swiftpackageindex.com/pointfreeco/swift-snapshot-testing/main/documentation/snapshottesting/customstrategies

## Plug-ins

  - [AccessibilitySnapshot](https://github.com/cashapp/AccessibilitySnapshot) adds easy regression
    testing for iOS accessibility.
    
  - [AccessibilitySnapshotColorBlindness](https://github.com/Sherlouk/AccessibilitySnapshotColorBlindness)
    adds snapshot strategies for color blindness simulation on iOS views, view controllers and images.

  - [GRDBSnapshotTesting](https://github.com/SebastianOsinski/GRDBSnapshotTesting) adds snapshot
    strategy for testing SQLite database migrations made with [GRDB](https://github.com/groue/GRDB.swift).

  - [Nimble-SnapshotTesting](https://github.com/tahirmt/Nimble-SnapshotTesting) adds 
    [Nimble](https://github.com/Quick/Nimble) matchers for SnapshotTesting to be used by Swift
    Package Manager.

  - [Prefire](https://github.com/BarredEwe/Prefire) generating Snapshot Tests via
    [Swift Package Plugins](https://github.com/apple/swift-package-manager/blob/main/Documentation/Plugins.md)
    using SwiftUI `Preview`
  
  - [PreviewSnapshots](https://github.com/doordash-oss/swiftui-preview-snapshots) share `View`
    configurations between SwiftUI Previews and snapshot tests and generate several snapshots with a
    single test assertion.

  - [swift-html](https://github.com/pointfreeco/swift-html) is a Swift DSL for type-safe,
    extensible, and transformable HTML documents and includes an `HtmlSnapshotTesting` module to
    snapshot test its HTML documents.

  - [swift-snapshot-testing-nimble](https://github.com/Killectro/swift-snapshot-testing-nimble) adds
    [Nimble](https://github.com/Quick/Nimble) matchers for SnapshotTesting.

  - [swift-snapshot-testing-stitch](https://github.com/Sherlouk/swift-snapshot-testing-stitch/) adds
    the ability to stitch multiple UIView's or UIViewController's together in a single test.

  - [SnapshotTestingDump](https://github.com/tahirmt/swift-snapshot-testing-dump) Adds support to
    use [swift-custom-dump](https://github.com/pointfreeco/swift-custom-dump/) by using `customDump`
    strategy for `Any`

  - [SnapshotTestingHEIC](https://github.com/alexey1312/SnapshotTestingHEIC) adds image support
  using the HEIC storage format which reduces file sizes in comparison to PNG.

  - [SnapshotVision](https://github.com/gregersson/swift-snapshot-testing-vision) adds snapshot
    strategy for text recognition on views and images. Uses Apples Vision framework.

Have you written your own SnapshotTesting plug-in?
[Add it here](https://github.com/pointfreeco/swift-snapshot-testing/edit/master/README.md) and
submit a pull request!

## Related Tools

  - [`iOSSnapshotTestCase`](https://github.com/uber/ios-snapshot-test-case/) helped introduce screen
    shot testing to a broad audience in the iOS community. Experience with it inspired the creation
    of this library.

  - [Jest](https://jestjs.io) brought generalized snapshot testing to the JavaScript community with
    a polished user experience. Several features of this library (diffing, automatically capturing
    new snapshots) were directly influenced.

## Learn More

SnapshotTesting was designed with [witness-oriented programming](https://www.pointfree.co/episodes/ep39-witness-oriented-library-design).

This concept (and more) are explored thoroughly in a series of episodes on
[Point-Free](https://www.pointfree.co), a video series exploring functional programming and Swift
hosted by [Brandon Williams](https://twitter.com/mbrandonw) and
[Stephen Celis](https://twitter.com/stephencelis).

Witness-oriented programming and the design of this library was explored in the following
[Point-Free](https://www.pointfree.co) episodes:

  - [Episode 33](https://www.pointfree.co/episodes/ep33-protocol-witnesses-part-1): Protocol Witnesses: Part 1
  - [Episode 34](https://www.pointfree.co/episodes/ep34-protocol-witnesses-part-1): Protocol Witnesses: Part 2
  - [Episode 35](https://www.pointfree.co/episodes/ep35-advanced-protocol-witnesses-part-1): Advanced Protocol Witnesses: Part 1
  - [Episode 36](https://www.pointfree.co/episodes/ep36-advanced-protocol-witnesses-part-2): Advanced Protocol Witnesses: Part 2
  - [Episode 37](https://www.pointfree.co/episodes/ep37-protocol-oriented-library-design-part-1): Protocol-Oriented Library Design: Part 1
  - [Episode 38](https://www.pointfree.co/episodes/ep38-protocol-oriented-library-design-part-2): Protocol-Oriented Library Design: Part 2
  - [Episode 39](https://www.pointfree.co/episodes/ep39-witness-oriented-library-design): Witness-Oriented Library Design
  - [Episode 40](https://www.pointfree.co/episodes/ep40-async-functional-refactoring): Async Functional Refactoring
  - [Episode 41](https://www.pointfree.co/episodes/ep41-a-tour-of-snapshot-testing): A Tour of Snapshot Testing 🆓

<a href="https://www.pointfree.co/episodes/ep41-a-tour-of-snapshot-testing">
  <img alt="video poster image" src="https://d3rccdn33rt8ze.cloudfront.net/episodes/0041.jpeg" width="480">
</a>

## License

This library is released under the MIT license. See [LICENSE](LICENSE) for details.
