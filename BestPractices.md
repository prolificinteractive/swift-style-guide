# BEST PRACTICES #

This is Prolific's Swift best practices document. 

The purpose of these best practices is to help you keep clean and bug free Swift code that is recommended to follow in your daily work.

### Nil Checking ###

Though we have conventions for nil checking in the swift style guide, below are some suggestions for nil checking in certain situations.

If there's a case where you want to enter a scope if AT LEAST ONE item is non-nil then use the `??` operator.

```swift
if let _ = item ?? item1 {
	// At least one of these is not nil
}
```

If you want to check for non-nil AND evaluate a bool property or series of bool properties, use the `where` keyword.

```swift
if let _ = item where propertyBoolOne && !propertyBoolTwo {
	// Bool properties have been evaluated.
}
```

If you want to enter a scope, only if an object is nil, then you can check directly for nil.

```swift
if item == nil {
	// Do stuff..
}
```

### Assertion management ###

When working with optional values you have to make sure that the variable you work with doesn’t have a `nil` value. To do so you can use the different unwrapping techniques provided by the standard library, but in certain scenarios it can make sense to force unwrap your value and terminate your program (for example a wrong view controller type after instantiating from a Storyboard). In this case your app better terminate in order to avoid unexpected behaviors.

The standard Swift library provides you different assertion functions that affect your code differently:

* assert
* assertionFailure
* precondition
* preconditionFailure
* fataError

#### assert ####
`assert` is only evaluated in debug mode, it means that the line will be removed in release and will not be executed.

#### assertionFailure ####
`assertionFailure` acts like `assert` but provides some context to the compiler.

#### precondition ####
`precondition` ensures that the given condition is meet. If not the app will terminate. `precondition` works for both debug and release.

#### preconditionFailure ####
`preconditionFailure` means a fatal error and will terminate in both debug and release mode, except for unchecked builds (`-Ounchecked`), then it will never be executed.

#### fatalError ####
`fatalError` acts like `preconditionFailure` but is not affected by the unchecked build flag. It will always terminate your app in both debug and release mode.

#### Production ####

Although crashing on production is not always the ideal. Often you prefer crashing on debug but in production handle the error with a default value like 0 or empty array. To do so, we recommand implementing a function that takes an optional value as well as a tuple containing the default value of the same type and a error message for the context.

```swift
func nilOrDefault<T>(value: T?, @autoclosure defaultValue: () -> (value: T, text: String)) -> T {
    assert(value != nil, defaultValue().text)
    return value ?? defaultValue().value
}

let int2 = nilOrDefault(Int("s"), defaultValue: (value: 0, text: "Expected int not working"))
// crash in Debug
// 0 in Release
```

You can also define a custom operator doing the same thing if you want a more concise syntax:

```swift
infix operator ?! {}

func ?!<T>(wrapped: T?, @autoclosure nilDefault: () -> (value: T, text: String)) -> T {
    assert(wrapped != nil, nilDefault().text)
    return wrapped ?? nilDefault().value
}

let integer = Int(string) ?! (0, “Expected integer, got \(string)”)
// crash in Debug
// 0 in Release
```

### Property Observers ###
Cf Apple documentation:

```
Property observers observe and respond to changes in a property’s value. Property observers are called every time a property’s value is set, even if the new value is the same as the property’s current value.
```

You can use 2 types of property observer:

* *willSet* is called just before the property value is stored.
* *didSet* is called right after the property value is stored.

```swift
var intValue: Int = 0 {
	willSet(newIntValue) {
		print("About to set intValue to \(newIntValue)")
  	}
    didSet {
    	print("Set intValue to \(intValue)")
    }
}
```

`didSet` is very useful when working with a data source array so you can automatically refresh the table view or collection view associated with the array.

```
var dataSource: [String] {
	didSet {
		self.tableView.reloadData()
	}
}
```

Be careful to use *didSet* only on an initialized property. A typical example where it is dangerous to use *didSet* is to set an IBOutlet value before the view loaded.

```swift
class myViewController: UIViewController {
	@IBOutlet weak var label: UILabel!
	
	var text: String {
		didSet {
			self.label.text = text
		}
	}
}

let myViewController = UIViewController()
myViewController.text = "Prolific" // Crash because the view controller label has not been initialized yet
```

### Retain Cycle ###

To avoid retain cycle in Swift -- meaning when two objects both have strong references to each other -- use **weak** and **unowned** on your references to avoid having a strong reference on both sides.

* A **weak** reference is a pointer to an object that **doesn't protect** the object from being deallocated by [ARC](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html). The object **can** be nil.
* A **unowned** reference is a pointer to an object that **doesn't protect** the object from being deallocated by [ARC](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html). The object **can't** be nil.

According to Apple's [documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html):

```Use a weak reference whenever it is valid for that reference to become nil at some point during its lifetime. Conversely, use an unowned reference when you know that the reference will never be nil once it has been set during initialization.```

##### Example #####
```swift
internal final class MyClass {
	let notNilInstance = Instance()
    weak var delegate: MyDelegate?

    func myFunction() {
        delegate?.doSomething()
    }
    
    func myFunctionWithClosure() {
    	let closure = { [weak self, unowned notNilInstance] in
	    self?.doSomething() // weak variables are optionals
    	notNilInstance.doSomethingElse() // unowned variables are not
    }
}
```

##### Debug #####

[Here](http://applifebalance.com/posts/retain-cycle-instruments/) is an article on how to diagnose retain cycle bugs in your app using Instruments. Another easy way is to print inside the `deinit` function of your objects and see if they get deallocated.

##### Note #####

Retain cycle doesn't apply to Swift structs since they are passed by value and not by reference.

*Rationale* Retain cycle bugs are very easy to reproduce, being very careful when manipulating pointers is crucial to build a solid app.
