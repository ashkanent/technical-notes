# 7 Lambdas and Streams

## Item 42
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


## Item 43
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

## Item 44
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

## Item 45
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

## Item 46
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

## Item 47
- Prefer collection to stream as a return type
- when writing a public API that returns a sequence of elements, we should provide for both users who want to use it in a stream pipeline as well as those who want to iterate on them.
- Best option here is to return a Collection that provides both
- the only thing to notice here is that if a return type is small enough to be kept in memory, we can use standard collection implementations such as `ArrayList` or `HashSet`. If it is large, we should consider returning a custom collection
  - one example is returning power set (all the sub-sets) of a given set: its space requirement is 2<sup>n</sup> space. Instead we can create a custom one based on binary representation of it (page 218 of the book for the implementation)

## Item 48
- Use caution when making streams parallel
- Java has always tried to facilitate concurrent programming. First version of Java was released with thread support, Java 5 introduced `java.util.concurrent` library with concurrent collections. Java 8 introduced streams which can be parallelized with a single call to the `parallel()` method
- although it is very easy to call the parallel method we should be very careful as it won't be very helpful in most of the cases and in some cases it can result in failure or performance disaster
- in general whenever we want to use it we have to have good reasons and we have to prove that the code remains correct. Then we have to measure the performance before and after to make sure it actually improved
- performance gains from parallelism are best on streams over ArrayList, HashMap, ConcurrentHashMap, arrays, int ranges and long ranges.
