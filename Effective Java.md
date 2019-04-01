# Item 1
- Use static factory methods instead of constructors
- _static factory method_: a static method that returns an instance of the class
- a few examples:
```java
  FileStore fileStore = Files.getFileStore(path);
  Date date = Date.from(instant);
  BufferedReader br = Files.newBufferedReader(path);
```
- benefits:
  - unlike constructors, they have names
    - compare this constructor: `BigInteger(int, int, Random)` w/ this static factory method: `BigInteger.probablePrime(int, int, Random)`. Here it is more clear that we are returning a BigInteger that is probably prime!
  - they don't create a new object each time they are called
    - e.g. `Boolean.valueOf(boolean)` doesn't create a new Boolean object  (this allows immutable classes —item 17)
  - Unlike constructors, they can return any subtype of their return type (the object)
    - e.g., `EnumSet` has only static factories which return `RegularEnumSet` if there are less that 64 elements, otherwise they return `JumboEnumSet` which is backed by a long array. User doesn't need to know about this and just uses it!

# Item 2
- Use builder when faced w/ many constructor parameters.
- maybe for more than 4 parameters in a constructor! or when you know you'll have more class members in future
- Compare these two:
```java
 NutritionFacts cocaCola = new NutritionFacts(270, 80, 100, 0, 35, 27);
 // vs.
 NutritionFacts cocaCola = new NutritionFacts.Builder(270, 80).
                                calories(100).sodium(35).carbohydrate(27).build();
```
  - as you can see, builder is more readable, arguments are more clear (we know 100 is for calories), we don't need to pass unnecessary parameters for those we don't have a value for (passing default values like '0') and it is more scalable (imagine we had 14 parameters instead of 6!!)
- Simple implementation of the above builder pattern:
  ```java
  public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;

    public class Builder {
      // Required parameters
      private final int servingSize;
      private final int servings;

      // Optional parameters, initialized to default values:
      private int calories = 0;

      public Builder(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings = servings;
      }

      public Builder calories(int val) {
        calories = val;
        return this;
      }

      public NutritionFacts build() {
        return new NutritionFacts(this);
      }
    }

    private NutritionFacts(Builder builder) {
      servingSize = builder.servingSize;
      servings = builder.servings;
      calories = builder.calories;
    }
  }
  ```

  # Item 3

  - Enforce the singleton property with a private constructor *or* an enum type (preferred!)
  - a *singleton* is a class that is instantiated exactly once
  - try on of these approaches:
  ```java
  // singleton w/ static factory (item 1)
  public class Elvis {
      private static final Elvis INSTANCE = new Elvis();

      private Elvis() {...}

      public static Elvis getInstance() {
        return INSTANCE;
      }

      // rest of class implementation...
  }
  ```
  OR:

    ```java
    // Enum singleton (preferred):
    public enum Elvis {
      INSTANCE;

      // rest of implementation...
      public void leaveTheBuilding() {...}
    }
    ```
  - only limitation w/ enum type is you can't use enum if you need to extend a superclass

# Item 4
- Enforce *non-instantiability* with a private constructor
- sometimes we need a class that is a collection of static methods (like `java.util.Arrays`)
- these classes shouldn't have a constructor (not to be instantiated). not writing any constructor will create the default constructor. the solution is a private constructor like this:
```java
public class UtilityClass {
    private UtilityClass() {
      throw new AssertionError();
    }
    ...
}
```
- the only side effect is that this prevents the class from being subclassed, since the subclass would not have an accessible superclass constructor to invoke.

# Item 5
- Prefer dependency injection to hardwiring resources
- not every class needs to be singleton or noninstantiable. sometimes classes behavior is parameterized by underlying resources. In these cases we need to pass the resource to the class
- One common way of doing this is passing the resource through constructor when creating a new instance
```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
      this.dictionary = Objects.requireNonNull(dictionary);
    }
    ...
}
```
- DI provides flexibility and testability
- in big projects w/ lots of dependencies, handling this can be a bit problematic. In this case *dependency injection frameworks* can be used to eliminate the clutter
  - such as Guice, Dagger or Spring

# Item 6
- Avoid creating unnecessary objects
- this basically says don't create a new object when you can reuse an existing one!
- One simple example:
```java
String s = new String("hello"); // don't do this!
String s = "hello"; // this will reuse the string instead of creating a new one
```

# Item 7
- Eliminate obsolete object references
- Java has garbage collection but in some situation we may leave an unnecessary reference to an object which prevents it from being garbage collected!
- for example look at this stack implementation:
```Java
public class Stack {
    private Object[] elements;
    private int size = 0;
    // ...

    public void push(Object element) {
        elements[size++] = element;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }
    // ...
}
```
  - in this example, our stack cares about the first "`size`" elements of the stack although other elements are still referring to objects. Java has no idea about that those references are not needed. So it won't be garbage collected and we will have a memory leak here.
  - in this case we can do this `elements[size] = null;` to eliminate the obsolete reference.
  - another benefit of doing this is that if by mistake we dereference the object, it will throw an `NPE`


# Item 8
- Avoid finalizers and cleaners
- as of Java 9, finalizers have been deprecated and replaced w/ cleaners
- they are somehow like destructors in C++ and used to perform some cleanup before garbage collection happens for a particular object
- they are both problematic, execution is not guaranteed, they can cause deadlocks
- finalizers have even more issues, security issues (known as *finalizer attacks*) and they ignore exceptions!
- what should we do instead?
  - best approach is to make the class implement `AutoClosable` and require its clients to call `close()` or even better use them in `try-with-resource` blocks

# Item 9
- Prefer `try-with-resources` to `try-finally`
- there are many java resources that must be closed manually
  - `InputStream`, `BufferedReader`, `java.sql.Connection`, etc.
- in these situations consider using `try-with-resources` because:
  - it is more concise and readable (specially if there are more than one resources)
  - in `try-finally` if the code in `try` throws an exception and then the code in `finally` throws another exception, there is no record of the first exception in the exception stack trace
- to be usable w/ this construct, a resource must implement `AutoClosable`
  - many classes and interfaces in Java libraries implement or extend `AutoClosable`
  - if you write a class that represents a resource that needs to be closed, make sure you implement `AutoClosable` too.
  ```Java
  static void copy(String src, String dst) throws IOException {
      try (InputStream in = new FileInputStream(src);
           OutputStream out = new FileOutputStream(dst)) {
               byte[] buffer = new byte[BUFFER_SIZE];
               int n;
               while ((n = in.read(buffer)) >= 0)
                   out.write(buffer, 0, n);
           }
  }
  ```

# Item 10
- Obey the general contract when overriding equals
- first make sure that you need to do that! in these cases you don't need to override it:
  - each instance of the class is inherently unique (like `Thread`)
  - there is no need for the class to provide a "logical equality" test
    - like `java.util.regex.Pattern`
  - a superclass has already overriden `equals`
    - e.g. most `Set` implementations inherit their `equals` from `AbstractSet`
- if you override, you must adhere to its general contract:
  1. *Reflexive*: x.equals(x) must return true
  2. *Symmetric*: x.equals(y) is true if and only if y.equals(x) is true
  3. *Transitive*: x.equals(y) and y.equals(z) then x.equals(z) must be true as well
  4. *Consistent*: x.equals(y) must consistently return true or false
  5. for non-null reference value x, x.equals(null) must return false
- usually it will be easy to follow this contract but if this doesn't happen we may break so many functionalities with _Collections_ and other things.
- There is no way to extend an instantiable class and add a value component while preserving the equals contract
  - one solution can be using composition instead of inheritance, or:
  - having an abstract class and overriding its sub-classes
- When you have to override `equals` consider the following:
  1. for performance optimization use `==` to check if the argument is a reference to this object (if so, return `true`)
  2. use `instanceof` to check if the argument has the correct type (if not, return `false`)
  3. cast the argument to the correct type

# Item 11
- You must override `hashCode` when you override `equals`
- general contract for `hashCode`:
  1. invoking multiple times should always return same hash value
  2. if two objects are equal (according to `equals`) then calling `hashCode` on them should return same integer value
  3. it is not required to return different hash values for different objects, but doing so will help with performance
- if you don't override `hashCode` when you override `equals`, you can break item 2 above
- to override the hashCode, you have to calculate the value starting from the first significant field, you add the calculated value of each field and result would be the hash value for that object
- define an integer called `result` and for each field, based on its type calculate the value like this:
  - if field is a primitive type, use `Type.hashCode(field)`
  - if field is an object, use `hashCode()` on that function. (if it is null, use 0)
  - for arrays, if all fields are significant, use `Arrays.hashCode`, else calculate the value for each item in the array based on its type (one of the previous 2 steps)
- add the calculated hash code `c` for each field into `result`:
  - `result = 31 * result + c;`
- based on above explanations, for `PhoneNumber` class (has areaCode, prefix and lineNum) you can do this:
```Java
@override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```
- if performance is not a concern we can use `Objects.hash()` (previous example is still better):
```Java
@override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```
#### Notes
- the number `31` above: it is an odd prime, if it was even and multiplication overflowed, information would be lost. advantage of using a prime is less clear but it's traditional. also for the sake of optimization: `31 * i == (i << 5) - i`
- write tests for your hashCode, for example make sure multiple calls on the same object returns same value. if you use AutoValue you don't need testing it.

# Item 12
- always override `toString`
- not as critical as last two items, but very useful and important
  - used by default when printing the object, logging, debugging, asserting, ...
- by default all objects will return `objectName` + `@` + `hashValue` (e.g. `PhoneNumber@158b39`)
- general contract for `toString` is: a concise but informative representation that is easy to read
  - e.g. for `PhoneNumber` we can return `416-421-7373`
- when practical, `toString` should return all of the interesting information, otherwise a meaningful summary
- always try to specify the format in a comment when overriding `toString`. if it is subject to change, specify that (like you add more fields in future or just change the format):

```Java
/**
* Returns the String representation of this phone number.
* The string contains 12 characters whose format is ....
*/
@override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```
#### Notes
- if you show something in `toString`, provide programmatic access to it so user doesn't have to parse the string

# Item 13
- Override `clone` judiciously
- if a class implements `Cloneable`, Object's clone method returns a field-by-field copy of the object ow it throws `CloneNotSupportedExeption`
  - highly atypical use of interface! instead of defining behavior for class, changes behavior of a protected method on a superclass
- a class implementing `Cloneable` is expected to provide a proper `Clone` method
- always call `super.clone();` first. If your class only contains primitive values or reference to immutable objects, that's all you need:
```Java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedExeption e) {
        throw new AssertionError(); // Can't happen
    }
}
```
- but if class has other types, you have to fix them in Clone method after calling `super.clone()`. For example in `Stack` example that we have an array of object (`Object[] elements;`), calling `super.clone()` returns a replica in which the `elements` refers to the same array as the original, in this case we can do this:
```Java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedExeption e) {
        throw new AssertionError();
    }
}
```
- For more complicated objects it will get harder to create a proper copy:
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            ...
        }

        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i=0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = buckets[i].deepCopy();
                }
                return result;
            }
        } catch(...) {...}
    }
}
```
- a better approach to object copying is to provide a copy constructor or copy factory
```Java
public Yum(Yum yum) {...}  // copy constructor
public static Yum newInstance(Yum yum) {...} // copy factory
```

# Item 14
- Consider implementing `Comparable`
  - when you implement a value class that has a sensible ordering
- by implementing it, a class indicates that its instances have a natural ordering
- by implementing Comparable, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on it
- the general contract for `compareTo` is similar to that of `equals`
  - `sgn(x.compareTo(y))` == `-sgn(y.compareTo(x))`
    - if one throws exception the other one throws exception as well
  - `x.compareTo(y) > 0` and `y.compareTo(z) > 0` ==> `x.compareTo(z) > 0`
  - `x.compareTo(y) == 0` ==> `sgn(x.compareTo(z))` == `sgn(y.compareTo(z))` for all z
- if a class has multiple significant fields, start from the most to the least significant:
```Java
public int compareTo(PhoneNumber phoneNumber) {
    int result = Short.compare(areaCode, phoneNumber.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, phoneNumber.prefix);
        if (result == 0) {
            result = Short.compare(lineNum, phoneNumber.lineNum);
        }
    }
    return result;
}
```
- Comparator based on static compare method:
```Java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode, o2.hashCodeOrder);
    }
};
```
**Notes**
- in implementations of `compareTo` never use `<` or `>` and always use static compare methods

# Item 15
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

# Item 16
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

# Item 17
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

# Item 18
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

# Item 19
- Design and document for inheritance or else prohibit it
- the class must document its self-use of overridable methods
- use the java doc to include these details. Since Java 8 there are new tags including `@implSpec` that should be used of *Implementation Requirements*. The person overriding these methods should review these documents to make sure he is not breaking things.
- The only way to test a class designed for inheritance is to write subclasses
- Constructors must not invoke overridable methods.
  - this can lead to serious bugs, avoid it!
- designing class for inheritance is hard work! you have to document all of its self-use patterns and then commit to them for the life of the class. If you change them in future, you may break other ppl code!

# Item 20
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


  # Item 21
  - Design interfaces for posterity
  - this item mostly talks about the default implementation of interface methods added in Java 8 (mainly to facilitate the use of lambdas)
  - although it is a powerful and helpful feature but we should be very careful and try not to change interfaces if possible
  - Java library has some changes to interfaces with new default methods that in some scenarios will break (like `removeIf` in `synchronizedCollection`)


  # Item 22
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

# Item 23
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

# Item 24
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

# Item 25
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

# Item 26
- Don't use raw types
- a class/interface whose declaration has _type parameters_ is a generic class/interface
  - `List<E>` --> list of E (elements of type E)
  - `List<String>` --> list of String (elements of type String)
  - `List` --> raw type: list can contain anything (old Java)
- main issue of raw types is we may get run-time exceptions. With generic types we will get compile-time exception if we attempt to put an incompatible type into our collection.
  ```Java
  // Raw collection type - don't do this:
  private final Collection stamps = ...;
  ```
  - here we can add something other than `Stamp` to this collection but later on if do something like `Stamp stamp = (Stamp) stamps.get(0);` and its first element is not `Stamp` it will throw `ClassCastException`
  - also we have to cast everything that we retrieve from this raw type collection. The correct way of declaring that collection is:
  ```Java
  private final Collection<Stamp> stamps = ...;
  ```
- Wildcard types are type arguments in the form `<?>` which can have lower bound and upper bound. They are unknown types and give us some flexibility when working with types that we don't know and at the same time type safety so if we have `List<?> myList` then the only thing that can be added to this list is `null` (which is member of every type).

# Item 27
- Eliminate unchecked warnings
- When working w/ generics you see many unchecked warnings. Eliminate every unchecked warning that you can and suppress the rest.
- When you can't eliminate the warning but you can prove that the code generating it is type safe, suppress it with `@SupressWarnings("unchecked")`
  - use it on smallest scope possible
  - don't use it on entire class or a method declaration
- every time you use this tag, explain in a comment why it is safe to do so
```Java
// Adding local variable to reduce scope of @SuppressWarnings
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // This cast is correct because the array we're creating
        // is of the same type as the one passed in, which is T[].
        @SuppressWarnings("unchecked") T[] result =
            (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

# Item 28
- Prefer lists to arrays
- arrays are *covariant*, generics are *invariant*
  - ***Covariant***: if *Sub* is a subtype of *Super*, then the array type `Sub[]` is a subtype of `Super[]`
  - ***invariant***: for any two distinct types `Type1` and `Type2`, `List<Type1>` is neither subtype nor a supertype of `List<Type2>`
- arrays are deficient:
```Java
// fails at runtime
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in!"; // throws ArrayStoreException
```
but this will give you compile time error:
```Java
// won't compile
List<Object> objectList = new ArrayList<Long>(); // incompatible types
```
- arrays and generics do not mix well, these expressions are illegal:
  - `List<E>[]`, `new List<String>[]` or `new E[]`
  - the reason is they are not type safe
