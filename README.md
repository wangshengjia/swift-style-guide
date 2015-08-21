A guide to our Swift style and conventions.

#### Whitespace

 * Tabs, not spaces.
 * End files with a newline.
 * Make liberal use of vertical whitespace to divide code into logical chunks.
 * `{` should stay in the same line
 * Don’t leave trailing whitespace.
   * Not even leading indentation on blank lines.

#### Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

#### Avoid Using Force-Unwrapping of Optionals

If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, prefer this:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Alternatively, you might want to use Swift's Optional Chaining in some of these cases, such as:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

_Rationale:_ Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.

#### Avoid Using Implicitly Unwrapped Optionals

Where possible, use `let foo: FooType?` instead of `let foo: FooType!` if `foo` may be nil (Note that in general, `?` can be used instead of `!`).

_Rationale:_ Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

#### Prefer implicit getters on read-only properties and subscripts

When possible, omit the `get` keyword on read-only computed properties and
read-only subscripts.

So, write these:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… not these:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_Rationale:_ The intent and meaning of the first version is clear, and results in less code.

#### When specifying a type, always associate the colon with the identifier

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_Rationale:_ The type specifier is saying something about the _identifier_ so
it should be positioned with it.

Also, when specifying the type of a dictionary, always put the colon immediately
after the key type, followed by a space and then the value type.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Only explicitly refer to `self` when required

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_Rationale:_ This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

#### Prefer structs over classes

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

could be refactored into these definitions:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ Value types are simpler, easier to reason about, and behave as expected with the `let` keyword.

#### Make classes `final` by default

Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

_Rationale:_ Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.

#### Use whitespace around operator definitions

Use whitespace around operators when defining them. Instead of:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

write:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_Rationale:_ Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.

#### Prefer to use `guard` instead of `if`

Use `guard` to do the quick fail. Instead of:

```swift
if let foo = foo {
  // do something
}
```
or

```swift
if foo == nil {
  // throw error 
}
```
write:

```swift
guard let foo = foo else {
  // throw error 
}
// do somethings
```

_Rationals:_ Quick fail means less pyramide of doom which brings easier read code.

#### Always chain condition after `if let` and `guard let` when possible

Instead of:

```swift
if let foo = foo {
  if let bar = bar {
    // do somethings with foo & bar
  }
}
```

write:

```swift
if let foo = foo, let bar = bar {
    // do somethings with foo & bar
}
```

#### Prefer nested type instead of global type

Instead of:

```swift
enum Bar {
  case bar1, bar2, bar3
}

static let ConstantKey = "ConstantKey"

struct Foo {}
```

write:

```swift
struct Foo {
  static let ConstantKey = "ConstantKey"
  enum Bar {
    case bar1, bar2, bar3
  }
}
```

_Rationals:_ In this case, we have the natural namespace.

#### If a function has a closure as parameter, always prefer put it to the last. Then use trailing closure.

Instead of:

```swift
foo.bar({ // do something })
```

write:

```swift
foo.bar() {
  // do something
}
```

#### Prefer named argument instead of shorthand `$0`, `$1`

Instead of:

```swift
elements.map { /* transform with $0 */ }
```

write:

```swift
elements.map { element in /* transform with element */ }
```

#### Prefer `if let param = param as? type` instead of `if param is type ... (param as! type).method()`

Instead of:

```swift
if foo is FooType {
  (foo as! FooType).bar()
}
```

write:

```swift
if let foo = foo as? FooType {
  foo.bar()
}
```

#### Prefer short dot syntax when using enum if possible

Instead of:

```swift
view.contentMode = UIViewContentMode.Center
```

write:

```swift
view.contentMode = .Center
```

#### Prefer `private` or `private(set)` when possible

#### Non class prefixe

#### Prefer to use `CGGeometry` way

Instead of:

```swift
CGRectMake(0, 0, 100, 100)
```

write:

```swift
CGRect(x: 0, y: 0, width: 100, height: 100)
```

