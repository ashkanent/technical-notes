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

# Item 71
- Avoid unnecessary use of checked exceptions
- when used sparingly they increase reliability of programs, when overused, they make APIs painful to use
- we need to use them when we believe user can recover the situation when getting the exception
  - if they just log the stack trace or ignore it (as in like they can do nothing else) it doesn't make sense to make it a checked exception. This just makes it harder to use as you have to use a try/catch or propagate it outward!

# Item 72
- Favor the use of standard exceptions
- Java libraries provide a set of exceptions that covers most of exception throwing needs of most APIs
- we should try to use standard and appropriate exception types which makes our code cleaner and easier to read and understand
- the most commonly reused exceptions are:
  - `IllegalArgumentException`: non-null parameter value is inappropriate
    - like passing a negative number to an argument expecting number of elements
  - `IllegalStateException`: Object state is inappropriate for method invocation
    - like Object is not initialized
  - `NullPointerException`: parameter value is null where prohibited
  - `IndexOutOfBoundException`: index parameter value is out of range
  - `ConcurrentModificationException`: concurrent modification of object has been detected where it is prohibited
  - `UnsupportedOperationException`: object does not support method

# Item 73
- Throw exceptions appropriate to the abstraction
- when exceptions propagate outward, we may get one that has no apparent connection to the task that it performs.
- to avoid this problem we can do *exception translation* which is basically catching the exception (when appropriate) and throwing a more relevant exception to the higher level instead
- this is an example from `AbstractSequentialList`:
```Java
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
}
```
- a special form of exception translation is called *exception chaining* which is when lower level exception might be useful to someone debugging the higher level exception. in this case, we pass the lower level exception to the higher level exception:
```Java
try {
    ...
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```
  - most standard exceptions have chaining-aware constructors which pass the "cause" to a higher level constructor which can be accessed later.
  - for exceptions, we can use `Throwable`'s `initCause` method which later can be accessed using `getCause`

**Notes**
- while exception translation is superior to mindless propagation of exceptions from lower layers, it should not be overused

# Item 74
- Document all exceptions thrown by each method
- always declare checked exceptions individually and document precisely the conditions under which each one is thrown
- use JavaDoc's `@throws` tag
- it is also a good practice to document unchecked exceptions that the method can throw. These unchecked exceptions are usually programmer's error and doing this, they will know what to expect and how to properly use that method or interface.

# Item 75
- Include failure-capture information in detail messages
- to capture a failure, the detail message of an exception should contain all the values for all the parameters and fields that contributed to the exception
  - e.g. if we get `IndexOutOfBoundException`, it would be very helpful to have the index value as well as the lower bound and upper bound
  - an exception to this (for security purposes) is passwords, encryption keys and the like which we don't wanna include in the detail message

# Item 76
- Strive for failure atomicity
- it means a failed method invocation should leave the object in the state that it was in prior to the invocation
- there are different ways to achieve this goal. For example in stack implementation from before:
```Java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    ...
}
```
so here we check the size first and throw exception instead of making size `-1` and then wait for the exception to be thrown when accessing `elements[-1]`
- another way to accomplish failure atomicity is to change the order of the computations if possible, so the part that may cause exception comes first.
- Another approach is to perform operations on a copy of the object and then if everything was successful apply them back to the original
- it should be mentioned that failure atomicity is not always achievable (like to threads without proper synchronization modify something) or sometimes making something atomic will hugely complicate the code. In these cases we should mention it in the API documentation that what is the broken state that may happen by a method invocation for example

# Item 77
- Don't ignore exceptions
- an empty catch block defeats the purpose of exceptions. We should always take proper actions to address exceptions
- in some rare cases it might be ok to ignore them, in which case we should put a comment in catch block with explanations:
```Java
int numColors = 4; // Default
try {
    numColors = getNumColors();
} catch (ExecutionException | TimedoutException ignored) {
    // in this case just use default value
}
```

# Item 78
- Synchronize access to shared mutable data
- it is better not to share mutable data between multiple threads and share only immutable data.
- if we have a mutable object, we can share it with only one thread
- if we have to share mutable data, we should use Synchronize which guarantees that no method will ever observe the object in an inconsistent state.
  - the language specification guarantees that read/write is atomic unless variable is `long` or `double` (there is `AtomicLong` that we can use)
- one example of using synchronization:
```Java
public class StopThread() {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while ( !stopRequested() ) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```
- synchronization can be used for:
  - 'communication': if we remove synchronization code above, our thread won't have access to updated value of `stopRequested`, so we should use it to make sure it does get the updated value as we expect it
  - 'mutual exclusion': so we won't have race condition or having one thread reading value while the other is writing a new value to it.
- in the above example, since synchronization is only used fo 'communication', we can remove the synchronized methods for read and write and instead declare the variable like this:
`private static volatile boolean stopRequested;` this will make the thread check for the updated value every time it reads it.
- synchronization is not guaranteed unless both read/write methods are synchronized.

# Item 79
- Avoid excessive synchronization
- inside a synchronized method or block, never cede control to the client
  - like calling a method that is designed to be overridden
  - or calling a method provided by the client
  - from the perspective of the class with the synchronized region, these methods are called **alien**
- we may get unwanted exceptions or deadlocks
- as a rule, we should do as little work as possible inside synchronized regions
- over-synchronization can have its own issues

# Item 80
- Prefer executors, tasks, and streams to threads
- java.util.concurrent package contains *Executor Framework* which is a flexible interface-based task execution facility. Instead of creating a work queue to execute tasks asynchronously and handling that queue manually, it is better to use this framework:
```Java
ExecutorService exec = Executors.newSingleThreadExecutor();
// submitting a runnable for execution:
exec.execute(runnable);
// terminating executor gracefully:
exec.shutdown();
```
- you can do many more with this service. You can wait for all the tasks or a particular task to complete, you can retrieve results of tasks as they complete, you can schedule them to run at a particular time or to run periodically, and so on.
- in essence, *Executor Framework* does for execution what *Collections Framework* did for aggregation.

# Item 81
- Prefer concurrency utilities to `wait` and `notify`
- the higher level utilities in `java.util.concurrent` fall into three categories:
  1. Executor Framework
  2. concurrent collections
  3. synchronizers
- concurrent collections are high-performance concurrent implementation of standard collection interfaces such as List, Queue and Map (like `ConcurrentHashMap`)
- *Synchronizers* are objects that enable threads to wait for one another, allowing them to coordinate their activities. the most common ones are `CountDownLatch` and `Semaphore` and the most powerful one is `Phaser`.
- quick explanation of `CountDownLatch`: it has a constructor that gets an int which is number of times the `countDown` method must be invoked on it before all waiting threads can proceed:
```Java
CountDownLatch latch = new CountDownLatch(2);
latch.await();
System.out.println("done!");
```
so here if in another thread for example we invoke `latch.countDown` two times, the above code will proceed and print "done!"
- in summary, this item (and the previous one) introduced us to three main utilities of `java.util.concurrent`. There are old alternatives for each of them that require lots of code and maintenance (for \#2 above, there are `Synchronized Collections` that are slower. For \#3 there is `wait` and `notify`/`notifyAll` which is messy and needs more handling). These last two items introduce these new alternatives and encourage us to use them instead. There are lots of details for each of them and if we need them, we should study them in more depth.

# Item 82
- Document thread safety
- How a class behaves when its methods are called concurrently is an important part of its API contract
- to enable safe concurrent use, a class must clearly document its level of thread safety:
  - **Immutable**: Instances are constant, no external synchronization is necessary
  - **Unconditionally thread-safe**: instances of this class are mutable but the class has sufficient internal synchronization
  - **Conditionally thread-safe**: like previous one but some of its methods need external synchronization
  - **Not thread-safe**: to use them safely, client must surround each method invocation with external synchronization
  - **Thread hostile**: unsafe for concurrent use even if everything is surrounded by synchronization. This usually results from modifying static data without internal synchronization, external synchronization won't help here!
- lock fields should always be declared final:

    ```Java
    private final Object lock = new Object();

    public void foo() {
        synchronized(lock) {
            ...
        }
    }
    ```

# Item 83
- Use lazy initialization judiciously
- **Lazy initialization**: the act of delaying the initialization of a field until its value is needed
- as is the case for most optimizations, don't do it unless you need to
- if a field is only accessed on a fraction of the instances and it's costly to initialize it, it's better to lazily initialize it

- normal lazy initialization which needs to be synchronized:
```Java
private FieldType field;
private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}
```
- *lazy initialization holder class idiom*, to use on a static field:
```Java
private static class FieldHolder {
    static final FieldType field = computeFieldValue(); // this will be ignored until accessor is called
}
private static FieldType getField() {
    return FieldHolder.field;
}
```
- to use lazy initialization for performance on an instance field, use the *double-check* idiom:

    ```Java
    private volatile FieldType field;

    private FieldType getField() {
        FieldType result = field;
        if (result == null) { // first check, no locking
            synchronized(this) {
                if (field == null) { // second check with locking
                    field = result = computeFieldValue();
                }
            }
        }
    }
    ```
