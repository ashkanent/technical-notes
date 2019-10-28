# 3 Methods Common to All Objects

## Item 10
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

## Item 11
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

## Item 12
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

## Item 13
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

## Item 14
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
