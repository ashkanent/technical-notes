# 8 Methods

## Item 49
- Check parameters for validity
- most methods and constructors have some restrictions on their parameters. We should properly document these and also check for their validity as soon as we can.
- for public or protected methods use the Javadoc @throws tag to document the exception
- the `Objects.requireNonNull()` is convenient and easy to use (for null checks)

## Item 50
- Make defensive copies when needed
- you must program defensively with the assumption that clients of your class will do their best to destroy it!
- as an example of how things can go wrong and how we can prevent them, consider the following class:
```Java
public final class Period {
    private final Date start;
    private final Date end;

    // 'start' must be before 'end'
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException();
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public date end() {
        return end;
    }
}
```
- first of all never use `Date` as it is mutable and instead use `java.time.Instant`. The following code can break the validity of our `Period` instance:
```Java
Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
end.setYear(78); // modifies internals of period!
```
- to fix this, we should make defensive copies in constructor:
```Java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException();
    }
}
```
in the above code since we make a copy, it is impossible for the client to change the parameters. we also moved the validity check to after defining the defensive copies, this is to prevent scenarios like this: let's say another thread for example changes the value of parameters after we check their validity so by the time we assign them, they become invalid. but after we make our defensive copies, this can't happen and we can be sure that our Period instance is valid (and will remain valid)
- there is one more thing we need to fix in this class to make it completely error prone. The getters we have defined will return the Date instances of the class and again their values can be altered out of the class (the variables are final but that's just the object reference, i.e. its value can change). We apply the same fix:
```Java
public Date start() {
    return new Date(start.getTime());
}
// we do the same for end()
```
- same rules apply when you use a mutable object in your class and you know your class can't tolerate mutability in that reference (like you use the referenced object in a map as a key or in a set).

## Item 51
- Design method signatures carefully
- choose proper method names
  - if there are naming conventions, obey them
  - choose the best name that conveys the purpose of that method clearly
  - ideally avoid very long method names
- avoid long parameters list (maximum 4 parameters). There are different ways of making parameter lists shorter:
  - breaking the method into multiple methods, each with fewer parameters list
  - using helper classes/DTOs to pass around different parameters
  - using Builder patter (*item 2*) from object construction to method invocation
- for parameter types, favor interfaces over classes
  - using classes will put unnecessary restrictions
  - e.g. as an input instead of accepting `HashMap` we can accept `Map` (someone may want to pass a `TreeMap` or a `ConcurrentHashMap`)
- prefer two element enum types to boolean parameters
  - `public enum TemperatureScale { FAHRENHEIT, CELSIUS }`
  - `Thermometer.newInstance(TemperatureScale.CELSIUS)` versus `Thermometer.newInstance(true)`
  - we can also later expand the types under our enum type

## Item 52
- Use overloading judiciously
- overloading is when two (or more) methods have the same name and different signatures. If not used carefully, it can get really confusing and generate unexpected results.
- Choice of overloading (which overloaded method to be used) is determined at compile time and can cause issues:

    ```Java
    public String someMethod(Set<?> daSet) {
        return  "Set!";
    }

    public String someMethod(Collection<?> daCollection) {
        return "Collection";
    }
    ```
    in an example like this, user may call `someMethod()` with a set and expecting it to call the first method, while it may call the second one!
- it can get really confusing when there is same number of parameters. One alternative in these situations which makes it very clear and error prone is to use different names:
  - like `readInt()`, `readBoolean()`, `readLong()` (instead of three overloaded `read()`s)

## Item 53
- Use varargs judiciously
- vararg methods accept zero or more arguments of a specified type
  - they are life saver when we need variable number of arguments
  ```Java
  static int sum(int... args) {
      int sum = 0;
      for (int arg : args)
          sum += arg;
      return sum;
  }
  ```
- couple of small notes about varargs and their usage:
  1. if your method requires them to have at least one argument, we can check the argument size first thing and throw exception if it was zero:
  ```Java
  if (args.legth == 0) {
      throw new IllegalArgumentException("too few arguments");
  }
  ```
  but its messy and also more important, it fails at runtime (while it could've been caught at compile time). Best approach here is:
  ```Java
  static int min(int firstArg, int... remainingArgs) {
      ...
  }
  ```
  2. we need to be careful in performance-critical situations as each each use of varargs causes an array allocation and initialization. In this situation if we know that 90% of the cases are 2 arguments or less and we want to optimize the program, we can do this:
  ```Java
  public void foo() { }
  public void foo(int a1) { }
  public void foo(int a1, int a2) { }
  public void foo(int a1, int a2, int... rest) { }
  ```

## Item 54
- Return empty collections or arrays, not nulls
- There is no reason to return null when the array or collection that we were supposed to return has no members. It makes code messy, on the client side we need to add extra unnecessary code to handle `null` case first and then if it was not null, process the returned values. If we forget, we may get `NPE`. It also adds extra unnecessary code on our side as well to see if there are no members return null, if not return the collection (while we should've just returned that collection, even if it is empty!)
- in very rare cases we can argue that returning null is more memory efficient than allocating an empty array or collection. First it is inadvisable to worry about performance at this level and usually unnecessary. In the unlikely event that you have evidence suggesting that allocating empty collections is harming the performance, you can avoid it by returning `Collections.emptyList()`, `Collections.emptySet()`, `Collections.emptyMap()`, etc.
  - in case of arrays, we can have an empty array and returning the same zero-length array repeatedly (zero length arrays are immutable):

      ```Java
      private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

      public Cheese[] getCheeses() {
          return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY); // if we want an empty array here
      }
      ```

## Item 55
- Return optionals judiciously
- Optionals represent values that can be either present or absent. Prior to Java 8, if a method was not able to return a value it could either throw an exception or return null. Optionals were added in Java 8 (`java.util.optional`), they are immutable containers that can contain either a non-null object reference or nothing at all (empty optional)
```Java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> collection) {
    if (collection.isEmpty()) {
        return Optional.empty();
    }

    E result = //find the maximum element
    return Optional.of(result);
}
```
- passing `null` to `Optional.of(value)` will throw an exception.
- many terminal operations on streams return optionals. Using stream in the above example will generate the optional for us:
```Java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> collection) {
    return collection.stream.max(Comparator.naturalOrder());
}
```
- On the client side, they can decide what to do with an optional returned value. such as:

    ```Java
    // use a default value if optional is empty:
    String myString = max(words).orElse("No words...");

    // or throw an exception if it is empty:
    Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);

    // if we know it's not empty we can just get the value:
    Element lastNobleGas = max(Elements.NOBLE_GAS).get();
    ```
- if we have a stream of optionals and we want to get those elements of this stream that are not empty:
```Java
streamOfOptionals.filter(Optional::isPresent).map(Optional::get);
```
Java 9 added `stream()` method to `Optional` which returns only those elements that have value, so this last code snippet can be even shorter:
```Java
streamOfOptionals.flatMap(Optional::stream);
```

**Notes**
- never return null from an optional-returning method
- never return an optional of a boxed primitive type
  - this is expensive compared to returning primitive type because the optional has two levels of boxing instead of zero
  - you can use `OptionalInt`, `OptionalLong` and `OptionalDouble` if needed

## Item 56
- Write doc comments for all exposed  API elements
- to document the API properly, you must precede every exported class, interface, constructor, method and field declarations with a doc comment
- The complete guide for writing java docs is [here](https://www.oracle.com/technetwork/articles/java/index-137868.html). This doc is a bit old (Java 4), since then, these tags have been added: `{@index}`, `{@implSpec}`, `{@literal}` and `{@code}`
- the text following @param and @return should be a noun phrase, summary description should be a verb phrase. @throws should consist of the word "if" followed by the condition under which the exception is thrown:

    ```Java
    /**
    * Returns the element at the specified position in this list.
    *
    * <p>This method is <i>not</i> guaranteed to run in constant time. In some
    * implementations it may run in time proportional to the element position.
    *
    * @param index index of element to return; must be non-negative and less
    *              than the size of this list
    * @return the element at the specified position in this list
    * @throws IndexOutOfBoundsException if the index is out of range
    *         ({@code index < 0 || index >= this.size()})
    */
    E get(int index);
    ```
- there are some HTML meta-characters that should be skipped (such as `&` and `<`). also first sentence of each doc comment becomes *summary description* and this will be determined by a dot followed by space, so in this sentence we need to skip the second dot as well:
```A college degree, such as B.S., M.S. or Ph.D.```  
in these situations we can use @literal or @code tags to skip these characters and render them properly

    ```Java
    /**
    * A college degree, such as B.S., {@literal M.S.} or Ph.D.
    */
    public class Degree {...}
    ```
- Doc comments should be readable both in the source code and in the generated documentation.
