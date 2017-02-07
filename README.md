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
* Templates that may or may not compile properly depending on the type arguments provided
* Long iteration times due to slow compile speed
* Having to implement object pooling yourself in order to avoid hiccups in garbage collected environments

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
    if(x==0) return x;
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

union Choices {
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
