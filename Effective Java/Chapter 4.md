# 4 Classes and Interfaces

## Item 15
- Minimize the accessibility of classes and members
- make each class or member as inaccessible as possible
- ***Information Hiding*** or ***Encapsulation***:
  - components communicate only through their APIs and are oblivious to each other's inner workings
- Four possible access levels (most restrictive to least restrictive):
  1. _private_: only accessible in the class it is defined
  2. _package-private_: also known as default access, member is accessible from any class in the package
  3. _protected_: accessible from subclasses of the class where it is defined + any class in the package
  4. _public_: accessible from anywhere
- by default everything should be declared as private, sometimes it is fine to make them package-private (like for testing) but moving to protected is a major change as they will become part of class's public API and must be supported forever!
- it is wrong to have a public static final array field (or an accessor that returns such field):
```Java
// potential security hole:
public static final Thing[] VALUES = {...};
//
// one alternative can be:
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

**Notes**:
- default access is always package-private except interfaces where it is public
- classes w/ public mutable fields are not generally thread-safe

## Item 16
- In public classes, use accessor methods not public fields
```java
// Encapsulation of data using getters/setters (accessor/mutators):
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

## Item 17
- Minimize mutability
- immutable class is one that its instances can not be modified
- all classes should be declared immutable unless there is a good reason to make them mutable
  - in this case we should limit its mutability as much as possible
- follow these rules to make your classes immutable:
  1. don't provide methods that modify the object (mutators)
  2. ensure that class can not be extended
  3. make all fields final
  4. make all fields private
  5. ensure exclusive access to any mutable components
    - e.g. if your class has a field referring to a mutable component, make sure it is private and when returning that through a getter, we can make a defensive copy
- some of the advantages of immutable classes:
  - They are very simple
    - they have exactly one state, the one that was assigned at creation time
  - inherently thread-safe, require no synchronization
  - can be shared freely
    - these classes should encourage clients to reuse existing instances whenever possible
    - they can even share their internals (like `BigInteger` that has `sign` and `magnitude`, negative value share the same `magnitude` and have different `sign`s)
  - they make great building blocks for other objects
    - specially as map keys or set elements
  - They provide failure atomicity for free
    - their state never change and there won't be any temporary inconsistencies or random failures!
  - you can use _lazy initialization_ on some of its fields
    - e.g. hash value of an immutable object never changes, so if its hash value is requested we calculate it once and if its requested again, we just return that same value moving forward (better performance)
- the major disadvantage is that they require a separate object for each distinct value
  - if you have a million bit BigInteger and want to change its low-order bit
  - there are different ways to cope with these issues, for example BigInteger has a companion package-private class that it uses to enhance its performance and memory usage while client can't see and use that mutable class.
- simple example of an immutable class:
```Java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.realPart(), im + c.imaginaryPart());
    }

    ...
}
```

## Item 18
- Favor composition over inheritance
- unlike method invocation, inheritance violates encapsulation
  - subclass depends on the implementation details of its super class
- it is safe to
  - use inheritance when it is within the same package (controlled by same programmer)
  - use inheritance when extending classes that are designed and documented for extension (_item 19_)
- in other cases, many things can go wrong that breaks your code, such as:
  - you wanna implement an instrumented HashSet that counts how many items have been added (not the size, including those removed). HashSet has `add()` and `addAll()` for which you can override and simply add number of elements to a counter each time any of these are called. Little you know that `addAll()` calls the `add()` behind the scene so if you add 2 elements to your set, counter will show 4!
  - sometimes they don't have these dependencies but in future updates the superclass may make these changes which still gives same input/output (valid updates) but it may break subclasses if they depend on these implementation details
  - let's say you have a collection with a few insert methods that you override them all to make them secure inserts and your clients use them. In future if that collection adds a new insert method your users can potentially use that new insert method and work around your security checks!
- These issues won't happen if both classes are controlled by one person (team) or when they are designed to be inherited. This won't also happen when the superclass provides implementation details so you can be careful when inheriting it.
- The best way to get around these issues and yet use the abilities of the superclass is to use **composition**. So instead of extending that class, you create a private instance of that class in your class and our methods can call the methods on this instance. (known as ***forwarding***)

**Notes**:
- Inheritance is appropriate only where the subclass is really a subtype of the superclass
  - when class B extends A, ask yourself is B really an A?

## Item 19
- Design and document for inheritance or else prohibit it
- the class must document its self-use of overridable methods
- use the java doc to include these details. Since Java 8 there are new tags including `@implSpec` that should be used of *Implementation Requirements*. The person overriding these methods should review these documents to make sure he is not breaking things.
- The only way to test a class designed for inheritance is to write subclasses
- Constructors must not invoke overridable methods.
  - this can lead to serious bugs, avoid it!
- designing class for inheritance is hard work! you have to document all of its self-use patterns and then commit to them for the life of the class. If you change them in future, you may break other ppl code!

## Item 20
- Prefer interfaces to abstract classes.
- as mentioned earlier inheritance has its own issues and when not necessary we better avoid them! In contrast interfaces don't have those issues and enable safe and powerful functionality enhancement
- A class can have one super class but can implement many interfaces
- default methods were introduced in Java 8 which let us change existing interfaces (still need to be careful). We add the new method with a default implementation, if it is not defined in the class implementing that interface, the default implementation will be used when needed.
- We can also combine an interface with an abstract class implementing that interface to maximize the benefits. These abstract classes are known as ***Skeletal Implementation***:
  - the way it works, you have your interface, you create an abstract class that implements that interface and provides implementation for the interface methods
  - now user can extend the skeletal implementation (their naming convention is Abstract*InterfaceName*) and for example only override one method and then use the default implementations in AbstractInterface for all other methods
  - if for example in another use case we need to override all the interface methods, user has the flexibility to just implement the interface directly (not using AbstractInterface) and override all methods
  - an example of this:

  ```Java
  /**
   * The Interface
   *
   */
  interface RedisConnection
  {
      int connect();
      boolean isConnected();
      int disconnect();
      int getDatabaseNumber();
  }

  /**
   * Abstract class which implements the interface.
   * This is called Abstract Interface known as Skeletal Implementation
   *
   */
  abstract class AbstractRedisConnection implements RedisConnection
  {
      @Override
      public final int connect()
      {
          //... lots of code to connect to Redis
      }

      @Override
      public final boolean isConnected()
      {
          //... code to check Redis connection
      }

      @Override
      public final int disconnect()
      {
          //... lots of code to disconnect from Redis and perform cleanup
      }
   }

  /**
   * A subclass which extends from the Abstract Interface
   *
   */
  class RedisOptOut extends AbstractRedisConnection {…}
  ```
  - here if we don't need to redefine those interface methods we just extend the `AbstractRedisConnection` and will have those default implementations.


## Item 21
  - Design interfaces for posterity
  - this item mostly talks about the default implementation of interface methods added in Java 8 (mainly to facilitate the use of lambdas)
  - although it is a powerful and helpful feature but we should be very careful and try not to change interfaces if possible
  - Java library has some changes to interfaces with new default methods that in some scenarios will break (like `removeIf` in `synchronizedCollection`)


## Item 22
  - Use interfaces only to define types
  - When a class implements an interface that should serve as a *type* that can be used to refer to instances of the class
  - there are some cases that interfaces are used to hold constants which is wrong and should be avoided:
  ```Java
  // bad use of interface:
  public interface PhysicalConstants {
      static final double AVOGADROS_NUMBER = 6.022_140_857e23;
      static final double BOLTZMANN_CONTANT = 1.380_648_52e-23;
      ...
  }
  ```
  - this constant patter is a poor use of interface. Alternatives are:
    1. make these constants part of the class that needs them (as a member)
    2. use an Enum
    3. have a utility class to keep them:

    ```Java
    package com.ashkan.science;

    public class PhysicalConstants {
        static final double AVOGADROS_NUMBER = 6.022_140_857e23;
        static final double BOLTZMANN_CONTANT = 1.380_648_52e-23;
    }
    ```
  then we can use these constants like `PhysicalConstants.AVOGADROS_NUMBER`. also if one class is heavily using these constants, in order to make it shorter and easier to use, we can use *static import*:
  ```Java
  import static com.ashkan.science.PhysicalConstants.*;

  public class Test {
      double atoms(double mols) {
          return AVOGADROS_NUMBER * mols;
      }
  }
  ```

**Note**:
- the underscore in numbers above (since Java 7) is used for readability and doesn't change the value of the numbers or break them.

## Item 23
- Prefer class hierarchies to tagged classes.
- tagged classes have a *tag* field that based on its value the class instance gets a different flavor. e.g.
```Java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    final Shape shape; // tag field

    // these fields used only if it's RECTANGLE:
    double length;
    double width;

    // this field used only if shape is CIRCLE:
    double radius;

    /*
    rest is omitted for brevity: basically different constructors based on Shape and area() method with a switch(shape) statement that behaves differently based on shape, so on and so forth
    */
}
```
- these classes are verbose, error-prone and inefficient (specially memory wise). They violate SRP and they are basically pallid imitation of class hierarchy
- avoid these type of classes and replaces them with proper hierarchies.

## Item 24
- Favor static member classes over non-static
- a nested class should exist to serve its enclosing class.
- if a nested class will be useful in some other context, then it should be a top-level class
- there are four kinds of nested classes:
  1. *static member classes*
    - one common use case is a helper class that can be used in conjunction w/ its outer class. Like `Calculator` class can have `Operation` class inside and we can refer to its operations like `Calculator.Operation.PLUS`.
    - if you declare a class that does not need access to its enclosing instance, make it static.
  2. *non-static member classes*
    - these classes are associated w/ their enclosing class and need to be instantiated. One common use of these non-static member classes is to define an ***Adapter***: this lets an instance of the enclosing class to be viewed as an instance of some other class (like viewing `Map`'s values or keys as a *collection view*)
  3. *anonymous classes*
    - they can be used anywhere that expressions are allowed
    - they have limitations, such as can not be instantiated or we can't use `instanceof` on them
    - before lambdas, they were more useful for creating small function objects and process objects but now wherever possible, we should use lambdas
  4. *local classes*
    - they are the least frequently used
- these four types should be considered 1 to 4, most to least preferable!

## Item 25
- Limit source files to a single top-level class
- Java let us define multiple top-level classes or interfaces in a single file. This item basically says never do this, it is confusing, messy and can result in errors depending on how you execute your program:

```Java
// Animal.java —don't do this
public class Dog {
    ...
}

public class Cat {
    ...
}
```
