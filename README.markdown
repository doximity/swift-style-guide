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
* [Types](#types)
* [Memory Management](#memory-management)
* [Access Control](#access-control)
* [Control Flow](#control-flow)
* [Guard Statements](#guard-statements)
* [Thread Safety with `mt` Convention](#thread-safety-using-mt_-hungarian-notation)
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
- strive to name functions in a way that reads like a sentence
- name methods for their side effects
  - methods with side effects should start with a verb
  - methods with no side effects should start with a noun
  - boolean types should read like assertions
  - protocols that describe _what something is_ should read as nouns (e.g., `Collection`)
  - protocols that describe _a capability_ should end in _-able_ or _-ible_ (e.g., `Equatable`)
- take advantage of default parameters in your methods


### Delegates

When creating custom delegate methods, an unnamed first parameter should be the delegate source. UIKit contains numerous examples of this.

**Preferred:**
```swift
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```

**Not Preferred:**
```swift
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```

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

Avoid block comments inline with code, as the code should be as self-documenting as possible.

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

Here's an example of a well-styled class definition:

```swift
class Circle: Shape {
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

  convenience init(x: Int, y: Int, diameter: Double) {
    self.init(x: x, y: y, radius: diameter / 2)
  }

  override func area() -> Double {
    return Double.pi * radius * radius
  }
}

extension Circle: CustomStringConvertible {
  var description: String {
    return "center = \(centerString) area = \(area())"
  }
  private var centerString: String {
    return "(\(x),\(y))"
  }
}
```

The example above demonstrates the following style guidelines:

 + Specify types for properties, variables, constants, argument declarations and other statements with a space after the colon but not before, e.g. `x: Int`, and `Circle: Shape`.
 + Indent getter and setter definitions and property observers.
 + Don't add modifiers such as `internal` when they're already the default. Similarly, don't repeat the access modifier when overriding a method.
 + Organize extra functionality (e.g. printing) in extensions.
 + Hide non-shared, implementation details such as `centerString` inside the extension using `private` access control.

### Use of Self

For conciseness and to prevent non-obvious retain cycles, avoid using `self` since Swift does not require it to access an object's properties or invoke its methods.

Use self only when required by the compiler (in `@escaping` closures, or in initializers to disambiguate properties from arguments). In other words, if it compiles without `self` then omit it.


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

## Function Declarations

Keep short function declarations on one line including the opening brace:

```swift
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

For functions with long signatures, add line breaks after each parameter. This helps with readability and decreases the need for line wrap.

```swift
func reticulateSplines(spline: [Double],
                       adjustmentFactor: Double,
                       translateConstant: Int,
                       comment: String) -> Bool {
    // reticulate code goes here
}
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

It makes sense to break logical chunks of the method into smaller pieces- there are a few ways to do approach this.

#### Option 1: Break the logical chunks of code into helper methods inside of the same type

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

#### Option 2: Use `performScopedOperation`

This approach breaks up the code into into smaller pieces, and limits those chunks of code to being called from within the scope of the enclosing method.  

`performScopedOperation` acts very similarly to nested functions: it can access variables previously defined outside of its brackets, and it can also define its own variables, which live only within the scope of its brackets. Unlike nested functions where you have to execute the function later in your code, `performScopedOperation` executes the block of code immediately.

However, the result is still a long `viewDidLoad` method.

```swift
override public func viewDidLoad() {
    super.viewDidLoad()

    // setupUpdatesButton
    performScopedOperation {
        updatesButton.layer.cornerRadius = 20.0
        updatesButton.layer.masksToBounds = true
        updatesButton.layer.shadowColor = UIColor.black.cgColor
        updatesButton.layer.shadowOffset = CGSize(width: 0, height: 2)
    }

    // setupDismissGestureRecognizer
    performScopedOperation {
        let dismissSwipe = UISwipeGestureRecognizer(target: self, action: #selector(didSwipeUpdateView(_:)))
        dismissSwipe.direction = .up
        updatesAvailableButton.addGestureRecognizer(dismissSwipe)
    }

    // setupTableView
    performScopedOperation {
        tableView.addSubview(refreshControl)
        tableView.rowHeight = UITableViewAutomaticDimension
        tableView.tableHeaderView = UIView(frame: CGRect(x: 0, y: 0, width: tableView.frameWidth, height: 0.01))
    }
}
```

Both options are supported, and it’s up to the developer to weigh out the options.

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

Use trailing closure syntax only if there's a single closure expression parameter at the end of the argument list. Otherwise, use a descriptive name for each closure parameter.

For method calls where closure arguments are specified by name (rather than using trailing closure syntax), add a newline before each parameter name (including the first)

**Preferred:**
```swift
UIView.animate(withDuration: 1.0) {
  self.myView.alpha = 0
}

UIView.animate(
  withDuration: 1.0,
  animations: 
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

Prefer compact code and let the compiler infer the type for constants or variables of single instances. Type inference is also appropriate for small (non-empty) arrays and dictionaries. When required, specify the specific type such as `CGFloat` or `Int16`.

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

IBOutlets should always be `strong`

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

Only explicitly use `open`, `public`, and `internal` when you require a full access control specification.

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

## Thread Safety Using `mt_` Hungarian Notation
Operations that affect the UI should be run on the main thread. Failing to obey this rule results in undefined behavior and untraceable bugs. To mitigate this issue, we use the `mt_` (short for `main thread`) prefix to indicate that something must be accessed from the main thread.

For more details, read our [wiki article](https://wiki.doximity.com/articles/the-mt_-convention-hungarian-notation-for-thread-safe-code) on the `mt_` convention.

### Overview
There are 3 ways in which an `mt_` operation may be carried out:

1. An `mt_` operation can be directly called by any `mt_` function
2. An `mt` operation can be directly called by another function that is guaranteed to run on the main thread. Examples of functions that are guaranteed to run on the main thread are lifecycle functions like `viewDidLoad`, and `IBAction`s, which should be named accordingly: e.g., `mt_didTapExitButton`
3. An `mt` operation can be called from a non-`mt` function, as long as you wrap it in a main-thread dispatch closure.

### Functions
A function which must be executed on the main thread must begin with the prefix `mt_`.

```swift
override func viewDidLoad() {
   super.viewDidLoad()

   // `viewDidLoad` is guaranteed to run on the main thread
   // So we can call `mt_updateUIBasedOnCurrentUser` directly
   mt_updateUIBasedOnCurrentUser()
}

func mt_presentNewUserFetchedAlert() {
   // We must name this method with `mt` because UI work takes place inside of it

   let alert = UIAlertController(title: "New User Fetched!",
                                 message: nil,
                                 preferredStyle: UIAlertControllerStyle.alert)

   presentViewController(alert, animated: true)
}

func updateCurrentUser(notification: Notification) {
   if let newCurrentUser = notification.payload as? User {

      // Since `updateCurrentUser` is not guaranteed to be called on the main thread,
      // We must dispatch onto the main thread before calling `mt_` functions.
      DispatchQueue.main.async{ 
         mtSet_currentUser = newCurrentUser
         mt_presentNewUserFetchedAlert()
      }
   }
}
```

### Properties
Properties can be both read and written to, which are 2 separate operations.
- If a property must be **read** on the main thread, use `mtGet_` as the prefix.
- If a property must be **set** on the main thread, use `mtSet_` as the prefix.
- If a property must be **read and set** on the main thread, use `mt_` as the prefix.

In the example below, since this is the `didSet` of an `mtSet` variable, we are guaranteed to be on the main thread. Therefore, we can directly call `mt_` functions.
     
```swift
var mtSet_currentUser: User? {
   didSet {
      mt_updateUIBasedOnCurrentUser()
   }
}
```

### Closures
If a closure must be **called** on the main thread, use `mtCall_` as the prefix. For `mtCall_` closures, always use named parameters instead of using trailing closure syntax.

```swift
func checkUserStatus(mtCall_success: () -> ()) {
  // We are not inside of a `mt` function
  // But the `mtCall_success` name indicates the closure must be called on the main thread
  // So we must call the closure inside of a main-thread dispatch.
  
  DispatchQueue.main.async{ mtCall_success() }
}

override func viewDidLoad() {
   super.viewDidLoad()

   checkUserStatus(
     mtCall_success: { // <- Don't use trailing-closure syntax
                       // <- Instead, explicitly name the argument with `mt`
         someLabel.text = "user status successfully set"
     }
   )
}
```

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
* [Doximity Wiki on `_mt` Convention](https://wiki.doximity.com/articles/the-mt_-convention-hungarian-notation-for-thread-safe-code)
