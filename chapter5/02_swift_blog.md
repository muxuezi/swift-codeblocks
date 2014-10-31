* Oct 20, 2014

##  [Failable Initializers](https://developer.apple.com/swift/blog/?id=17)

Swift version 1.1 is new in [Xcode 6.1](https://developer.apple.com/xcode/downloads/), and it introduces a new feature: failable initializers. Initialization is the process of providing initial values to each of the stored properties of a class or struct, establishing the invariants of the object. In some cases initialization can fail. For example, initializing the object requires access to a resource, such as loading an image from a file:
```swift    
    NSImage(contentsOfFile: "swift.png")
```

If the file does not exist or is unreadable for any reason, the initialization of the NSImage will fail. With Swift version 1.1, such failures can be reported using a failable initializer. When constructing an object using a failable initializer, the result is an optional that either contains the object (when the initialization succeeded) or contains nil (when the initialization failed). Therefore, the initialization above should handle the optional result directly:
```swift    
    if let image = NSImage(contentsOfFile: "swift.png") {
        // loaded the image successfully
    } else {
        // could not load the image
    }
```

An initializer defined with init can be made failable by adding a ? or a ! after the init, which indicates the form of optional that will be produced by constructing an object with that initializer. For example, one could add a failable initializer to Int that attempts to perform a conversion from a String:
```swift    
    extension Int {
        init?(fromString: String) { 
            if let i = fromString.toInt() {
                // Initialize
                self = i
            } else { 
                // return nil, discarding self is implied
                return nil
            }
        }
    }
```

In a failable initializer, return nil indicates that initialization has failed; no other value can be returned. In the example, failure occurs when the string could not be parsed as an integer. Otherwise, self is initialized to the parsed value.

Failable initializers eliminate the most common reason for factory methods in Swift, which were previously the only way to report failure when constructing this object. For example, enums that have a raw type provided a factory method fromRaw that returned an optional enum. Now, the Swift compiler synthesizes a failable initializer that takes a raw value and attempts to map it to one of the enum cases. For example:
```swift    
    enum Color : Int {
        case Red = 0, Green = 1, Blue = 2
    
        // implicitly synthesized
        var rawValue: Int { /* returns raw value for current case */ }
    
        // implicitly synthesized
        init?(rawValue: Int) {
            switch rawValue { 
                case 0: self = .Red
                case 1: self = .Green
                case 2: self = .Blue
                default: return nil
            }
        }
    }
```

Using the failable initializer allows greater use of Swift’s uniform construction syntax, which simplifies the language by eliminating the confusion and duplication between initializers and factory methods. Along with the introduction of failable initializers, Swift now treats more Cocoa factory methods — those with NSError arguments — as initializers, providing a more uniform experience for object construction.

You can read more about failable initializers in [The Swift Programming Language](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-XID_339).

* Oct 7, 2014

##  [Building Your First Swift App Video](https://developer.apple.com/swift/blog/?id=16)

So far the Swift blog has focused on advanced programming topics, including the design principles of the Swift language. We thought it would be helpful to provide content for programmers who are new to Swift and just trying Xcode for the first time. To make it more approachable for everyone, we put together a very short video that demonstrates how to build an iOS app in Swift from scratch, in less than ten minutes.

    * [Watch the video](https://developer.apple.com/swift/blog/?id=16)

* Sep 25, 2014

##  [Building assert() in Swift, Part 2: __FILE__ and __LINE__](https://developer.apple.com/swift/blog/?id=15)

Two occasionally useful features of C are the __FILE__ and __LINE__ magic macros. These are built into the preprocessor, and expanded out before the C parser is run. Despite not having a preprocessor, Swift provides very similar functionality with similar names, but Swift works quite differently under the covers.

### Built-In Identifiers

As described in [the Swift programming guide](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/LexicalStructure.html), Swift has a number of built-in identifiers, including __FILE__, __LINE__, __COLUMN__, and __FUNCTION__. These expressions can be used anywhere and are expanded by the parser to string or integer literals that correspond to the current location in the source code. This is incredibly useful for manual logging, e.g. to print out the current position before quitting.

However, this doesn’t help us in our quest to implement assert(). If we defined assert like this:
```swift    
    func assert(predicate : @autoclosure () -> Bool) { 
        #if DEBUG
            if !predicate() {
                println("assertion failed at \(__FILE__):\(__LINE__)")
                abort()
            }
        #endif
    }
```

The above code would print out of the file/line location that implements assert() itself, not the location from the caller. That isn’t helpful.

### Getting the location of a caller

Swift borrows a clever feature from the D language: these identifiers expand to the location of the caller _when evaluated in a default argument list_. To enable this behavior, the assert() function is defined something like this:
```swift    
    func assert(condition: @autoclosure () -> Bool, _ message: String = "",
        file: String = __FILE__, line: Int = __LINE__) {
            #if DEBUG
                if !condition() {
                    println("assertion failed at \(file):\(line): \(message)")
                    abort()
                }
            #endif
    }
```

The second parameter to the Swift assert() function is an optional string that you can specify, and the third and forth arguments are defaulted to be the position in the caller’s context. This allows assert() to pick up the source location of the caller by default, and if you want to define your own abstractions on top of assert, you can pass down locations from its caller. As a trivial example, you could define a function that logs and asserts like this:
```swift    
    func logAndAssert(condition: @autoclosure () -> Bool, _ message: StaticString = "",
        file: StaticString = __FILE__, line: UWord = __LINE__) {
    
        logMessage(message)
        assert(condition, message, file: file, line: line)
    }
```

This properly propagates the file/line location of the logAndAssert() caller down to the implementation of assert(). Note that StaticString, as shown in the code above, is a simple String-like type used to store a string literal, such as one produced by __FILE__, with no memory-management overhead.

In addition to being useful for assert(), this functionality is used in the Swift implementation of the higher-level XCTest framework, and may be useful for your own libraries as well.

* Sep 9, 2014

##  [Swift Has Reached 1.0](https://developer.apple.com/swift/blog/?id=14)

On June 2, 2014 at WWDC, the Swift team finally showed you what we had been working on for years. That was a big day with lots of excitement, for us and for developers around the world. Today, we’ve reached the second giant milestone:

Swift version 1.0 is now GM.

You can now submit your apps that use Swift to the App Store. Whether your app uses Swift for a small feature or a complete application, now is the time to share your app with the world. It’s your turn to excite everyone with your new creations.

### Swift for OS X

Today is the GM date for Swift on iOS. We have one more GM date to go for Mac. Swift for OS X currently requires the SDK for OS X Yosemite, and when Yosemite ships later this fall, Swift will also be GM on the Mac. In the meantime, you can keep developing your Mac apps with Swift by downloading the beta of [Xcode 6.1](https://developer.apple.com/xcode/downloads/).

### The Road Ahead

You’ll notice we’re using the word “GM”, not “final”. That’s because Swift will continue to advance with new features, improved performance, and refined syntax. In fact, you can expect a few improvements to come in Xcode 6.1 in time for the Yosemite launch. Because your apps today embed a version of the Swift GM runtime, they will continue to run well into the future.

* Sep 3, 2014

##  [Patterns Playground](https://developer.apple.com/swift/blog/?id=13)

In Swift, a pattern is a way to describe and match a set of values based on certain rules, such as: 

    * All tuples whose first value is 0
    * All numbers in the range 1...5
    * All class instances of a certain type

The learning playground linked below includes embedded documentation and experiments for you to perform. Download it for an interactive experience that will give you a jump start into using patterns in your own apps.

This playground requires the latest beta version of Xcode 6 on OS X Mavericks or OS X Yosemite beta.

    * [Patterns.playground](https://developer.apple.com/swift/blog/downloads/Patterns.zip)

* Aug 26, 2014

##  [Optionals Case Study: valuesForKeys](https://developer.apple.com/swift/blog/?id=12)

This post explores how optionals help preserve strong type safety within Swift. We’re going to create a Swift version of an Objective-C API. Swift doesn’t really need this API, but it makes for a fun example.

In Objective-C, NSDictionary has a method -objectsForKeys:notFoundMarker: that takes an NSArray of keys, and returns an NSArray of corresponding values. From the documentation: “the _N_-th object in the returned array corresponds to the _N_-th key in [the input parameter] keys.” What if the third key isn’t actually in the dictionary? That’s where the notFoundMarker parameter comes in. The third element in the array will be this marker object rather than a value from the dictionary. The Foundation framework even provides a class for this case if you don’t have another to use: NSNull.

In Swift, the Dictionary type doesn’t have an objectsForKeys equivalent. For this exercise, we’re going to add one — as valuesForKeys in keeping with the common use of ‘value’ in Swift — using an extension:
```swift    
    extension Dictionary {
        func valuesForKeys(keys: [K], notFoundMarker: V) -> [V] {
            // To be implemented
        }
    }
```

This is where our new implementation in Swift will differ from Objective-C. In Swift, the stronger typing restricts the resulting array to contain only a single type of element — we can’t put NSNull in an array of strings. However, Swift gives an even better option: we can return an array of optionals. All our values get wrapped in optionals, and instead of NSNull, we just use nil.
```swift    
    extension Dictionary {
        func valuesForKeys(keys: [Key]) -> [Value?] {
            var result = [Value?]()
            result.reserveCapacity(keys.count)
            for key in keys {
                result.append(self[key])
            }
            return result
        }
    }
```

NOTE: Some of you may have guessed why a Swift Dictionary doesn’t need this API, and already imagined something like this:
```swift    
    extension Dictionary {
        func valuesForKeys(keys: [Key]) -> [Value?] {
            return keys.map { self[$0] }
        }
    }
```

This has the exact same effect as the imperative version above, but all of the boilerplate has been wrapped up in the call to map. This is great example why Swift types often have a small API surface area, because it’s so easy to just call map directly.

Now we can try out some examples:
```swift    
    let dict = ["A": "Amir", "B": "Bertha", "C": "Ching"]
    
    dict.valuesForKeys(["A", "C"])
    // [Optional("Amir"), Optional("Ching")]
    
    dict.valuesForKeys(["B", "D"])
    // [Optional("Bertha"), nil]
    
    dict.valuesForKeys([])
    // []
```

### Nested Optionals

Now, what if we asked for the last element of each result?
```swift    
    dict.valuesForKeys(["A", "C"]).last
    // Optional(Optional("Ching"))
    
    dict.valuesForKeys(["B", "D"]).last
    // Optional(nil)
    
    dict.valuesForKeys([]).last
    // nil
```

That’s strange — we have two levels of Optional in the first case, and Optional(nil) in the second case. What’s going on?

Remember the declaration of the last property:
```swift    
    var last: T? { get }
```

This says that the last property’s type is an Optional version of the array’s element type. In _this_ case, the element type is also optional (String?). So we end up with String??, a doubly-nested optional type.

So what does Optional(nil) mean?

Recall that in Objective-C we were going to use NSNull as a placeholder. The Objective-C version of these three calls looks like this:
```swift    
    [dict valuesForKeys:@[@"A", @"C"] notFoundMarker:[NSNull null]].lastObject
    // @"Ching"
    
    [dict valuesForKeys:@[@"B", @"D"] notFoundMarker:[NSNull null]].lastObject
    // NSNull
    
    [dict valuesForKeys:@[] notFoundMarker:[NSNull null]].lastObject
    // nil
```

In both the Swift and Objective-C cases, a return value of nil means “the array is empty, therefore there’s no last element.” The return value of Optional(nil) (or in Objective-C NSNull) means “the last element of this array exists, but it represents an absence.” Objective-C has to rely on a placeholder object to do this, but Swift can represent it in the type system.

### Providing a Default

To wrap up, what if you _did_ want to provide a default value for anything that wasn’t in the dictionary? Well, that’s easy enough.
```swift    
    extension Dictionary {
        func valuesForKeys(keys: [Key], notFoundMarker: Value) -> [Value] {
            return self.valuesForKeys(keys).map { $0 ?? notFoundMarker }
        }
    }  
    
    dict.valuesForKeys(["B", "D"], notFoundMarker: "Anonymous")
    // ["Bertha", "Anonymous"]
```

While Objective-C has to rely on a placeholder object to do this, Swift can represent it in the type system, and provides rich syntactic support for handling optional results.

* Aug 19, 2014

##  [Access Control and protected](https://developer.apple.com/swift/blog/?id=11)

The response to support for access control in Swift has been extremely positive. However, some developers have been asking, “Why doesn’t Swift have something like protected?” Many other programming languages have an access control option that restricts certain methods from being accessed from anywhere except subclasses.

When designing access control levels in Swift, we considered two main use cases:

* keep private details of a class hidden from the rest of the app
* keep internal details of a framework hidden from the client app

These correspond to private and internal levels of access, respectively.

In contrast, protected conflates access with inheritance, adding an entirely new control axis to reason about. It doesn’t actually offer any real protection, since a subclass can always expose “protected” API through a new public method or property. It doesn’t offer additional optimization opportunities either, since new overrides can come from anywhere. And it’s unnecessarily restrictive — it allows subclasses, but not any of the subclass’s helpers, to access something.

As some developers have pointed out, Apple frameworks do occasionally separate parts of API intended for use by subclasses. Wouldn’t protected be helpful here? Upon inspection, these methods generally fall into one of two groups. First, methods that aren’t really useful outside the subclass, so protection isn’t critical (and recall the helper case above). Second, methods that are designed to be overridden but not called. An example is drawRect(_:), which is certainly used within the UIKit codebase but is not to be called outside UIKit.

It’s also not clear how protected should interact with extensions. Does an extension to a class have access to that class’s protected members? Does an extension to a subclass have access to the superclass’s protected members? Does it make a difference if the extension is declared in the same module as the class?

There was one other influence that led us to the current design: existing practices of Objective-C developers both inside and outside of Apple. Objective-C methods and properties are generally declared in a public header (.h) file, but can also be added in class extensions within the implementation (.m) file. When parts of a public class are intended for use elsewhere within the framework but not outside, developers create a second header file with the class’s “internal” bits. These three levels of access correspond to public, private, and internal in Swift.

Swift provides access control along a single, easy-to-understand axis, unrelated to inheritance. We believe this model is simpler, and provides access control the way it is most often needed: to isolate implementation details to within a class or within a framework. It may be different from what you’ve used before, but we encourage you to try it out.

* Aug 15, 2014

##  [Value and Reference Types](https://developer.apple.com/swift/blog/?id=10)

Types in Swift fall into one of two categories: first, “value types”, where each instance keeps a unique copy of its data, usually defined as a struct, enum, or tuple. The second, “reference types”, where instances share a single copy of the data, and the type is usually defined as a class. In this post we explore the merits of value and reference types, and how to choose between them. 

###  What’s the Difference? 

The most basic distinguishing feature of a _value type_ is that copying — the effect of assignment, initialization, and argument passing — creates an _independent instance_ with its own unique copy of its data:

```swift
  // Value type example
  struct S { var data: Int = -1 }
  var a = S()
  var b = a       // a is copied to b
  a.data = 42       // Changes a, not b
  println("\(a.data), \(b.data)") // prints "42, -1"
```

Copying a reference, on the other hand, implicitly creates a shared instance. After a copy, two variables then refer to a single instance of the data, so modifying data in the second variable also affects the original, e.g.:

```swift  
  // Reference type example
  class C { var data: Int = -1 }
  var x = C()
  var y = x       // x is copied to y
  x.data = 42       // changes the instance referred to by x (and y)
  println("\(x.data), \(y.data)") // prints "42, 42"
```

### The Role of Mutation in Safety

One of the primary reasons to choose value types over reference types is the ability to more easily reason about your code. If you always get a unique, copied instance, you can trust that no other part of your app is changing the data under the covers. This is especially helpful in multi-threaded environments where a different thread could alter your data out from under you. This can create nasty bugs that are extremely hard to debug.

Because the difference is defined in terms of what happens when you change data, there’s one case where value and reference types overlap: when instances have no writable data. In the absence of mutation, values and references act exactly the same way.

You may be thinking that it could be valuable, then, to have a case where a class is completely immutable. This would make it easier to use Cocoa NSObject objects, while maintaining the benefits of value semantics. Today, you can write an immutable class in Swift by using only immutable stored properties and avoiding exposing any APIs that can modify state. In fact, many common Cocoa classes, such as NSURL, are designed as immutable classes. However, Swift does not currently provide any language mechanism to enforce class immutability (e.g. on subclasses) the way it enforces immutability for struct and enum.

### How to Choose?

So if you want to build a new type, how do you decide which kind to make? When you’re working with Cocoa, many APIs expect subclasses of NSObject, so you have to use a class. For the other cases, here are some guidelines:

Use a value type when:

* Comparing instance data with == makes sense
* You want copies to have independent state
* The data will be used in code across multiple threads

Use a reference type (e.g. use a class) when:

* Comparing instance identity with === makes sense
* You want to create shared, mutable state

In Swift, Array, String, and Dictionary are all value types. They behave much like a simple int value in C, acting as a unique instance of that data. You don’t need to do anything special — such as making an explicit copy — to prevent other code from modifying that data behind your back. Importantly, you can safely pass copies of values across threads without synchronization. In the spirit of improving safety, this model will help you write more predictable code in Swift.

* Aug 8, 2014

##  [Balloons](https://developer.apple.com/swift/blog/?id=9)

Many people have asked about the Balloons playground we demonstrated when introducing Swift at WWDC. Balloons shows that writing code can be interactive and fun, while presenting several great features of playgrounds. Now you can learn how the special effects were done with this tutorial version of ‘Balloons.playground’, which includes documentation and suggestions for experimentation.

This playground uses new features of SpriteKit and requires the latest beta versions of Xcode 6 and OS X Yosemite.

* [Balloons.playground](https://developer.apple.com/swift/blog/downloads/Balloons.zip)

* Aug 5, 2014

##  [Boolean](https://developer.apple.com/swift/blog/?id=8)

The boolean Bool type in Swift underlies a lot of primitive functionality, making it an interesting demonstration of how to build a simple type. This post walks through the creation of a new MyBool type designed and implemented to be very similar to the Bool type built into Swift. We hope this walk through the design of a simple Swift type will help you better understand how the language works.

Let’s start with the basic definition. The MyBool type models two different cases, perfect for an enum:

```swift  
  enum MyBool {
    case myTrue, myFalse
  }
```

To reduce confusion in this post, we’ve named the cases myTrue and myFalse. We want MyBool() to produce a false value, and can do so by providing an init method:

```swift  
  extension MyBool {
    init() { self = .myFalse }
  }
```

Swift enum declarations implicitly scope their enumerators within their body, allowing us to refer to MyBool.myFalse and even .myFalse when a contextual type is available. However, we want our type to work with the primitive true and false literal keywords. To make this work, we can make MyBool conform to the BooleanLiteralConvertible protocol like this:

```swift
  extension MyBool : BooleanLiteralConvertible {
    static func convertFromBooleanLiteral(value: Bool) -> MyBool {
    return value ? myTrue : myFalse
    }
  }

  // We can now assign 'true' and 'false' to MyBool.
  var a : MyBool = true
```

With this set up, we have our basic type, but we still can’t do much with it. Booleans need to be testable within an if condition. Swift models this with the BooleanType protocol, which allows any type to be used as a logical condition:

```swift
  extension MyBool : BooleanType {
    var boolValue: Bool {
      switch self {
      case .myTrue: return true
      case .myFalse: return false
      }
    }
  }

  // Can now test MyBool in 'if' and 'while' statement conditions.
  if a {}
```

We also want anything that conforms to BooleanType to be castable to MyBool, so we add:

```swift  
  extension MyBool {
    // MyBool can be constructed from BooleanType
      init(_ v : BooleanType) {
        if v.boolValue {
          self = .myTrue
        } else {
          self = .myFalse
        }
      }
  }

  // Can now convert from other boolean-like types.
  var basicBool : Bool = true
  a = MyBool(basicBool)
```

Note that the use of _ in the initializer argument list disables the keyword argument, which allows the MyBool(x) syntax to be used instead of requiring MyBool(v: x).

Now that we have basic functionality, let’s define some operators to work with it, starting with the == operator. Simple enums that have no associated data (like MyBool) are automatically made Equatable by the compiler, so no additional code is required. However, you can make arbitrary types equatable by conforming to the Equatable protocol and implementing the == operator. If MyBool weren’t already Equatable, this would look like this:

```swift
  extension MyBool : Equatable {
  }

  func ==(lhs: MyBool, rhs: MyBool) -> Bool {
    switch (lhs, rhs) {
      case (.myTrue,.myTrue), (.myFalse,.myFalse):
      return true
      default:
      return false
    }
  }

  // Can now compare with == and !=
  if a == a {}
  if a != a {}
```

Here we’re using some simple pattern matching in the switch statement to handle this. Since MyBool is now Equatable, we get a free implementation of the != operator. Lets add binary operations:

```swift  
  func &(lhs: MyBool, rhs: MyBool) -> MyBool {
    if lhs {
      return rhs
    }
    return false
  }

  func |(lhs: MyBool, rhs: MyBool) -> MyBool {
    if lhs {
      return true
    }
    return rhs
  }

  func ^(lhs: MyBool, rhs: MyBool) -> MyBool {
    return MyBool(lhs != rhs)
  }
```

With the basic operators in place, we can implement a variety of helpful unary and compound assignment operators as well, for example:

```swift
  prefix func !(a: MyBool) -> MyBool {
  return a ^ true
  }

  // Compound assignment (with bitwise and)
  func &=(inout lhs: MyBool, rhs: MyBool) {
    lhs = lhs & rhs
  }
```

The &= operator takes the left operand as inout because it reads and writes to it, and the effect must be visible to the user of the operator. Swift gives you complete control over mutability of operations on value types like enum and struct.

With this, the simple MyBool type has all of the basic operations and operators. Hopefully this post gives you a few tips that you can apply to your own code when defining higher-level types.

* Aug 1, 2014

##  [Files and Initialization](https://developer.apple.com/swift/blog/?id=7)

By now, most of you have written a small Swift app or experimented in the playground. You may even have experienced an error after you copied code from a playground into another file and wondered, “What is actually going on? What is the difference between a playground file, and other Swift source files?” This post will explain how Swift deals with the files in your project, and how global data is initialized.

### Files in an App

A Swift app is composed of any number of files, each with the functions, classes, and other declarations that make up the app. Most Swift files in your app are **order-independent**, meaning you can use a type before it is defined, and can even import modules at the bottom of the file (although that is not recommended Swift style.)

However, top-level code is not allowed in most of your Swift source files. For clarity, any executable statement not written within a function body, within a class, or otherwise encapsulated is considered top-level. We have this rule because if top-level code were allowed in all your files, it would be hard to determine where to start the program.

### Playgrounds, REPL, and Top-Level Code

You may be wondering why the code below works perfectly in a playground. This example isn’t encapsulated in anything, so it must be top-level code:

```swift  
  println("Hello world")
```

The above single-line program works — with no additional code at all — because playground files do support the execution of top-level code. Code within a playground file is **order-dependent**, run in top-down lexical order. For example, you can’t use a type before you define it. Of course, Swift playground files can also define functions, classes, and any other legal Swift code, but they don’t need to. This makes it easy to learn the Swift language or try a new API without writing a lot of code to get started.

In addition to playgrounds, top-level code can also be run in the REPL (Read-Eval-Print-Loop) or when launching Swift files as scripts. To use Swift for scripting, you can use shebang-style launching by starting your Swift file with “#!/usr/bin/xcrun swift” or type “xcrun swift myFile.swift” within Terminal.

### Application Entry Points and “main.swift”

You’ll notice that earlier we said top-level code isn’t allowed in _most_ of your app’s source files. The exception is a special file named “main.swift”, which behaves much like a playground file, but is built with your app’s source code. The “main.swift” file can contain top-level code, and the order-dependent rules apply as well. In effect, the first line of code to run in “main.swift” is implicitly defined as the main entrypoint for the program. This allows the minimal Swift program to be a single line — as long as that line is in “main.swift”.

In Xcode, Mac templates default to including a “main.swift” file, but for iOS apps the default for new iOS project templates is to add @UIApplicationMain to a regular Swift file. This causes the compiler to synthesize a main entry point for your iOS app, and eliminates the need for a “main.swift” file.

Alternatively, you can link in an implementation of main written in Objective-C, common when incrementally migrating projects from Objective-C to Swift.

### Global Variables

Given how Swift determines where to start executing an app, how should global variables work? In the following line of code, when should the initializer run?

```swift  
  var someGlobal = foo()
```

In a single-file program, code is executed top-down, similar to the behavior of variables within a function. Pretty simple. The answer for complex apps is less obvious, and we considered three different options:

* Restrict initializers of global variables to be simple constant expressions, as C does.
* Allow any initializer, run as a static constructor at app load time, as C++ does.
* Initialize lazily, run the initializer for a global the first time it is referenced, similar to Java.

The first approach was ruled out because Swift doesn’t need constant expressions like C does. In Swift, constants are generally implemented as (inlined) function calls. And there are good reasons to use complex initializers, e.g. to set up singletons or allocate a dictionary.

The second approach was ruled out because it is bad for the performance of large systems, as all of the initializers in all the files must run before the application starts up. This is also unpredictable, as the order of initialization in different files is unspecified.

Swift uses the third approach, which is the best of all worlds: it allows custom initializers, startup time in Swift scales cleanly with no global initializers to slow it down, and the order of execution is completely predictable.

The lazy initializer for a global variable (also for static members of structs and enums) is run the first time that global is accessed, and is launched as dispatch_once to make sure that the initialization is atomic. This enables a cool way to use dispatch_once in your code: just declare a global variable with an initializer and mark it private.

### Summary

Swift is designed to make it easy to experiment in a playground or to quickly build a script. A complete program can be a single line of code. Of course, Swift was also designed to scale to the most complex apps you can dream up. With “main.swift” you can take complete control over initialization or you can let @UIApplicationMain do the startup work for you on iOS.

* Jul 28, 2014

##  [Interacting with C Pointers](https://developer.apple.com/swift/blog/?id=6)

Objective-C and C APIs often require the use of pointers. Data types in Swift are designed to feel natural when working with pointer-based Cocoa APIs, and Swift automatically handles several of the most common use cases for pointers as arguments. In this post we’ll look at how pointer parameters in C can be used with the variables, arrays, and strings in Swift.

### Pointers as In/Out Parameters

C and Objective-C don’t support multiple return values, so Cocoa APIs frequently use pointers as a way of passing additional data in and out of functions. Swift allows pointer parameters to be treated like inout parameters, so you can pass a reference to a var as a pointer argument by using the same & syntax. For instance, UIColor’s getRed(_:green:blue:alpha:) method takes four CGFloat* pointers to receive the components of the color. We can use & to collect these components into local variables:

```swift  
  var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0, a: CGFloat = 0
  color.getRed(&r, green: &g, blue: &b, alpha: &a)
```

Another common case is the Cocoa NSError idiom. Many methods take an NSError** parameter to save an error in case of failure. For instance, we can list the contents of a directory using NSFileManager’s contentsOfDirectoryAtPath(_:error:) method, saving the potential error directly to an NSError? variable:

```swift  
  var maybeError: NSError?
  if let contents = NSFileManager.defaultManager()
    .contentsOfDirectoryAtPath("/usr/bin", error: &maybeError) {
      // Work with the directory contents
    } else if let error = maybeError {
    // Handle the error
  }
```

For safety, Swift requires the variables to be initialized before being passed with &. This is because it cannot know whether the method being called tries to read from a pointer before writing to it.

### Pointers as Array Parameters

Pointers are deeply intertwined with arrays in C, and Swift facilitates working with array-based C APIs by allowing Array to be used as a pointer argument. An immutable array value can be passed directly as a const pointer, and a mutable array can be passed as a non-const pointer argument using the & operator, just like an inout parameter. For instance, we can add two arrays a and b using the vDSP_vadd function from the Accelerate framework, writing the result to a third result array:

```swift  
  import Accelerate

  let a: [Float] = [1, 2, 3, 4]
  let b: [Float] = [0.5, 0.25, 0.125, 0.0625]
  var result: [Float] = [0, 0, 0, 0]

  vDSP_vadd(a, 1, b, 1, &result, 1, 4)

  // result now contains [1.5, 2.25, 3.125, 4.0625]
```

### Pointers as String Parameters

C uses const char* pointers as the primary way to pass around strings. A Swift String can be used as a const char* pointer, which will pass the function a pointer to a null-terminated, UTF–8-encoded representation of the string. For instance, we can pass strings directly to standard C and POSIX library functions:

```swift  
  puts("Hello from libc")
  let fd = open("/tmp/scratch.txt", O_WRONLY|O_CREAT, 0o666)

  if fd < 0 {
    perror("could not open /tmp/scratch.txt")
  } else {
    let text = "Hello World"
    write(fd, text, strlen(text))
    close(fd)
  }
```

### Safety with Pointer Argument Conversions

Swift works hard to make interaction with C pointers convenient, because of their pervasiveness within Cocoa, while providing some level of safety. However, interaction with C pointers is inherently unsafe compared to your other Swift code, so care must be taken. In particular:

* These conversions cannot safely be used if the callee saves the pointer value for use after it returns. The pointer that results from these conversions is only guaranteed to be valid for the duration of a call. Even if you pass the same variable, array, or string as multiple pointer arguments, you could receive a different pointer each time. An exception to this is global or static stored variables. You can safely use the address of a global variable as a persistent unique pointer value, e.g.: as a KVO context parameter.
* Bounds checking is not enforced when a pointer to an Array or String is passed. A C-based API can’t grow the array or string, so you must ensure that the array or string is of the correct size before passing it over to the C-based API.

If you need to work with pointer-based APIs that don’t follow these guidelines, or you need to override Cocoa methods that accept pointer parameters, then you can work directly with raw memory in Swift using unsafe pointers. We’ll look at a more advanced case in a future post.

* Jul 23, 2014

##  [Access Control](https://developer.apple.com/swift/blog/?id=5)

In Xcode 6 beta 4, Swift adds support for access control. This gives you complete control over what part of the code is accessible within a single file, available across your project, or made public as API for anyone that imports your framework. The three access levels included in this release are:

* private entities are available only from within the source file where they are defined.
* internal entities are available to the entire module that includes the definition (e.g. an app or framework target).
* public entities are intended for use as API, and can be accessed by any file that imports the module, e.g. as a framework used in several of your projects.

By default, all entities have internal access. This allows application developers to largely ignore access control, and most Swift code already written will continue to work without change. Your framework code does need to be updated to define public API, giving you total control of the exposed interface your framework provides.

The private access level is the most restrictive, and makes it easy to hide implementation details from other source files. By properly structuring your code, you can safely use features like extensions and top-level functions without exposing that code to the rest of your project.

Developers building frameworks to be used across their projects need to mark their API as public. While distribution and use of 3rd-party binary frameworks is not recommended (as mentioned in a previous blog post), Swift supports construction and distribution of frameworks in source form.

In addition to allowing access specification for an entire declaration, Swift allows the get of a property to be more accessible than its set. Here is an example class that is part of a framework:

```swift  
  public class ListItem {

    // Public properties.
    public var text: String
    public var isComplete: Bool

    // Readable throughout the module, but only writeable from within this file.
    private(set) var UUID: NSUUID

    public init(text: String, completed: Bool, UUID: NSUUID) {
      self.text = text
      self.isComplete = completed
      self.UUID = UUID
    }

    // Usable within the framework target, but not by other targets.
    func refreshIdentity() {
      self.UUID = NSUUID()
    }

    public override func isEqual(object: AnyObject?) -> Bool {
      if let item = object as? ListItem {
        return self.UUID == item.UUID
      }
      return false
    }
  }
```

When mixing Objective-C and Swift, because the generated header for a framework is part of the framework’s public Objective-C interface, only declarations marked public appear in the generated header for a Swift framework. For applications, the generated header contains both public and internal declarations.

For more information, [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/) and [Using Swift with Cocoa and Objective-C](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/) books have been updated to cover access control. [Read the complete Xcode 6 beta 4 release notes here](https://developer.apple.com/devcenter/download.action?path=/Developer_Tools/xcode_6_beta_4_o2p8fz/xcode_6_beta_4_release_notes.pdf).

* Jul 18, 2014

##  [Building assert() in Swift, Part 1: Lazy Evaluation](https://developer.apple.com/swift/blog/?id=4)

UPDATE: This post has been updated to reflect a change in Xcode 6 beta 5 that renamed @auto_closure to @autoclosure, and LogicValue to BooleanType.

When designing Swift we made a key decision to do away with the C preprocessor, eliminating bugs and making code much easier to understand. This is a big win for developers, but it also means Swift needs to implement some old features in new ways. Most of these features are obvious (importing modules, conditional compilation), but perhaps the most interesting one is how Swift supports macros like assert().

When building for release in C, the assert() macro has no runtime performance impact because it doesn’t evaluate any arguments. One popular implementation in C looks like this:

```swift  
  #ifdef NDEBUG
  #define assert(e)  ((void)0)
  #else
  #define assert(e)  \
    ((void) ((e) ? ((void)0) : __assert (#e, __FILE__, __LINE__)))
  #define __assert(e, file, line) \
    ((void)printf ("%s:%u: failed assertion `%s'\n", file, line, e), abort())
  #endif
```

Swift’s assert analog provides almost all of the functionality of C’s assert, without using the preprocessor, and in a much cleaner way. Let’s dive in and learn about some interesting features of Swift.

### Lazy Evaluation of Arguments

When implementing assert() in Swift, the first challenge we encounter is that there is no obvious way for a function to accept an expression without evaluating it. For example, say we tried to use:

```swift  
  func assert(x : Bool) {
    #if !NDEBUG

    /*noop*/
    #endif
  }
```

Even when assertions are disabled, the application would take the performance hit of evaluating the expression:

```swift 
  assert(someExpensiveComputation() != 42)
```

One way we could fix this is by changing the definition of assert to take a closure:

```swift
  func assert(predicate : () -> Bool) {
    #if !NDEBUG
    if !predicate() {
      abort()
    }
    #endif
  }
```

This evaluates the expression only when assertions are enabled, like we want, but it leaves us with an unfortunate calling syntax:

```swift 
  assert({ someExpensiveComputation() != 42 })
```

We can fix this by using the Swift @autoclosure attribute. The auto-closure attribute can be used on an argument to a function to indicate that an unadorned expression should be implicitly wrapped in a closure to the function. The example then looks like this:

```swift  
  func assert(predicate : @autoclosure () -> Bool) {
    #if !NDEBUG
    if !predicate() {
      abort()
    }
    #endif
  }
```

This allows you to call it naturally, as in:

```swift   
  assert(someExpensiveComputation() != 42)
```

Auto-closures are a powerful feature because you can conditionally evaluate an expression, evaluate it many times, and use the bound expression in any way a closure can be used. Auto-closures are used in other places in Swift as well. For example, the implementation of short-circuiting logical operators looks like this:

```swift  
  func &&(lhs: BooleanType, rhs: @autoclosure () -> BooleanType) -> Bool {
    return lhs.boolValue ? rhs().boolValue : false
  }
```

By taking the right side of the expression as an auto-closure, Swift provides proper lazy evaluation of that subexpression.

### Auto-Closures

As with macros in C, auto-closures are a very powerful feature that must be used carefully because there is no indication on the caller side that argument evaluation is affected. Auto-closures are intentionally limited to only take an empty argument list, and you shouldn’t use them in cases that feel like control flow. Use them when they provide useful semantics that people would expect (perhaps for a “futures” API) but don’t use them just to optimize out the braces on closures.

This covers one special aspect of the implementation of assert in Swift, but there is more to come.

* Jul 15, 2014

##  [Swift Language Changes in Xcode 6 beta 3](https://developer.apple.com/swift/blog/?id=3)

The Swift programming language continues to advance with each new Xcode 6 beta, including new features, syntax enhancements, and behavioral refinements. Xcode 6 beta 3 incorporates some important changes, a few of which we’d like to highlight:

* Array has been completely redesigned to have full value semantics to match the behavior of Dictionary and String. Now a let array is completely immutable, and a var array is completely mutable.
* Syntax “sugar” for Array and Dictionary has changed. Arrays are declared using [Int] as short hand for Array<Int>, instead of Int[]. Similarly, Dictionary uses [Key: Value] for Dictionary<Key, Value>.
* The half-open range operator has been changed from .. to ..< to make it more clear alongside the ... operator for closed ranges.

Xcode 6 beta is free to Registered Apple Developers and can be downloaded on the [Xcode downloads page](https://developer.apple.com/xcode/downloads/). Read all about these and other changes in the complete [release notes for Xcode 6 beta 3.](https://developer.apple.com/library/prerelease/ios/releasenotes/DeveloperTools/RN-Xcode/)

* Jul 11, 2014

##  [Compatibility](https://developer.apple.com/swift/blog/?id=2)

One of the most common questions we heard at WWDC was, “What is the compatibility story for Swift?”. This seems like a great first topic.

### App Compatibility

Simply put, if you write a Swift app today and submit it to the App Store this Fall when iOS 8 and OS X Yosemite are released, you can trust that your app will work well into the future. In fact, you can target back to OS X Mavericks or iOS 7 with that same app. This is possible because Xcode embeds a small Swift runtime library within your app’s bundle. Because the library is embedded, your app uses a consistent version of Swift that runs on past, present, and future OS releases.

### Binary Compatibility and Frameworks

While your app’s runtime compatibility is ensured, the Swift language itself will continue to evolve, and the binary interface will also change. To be safe, all components of your app should be built with the same version of Xcode and the Swift compiler to ensure that they work together.

This means that frameworks need to be managed carefully. For instance, if your project uses frameworks to share code with an embedded extension, you will want to build the frameworks, app, and extensions together. It would be dangerous to rely upon binary frameworks that use Swift — especially from third parties. As Swift changes, those frameworks will be incompatible with the rest of your app. When the binary interface stabilizes in a year or two, the Swift runtime will become part of the host OS and this limitation will no longer exist.

### Source Compatibility

Swift is ready to use today, in brand new apps or alongside your proven Objective-C code. We have big plans for the Swift language, including improvements to syntax, and powerful new features. And as Swift evolves, we will provide tools in Xcode to help you migrate your source code forward.

We can’t wait to see what you build!

* Jul 11, 2014

##  [Welcome to Swift Blog](https://developer.apple.com/swift/blog/?id=1)

This new blog will bring you a behind-the-scenes look into the design of the Swift language by the engineers who created it, in addition to the latest news and hints to turn you into a productive Swift programmer.

Get started with Swift by downloading [Xcode 6 beta](https://developer.apple.com/devcenter/download.action?path=/Developer_Tools/xcode_6_beta_3_lpw27r/xcode_6_beta_3.dmg), now available to all Registered Apple Developers for free. The Swift Resources tab has a ton of great links to videos, documentation, books, and sample code to help you become one of the world’s first Swift experts. There’s never been a better time to get coding!

\- The Swift Team 
