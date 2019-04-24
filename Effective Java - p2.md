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
