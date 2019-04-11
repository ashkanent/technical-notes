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
