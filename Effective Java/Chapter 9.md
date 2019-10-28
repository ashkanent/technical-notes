# 9 General Programming

## Item 57
- Minimize the scope of local variables
- the most powerful technique is to declare them where they are first used
- if local variable declaration doesn't contain initializer, perhaps we should postpone its declaration to when we have enough information to initialize it.
- prefer for loops to while loops because they enforce local variables for iterators
- keep methods small and focused
  - having a method doing two things (let's say task 1 and task 2), we may define local variables that is used for task 1 but it's not used by task 2!
  - to prevent this, simply separate this method into two

## Item 58
- Prefer for-each loops to traditional for loops
- for different reasons such as being more succinct, less error prone and more flexibility we should use for-each loops when possible. Specially that they come with no performance penalty
- we should be able to use for-each loops in different situations except when we need to access the iterator index for different scenarios such as removing or modifying an element at a certain index
```Java
for (Element element : elements) {
    // do something
}
```

## Item 59
- Know and use the libraries
- By using a standard library, you take advantage of the knowledge of the experts who wrote it and the experience of those who used it before you
  - different functionalities in the libraries get developed and tested with all the edge cases and performance scenarios in mind, gets reviewed by the experts and then gets used by millions of developers. Discovered issues get fixed in future releases.
- We should be familiar with standard libraries as well as high-quality 3rd party libraries such as Google's guava library

## Item 60
- avoid `float` and `double` if exact answers are required
- these types are designed particularly for scientific and engineering calculations. They carefully provide accurate approximations quickly over a broad range of magnitudes.
- they are specially ill-suited for monetary calculations
```Java
System.out.println(1.03 - 0.42); // 09999999999999998
```
- use `BigDecimal`, `int` (or `long`) in these situations

## Item 61
- prefer primitive types to boxed primitives
- java has two-part type system:
  1. primitives: such as int, double and boolean
  2. reference types: such as String and List
- every primitive type has a corresponding reference type called *boxed primitives*
  - boxed primitives for `int`, `double` and `boolean` are `Integer`, `Double` & `Boolean`
- primitives only have values but boxed primitives have identities distinct from their values. Primitives have only fully functional values whereas boxed primitives have non-functional value which is `null`. also primitives are more time and space efficient.
- there are certain issues that we may encounter when using boxed primitives. for example:
  - if we compare them like this: `integerOne == integerTwo` almost always returns false when their values are equal! (because it compares their identities rather than their values)
  - when comparing `Integer` with `int`, it will automatically unbox the Integer to get its value and compare it with int. If Integer value is null, it will throw NPE!
  ```Java
  Integer i;
  if (i == 52) { // NullPointerException
      //...
  }
  ```
  - there situations where we can't use primitives. Such as putting elements (or keys or values) in collections. For type parameters also we can't use primitives:
    - `ThreadLocal<int>` is not allowed, you must use `ThreadLocal<Integer>`
- in summary, we should always use primitives where ever we have the choice.

## Item 62
- Avoid Strings where other types are appropriate
- Strings are designed to represent text, they do a fine job of it. at the same time, they are poor substitutes for other value types.
  - poor substitutes for enum types (discussed in *Item 34*)
  - poor substitutes for aggregate types ( strings like `'className#2#true'` that are meant to keep different values)
- Used inappropriately, strings are more cumbersome, less flexible, slower and more error prone

## Item 63
- Beware the performance of string concatenation
- using string concatenation operator on n strings requires time quadratic in `n`.
  - this is because strings are immutable
- don't use string concatenation to combine more than a few strings
- for a better performance, use `StringBuilder` in place of string:
```Java
public String statement() {
    StringBuilder stringBuilder = new StringBuilder(newItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        stringBuilder.append(lineForItem(i));
    }
    return stringBuilder.toString();
}
```

## Item 64
- Refer to objects by their interfaces

    ```Java
    // good
    Set<Son> sonSet = new LinkedHashSet<>();

    // bad
    LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
    ```
- If there is no appropriate interface, just use the least specific class that provides the required functionality
- if we are using a class-base framework we can use the base class which is usually an abstract class

## Item 65
- Prefer interfaces to reflection
- Using reflection, given a class object you can access its constructor, fields and methods and invoke them. This comes handy when you don't know the class to use until run time.
- when using reflection:
  - you lose all the benefits of compile time type checking
  - the code required for reflection is verbose (*see p. 283, 3rd edition for an example*)
  - performance suffers
- we should limit use of reflection as much as possible.
- if possible we should create a class instance reflectively and then access the object using an interface or superclass that is known at compile time (to limit use of reflection)

## Item 66
- Use native methods judiciously
- Java lets us call native methods written in native programming languages such as C/C++
  - they allow access to low level resources and platform specific facilities such as registries
  - they are also used to write performance-critical parts of applications
- this item basically says that Java has changed a lot and added more features to make us independent of these native methods. it has also improved performance a lot.
- we should rarely use native methods for improved performance
- if we use them to access native libraries or low level resources, we should use as little native code as possible

## Item 67
- Optimize judiciously
- this item says that we should avoid optimizing as much as possible! most of the effort is usually wasted and the results are usually negligible and sometimes worse!
- instead we should focus on having a good architecture, which in nature is flexible, scalable and fast.
- when you're done writing your application, use profiling tools to measure the performance
  - if it's fast enough, you're good
  - if not, locate source of problem, fix it and measure it afterwards to make sure you made improvements
- no amount of low-level optimization can make up for a poor choose of algorithm

## Item 68
- Adhere to generally accepted naming conventions
- there are generally two categories of conventions: typographical and grammatical
- **package** or **module**
  - all lower case
  - ideally less than 8 characters
  - meaningful abbreviations are encouraged (util instead of utilities)
  - start with organization's internet domain in reverse order
  - `org.junit.jupiter.api`, `com.google.common`
- **class** or **interface**
  - CamelCase
  - abbreviations are to be avoided except for common ones such as min and max
  - `Stream`, `FutureTask`, `LinkedHashSet`, `HttpClient`
- **method** or **field**
  - camelCase
  - except constants which are all caps, separated with under-score
  - `remove`, `groupingBy`, `getCrc`, `MIN_VALUE`
- local variables are lest restricted since they have a limited scope and no visibility in the api. you can use `i` or abbreviations such as `houseNum`
  - personal note: I still prefer getting into the habit of more readable and meaningful names, specially given that all the modern editors have auto-complete (as mentioned in *Clean Code*)
- type parameters are single letters, `E` for collection elements, `T` for an arbitrary type, `K` and `V` for key/value types in map, `R` for return types of methods and `X` for exception types. a sequence of arbitrary types are `T`, `U`, `V` or `T1`, `T2` and `T3`
  - personal note: lots of developers prefer using full words to make it more clear and meaningful, such as `Type` instead of `T` or `DataType` instead of `E`, etc. (lots of articles about it online)
- grammatical naming conventions are more flexible. none for package and module names. Class names (and enums) are usually singular noun (or noun phrases) such as `ChessPiece`, `Thread`,...
- non-instantiable utility classes are usually plural noun such as `Collectors` or `Collections`
- Interfaces are like classes (`Collection`) or end with `-ible`/`-able` like `Runnable` or `Accessible`
- methods are verbs such as `append`, `drawImage`. if returning boolean, usually start with 'is' like `isDigit`, `isEmpty` or sometimes like `hasSiblings`
  - methods returning non-boolean usually start with 'get' like `getTime` but sometimes it can be more readable to avoid get like `car.speed()`
