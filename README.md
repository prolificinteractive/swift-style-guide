# Prolific Swift Style Guide

This is Prolific's style guide for writing code using Swift.

The purpose of this guide is to develop a universal standard for Swift code that makes our codebases
consistent and easy to read.

This is a work in progress.
If there is anything you are looking for that is not covered here you should refer to [Github's style guide](https://github.com/github/swift-style-guide).

----

### Statement Termination ###

Unlike Objective-C, omit the use of `;` to terminate statements. Instead, simply use new lines to indicate the end of a statement. 

```swift
let myVar = 0
doSomething(myVar)

return
```

Avoid multiple statements on a single line.

```swift
guard let obj = myObj else { print("Something went wrong"); return; } // Wrong! Instead, place each item on its own line.
```

### Variable Declaration ###

For declaring variables, favor `let` instead of `var` unless you need a mutable object or container.

```swift
func formatDate(date: NSDate) -> String {
    let dateFormatter = NSDateFormatter() // In this case, use `let` since the variable `dateFormatter` never changes once set
    dateFormatter.dateStyle = .ShortStyle
    return dateFormatter.stringFromDate(date)
}

func arrays() {
    let array = ["Hello", "Ciao", "Aloha"] // use let here since this is an immutable container

    var mutableArray = [String]() // Use var here since this container is mutable
    mutableArray.append("Farewell")
    mutableArray.append("Arrivederci")
}

```

### Self ###

Never use the `self` modifier except in cases where it is necessary by the compiler or to alleviate conflicts
with other variable declarations.

```swift

class Object {
	private var name = ""
	
	func useName() {
		// Let self be implied when it can be understood.
		otherObject.doSomethingWithName(name)
		setName("Will Smith")
	}
	
	func setName(name: String) {
		// Use self here to prevent conflicts with the `name` parameter being passed.
		self.name = name
	}
	
	func setNameAsync(newName: String) {
		// Use implicit self outside closures...
		otherObject.doSomethingWithName(name, then: { 
			// .. but within, you must use self to ease the compiler.
			self.setName("Jason")
		})
	}
}

```

*Rationale*: The idea behind this is that implicit use of self makes the conditions where you _must_ use self
(for instance, within closures) much more apparent and will make you think more on the reasons why you are using it.
In closures, think about: should `self` be `weak` instead of `strong`? Apple has even rejected a request to enforce use of `self` for this reason, [among others](http://ericasadun.com/2016/01/06/the-swift-evolution-proposal-se-0009-rejection/).

### Bracket Syntax ###

For brackets, prefer the Xcode-default syntax of having the opening brace be on the same line as the statement opening it:

```swift
internal final class MyObject {
}

internal enum MyEnum {
}

func doSomething() {
}

if true == false {
}

let doSomething: () -> Void = { 
}

```

*Rationale*: Simply put, this is the Xcode default and standard, and it's not worth fighting. This keeps things consistent
across the board and makes our lives as developers considerably easier.

### Force Unwrap ###

Unless there is a situation that absolutely calls for it, usage of the force-unwrap operator `(!)` should
be minmized, if not eliminated all together. With the many and varied ways of unwrapping optionals, it is
safer and simpler to declare variables as optional and unwrap them when needing their values than it is
to force-unwrap them and potentially introduce runtime errors into the code base.

```swift

if let text = self.textLabel?.text {
    doSomethingWithText(text)
}

```

If you are interoping from Objective-C and the declaration automatically translates into force-unwrapped
parameters, replace them with `?` parameters instead.

The one exception to this rule are IBOutlets. @IBOutlets may be declared using `!` so long as they are expected
to exist for the lifetime of the view controller object:

```swift
@IBOutlet private weak var textLabel: UILabel!

```


### Access Modifiers ###

Always specify access modifiers to top-level definitions. Do not specify them otherwise unless
the modifier is needed:

```swift
internal final class Object {
    var myInt: Int

    private func doWork() {
        ...
    }
}

```

### Type Declarations ###

When declaring types, the colon should be placed immediately after the identifier followed by one space
and the type name.

```swift

var intValue: Int

// Do NOT do this
var intValue : Int

```

In all use-cases, the colon should be associated with the left-most item with no spaces preceding and one space afterwards:

```swift
let myDictionary: [String: AnyObject] = ["String": 0]
```

### Nil-Checking ###

Favor `if-let` checking over direct nil checking in all cases except when the result of this check is required:

```swift
guard let item = myItem else {
	return
}

doSomethingWith(item)
```

```swift
if let _ = error { // Prefer this over `if error != nil`
	fatalError()
}
```

```swift
func isError(error: Error?) -> Bool {
	return (error != nil) // In this case, we need the result of the bool, and this is much cleaner than the other options.
}
```

### Enums ###

For enum declarations, declare each enum case on a new line with its own `case` statement instead of a comma-separated list.

```swift
internal enum State {
	case Open
	case Closed
	case Pending
	case Faulted
}
```

Prefer singular case for enum names instead of plural: `State` vs. `States`:

```swift
var currentState = State.Open
var previousState = States.Closed // Reads less clearly than the previous option.
```

For enums with associated values, declare the associated value on the same line as its declaration:

```swift
internal enum HTTPMethod: String {
	case Get = "GET"
	case Post = "POST"
}
```

For any other functions or properties associated with the enum, place them after the last case item in the enum list:

```swift
internal enum State {
	case Open
	case Closed
	case Pending
	case Faulted
	
	func nextState() -> State {
		...
	}
}
```

In cases where the enum's type name can be omitted, do so:

```swift
let state = State.Open

if state == .Closed { ... // Prefer .Closed instead of State.Closed

```
