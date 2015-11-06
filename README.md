# Prolific Swift Style Guide

This is Prolific's style guide for writing code using Swift.

The purpose of this guide is to develop a universal standard for Swift code that makes our codebases
consistent and easy to read. 

----

### Variable Declaration ###

For declaring variables, favor `let` instead of `var` unless you need a mutable object or container.

```swift
func formatDate(date: NSDate) -> String 
{
    let dateFormatter = NSDateFormatter() // In this case, use `let` since the variable `dateFormatter` never changes once set
    dateFormatter.dateStyle = .ShortStyle
    return dateFormatter.stringFromDate(date)
}

func arrays()
{
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

class Object
{
	private var name = ""
	
	func useName()
	{
		// Let self be implied when it can be understood.
		otherObject.doSomethingWithName(name)
		setName("Will Smith")
	}
	
	func setName(name: String)
	{
		// Use self here to prevent conflicts with the `name` parameter being passed.
		self.name
	}
	
	func setNameAsync(newName: String)
	{
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
In closures, think about: should `self` be `weak` instead of `strong`?

### Force Unwrap ###

Unless there is a situation that absolutely calls for it, usage of the force-unwrap operator `(!)` should
be minmized, if not eliminated all together. With the many and varied ways of unwrapping optionals, it is
safer and simpler to declare variables as optional and unwrap them when needing their values than it is 
to force-unwrap them and potentially introduce runtime errors into the code base.

```swift
@IBOutlet private weak var textLabel: UILabel?

...

if let text = self.textLabel?.text {
    doSomethingWithText(text)
}

```

If you are interoping from Objective-C and the declaration automatically translates into force-unwrapped 
parameters, replace them with `?` parameters instead.


### Access Modifiers ###

Always specify access modifiers to top-level definitions. Do not specify them otherwise unless
the modifier is needed:

```swift
internal final class Object {
    
    var myInt: Int

    private func doWork() 
    {
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

