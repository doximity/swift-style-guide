# Doximity's Swift Style Guide

As our team grows, its important to maintain a consistent coding style. Consistency makes our code more readable, understandable, and enjoyable to work with! It can also save time during code reviews and help avoid developer errors.

That being said, we understand that each contributor has his/her own personal style- This style guide attempts to cover most common styling decisions while still leaving much open to developer discretion.

Our overarching goals are correctness, clarity, consistency, and brevity, in that order.

## Table of Contents

* [Correctness](#correctness)
* [Naming](#naming)
* [Code Organization](#code-organization)
* [Spacing](#spacing)
* [Comments](#comments)
* [Classes and Structures](#classes-and-structures)
* [Function Declarations](#function-declarations)
* [Closure Expressions](#closure-expressions)
* [Void Inputs in Closures](#void-inputs-in-closures)
* [Types](#types)
* [Memory Management](#memory-management)
* [Access Control](#access-control)
* [Control Flow](#control-flow)
* [Guard Statements](#guard-statements)
* [Catching Errors with `gentlePreconditionFailure`](#catching-errors-with-gentlepreconditionfailure)
* [General Syntax](#general-syntax)

## Correctness

Correctness is first and foremost. Code should compile without warnings.

## Naming

Descriptive and consistent naming makes software easier to read and understand. Please read through the Swift naming conventions described in the [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). Some key takeaways include:

- prioritize clarity over brevity
  - dont use obscure terminology
  - avoid abbreviations
  - label closure and tuple parameters used in your method declarations
- use camel case (not snake case)
- use uppercase for types and protocols, lowercase for everything else
- name functions and parameters based on role, not on type
- name methods for their side effects
  - methods with side effects should start with a verb
  - methods with no side effects should start with a noun
  - boolean types should read like assertions
  - protocols that describe _what something is_ should read as nouns (e.g., `Collection`)
  - protocols that describe _a capability_ should end in _-able_ or _-ible_ (e.g., `Equatable`)
- take advantage of default parameters in your methods

### Generics

Generic type parameters should be descriptive, upper camel case names. When a type name doesn't have a meaningful relationship or role, use a traditional single uppercase letter such as `T`, `U`, or `V`.

**Preferred:**
```swift
struct Stack<Element> { ... }
func write<Target: OutputStream>(to target: inout Target)
func swap<T>(_ a: inout T, _ b: inout T)
```

**Not Preferred:**
```swift
struct Stack<T> { ... }
func write<target: OutputStream>(to target: inout target)
func swap<Thing>(_ a: inout Thing, _ b: inout Thing)
```

## Code Organization
### Extensions

Use extensions to organize your code into logical blocks of functionality.

Extensions can live in the same file, or can be broken out into a separate file. As a general rule of thumb, if the extension is long and provides a larger set of functionality, we should move it to a new file.

For extensions in the same file, use a `// MARK: -` comment to separate each one. This keeps things well-organized and searchable from the jump bar (`ctrl` + `6`).

For UIKit view controllers, consider grouping lifecycle, custom accessors, and IBAction in separate class extensions.


### Protocol Conformance

In particular, when adding protocol conformance to a model, prefer adding a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods.

**Preferred:**
```swift
class MyViewController: UIViewController {
    // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
    // scroll view delegate methods
}
```

**Not Preferred:**
```swift
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
    // all methods
}
```

### Unused Code

Unused (dead) code should be removed. This includes commented out code, methods that simply forward to the superclass, and empty/unused delegate methods.

### Minimal Imports

Keep imports minimal. For example, don't import `UIKit` when importing `Foundation` will suffice.

## Spacing

### Braces
Method braces and other braces (`if`/`else`/`switch`/`while` etc.) always open on the same line as the statement but close on a new line.

Tip: You can re-indent by selecting some code (or `⌘A` to select all) and then `ctrl` + `i`.

**Preferred:**
```swift
if user.isHappy {
    // Do something
} else {
    // Do something else
}
```

**Not Preferred:**
```swift
if user.isHappy
{
    // Do something
}
else {
    // Do something else
}
```

A potential exception to this is when the return statement is a single statement, then the open and close braces can be on the same line

```swift
guard let user.isHappy else { return }
```

### Newlines
There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but having too many sections in a method often means you should refactor into several methods.

### Colons
Colons always have no space on the left and one space on the right. Exceptions are the ternary operator `? :`, empty dictionary `[:]` and `#selector` syntax for unnamed parameters `(_:)`.

**Preferred:**
```swift
class TestDatabase: Database {
    var data: [String: CGFloat] = ["A": 1.2, "B": 3.2]
}
```

**Not Preferred:**
```swift
class TestDatabase : Database {
    var data :[String:CGFloat] = ["A" : 1.2, "B":3.2]
}
```

* Long lines should be wrapped at around 120 characters. A hard limit is intentionally not specified.

* Avoid trailing whitespaces at the ends of lines.


## Comments

When they are needed, use inline comments to explain **why** a particular piece of code does something. Comments must be kept up-to-date or deleted.

For documenting methods and properties, use the auto-generated template (`opt` + `cmd` + `/`) to describe expected parameters and return values.

```swift
struct Point {
        var x = 0.0
        var y = 0.0

        /// Update the caller's position. This function is mutating, so it actually changes the Point's properties.
        ///
        /// - Parameters:
        ///   - deltaX: the amount to move in the horizontal direction
        ///   - deltaY: the amount to move in the vertical direction
        mutating func moveBy(x deltaX: Double, y deltaY: Double) {
            x += deltaX
            y += deltaY
        }
    }
```

## Classes and Structures

### Which one to use?

Remember, structs have [value semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_144). Classes have [reference semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_145).

Structs are generally safer.
- Structs are always copied, so you don't need to worry about a separate thread mutating your object unexpectedly
- With structs, we generally don't need to worry about memory management, which helps avoid retain cycles and memory leaks

However you may want to use a class in certain instances. Use a class if:
- It needs to inherit from another class
- It needs to class properties
- Changes should propagate to all references to the same object
- Generally, this is the case if the object has identity. You would model a person as a class because two person objects are two different things. Just because two people have the same name and birthdate, doesn't mean they are the same person. But the person's birthdate would be a struct because a date of 3 March 1950 is the same as any other date object for 3 March 1950. The date itself doesn't have an identity.


### Example definition

Here's an example of a well-styled struct definition:

```swift
protocol Shape {
    func area() -> Double
    func perimeter() -> Double
}

struct Circle {
  var x: Int
  var y: Int
  var radius: Double
  var diameter: Double {
    get {
      return radius * 2
    }
    set {
      radius = newValue / 2
    }
  }

  init(x: Int, y: Int, radius: Double) {
    self.x = x
    self.y = y
    self.radius = radius
  }

  init(x: Int, y: Int, diameter: Double) {
    self.init(x: x, y: y, radius: diameter / 2)
  }
}

extension Circle: Shape {
    func area() -> Double {
        return Double.pi * radius * radius
    }

    func perimeter() -> Double {
        return 2 * Double.pi * radius
    }
}

extension Circle: CustomStringConvertible {
    var description: String {
        return "center = \(centerString), area = \(area()), perimeter = \(perimeter())"
    }

    private var centerString: String {
        return "(\(x),\(y))"
    }
}
```

The example above demonstrates the following style guidelines:
 + Use a struct instead of a class because a Circle doesn't have identity
 + Use a protocol for `Shape` to specify expected functionality of shapes, such as `area` and `perimeter`. Using a base class `Shape` wouldnt make sense, because there would be no meaningful `area` or `perimeter` for just an abstract `Shape`
 + Specify types for properties, variables, constants, argument declarations and other statements with a space after the colon but not before, e.g. `x: Int`, and `Circle: Shape`.
 + Indent getter and setter definitions and property observers.
 + Don't add modifiers such as `internal` when they're already the default. Similarly, don't repeat the access modifier when overriding a method.
 + Organize extra functionality (e.g., `Shape` protocol conformance, printing) in extensions.
 + Hide non-shared, implementation details such as `centerString` inside the extension using `private` access control.

### Use of Self

For conciseness and to prevent non-obvious retain cycles, avoid using `self` since Swift does not require it to access an object's properties or invoke its methods.

Use self only when required by the compiler (in `@escaping` closures, or in initializers to disambiguate properties from arguments). In other words, if it compiles without `self` then omit it. When the compiler forces you to use `self` inside of closures, please look at the [Memory Management](https://github.com/doximity/swift-style-guide#memory-management) section of this guide to see how to use `self` safely.

Example of using `self` inside of an initializer 👌:
```swift
class Book {
    let title: String
    let author: String

    init(title: String, author: String) {
        self.title = title
        self.author = author
    }
}
```

### Computed Properties

For conciseness, if a computed property is read-only, omit the get clause. The get clause is required only when a set clause is provided.

**Preferred:**
```swift
var diameter: Double {
    return radius * 2
}
```

**Not Preferred:**
```swift
var diameter: Double {
    get {
        return radius * 2
    }
}
```

### Final

Use of `final` can clarify your intent. In the below example, `Box` has a particular purpose and customization in a derived class is not intended. Marking it `final` makes that clear.

```swift
// Turn any generic type into a reference type using this Box class.
final class Box<T> {
    let value: T
    init(_ value: T) {
        self.value = value
    }
}
```

## Functions

### Function Spacing
Keep short function declarations on one line including the opening brace:

```swift
func reticulateSplines(spline: [Double]) -> Bool {
    // reticulate code goes here
}
```

For functions with long signatures, add line breaks before the first and after each parameter. This helps with readability and decreases the need for line wrap.

```swift
func reticulateSplines(
    spline: [Double],
    adjustmentFactor: Double,
    translateConstant: Int,
    comment: String
) -> Bool {
    // reticulate code goes here
}
```

Use the same rule when calling functions with many parameters. If the function call doesn't fit on one line, add a newline between each parameter.

```swift
    let john = Person(
    fullName: "John Doe",
    gender: .male,
    location: "San Francisco, CA",
    occupation: .doctor
)
```

### Method Organization
When a single method encompasses a lot of functionality, it can get long and cluttered. A common case of this is when there is a ton of set up in `viewDidLoad`.

```swift
override public func viewDidLoad() {
    super.viewDidLoad()

    updatesButton.layer.cornerRadius = 20.0
    updatesButton.layer.masksToBounds = true
    updatesButton.layer.shadowColor = UIColor.black.cgColor
    updatesButton.layer.shadowOffset = CGSize(width: 0, height: 2)

    let dismissSwipe = UISwipeGestureRecognizer(target: self, action: #selector(didSwipeUpdateView(_:)))
    dismissSwipe.direction = .up
    updatesAvailableButton.addGestureRecognizer(dismissSwipe)

    tableView.addSubview(refreshControl)
    tableView.rowHeight = UITableViewAutomaticDimension
    tableView.tableHeaderView = UIView(frame: CGRect(x: 0, y: 0, width: tableView.frameWidth, height: 0.01))
}
```

#### Preferred approach: Break the logical chunks of code into helper methods inside of the same type

This approach significantly cleans up the `viewDidLoad` call, making it very easy to understand and digest in pieces. However, it leaves the helper methods available for use within the scope of the whole view controller class. This may or may not be desirable.

```swift
override public func viewDidLoad() {
    super.viewDidLoad()

    setupUpdatesButton()
    setupDismissGestureRecognizer()
    setupTableView()
}

private func setupUpdatesButton() {
    updatesButton.layer.cornerRadius = 20.0
    updatesButton.layer.masksToBounds = true
    updatesButton.layer.shadowColor = UIColor.black.cgColor
    updatesButton.layer.shadowOffset = CGSize(width: 0, height: 2)
}

private func setupDismissGestureRecognizer() {
    let dismissSwipe = UISwipeGestureRecognizer(target: self, action: #selector(didSwipeUpdateView(_:)))
    dismissSwipe.direction = .up
    updatesAvailableButton.addGestureRecognizer(dismissSwipe)
}

private func setupTableView() {
    tableView.addSubview(refreshControl)
    tableView.rowHeight = UITableViewAutomaticDimension
    tableView.tableHeaderView = UIView(frame: CGRect(x: 0, y: 0, width: tableView.frameWidth, height: 0.01))
}
```

Where appropriate a helper `var`, especially `lazy var`, is also a great way to break out composition of the object.

### Functions vs Methods

Free functions, which aren't attached to a class or type, should be used sparingly. When possible, prefer to use a method instead of a free function. This aids in readability and discoverability.

Free functions are most appropriate when they aren't associated with any particular type or instance.

**Preferred**
```swift
let sorted = items.mergeSorted()  // easily discoverable
rocket.launch()  // acts on the model
```

**Not Preferred**
```swift
let sorted = mergeSort(items)  // hard to discover
launch(&rocket)
```

**Free Function Exceptions**
```swift
let tuples = zip(a, b)  // feels natural as a free function (symmetry)
let value = max(x, y, z)  // another free function that feels natural
```

## Closure Expressions

For method calls where closure arguments are specified by name (rather than using trailing closure syntax), add a newline before each parameter name (including the first)

**Preferred:**
```swift
UIView.animate(withDuration: 1.0) {
    self.myView.alpha = 0
}

UIView.animate(
    withDuration: 1.0,
    animations: {
        self.myView.alpha = 0
    },
    completion: { finished in
        self.myView.removeFromSuperview()
    }
)
```

**Not Preferred:**
```swift
UIView.animate(withDuration: 1.0, animations: {
    self.myView.alpha = 0
})

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}) { f in
    self.myView.removeFromSuperview()
}
```

Chained methods using trailing closures should be clear and easy to read in context. Decisions on spacing, line breaks, and when to use named versus anonymous arguments is left to the discretion of the author. Examples:

```swift
let value = numbers.map { $0 * 2 }.filter { $0 % 3 == 0 }.index(of: 90)

let value = numbers
    .map {$0 * 2}
    .filter {$0 > 50}
    .map {$0 + 10}
```

## Types

Always use Swift's native types when available. Swift offers bridging to Objective-C so you can still use the full set of methods as needed.

**Preferred:**
```swift
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String
```

**Not Preferred:**
```swift
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```

In Sprite Kit code, use `CGFloat` if it makes the code more succinct by avoiding too many conversions.

### Constants

Constants are defined using the `let` keyword, and variables with the `var` keyword. Always use `let` instead of `var` if the value of the variable will not change.

**Tip:** A good technique is to define everything using `let` and only change it to `var` if the compiler complains!

You can define constants on a type rather than on an instance of that type using type properties. To declare a type property as a constant simply use `static let`. Type properties declared in this way are generally preferred over global constants because they are easier to distinguish from instance properties. Example:

**Preferred:**
```swift
enum Math {
    static let e = 2.718281828459045235360287
    static let root2 = 1.41421356237309504880168872
}

let hypotenuse = side * Math.root2

```
**Note:** The advantage of using a case-less enumeration is that it can't accidentally be instantiated and works as a pure namespace.

**Not Preferred:**
```swift
let e = 2.718281828459045235360287  // pollutes global namespace
let root2 = 1.41421356237309504880168872

let hypotenuse = side * root2 // what is root2?
```

### Optionals

Declare variables and function return types as optional with `?` where a nil value is acceptable.

**Avoid using implicitly unwrapped optionals declared with `!`**
They should be used as little as possible! One exception to this is for `IBOutlet`s, which are guaranteed to be initialized before use

When accessing an optional value, use optional chaining if there are many optionals in the chain:

```swift
self.textContainer?.textLabel?.setNeedsDisplay()
```

Use optional binding when it's more convenient to unwrap once and perform multiple operations:

```swift
if let textContainer = self.textContainer {
    // do many things with textContainer
}
```

When naming optional variables and properties, avoid naming them like `optionalString` or `maybeView` since their optional-ness is already in the type declaration.

For optional binding, shadow the original name when appropriate rather than using names like `unwrappedView` or `actualLabel`.

**Preferred:**
```swift
var subview: UIView?
var volume: Double?

// later on...
if let subview = subview, let volume = volume {
  // do something with unwrapped subview and volume
}
```

**Not Preferred:**
```swift
var optionalSubview: UIView?
var volume: Double?

if let unwrappedSubview = optionalSubview {
    if let realVolume = volume {
        // do something with unwrappedSubview and realVolume
    }
}
```

### Lazy Initialization

Consider using lazy initialization for finer grain control over object lifetime. This is especially true for `UIViewController` that loads views lazily. You can either use a closure that is immediately called `{ }()` or call a private factory method. Example:

```swift
lazy var locationManager: CLLocationManager = self.makeLocationManager()

private func makeLocationManager() -> CLLocationManager {
    let manager = CLLocationManager()
    manager.desiredAccuracy = kCLLocationAccuracyBest
    manager.delegate = self
    manager.requestAlwaysAuthorization()
    return manager
}
```

**Notes:**
  - `[unowned self]` is not required here. A retain cycle is not created.
  - Location manager has a side-effect for popping up UI to ask the user for permission so fine grain control makes sense here.


### Type Inference

Prefer compact code and let the compiler infer the type for constants or variables of single instances when expressions are simple. For more complex expressions, types can make the code more readable and faster to compile. If it's hard to immediately understand what type an expression is using or returns it's probably worth including some types. If any type inference time warnings appear it's also necessary to add more types. This is especially common in heavily chained reactive expressions. Type inference is also appropriate for small (non-empty) arrays and dictionaries. When required, specify the specific type such as `CGFloat` or `Int16`.

**Preferred:**
```swift
let message = "Click the button"
let currentBounds = computeViewBounds()
var names = ["Mic", "Sam", "Christine"]
let maximumWidth: CGFloat = 106.5
```

**Not Preferred:**
```swift
let message: String = "Click the button"
let currentBounds: CGRect = computeViewBounds()
let names = [String]()
```

#### Type Annotation for Empty Arrays and Dictionaries

For empty arrays and dictionaries, use type annotation. (For an array or dictionary assigned to a large, multi-line literal, use type annotation.)

**Preferred:**
```swift
var names: [String] = []
var lookup: [String: Int] = [:]
```

**Not Preferred:**
```swift
var names = [String]()
var lookup = [String: Int]()
```

### Syntactic Sugar

Prefer the shortcut versions of type declarations over the full generics syntax.

**Preferred:**
```swift
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```

**Not Preferred:**
```swift
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```

### Use Type Inferred Context

Use compiler inferred context to write shorter, clear code.  (Also see [Type Inference](#type-inference).)

**Preferred:**
```swift
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)
```

**Not Preferred:**
```swift
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

## Memory Management

Code should not create reference cycles. Analyze your object graph and prevent strong cycles with `weak` and `unowned` references. Alternatively, use value types (`struct`, `enum`) to prevent cycles altogether.

### Extending Object Lifetime

Extend object lifetime using the `[weak self]` and `guard let 'self' = self else { return }` idiom. `[weak self]` is preferred to `[unowned self]` where it is not immediately obvious that `self` outlives the closure. Explicitly extending lifetime is preferred to optional unwrapping.

**Preferred**
```swift
resource.request().onComplete { [weak self] response in
    guard let 'self' = self else {
        return
    }
    let model = self.updateModel(response)
    self.updateUI(model)
}
```

**Not Preferred**
```swift
// might crash if self is released before response returns
resource.request().onComplete { [unowned self] response in
    let model = self.updateModel(response)
    self.updateUI(model)
}
```

**Not Preferred**
```swift
// deallocate could happen between updating the model and updating UI
resource.request().onComplete { [weak self] response in
    let model = self?.updateModel(response)
    self?.updateUI(model)
}
```

### IBOutlets

IBOutlets should be `strong` and `private` when appropriate

**Preferred**
```swift
@IBOutlet var notificationsView: UIView!
```

**Not Preferred**
```swift
@IBOutlet weak var notificationsView: UIView!
```

## Access Control

Using `private` and `fileprivate` appropriately, however, adds clarity and promotes encapsulation. Prefer `private` to `fileprivate` when possible. Using extensions may require you to use `fileprivate`.

Only explicitly use `open`, `public`, and `internal` when you require a full access control specification. Only include at the access level actually needed as these are expensive lookups across the project, especially when added to common `UIKit` and `Foundation` base types.

Use access control as the leading property specifier. The only things that should come before access control are the `static` specifier or attributes such as `@IBAction`, `@IBOutlet` and `@discardableResult`.


## Control Flow

Prefer the `for-in` style of `for` loop over the `while-condition-increment` style.

**Preferred:**
```swift
for _ in 0..<3 {
    print("Hello three times")
}

for (index, person) in attendeeList.enumerated() {
    print("\(person) is at position #\(index)")
}
```

**Not Preferred:**
```swift
var i = 0
while i < 3 {
    print("Hello three times")
    i += 1
}

var i = 0
while i < attendeeList.count {
    let person = attendeeList[i]
    print("\(person) is at position #\(i)")
    i += 1
}
```

## Guard Statements

### Avoid Nested `if` Statements

When coding with conditionals, the left-hand margin of the code should be the "golden" or "happy" path. That is, don't nest `if` statements. Multiple return statements are OK. The `guard` statement is built for this.

When using `guard` statements, keep the `else {` on the same line as the last condition

**Preferred:**
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {

    guard let context = context else {
        throw FFTError.noContext
    }
    guard let inputData = inputData else {
        throw FFTError.noInputData
    }

    // use context and input to compute the frequencies
    return frequencies
}
```

**Not Preferred:**
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {

    if let context = context {
        if let inputData = inputData {
            // use context and input to compute the frequencies

            return frequencies
        } else {
            throw FFTError.noInputData
        }
    } else {
        throw FFTError.noContext
    }
}
```

### Compound `guard` or `if let` Statements

When multiple optionals are unwrapped either with `guard` or `if let`, minimize nesting by using the compound version when possible. When there are multiple `let` statements compounded into the same `guard` or `if`, separate them onto new lines and left align them.

**Preferred:**
```swift
guard let number1 = number1,
      let number2 = number2,
      let number3 = number3 else {
    fatalError("impossible")
}
// do something with numbers
```

**Not Preferred:**
```swift
if let number1 = number1 {
    if let number2 = number2 {
        if let number3 = number3 {
            // do something with numbers
        } else {
            fatalError("impossible")
        }
    } else {
        fatalError("impossible")
    }
} else {
    fatalError("impossible")
}
```

### Failing Guards

Guard statements are required to exit in some way. Generally, this should be simple one line statement such as `return`, `throw`, `break`, `continue`, and `fatalError()`. Large code blocks should be avoided. If cleanup code is required for multiple exit points, consider using a `defer` block to avoid cleanup code duplication.

### Switch Statements

When at all possible, avoid using the `default` clause in `switch` statements, instead prefer to explicitly enumerate the unexpected conditions. This is especially true with `enum` data types. The rationale being if a new `case` is added to an `enum`, we want the compiler to generate an error informing us of the unhandled case. If we used a `default` clause, the new case would fall through to that clause and that default action may not be how that case should be handled. Reasonable exceptions include test-only code where handling many cases outside of the test at hand will never execute and add little value. Use in application code should have a comment explaining why so the next developer can decide if more explicit handling becomes warranted.

**Preferred:**
```swift
switch someDirection {
case west:
    print("Go west, young man!")
case north,
     south,
     east:
    print("Go somewhere else!")
}
```

**Not Preferred**
```swift
switch someDirection {
case west:
    print("Go west, young man!")
default:
    print("Go somewhere else")
}

```

A more complex example may illustrate the rationale more clearly. Take the case of conforming to `Equatable`:

```swift
public static func == (lhs: Direction, rhs: Direction) -> Bool {
    switch (lhs, rhs) {
    case (.north, .north),
         (.west, .west),
         (.south, .south),
         (.east, .east):
        return true

    case (.north, _),
         (.west, _),
         (.south, _),
         (.east, _):
        return false
    }
}
```

Were we to add a new cases for in between directions (e.g. `northEast`) and we relied on the `default` clause to inform us of the non-equal state, then `northEast == south` which isn't true, and neither would the compiler inform us of this condition.

Using the `default` clause may be acceptable when switching over other data types.


## Catching Errors with `gentlePreconditionFailure`
Use `gentlePreconditionFailure` to catch instances where the app reaches an unexpected state.

In development or test builds (`DEBUG` and `ADHOC` configurations), `gentlePreconditionFailure` behaves like `preconditionFailure` and triggers a crash.

In a production build (`RELEASE` configuration), `gentlePreconditionFailure` will log that an unexpected state was encountered, but will not crash. If your app expects a returned value where the unexpected state was encountered, `gentlePreconditionFailure` also allows you to provide a fallback return value.

```swift
override public func tableView(_ tableView: UITableView,
                               cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let colleague = colleagues[safe: indexPath.row],
          let cell = tableView.dequeueReusableCell(withIdentifier: ColleagueCell.reuseIdentifier()) as? ColleagueCell else {
        return gentlePreconditionFailure() { return UITableViewCell() }
    }

    cell.colleague = colleague
    return cell
}
```

## General Syntax
### Parentheses

Parentheses around conditionals are not required and should be omitted.

**Preferred:**
```swift
if name == "Hello" {
    print("World")
}
```

**Not Preferred:**
```swift
if (name == "Hello") {
    print("World")
}
```

In larger expressions, optional parentheses can sometimes make code read more clearly.

**Preferred:**
```swift
let playerMark = (player == current) ? "X" : "O"
```

## References

* [The Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
* [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/index.html)
