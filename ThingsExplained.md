# Order of Complexity
* Big O notation is the upper bound, Omega is the lower bound and theta is a two-sided bound.
* Worst case scenario of an algorithm
* _Big-O_ shows the relative Complexity of an algorithm and how it scales.
 * __How it scales__: e.g. algorithm does one operation for one item, 10 operations for 10 items, 100 operations for 100 items, and so on... (so its linear)
* Different complexities:
 * Constant Complexity —> O(1)
 * Logarithmic Complexity —> O(log n)
   * Binary Search
 * Linear Complexity —> O(n)
 * Quadratic Complexity —> O(n^2)
 * Factorial Complexity —> O(n!)
   * TSP or Traveling Salesman Problem
 * Polynomial Complexity —> O(n^a)

# Java Terms - Tips
### JDK > JRE > JVM
  * JVM translates byte-code into executables (based on OS)
  * JRE is what we need for running jar files. It contains libraries, etc.
  * JDK is what we need to do java development, debugging
  * **JDK** has **JRE** inside it. **JRE** has **JVM** inside it

### Wrapper class:
  * wraps around a data type and gives it an object-like appearance
    * e.g. `Integer` around `int` or `Boolean` around `boolean` or `Double` around `double`
  * it also provides utility methods
  * also you can't put primitive data types in _collections_. You can only put objects.
    * so you can use their wrappers
  * wrapper objects are immutable

### Strings:
  * they are immutable.

  ```Java
  String str3 = "value1";
  str3.concat("value2");
  System.out.println(str3); // prints "value1"
  ```

  * all of String class methods, such as `concat()`, `replace()`, etc. return a String. Which means they don't change the original value, they create a new instance and return it
  * `StringBuffer` is dynamic and faster.
  * `StringBuilder` is same as `StringBuffer` with the exception that it's not thread-safe and it performs better in situations where thread safety is not required

### Functional Programming
* _Functions_ are not the same as methods! Methods like those we define in a class are dependent on the class.
  * Functions live independently and unlike methods, they don't bound to objects or classes.
  * All values in functions are immutable. If it want to change something, it creates a copy and modifies that and returns that copy.
  * methods may change value of its arguments or class variables but functions do not change anything.
  * Functions return only a value or another function.
  * functions can be composed together regardless of context and form a chain of functions. This can not be done with methods.
* there is **no iteration** in functional programming. Operations are done by recursion.

### Simple Design Principles
1. Should run all tests
  * testable code will lead to better Design
2. Avoid Duplication
3. Maximize Clarity
4. Keep it small
  * if a new component (package, class, method) does not help improving clarity or avoiding duplication, think if it is needed!
