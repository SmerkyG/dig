# The DIG Language
Simple efficient and safe

## Rationale
The goal is to provide a safe yet efficient portable alternative to C++, which is powerful yet easy to learn and use, and can cross compile to virtually any other language. This is accomplished by taking the best features from modern programming languages and combining them, while removing extraneous syntax and dangerous options.

The hope is that Dig will remove the requirement of choosing a language when choosing a project. Just use Dig and later cross compile to anything you like. This lets you to reuse your code in the future on other products that may not even run in the same kind of environment. Write less code to get the same work done.

## Features
* Simple recognizable syntax
* Cross-compile to efficient C++, Java, C#, and more
* Type inference system - strong type safety guarantees with less code
* Lambda functions, with optional automatic inlining when passed as a function argument
* Algebraic Data Types w/ direct support for Tagged Unions
* Extension methods and UMC (Universal Method Call) syntax e.g. `foo(a, b)` is the same as `a.foo(b)`
* Stack allocable and embeddable complex Value-type support
* Syntactic support for both Optionally NULL and Definite types
* Pattern matching
* Type generics that compile quickly to efficient output
* Function generics
* Partial specialization for both type and function generics
* Function overloads e.g. foo(i:Int) can coexist with foo(b:Boolean)
* Function default parameters e.g. foo(i:Int = 4, b:Bar? = null)
* Member setter/getter functions
* Operator overloading without weird syntax
* Object Pool based memory allocator
* Optional Temporary Arena memory allocator
* Optional reference counting
* Optional user-controlled marking garbage collection
* String interpolation
* Multiline strings
* Simple efficient reflection
* Slice and Range based iteration and array manipulation - efficient, yet succinct and functionally chainable
* Standard Libraries that elegantly and efficiently encapsulate common algorithms and data structures
* Dynamic length object support e.g. represent blocks in a file such as a PNG without giving up memory safety
* Deep macro system
* Aggressive inlining support allows you to write reusable code that still compiles to an efficient implementation
* Generates C++ code that compiles very quickly even for large codebases that use advanced features like generics

## Things it avoids
* Inefficient abstractions
* Complex syntax
* Accidental numeric type conversion
* Pointless syntactic differences between * and ->, as well as excess pointer/reference type naming like X*, X&, and X&&
* Templates that may or may not compile properly depending on the type arguments provided
* Long iteration times due to slow compile speed
* Having to write your own allocator just to support a large number of objects
* Having to implement object pooling yourself in order to avoid hiccups in garbage collected environments for large numbers of objects
* Major tradeoffs between ease of use and efficiency

## Example code

```
struct Example extends Foo implements Bar {
  var someInts = Slice<Int>();
  fn Example() {
    foreach(Range(0,10)) {
      someInts.push(it);
    }
  }
  
  static fn factorial(x:Int) {
    if(x<=2) return 2;
    // tail call optimization converts this into a loop, allowing this entire function to be inlined
    return x*factorial(x-1);
  }

  // imperatively calculate the sum of the factorials of 1 through 10
  fn imperativeTest() {
    var rv=0;
    Range(1,11).foreach {
      rv=rv+it.factorial;
    }
    return rv;
  }

  // functionally/declaratively calculate the sum of the factorials of 1 through 10
  // because of inlining, this generates the same efficient code as imperativeTest
  fn functionalTest() Range(1,11).map(factorial).reduce(operator_plus)  
  
  rc struct InnerRefCounted {
  }
  defer_rc struct InnerDeferRefCounted {
    var other = InnerRefCounted();
  }
  fn testRC() {
    {
      var rcd = InnerRefCounted();
      // rcd is freed here
    } 
    {
      var drcd = InnerDeferRefCounted();
      // drcd and its member 'other' are freed whenever we next ask the deferred refcounting service to test the stack
    }
  }
}

union Choice {
  Choice1,
  Choice2,
  Choice3 { var value:Int; }
}

static fn choiceTest(choice:Choice):String {
  switch(choice) {
    case Choice1: return "Chose 1";
    case Choice2: return "Chose 2";
    case Choice3 where value==0: return "Chose 3 but had no apples";
    case Choice3 where value==1: return "Chose 3 with one apple";
    case Choice3: return "Chose 3 with <<value>> apples";
  }
}

// alternative implementation
dispatch(Choice) fn test(choice:Choice1) "Chose 1"
dispatch(Choice) fn test(choice:Choice2) "Chose 2"
dispatch(Choice) fn test(choice:Choice3) {
  switch(value) {
    case 0: "Chose 3 but had no apples";
    case 1: "Chose 3 with one apple";
    default: "Chose 3 with <<value>> apples";
  }
}

static fn choiceTest2(choice:Choice):String {
  return "The result is: <<choice.test()>>";
}

```

## Why not just use language X

Here's why we prefer the promise of Dig to each of the following languages:

C#
----------
* Requires a large runtime that is not available on all platforms
* Mono has major drawbacks
* Poor inlining support
* Cannot avoid GC
* GC workarounds are painful to implement/refactor towards
* Poor/no macro support
* No support for non-nullable references
* Difficult interoperability with systems languages and native libraries

C++
----------
* No safety guarantees whatsoever
* Easy to accidentally write to memory you shouldn't, including freed memory regions
* Cross platform compatibility is possible but difficult and time consuming to implement, test and maintain
* Lots of things that the compiler could take care of have to be done manually by the programmer, leading to bloated code, reduced safety, and increased refactoring and testing time
* Easy to accidentally leave memory uninitialized that should have been initialized
* Hugely varied syntax with excess specifiers and many ways to do the same thing means it's slow to read other people's code (or even your own older code)
* Missing language features like safe discriminated unions can be emulated but are difficult and error prone
* Missing language features like extended dispatching and pattern matching
* Templates are a nightmarish version of Generics that are not even guaranteed to compile for a given set of type parameters
* Hideous lambda syntax with complex highly error-prone closure cleanup considerations
* No null safety, not even a standardized way to easily tell if a pointer might be able to be null
* Slow compile times unless you invest heavily in making your code fit very specific and limiting patterns (Dig emits C++ code that does not suffer from the problems that often lead to slow compilation)
* Verbose - no null conditional operator (?.) etc
* Macro system that results in error-prone hard to debug and hard to read code
* Memory management is slow by default, and requires specialized implementations to be reasonable for large numbers of allocations
* Difficult interoperability with garbage collected languages

C
----------
* No safety guarantees whatsoever
* Easy to accidentally write to memory you shouldn't, including freed memory regions
* Cross platform compatibility is possible but difficult and time consuming to implement, test and maintain
* Lots of things that the compiler could take care of have to be done manually by the programmer, leading to bloated code, reduced safety, and increased refactoring and testing time
* Easy to accidentally leave memory uninitialized that should have been initialized
* Missing major modern language features, including major code safety and reuse features like generics and dynamic type checks
* Extremely simplified type system
* Verbose
* Memory management is slow by default, and requires specialized implementations to be reasonable for large numbers of allocations
* Difficult interoperability with garbage collected languages

Java
-----------
* Requires a large runtime that is available on many but not all platforms (such as Game Consoles)
* Poor inlining support
* Cannot avoid GC
* GC workarounds are painful to implement/refactor towards
* Poor/no macro support
* No support for non-nullable references
* Lack of Value-type support leads to heavy dependence on the garbage collector
* Type-erasure based generics do not allow many of the most desirable uses for generics
* Slow runtime performance compared to systems languages
* No multiline strings or string interpolation
* Verbose - no null conditional operator (?.), no automatic casts based on comparisons, etc
* Difficult interoperability with systems languages and native libraries
* Lack of support for static (or non-static) extension methods

Haxe
-----------
* C++ target requires a runtime that may not be available on your target platform and may be inappropriate for embedded systems
* No inlining support for lambdas
* Cannot avoid GC
* GC workarounds are painful to implement/refactor towards
* No support for non-nullable references
* Lack of cross-target Value-type support leads to heavy dependence on the garbage collector
* Debugging is unpleasant for anything other than SWF target
* Slow runtime performance compared to systems languages
* Same code leads to different results in different target languages (including errors and differently initialized data!)
* Bugs in desirable core features like @generic
* Verbose - no null conditional operator (?.), no automatic casts based on comparisons, etc
* Complex and uninituitive language specification (e.g. method accessibility modifiers, abstracts, getter/setter/underlying value)
* Iteration mechanism is low performance
* Slow compile times for C# target
* Lack of support for function overloading
* Lack of support for partial specialization for generics
* Widely varied target implementation quality (e.g. optional function parameters in C#)
