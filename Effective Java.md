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
