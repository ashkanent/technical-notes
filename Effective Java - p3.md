# Item 69
- Use exceptions only for exceptional conditions
- we should not use them for ordinary conditional flow like this:
```Java
try {
    int i = 0;
    while (true) {
        range[i++].climb();
    }
} catch (ArrayIndexOutOfBoundsException e) {}
```
- this approach of using exceptions is wrong on different levels:
  - it is confusing as it's not the intention of the exceptions
  - JVM will not optimize these (compared to when we use Arrays.length() for instance)
  - we may catch an actual exception here unintentionally without realizing it
- a well designed API must not force its clients to use exceptions for ordinary workflow
  - e.g. Iterator has `hasNext()` which you can use before calling `next()`, so you don't need try/catch here
  - in general, when a method is state-dependant, we should have a state-testing method to call before calling our method to make sure we can safely call it.

# Item 70
- Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
- Java provides three kinds of throwables:
  1. *checked exceptions*
    - when we declare a method throws an exception
    - the caller needs to catch it and resolve it or propagate it upwards
    - they are meant to be used in conditions from which the caller can reasonably be expected to recover
    - it is a bad idea to just catch these exceptions and ignore them.
    - provide methods on checked exceptions to aid in recovery. e.g. we have a payment w/ credit card method that throws exception if funds are not sufficient. We can provide an accessor method to query the amount of shortfall. This enables the caller to relay the amount to the shopper
  2. *runtime exceptions*
    - these exceptions are usually result of precondition violation which is client failure to adhere to the contract established by the API specifications
    - they are result of programming errors
    - e.g. the contract for array access specifies the indices to be between 0 and array's length minus 1, using something outside this range will throw `ArrayIndexOutOfBoundsException`
  3. *errors*
    - there is a strong convention that errors are reserved for use by the JVM
    - such as `OutOfMemoryError`, `ThreadDeath`, etc.
    - all the unchecked exceptions you implement should subclass `RuntimeException`
