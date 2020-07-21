# Code Architecture

## Intended Platforms
* `iOS`
* `macOS`

## Version
`1.0.0`

## Introduction
Here are [many interesting architectures](https://github.com/onmyway133/awesome-ios-architecture). Some well-known architectures from this include: [VIPER](https://www.objc.io/issues/13-architecture/viper/), [Clean Swift Architecture](https://clean-swift.com/) and [MVVM](https://www.raywenderlich.com/34-design-patterns-by-tutorials-mvvm).

Architecture seems the most artistic part of coding. Whilst simple ideas are often the best, there's something satisfying about clean design patterns. They take something mundane, and make it humane. However, the battle is often over finding the impossible middle-ground between simple and over-engineered, dynamic, yet consistent. One foot in order, and the other in chaos. The illusion of perfect architecture can easily bring one to the brink of madness.

From reading through the examples above, seeing real-world implementations, and of course, by making mistakes and subsequent refactors, herein contains a brief description of design patterns used in:

[EasyMusicPlayer](https://github.com/larromba/EasyMusicPlayer)

[EasyLife](https://github.com/larromba/EasyLife)

[DigiDocs](https://github.com/larromba/DigiDocs)

[Graffiti-Backgrounds](https://github.com/larromba/graffiti-backgrounds)

## Overview
[MVVM](https://www.raywenderlich.com/34-design-patterns-by-tutorials-mvvm) is probably the most accessible architecture, and what is used professionally. Nonetheless, I've always disapproved of `UIViewController` instances owning business logic (`ViewModel`). Whilst it does play well with Apple's eco-system, it seems more intuitive to reverse the direction, and have something owning a `UIViewController`, to keep it as thin as possible. However this is at the expense of battling Apple's eco-system - a slippery slope.

Taking this principle, the following architecture is my conclusion. If you find it useful, that's great, but remember to question everything, and ultimately do what makes sense to you. Often with the niche knowledge that comes with architecture, or new frameworks, you'll find ego. The reality is, especially with simple projects, the compiler won't care that much. Architecture should be about, your future-self's sanity, and the sanity of others, which of course is very subjective. Either way, the following brought consistency to my code, especially when I didn't know what to do. I find it repeatably testable, and whilst a little verbose, satisfying to use.

It's still a work in progress, and probably has many areas to improve, but it's become my default for personal projects. As there's no perfect solution, if it works, is durable to change, makes sense on Monday, and doesn't crash - it's probably good enough.

## Parts

### `AppDelegate` 
* Entry point to most iOS / macOS apps.
* Holds an `App` object, and forwards on all relevant `NSApplicationDelegate` / `UIApplicationDelegate` methods.

### `AppFactory`
* Creates an `App` object.

### `App`
* Essentially the `UIApplicationDelegate` delegate. The intentional separation allows UI testing via conventional unit tests. This is controversial, but I find it more effective that Apple's UI testing framework, as internal objects can be mocked / stubbed where necessary.
* Holds `Router` instances. I haven’t yet written an app with more than one `Router`, so there may be a blind-spot here.

### `Router`
* Holds `Coordinator` instances. 
* Handles routing logic between `Coordinator` instances, such as injecting `UIViewController` instances that come from `UISegue`.

### `Coordinator`
* Manages the lifecycle of `Controller` instances.
* Holds one `Controller`, or series of `Controller` instances that form a conceptual 'flow' (e.g. a sign-up flow).

### `Controller`
* Holds one `UIViewController`, and any `Controller` instances that wrap the child `UIViewController` components.
* Manages connection between `UIViewController` and business logic objects (managers, services, etc).
* Injects `ViewState`.

### `ViewController`
* Binds a `ViewState` object to the `UIView`. 
* Handles `UIView` animations.

### `ViewState`
* Always a struct
* Holds all the state of a `UIViewController` at any one time. 
* Should mostly be `let` properties. Can contain `var` properties when a `UIViewController` needs to reflect UI changes. There's probably a better way of doing this using [RxSwift](https://github.com/ReactiveX/RxSwift) or now [Combine](https://developer.apple.com/documentation/combine).
* Contains data and simple data logic. Is not injected with business logic objects (e.g. managers / services, etc).

### `Utility`
* Business logic utilities to help make code more reusable.
* `Manager`, `Service`, `Object`. Names are usually tricky, but there's no real convention - just whatever makes sense. 

Here's some articles:

[avoiding managers](https://stackoverflow.com/questions/1866794/naming-classes-how-to-avoid-calling-everything-a-whatevermanager)

[the trouble with managers](https://sandofsky.com/patterns/manager-classes/)

## Protocols
All parts should be defined with a protocol ending in `able` or `ing` - whatever reads better. This creates an interface that allows each part to be swapped out for mock objects. 

For example:
```
protocol MyObjecting {
    // define the public interface here
}

final class MyObject: MyObjecting {
}

final class Foo {
    let object: MyObjecting // always reference your object by the interface
    
    init(object: MyObjecting) {
        self.object = object
    }
}
```

## Style
There's a [SwiftLint](https://github.com/realm/SwiftLint) with each project, however, [this](https://github.com/raywenderlich/swift-style-guide) style guide is often aimed for.

## Testing the UI with unit tests
As mentioned above, the `UIAppDelegate` can be intercepted by creating an `AppTestEnvironment`. Here's an example:

```swift
final class AppTestEnvironment {
    // objects that can be stubbed before injecting (like managers / services)
    var app: FooService!
    
    // objects that that can't be stubbed as they're created on injection (like coordinators / controllers)
    private(set) app: Apping!
    
    init(foo: FooService) {
        self.foo = foo
    }
    
    func inject() {
        app = App(foo: foo)
    }
    
    // lots of useful shortcut functions to help with testing
    
    func usefulFunction1() {
    }
    
    func usefulFunction2() {
    }
}
```

In your test, you can then write code like this:

```
func test_foo_whenButtonPressed_expectSomethingHappens() {
    // mocks
    let foo = MockFoo()
    env = AppTestEnvironment(foo: foo)
    env.usefulFunction1()
    env.usefulFunction2()
    env.inject() // inject when ready
    
    // sut
    env.someViewController.button.fire()
    
    // test
    XCTAssertEqual(...)
}
```

## Discussion
Pros:
* Allows for UI Unit testing without using Apple's UI testing framework.
* Consistency between apps.
* Helps explore principles of code architecture. Experimenting with your own system is a great way to gain insight as a developer.

Cons:
* One-fit solutions are never perfect for all solutions.
* It's arguably over-engineered, especially for beginners.
* New systems are confusing for unfamiliar developers.
* Possibly re-inventing the wheel.

## Points of Interest
Objects are unnecessarily created in the `AppFactory` instance. It's 'more precise' to create & inject these only when necessary (such as at the `Router`, or `Coordinator` level). However, as the memory footprint is often only from `Data` or `View` objects, which are still allocated at point of use, it's not really a problem.

The advantage outweighs this inaccuracy, as it becomes easier to test from higher levels. For example, if the business logic for downloading an image works, but the button triggering it does not, the app won't work, even if the business logic is passing its own test. So by testing the button press (input), and the UI expectation (output), rather than the explicit business logic (that may even be split over multiple classes), we achieve a more accurate test. By not caring for the underlying implementation - refactors are also easier. It's intuitive to think of testing this way - e.g. when I tap this button, I expect something to happen. These tests often achieve 70-80% code coverage.
