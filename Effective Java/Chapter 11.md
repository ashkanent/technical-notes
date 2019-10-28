# 11 Concurrency

## Item 78
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

## Item 79
- Avoid excessive synchronization
- inside a synchronized method or block, never cede control to the client
  - like calling a method that is designed to be overridden
  - or calling a method provided by the client
  - from the perspective of the class with the synchronized region, these methods are called **alien**
- we may get unwanted exceptions or deadlocks
- as a rule, we should do as little work as possible inside synchronized regions
- over-synchronization can have its own issues

## Item 80
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

## Item 81
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

## Item 82
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

## Item 83
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

## Item 84
- Don't depend on the thread scheduler
- when there are many runnable threads at the same time, thread scheduler determines which one gets to run for how long and this can be different in different systems. Any program that relies on thread scheduler for correctness and performance is likely to be nonportable.
- threads should not run if they are not doing useful work (like waiting for something)
- resist the temptation to use `Thread.yield` or thread priorities which are among least portable features of Java
