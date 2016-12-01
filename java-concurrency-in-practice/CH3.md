## 3.2 Publication and escape

> **Publishing** an object means making it available outside its current scope.
> An object that is published when it should not have been is said to have **escaped**.

There are several ways to publish an object:

1. Storing object references in a public static field.
2. Returning a reference from a nonprivate method.
3. Publishing an object that also publishes any objects referred to by its nonprivate fields.
4. Calling *alien* methods

>From the point of view of class A, an alien method is a method not fully "understood" by A.

* Calling an alien method from a synchronized block can cause excessive synchronization or liveliness issues.
* Alien methods can be methods from outside classes or overrideable methods in same class.

5. Publishing an inner class instance, because inner classes contain a hidden reference to the enclosing instance.





