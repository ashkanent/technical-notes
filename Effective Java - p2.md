# Item 34
- Use enums instead of int constants
- When we have an enumerated type whose values are a fixed set of constants we should use `enum`. Bad alternatives are constant strings or ints like this:

```Java
// bad
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_BLOOD = 1;
```

instead we can use something like this:

```Java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, BLOOD, TEMPLE }
```
- this item also talks about various disadvantages of using constant ints or strings. For example in above example int values are used for both orange and apple types, later we can do math operations with these ints or can potentially compare these ints which is mathematically allowed but we are basically comparing apples to oranges! They are also verbose (long prefixes before each constant —`APPLE_`)
- enums are designed for this and other than simple example above (which is what we need most of the times) we can make them more advanced with extra functionalities:
```Java
public enum Planet {
    MERCURY(3.302e+23, 2.439e5),
    VENUS(4.859e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    ...;

    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    // constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = 6.6e-11 * mass / (radius*radius);
    }

    public double mass() {return mass;}
    public double radius() {return radius;}
    public double surfaceGravity() {return surfaceGravity;}

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```
- in the above code sample we are associating data with enum constants (mass, radius,..) and to do that we should add a constructor. we've also added a shared functionality to easily calculate the surface weight for each constant. Look at these sample usages of the above enum:
```Java
double earthWeight = 158; // or read from input
double mass = earthWeight / Planet.EARTH.surfaceGravity();
```
or
```Java
for (Planet planet : Planet.values()) {
    System.out.println("Weight on " + planet + " is " + planet.surfaceWeight(mass));
}
```
- note that all enums have static `values()` that returns an array of their values. they also have a default `toString()` which returns the declared name of each enum value. We can override this if we want.
- If enums are only needed in a class we should define them as a member class of that class, but if they are helpful in other places we should declare them as a top-level class.
- in `Planet` enum example we had a shared functionality (`surfaceWeight()`) that was similar for all of its constants. If we want a method that behaves different for each constant, best approach is to define an abstract method and implement it like this:
```Java
public enum Operation {
    PLUS("+") {public double apply(double x, double y){return x + y;} },
    MINUS("-") {public double apply(double x, double y){return x - y;} },
    TIMES("*") {public double apply(double x, double y){return x * y;} },
    DIVIDE("/") {public double apply(double x, double y){return x / y;} };

    public abstract double apply(double x, double y);

    public final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
}
```

# Item 35
- Use instance fields instead of ordinals
- enums have an ordinal method that returns an int for each member (in order they are defined, 0-index)
- we might get tempted in some situations to get the `ordinal()` value and use it. Don't do this!!
- if you need to associate a value with each member, even an int, use instance fields as explained in *item 34*.

# Item 36
- Use EnumSet instead of bit fields
- when the enum elements are meant to be used in a set, traditionally many ppl use int enum pattern and instead of enum, for each element they define a different power of 2 and to pass them around they use *bit field*:
```Java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 0; // 2
    public static final int STYLE_UNDERLINE = 1 << 0; // 4
    // ...

    public void applyStyles(int styles) {...}
}
```
and to use them and passing them around, instead of set we can do this: `text.applyStyles(STYLE_BOLD | STYLE_ITALIC);` (we pass something like `100101` in which every `1` says we should apply that style)
- Java.util package has `EnumSet` which should be used instead of these bit fields. it provides type safety and interoperability of sets and internally uses bit vector which makes its performance comparable with bit fields:
```Java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) { ... }
}
```
and then we can use it like this:
```Java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
note that `applyStyles()` takes `Set` instead of `EnumSet`, this is a best practice to accept the interface type rather than the implementation type (*item 64*), but it works fine if we accept `EnumSet`.

# Item 37
- Use EnumMap instead of ordinal indexing
- it is rarely appropriate to use oridnals to index into arrays, use `EnumMap` instead.
  - if the relationship being represented is multidimensional, nested `EnumMap`s should be used:  
  `EnumMap<..., EnumMap<...>>`
- One example for using EnumMaps: consider this class representing a Plant:
```Java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    // implement constructor and override toString to return name
}
```
- so here basically each plant has a name and a life cycle. Now we want to have an array of plants representing a garden and we want to organize these plants by there life cycle. We can have an array of set (`Set<Plant>[]`) and use `LifeCycle` ordinal value to put each plant in one of the possible 3 sets or we can use `EnumMap`:
```Java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifecycle = Arrays.stream(garden)
                    .collect(gropuingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class),
                        toSet() ));
```

# Item 38
- Emulate extensible enums with interfaces
- enum types are better that alternative enum patterns (such as int enum patterns). There is however one limitation which is one enum type can not extend another (generally not a good idea!)
- in some cases such as operation codes, it might be still valid to have extensible enums. To emulate the extension, we can use interfaces like this:

```Java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() { return symbol; }
}

public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) { return Math.pow(x, y); }
    },
    REMAINDER("%") {
        public double apply(double x, double y) { return x % y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() { return symbol; }
}
```
- this pattern lets clients to write their own enums and we can have methods using these enums that rely on their standard methods, for example we can have a method that accepts enums extending `Operation` and we call `apply()` method on its enum constants, knowing they all implement it
- The only issue here is that there are some duplicated code for `toString()`. we can potentially use static helper methods to reduce code duplication in these situations

# Item 39
- Prefer annotations to naming patterns
- Annotations in java are used to provide supplement information about a program. They help us associate metadata (information) with the program elements.
  - there are standard ones we can use (like `@SupressWarnings`, `@Override`, `@Deprecated`)
  - we can also define our own annotations
- One example of defining our test annotation for a test framework:  

    ```Java
    import java.lang.annotation.*;

    @Retention(RententionPolicy.RUNTIME) // it will be retained at runtime
    @Target(ElementType.METHOD) // it can be used only on methods (not classes, etc.)
    public @interface Test {
    }
    ```
- now we can write tests like this:
```Java
public class Sample {
    @Test
    public static void testMethod() {}
}
```
- now we can have a framework like this to find the annotated methods and run the tests:

    ```Java
    public class TestRunner {
        public static void main(String[] args) throws Exception {
            int tests = 0;
            int passed = 0;
            Class<?> testClass = Class.forName(args[0]);
            for (Method m : testClass.getDeclaredMethods()) {
                if (m.isAnnotationPresent(Test.class)) {
                    tests++;
                    try {
                        m.invoke(null);
                        passed++;
                    } catch() {
                        ... // see full implementation on page 182, 3rd edition
                    }
                }
            }
        }
    }
    ```
- we can also define our test annotation to have some associated values (e.g. for exception):
```Java
@Retention(RententionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
    Class<? extends Throwable> value();
}
```
- we can use it on a method like this:
```Java
@Test(ArithmeticException.class)
public static void badMethod() {
    int i = 3 / 0;
}
```
and when calling it in our test runner, we can check its value like this:
```Java
method.getAnnotation(Test.class).value(); // here it should return ArithmeticException
```
- our custom annotations can contain more information:
```Java
@Documented
@Retention(RententionPolicy.RUNTIME)
public @interface User {
    String name() default "Ashkan";
    Long id();
}
```
and then we can use it like this:
```Java
@User(name="Ashkan", id=123L)
public AdminUser adminUser = new AdminUser();
```

**Note**
- this item recommends using annotations
- there used to be some naming conventions (like JUnit 3 was looking for methods defined in a certain way -start with test) but now they use annotations which is much better.

# Item 40
- Consistently use the `Override` annotation
- the `@Override` annotation may be used on method declarations that override declarations from interfaces as well as classes
  - with the advent of default methods, it is good practice to use `Override` on implementations of interface methods.
- in an abstract class or an interface, it is worth annotating all methods that override superclass or interface methods, it provides extra checks and if by mistake you add a new method (e.g. by using a different method signature), it will warn you that what you are overriding does not exist in the parent

# Item 41
- Use marker interfaces to define types
- a *marker interface* is an interface that contains no method declarations but merely designates/marks a class a class implementing it as having some property
- if you find yourself writing a marker annotation type whose target is `ElementType.TYPE` (means any class or interface), make sure that you can't use a marker interface instead!
  - if you wanna add methods that accept only objects that have this marking, you should use interfaces
- this is basically inverse of *Item 22* (don't use an interface if you are not defining a type) and it says use an interface if you want to define a type

# Item 42
- Prefer lambdas to anonymous classes
- historically interfaces with single abstract methods were used as functional types. Their instances are known as *function objects* which represent functions or actions.
- Since JDK 1.1 the primary means of creating these function objects was the *anonymous class* (refer to *Item 24*). Since Java 8 it is made possible to create these *functional interfaces* in a much more concise way using *lambda expressions*:

    ```Java
    // using anonymous classes as a function object (obsolete!)
    Collections.sort(words, new Comparator<String>() {
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
    });

    // now using lambda expression as function object (instead of anonymous class)
    Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));

    // even better using comparator construction method instead of lambda:
    Collections.sort(words, comparingInt(String::length));

    // even more succinct using sort method added to List interface in Java 8:
    words.sort(comparingInt(String::length));
    ```
- remember from *Item 34* in enum type `Operation` we used constant-specific class bodies and overrode `apply()` for each enum constant (since behavior was different for each constant). now instead of function objects we used in those examples of *Item 34* to override `apply()`, we can simply do this:
```Java
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("+", (x, y) -> x + y); // omitted other operations for brevity

    private final static symbol;
    private final static DoubleBinaryOperation op;

    Operation(String symbol, DoubleBinaryOperation op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

**Notes**
- Lambdas lack names and documentation, one to three lines are reasonable for a lambda expression, don't put more than that in a lambda since they are not very self-explanatory and may result in confusion and harm the readability.
- you should rarely (if ever) serialize a lambda.


# Item 43
- Prefer method references to lambdas
- the primary advantage of lambdas over anonymous classes is that they are more succinct
- Java provides a way to generate function objects even more succinct: *method references*

    ```Java
    // lambda expression
    map.merge(key, 1, (count, incr) -> count + incr);

    // method reference:
    map.merge(key, 1, Integer::sum);
    ```
- in the above example, lambda expression is a bit verbose and all it is saying is that the function returns the sum of its two arguments. we accomplish that by just using `Integer::sum`
- in some situations lambda expression can be shorter that method reference (usually when the method is in the same class as the lambda). in these situations we better use the lambda version:
```Java
service.execute(GoshThisClassNameIsHumongous::action);
// vs.
service.execute(() -> action());
```
- there are other cases like `Function.identity()` which is again better to use the lambda equivalent `x -> x` as it is shorter and more clear.
- so whenever it is shorter and more clear, we should use method references, o.w. we should use lambdas.

# Item 44
- Favor the use of standard functional interfaces
- now that Java has lambdas, best practices of writing APIs has changed considerably
- now instead of having a subclass overriding a primitive method in superclass, we can provide a static factory/constructor that accepts a function object
- when you want to accept functional interfaces you don't have create the interface every time. First make sure it doesn't exist in `java.util.function` (unless you have a good reason not to use them)
- there are 43 interfaces in this package but the main 6 are:
```
Interface               Function signature          Example
--                      
UnaryOperator<T>        T apply(T t)                String::toLowerCase
BinaryOperator<T>       T apply(T t1, T t2)         BigInteger::add
Predicate<T>            boolean test(T t)           Collection::isEmpty
Function<T, R>          R apply(T t)                Arrays::asList
Supplier<T>             T get()                     Instant::now
Consumer<T>             void accept(T t)            System.out::println
```
- make sure you use these interfaces unless there are good reasons like
  - it will be commonly used and can benefit from a more descriptive name
  - it has a strong contract associated with it
  - it would benefit from custom default methods
- one example is Comparator<T> (where we don't use an existing interface)
- when defining functional interfaces, use `@FunctionalInterface` annotation:
```Java
// for demonstration only, use the standard one under java.util.function:
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

# Item 45
- Use streams judiciously
- Streams API was added in Java 8 and contains two key abstractions:
  1. the stream, which represents finite/infinite sequence of data elements
  2. the stream pipeline, which represents a multistage computation on these elements
- streams are sufficiently versatile that practically any computation can be performed using streams.
  - but this doesn't mean we should!
  - when used appropriately, they make programs shorter and cleaner
  - otherwise they make programs difficult to read and maintain
- as an example look at one implementation of program that reads words and prints them based on *anagram* groups (words with same letters, different combinations):
```Java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word))).values()
            .stream().filter(group -> group.size() >= minGroupSize)
            .forEach(group -> System.out.println(group.size() + ": " + group));
        }
    }

    private static String alphabetize(String str) {
        char[] a = str.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- things to consider in the above code:
  - we could make everything iterative or we could use streams everywhere. But this is the best and most concise implementation that combines streams and imperative paradigms:
    - we could implement `alphabetize()` using streams, but would've been less readable and less performant (streams are not efficient on chars as they don't support them)
  - we also extracted `alphabetize()` as a method and used it in our stream. Alternatively it could have been implemented using streams and used in the same stream, but it would have made the entire thing more complex, less readable and less efficient
  - we tried to use meaningful names in streams which clearly express the elements we are iterating on (e.g. `word` and `group`)
- there are certain things we can't do from function objects (in streams) such as defining local variables and reading/modifying them in scope. Also we can't return from the enclosing method or break/continue or throw a checked exception:
  - in these situations we should avoid streams.
- also there are situations in which streams usually makes more sense:
  - uniformly transform sequences of elements
  - filter sequences of elements
  - search sequence of elements for an element satisfying some criterion
  - accumulate sequences of elements into a collection (perhaps grouping them by some common attributes)
  - combine sequences of elements using a single operation
- there are also some situations which it is not clear that which way is better or more efficient or more readable! in these situations we can go either way!


**Notes**
- Overusing streams makes programs hard to read and maintain
- in the absence of explicit types, careful naming of lambda parameters is essential for readability of stream pipelines.
- using helper methods is even more important for readability in stream pipelines than in iterative code!

# Item 46
- Prefer side-effect-free functions in streams
- Streams aren't just an api, they are a paradigm based on functional programming
- they should have a sequence of transformations where the result of each stage is as close as possible to a *pure function*:
  - its result depends only on its input, it doesn't depend on any mutable state, nor does it update any state
- compare the following two examples, the first one doesn't follow the paradigm but the second one is modified to follow the paradigm:
```Java
// uses streams but not the paradigm:
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
// proper use of streams:
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
the problem with the first example above is that we use forEach to mutate a an external state (frequency table). But second example is modified so that it accepts a table and initializes the given table
- the forEach operation should be used only to report the result of a stream computation, not to perform the computation
- In order to use streams properly, you have to know about collectors. The most important collector factories are `toList`, `toSet`, `toMap`, `groupingBy` and `joining`.

# Item 47
- Prefer collection to stream as a return type
- when writing a public API that returns a sequence of elements, we should provide for both users who want to use it in a stream pipeline as well as those who want to iterate on them.
- Best option here is to return a Collection that provides both
- the only thing to notice here is that if a return type is small enough to be kept in memory, we can use standard collection implementations such as `ArrayList` or `HashSet`. If it is large, we should consider returning a custom collection
  - one example is returning power set (all the sub-sets) of a given set: its space requirement is 2<sup>n</sup> space. Instead we can create a custom one based on binary representation of it (page 218 of the book for the implementation)

# Item 48
- Use caution when making streams parallel
- Java has always tried to facilitate concurrent programming. First version of Java was released with thread support, Java 5 introduced `java.util.concurrent` library with concurrent collections. Java 8 introduced streams which can be parallelized with a single call to the `parallel()` method
- although it is very easy to call the parallel method we should be very careful as it won't be very helpful in most of the cases and in some cases it can result in failure or performance disaster
- in general whenever we want to use it we have to have good reasons and we have to prove that the code remains correct. Then we have to measure the performance before and after to make sure it actually improved
- performance gains from parallelism are best on streams over ArrayList, HashMap, ConcurrentHashMap, arrays, int ranges and long ranges.

# Item 49
- Check parameters for validity
- most methods and constructors have some restrictions on their parameters. We should properly document these and also check for their validity as soon as we can.
- for public or protected methods use the Javadoc @throws tag to document the exception
- the `Objects.requireNonNull()` is convenient and easy to use (for null checks)

# Item 50
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
Period period = new Period();
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

# Item 51
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

# Item 52
- Use overloading judiciously
- 
