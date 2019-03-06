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
