# The DIG Language
Simple efficient and safe

## Rationale
The goal is to provide a safe yet efficient portable alternative to C++, which is powerful yet easy to learn and use, and can transpile/cross-compile to virtually any other language. This is accomplished by taking the best features from modern programming languages and combining them, while removing extraneous syntax and dangerous options.

The hope is that Dig will remove the requirement of choosing a language when choosing a project. Just use Dig and later transpile/cross-compile to anything you like. This lets you to reuse your code in the future on other products that may not even run in the same kind of environment. Write less code to get the same work done.

## Features
* Simple recognizable syntax
* Transpile to efficient C++, Java, C#, and more
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
// 'struct' keyword is analagous to 'class'
struct Example extends Foo implements Bar {
  // notice the implicit type of Slice<Int> for someInts
  var someInts = Slice<Int>();
  
  // constructor
  fn Example() {
    foreach(Range(0,10)) {
      someInts.push(it);
    }
  }
  
  // notice the implicit return type of Int
  static fn factorial(x:Int) {
    if(x<=2) return 2;
    // tail call optimization converts this into a loop, allowing this entire function to be inlined
    return x*factorial(x-1);
  }

  // imperatively calculate the sum of the factorials of 1 through 10
  fn imperativeTest() {
    // notice the implicit type of Int for rv
    var rv=0;
    Range(1,11).foreach {
      rv+=it.factorial;
    }
    return rv;
  }

  // functionally/declaratively calculate the sum of the factorials of 1 through 10
  // because of inlining, this generates the same efficient code as imperativeTest
  fn functionalTest() Range(1,11).map(factorial).reduce(operator_plus)

  // rc means this type is reference-counted
  rc struct InnerRefCounted {
  }
  // rc means this type is deferred-reference-counted
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

// a 'discriminated union' that can be one of several subtypes and knows its current subtype
union Choice {
  Choice1,
  Choice2,
  Choice3 { var value:Int; }
}

static fn choiceTest(this:Choice):String {
  switch(this) {
    case Choice1: return "Chose 1";
    case Choice2: return "Chose 2";
    case Choice3 where value==0: return "Chose 3 but had no apples";
    case Choice3 where value==1: return "Chose 3 with one apple";
    case Choice3: return "Chose 3 with <<value>> apples";
  }
}

// alternative implementation
dispatch(Choice) fn test(this:Choice1) "Chose 1"
dispatch(Choice) fn test(this:Choice2) "Chose 2"
dispatch(Choice) fn test(this:Choice3) {
  switch(value) {
    case 0: return "Chose 3 but had no apples";
    case 1: return "Chose 3 with one apple";
    default: return "Chose 3 with <<value>> apples";
  }
}

static fn choiceTest2(this:Choice):String {
  // the proper test function is dynamically dispatched here, depending on choice's specific subtype
  return "The result is: <<this.test()>>";
}

// generic tree data structure
struct TreeNode<Key implements IComparable<Key>, Value> {
  var key:Key;
  var value:Value;
  var left:TreeNode;
  var right:TreeNode;
  // return value is an optional type - can be null
  fn find(needle:Key):TreeNode? {
    var result=needle.cmp(key);
    // note the use of the optional chaining operator - this will return null if left is null
    if(result<0) return left?.find(needle);
    if(result>0) return right?.find(needle);
    return this;
  }
}

// static extension method
static fn findNearest<Key implements IComparable<Key>,Value>(this:TreeNode<Key, Value>, needle:Key):TreeNode<Key,Value> {
  var result=needle.cmp(key);
  if(result<0) return left?left.find(needle):this;
  if(result>0) return right?right.find(needle):this;
  return this;
}

// method specialization (optimization for Strings that keeps track of the max character index compared so far)
static fn findNearest<String,Value>(this:TreeNode<String, Value>, needle:String):TreeNode<String,Value> {
  return this.findNearestHelper(needle);
}
static fn findNearestHelper<String,Value>(this:TreeNode<String, Value>, needle:String, fromCharIndex:Int=0):TreeNode<String,Value> {
  if(fromCharIndex>=needle.length || fromCharIndex>=key.length) return this;
  var result=needle.charAt(fromCharIndex)-key.charAt(fromCharIndex);
  var minLength=min(needle.length, key.length);
  while(minLength>fromCharIndex && needle.charAt(fromCharIndex)==key.charAt(fromCharIndex)) {
    fromCharIndex+=1;
  }
  
  // emit is a way to make expressions out of declarative statements like if, switch, etc. by having them emit values
  var result:Int = {
    if(minLength>fromCharIndex) {
      emit (fromCharIndex>needle.length ? -1 : 1);
    } else {
      emit (needle.charAt(fromCharIndex)<key.charAt(fromCharIndex)) ? -1 : 1;
    }
  }
  if(result<0) return left?left.find(needle, fromCharIndex):this;
  if(result>0) return right?right.find(needle, fromCharIndex):this;
  return this;  
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
* Verbose - very limited type inference, no null conditional operator (?.) etc
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
* Verbose - no type inference, no null conditional operator (?.), no automatic casts based on comparisons, etc
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




## Mechanisms

The following implementation details are used to provide memory safety, cross-target compatibility, and other related features:

* Object pooling for memory safety

    The allocator working in tandem with the compiler take special care to ensure that no reference can ever lead to an object of a different type than was expected. This, in turn, allows us to know that all member accesses are also valid. Manually freeable objects are always allocated from object pools, which guarantees that they can never contain references to an object of the wrong type. When such an instance is freed, its contained pointers are zeroed or pointed to a special dummy object of the proper type, and it goes back into its pool's freelist. Null pointers are normally checked before access, but we may relax this rule while forcing such accesses to hit a reserved low region of memory that is protected.
    
    A sweeping garbage collector may optionally be run very slowly on all of memory (including manually freeable object pools) to determine which manually freeable objects may be automatically freed. This can help clean up infrequent exceptional cases such as when an exception is triggered during a constructor, cyclic reference counts, or certain kinds of closures.
    
    The user can clear entire object pools when they become entirely freed, or can explicitly make a call to compact certain object pools. Compacting the pools takes time but the marking operation is limited by the compiler to only search objects that can be in the reference graph of the pools in question, as determined by which types contain what types of references. 
    
    Manually freed objects CAN be accessed accidentally if references to them remain. There are compilation modes to help detect these errors, and even if they occur no reference can ever exist to data of the wrong type. Therefore, pointers are always safe.

* Deferred reference counting

    Deferred reference counting has a significantly cheaper amortized cost than traditional RC. It ignores pointers on the stack and defers actual incrementing an decrementing of counters until such time as a deferred collection is called by the user. This may be done on a thread or at specific moments, such as between rendering frames. The deferred collection process may be invoked partially or completely.
    
* Arrays

    Arrays are currently always reference counted, since they are not well-suited to being object pooled and otherwise the user might quickly exhaust memory. This may change.

* Managed Target Environments

    Managed target environments use object pools to simulate manually freeable objects, similarly to how they are actually implemented on unmanaged targets like C++. Reference counted types are emulated via object pools and reference counts. Deferred reference counted types are gc'd.
    
* Value Types    
    
    Targets that do not inherently support Value types emulate Value types using multple stack variables and extra function parameters.
