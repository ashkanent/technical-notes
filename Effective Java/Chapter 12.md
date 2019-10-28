# 12 Serialization

## Item 85
- Prefer alternatives to Java serialization
- There are many security risks in serialization. the best way to avoid serialization exploits is to never deserialize anything
- there is no reason to use Java serialization in any new code you write
- two good alternatives is to use JSON or protobuf as a cross platform structured data representation

## Item 86
- Implement `Serializable` with a great caution
- you can simply make any class to implement `Serializable` but there are a few things to be aware of
  - it decreases class's flexibility to change once it has been released. basically the byte stream of its internals are now part of the exported API
  - it increases the likelihood of bugs and security holes
  - it increases the testing burden. It should be possible to serialize the old version of a class and deserialize it using its new version and vice versa. These should be all tested
  - Classes designed for inheritance should rarely implement `Serializable` and interfaces should rarely extend it
  - inner classes should not implement `Serializable`
- when you make a serializable class, if you don't declare a static final long field called `serialVersionUID`, the system automatically generates it at runtime

## Item 87
- Consider using a customer serialized form
- Do not accept default serialized form without considering whether it is appropriate or not.
- default serialization is an efficient encoding of the *physical* representation of the object graph
- if this physical representation is identical to its logical content, then it is appropriate to use the default serialization. The following class is an example:
```Java
public class Name implements Serialization {
    /**
    * Last name, must be non-Null
    * @serial
    */
    private final String lastName;

    /**
    * First name, must be non-Null
    * @serial
    */
    private final String firstName;

    /**
    * Middle name, or null if there is none
    * @serial
    */
    private final String middleName;
}
```
- even if you decide that the default serialized form is appropriate, you often must provide a readObject method
- On the other end of the spectrum, here is a terrible candidate for default serialization:
```Java
public final class StringList implements Serializable {
    private int size = p;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    //...
}
```
logically speaking, this class represents a sequence of strings but physically it is a sequence of doubly linked list. A reasonable serialization form of this class is simply a number, representing number of strings in the list, followed by the strings themselves:
```Java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // appends the specified string to the list
    public final void add(String s) {...}

    /**
    * Serialize this {@code StringList} instance
    *
    * @serialData The size of the list (the number of strings it contains) is
    * emitted ({@code int}), followed by all of its elements (each a {@code String}
    * ), in the proper sequence
    */
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // write out all elements in the proper order
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream s) throws IOException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // read in all elements and insert them in list
        for (int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }

    // remainder omitted
}
```

## Item 88
- Write readObject methods defensively
- default serialization and deserialization of classes can be overridden using `readObject` and `writeObject` methods.
- these can be used for different scenarios, for example initializing transient (non-serializable) fields once we deserialized the object
- another use case which is the topic of this item is protecting immutable classes when we deserialize them
- as a reminder, for classes that have modifiable objects, we make them immutable by making them private final and also defensively copying them everywhere:
```Java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException();
        }
    }

    // getters and no setters
}
```
other than defensive copying, we check for validations as well to make sure state is valid.
- This item explains that `readObject` is effectively another public constructor which demands all of the same care as any other constructor
  - there are a few examples in the book that modifies the serialized byte stream to break things that are normally not possible on the above `Period` class. (examples are for when we make `Period` to implement Serializable and we don't provide a proper `readObject`)
  - in short, the proper `readObject` should defensively copy the fields and check for the validations
  - here is a proper implementation of it:
  ```Java
  private void readObject(ObjectInputStream is) throws IOException, ClassNotFoundException {
      is.defaultReadObject();

      start = new Date(start.getTime());
      end = new Date(end.getTime());

      if (start.compareTo(end) > 0) {
          throw new InvalidObjectException();
      }
  }
  ```
  note that here we have to make the fields non-final to be able to make a defensive copy right after deserialization.
- same as immutable classes, these `readObject` methods should not invoke overridable methods

## Item 89
- For instance control, prefer enum types to readResolve
- another risk with using serialization is singleton classes. They look like this:
```Java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
}
```
if we make it to implement serializable, it no longer will be a singleton.
- As a fix we can use `readResolve` method. This method takes object after deserialization and the object returned by this method is the final deserialized object (so we can replace the serialized object or modify it however we want before it gets to the client). The fix using `readResolve` will look something like this:
```Java
private Object readResolve() {
    // return the only instance of Elvis and garbage collector will take care of
    // the deserialized Elvis impersonator which will be ignored here:
    return INSTANCE;
}
```
- if we depend on `readResolve` for instance control, all instance fields with object reference types must be declared transient, if not there can be a potential for an attack.
- We can prevent this attack with a more preferred approach which is using enum:
```Java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = {"a", "b"};

    public void printFavourites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
## Item 90
- Consider serialization proxies instead of serialized instances
- consider this pattern whenever you find yourself having to write readObject or writeObject on a class that is not extendable by its clients
- as mentioned in previous items, using serialization increases the likelihood of bugs and security problems. *Serialization Proxy Pattern* greatly reduces these risks
- first, design a private static nested class that concisely represents the logical state of an instance of the enclosing class (this nested class is *serialization proxy* of the enclosing class)
- this class should have one constructor whose parameter type is enclosing class.
- both enclosing class and serialization proxy must declare `Serialization`
- this is a serialization proxy for the `Period` class written in item 50:
```Java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period period) {
        this.start = period.start;
        this.end = period.end;
    }

    private static final long serialVersionUID = 23413487423847742L; // any number will do
}
```
- now we have to add the `writeReplace` method to the enclosing class:
```Java
private Object writeReplace() {
    return new SerializationProxy(this);
}
```
- with this pattern, we should not generate a serialized version of the enclosing class, to safe gaurd it against attackers, add this to the enclosing class:
```Java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("Proxy required!");
}
```
- finally, provide `readResolve` method on the serialization proxy class that returns a logically equivalent instance of the enclosing class:
```Java
private Object readResolve() {
    return new Period(start, end);
}
```
- the key here is that at the end, this pattern uses the public constructor in the `readResolve` method which makes it safe (normally serialization uses extralinguistic mechanism in place of ordinary constructors)
