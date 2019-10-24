# 2 Creating and Destroying Objects

## Item 1
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

## Item 2
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

## Item 3

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

## Item 4
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

## Item 5
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

## Item 6
- Avoid creating unnecessary objects
- this basically says don't create a new object when you can reuse an existing one!
- One simple example:
```java
String s = new String("hello"); // don't do this!
String s = "hello"; // this will reuse the string instead of creating a new one
```

## Item 7
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


## Item 8
- Avoid finalizers and cleaners
- as of Java 9, finalizers have been deprecated and replaced w/ cleaners
- they are somehow like destructors in C++ and used to perform some cleanup before garbage collection happens for a particular object
- they are both problematic, execution is not guaranteed, they can cause deadlocks
- finalizers have even more issues, security issues (known as *finalizer attacks*) and they ignore exceptions!
- what should we do instead?
  - best approach is to make the class implement `AutoClosable` and require its clients to call `close()` or even better use them in `try-with-resource` blocks

## Item 9
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
