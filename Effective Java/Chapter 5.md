# 5 Generics

## Item 26
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

## Item 27
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

## Item 28
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
- when using arrays with generics, we may get compile errors or unchecked cast warnings. when this happens the best solution often is to use `List<E>` (replace arrays with lists)

## Item 29
- Favor generic types
- generic types are safer and easier to use than the types that require casts in client code
- we should design our code and types to be generic for this reason
  - as an example, this item changes the `Stack` implementation of *Item 7* to use generics instead of `Object`
  - when we do this we will get one error in the constructor (`elements = new E[...];`)
  - it resolves it by changing it to `elements = (E[]) new Object[...];`
  - after this we will get a warning but we can justify that `elements` is internal and will be safe to do so, then based on *Item 26* we suppress the warning.
  - *takeaway*: if we can avoid using lists (*item 28*) it will be more efficient to use arrays! otherwise as *item 28* suggests, use lists.
- ***Bounded type***: when working w/ generics, we can define a type like `<E extends Delayed>` which means that we have a generic type that is a subtype of `java.util.concurrent.Delayed`. The advantage is that we can use methods of `Delayed` on elements of our generified class (which are of type `E`).

## Item 30
- Favor generic methods
- same as classes, methods can be generic and we need to use them specially when we have a method whose use requires casting!
- to make a method generic, you need to put the type parameter between method's modifier and its return type:
```Java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    ...
}
```
- generic methods can be used in *generic singleton factory* pattern:

    ```Java
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SupressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }

    // now we can use it on any type:
    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number number : numbers) {
        System.out.println(sameNumber.apply(number));
    }
    // or (my example which "I think" should be correct and makes more sense to me):
    BigDecimal b = new BigDecimal("1234.1241234");
    UnaryOperator<BigDecimal> sameDecimal = identityFunction();
    BigDecimal b1 = sameDecimal.apply(b);
    ```

## Item 31
- Use bounded wildcards to increase API flexibility
- parameterized types are *invariant* (two distinct types `Type1`, `Type2` -> `List<Type1>` is not sub/super type of `List<Type2>`)  
  - look at this Stack implementation (here only included the public API):
  ```Java
  public class Stack<E> {
      public Stack();
      public void push(E element);
      public E pop();
  }
  ```
  we now add the `pushAll()` method to this class:
  ```Java
  public void pushAll(Iterable<E> src) {
      for (E element : src) {
          push(element);
      }
  }
  ```
  - since parameterized types are *invariant*, this won't work:
  ```Java
  Stack<Number> numberStack = new Stack<>();
  Iterable<Integer> integers = ...;
  numberStack.pushAll(integers);
  ```
  - although `Integer` is a subtype of `Number` but list of it is not! to fix it we need to change `pushAll` to this:
  ```Java
  public void pushAll(Iterable<? extends E> src) {...}
  ```
  - for the exact same reason, `popAll()` should be implemented like this:
  ```Java
  public void popAll(Iterable<? super E> dst) {
      while (!isEmpty()) {
          dst.add(pop());
      }
  }
  ```
- `<?>` is a wildcard type and when used with extends/super is called a *bounded wildcard type*
  - `<? extends E>`: any subtype of E
  - `<? super E>`: any supertype of E
- for maximum flexibility of our API, we should use bounded wildcard types on input parameters that represent producers or consumers
  - **PECS** stands for producer-extends, consumer-super
  - so `pushAll()` is producing (for internal stack) so it extends
  - and `popAll()` is consuming (from internal stack) so it uses super

***Notes***
- do not use bounded wildcard types as return types
- all comparables and comparators are consumers

## Item 32
- Combine generics and varargs judiciously
- *varargs* (variable legnth arguments) was introduced in Java 5 (same as generics) but they don't work well with each other!
  - we can have a method like `public void func(int... input) {...}` and then call it like `func(2);` or `func(1, 4, 3);` etc.
  - when we invoke varargs, an array is created to hold the parameters.
  - as mentioned in *Item 28*, arrays and generics do not mix well!
- whenever we use varargs on a method with parameterized type, we will get a warning (possible heap pollution warning). To fix this we have to do one of the following:
  1. make sure it is type safe and then use `@SafeVarargs`, or:
  2. replace varargs with a list (sth like *item 28* to use list instead of arrays)
- varargs methods are *safe* if:
  1. it doesn't store anything in the varargs parameter array, and
    - i.e. we don't assign anything to the passed in parameter that is a vararg
  2. it doesn't make the array (or a clone) visible to the untrusted code
    - for example getting the varargs parameter and returning it, so another code can get it, use it and break it  

**Note**
- `@SafeVarargs` is legal only on methods that can not be overriden (o.w. those methods may break its safety!)
- use `@SafeVarargs` on every method with a varargs parameter of a *generic* or *parameterized* type (so users won't be burdened with warnings)

## Item 33
- Consider typesafe heterogeneous containers
- Common uses of generic collections are like `Set<E>` and `Map<K, V>` which is a parametrized container that contains elements of one type.
- Usually this is what we want but sometimes we may need a typesafe container that can store and return elements of different types.
  - we parametrize the key instead of the container!
- before we take a look at one example, consider the following:
  - the type of a class literal is `Class<T>`:
    - type of `String.class` is `Class<String>`
  - when a class literal is passed around for type information it is called a **type token**
- here is a typesafe heterogeneous container:
```Java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance));
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
- a sample program to use the above container:
```Java
Favorites favorites = new Favorites();
favorites.putFavorite(String.class, "Java");
favorites.putFavorite(Integer.class, 0xcafe);
int favInt = favorites.getFavorite(Integer.class);
```
