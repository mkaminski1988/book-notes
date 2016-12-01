## 3.2 Publication and escape

> **Publishing** an object means making it available outside its current scope.
> An object that is published when it should not have been is said to have **escaped**.

There are several ways to publish an object:

1. Storing object references in a public static field.
2. Returning a reference from a nonprivate method.
3. Publishing an object that also publishes any objects referred to by its nonprivate fields.
4. Calling *alien* methods.
5. Publishing an inner class instance, because inner classes contain a hidden reference to the enclosing instance.

> From the point of view of class A, an alien method is a method not fully "understood" by A.

* Calling an alien method from a synchronized block can cause excessive synchronization or liveliness issues.
* Alien methods can be methods from outside classes or overrideable methods in same class.

> A better explanation of alien methods can be found in Joshua Bloch's *Effective Java Second Edition* on page 265.

### 3.2.1 Safe construction practices

* If a `this` reference escapes during construction, the object is not properly constructed.
* Don't start a thread from a constructor!
* Registering an event listener or starting a thread in a constructor can be safely handled if the constructor is private and the instance is created in a static factory. 

> A good example of the problem on Stack Overflow -> http://stackoverflow.com/questions/1588420/how-does-this-escape-the-constructor-in-java

## 3.3 Thread confinement

* The easiest way to to avoid synchronization is to not share mutable data.

> For example, pooled JDBC connection objects are not thread safe. Instead, the objects are removed by a thread and then returned. The connection manager does not give the same connection to another thread until it is returned to the pool.

### 3.3.1 Ad-hoc thread confinement

* A fragile way to confine an object to a target thread that does not use any of the language features like visibility modifiers or local variables.
* Thread confinement can be accomplished using volatile variables, provided only one thread performs the read-modify-write operations. Other read-only threads can see the latest values without synchronization.
* Use this method sparingly—prefer stack confinement or `ThreadLocal`.

### 3.3.2 Stack confinement

* Create local references inside methods and don't return them.
* Works even for non-threadsafe objects.
* The requirement that the object be confined locally is not usually clear, so ensure that it gets documented.

### 3.3.3 `ThreadLocal`

* Stores values for each thread. Can be thought of as `Map<Thread, Value>`.
* *May* be useful when a frequently used operation uses a temporary buffer that is expensive to create on each invocation.
* Useful for porting single threaded applications to a multithreaded environment.
* `ThreadLocal` is easy to abuse as a type of global variable and can introduce hidden coupling that can harm reusability.





