---
title: "Java: Value Objects - Performance Like Primitives"
last_modified_at: 2024-10-21T16:20:02-05:00
categories:
  - Blog
  - Programming Language
tags:
  - Java 
classes: wide  
---

# Project Valhalla: *Codes Like A Class, Works Like An Int*

> [!NOTE]
> The project Valhalla is still very vivid. The things mentioned here are due to change even though the fundemantal ideas and the direction is very unlikely to change. This write-up is based on the related JEP and the talk of Brian Goetz.  

The [Valhalla project](https://openjdk.org/projects/valhalla) phrases its task as below:

> Project Valhalla is augmenting the Java object model with value objects, combining the abstractions of object-oriented programming with the performance characteristics of simple primitives. Supplementary changes to Java’s generics will carry these performance gains into generic APIs. 
>
> — Valhalla Project 

The road to the goal of the project turned out to be very rocky as the Java language architect puts it:

> Project Valhalla is the biggest refactor of the Java language and runtime we've ever taken.
>
> — Brian Goetz 

## Java Object Model

```java
record Point(int x, int y) { } E
record Rect(Point pi, Point p2) { }

Rect r = new Rect(new Point(1, 2),
new Point(3, 4));
```
Given the code above, the the object is laid out in memory as follows:

![Object Memory Layout](/blog/assets/images/java-object-memory-layout.png)

Implications of this layout:

- Not memory efficient
- Not flat - small islands of data connected by pointers
- Memory intensive - much memory used by object-headers and pointers

A flat memory layout would be much better:

![Better Object Memory Layout](/blog/assets/images/java-valhalla-object-memory-layout.png)

# Differences Between Primitives And Objects

To arrive at that goal, they tried to overcome the differences between objects and primtives, which are many:

- Classes are user-defined - primitives are built in
- Classes have members and supertypes - primitives don’t
- Objects have identity - primitives don’t
- Objects are polymorphic in layout and behavior - primitives are not
- Object references are nullable - primitives are not
- Primitives are usable without initialization (they come with reasonable
defaults) - objects must be constructed before use
- Primitives are flattened into object and array layouts - objects are not
- Longer primitives allow relaxed atomicity under data races - objects do not

That means the path to achieving the goals is apparent:

- Enable flatter and denser memory layouts for Java object graphs
- Unify the type system, healing the rift between primitives and objects
- Enable new numeric types to be added as /ibraries, not built-ins

# Impediments To Flattening And Density

The project members did realize that giving up **object identity** and **nullability** could make handling objects like primitives very easy. This insight did make causes for the very complex implementations disappear all at once.

Object identity implies an object lives in a well-defined place, basically forcing indirections for
heap objects. It's necessary for mutability (among other things), because there has to
be an authoritative place to store the object state. The equality operator == compares two object references by identity, which could be a source of bugs, where this is not desired. Additionally identity Increases the number of bit patterns that must be representable (e.g. a nullable long needs 65 bits -> practically 128 bits). 

Nullability, which is the second impediment, will be tackled the same way as object identity. The developer needs to declare if he needs object nullability. 

There are some JEPs dealing with nullability:

  - [Null-Restricted and Nullable Types](https://openjdk.org/jeps/8303099)  
  - [Null-Restricted Value Class Types](https://openjdk.org/jeps/8316779)  

# Identity-Free Objects: value classes

Value classes allow the developer to state that the object does not need identity. 
They are immutable by default.

Value classes are defined with the keyword `value` in front of a class:

```java
value class Point {
/* final */ int x, y;
...
}
```
# Characteristics Of Value classes

- Value classes can have pretty much everything that identity classes can
(fields, methods, constructors, supertypes, etc)
- They differ from identity classes in only a few ways:
  - The value class, and its fields, are implicitly final 
  - Value objects are compared for equality (==) by their state, not by identity
  - Value objects have no intrinsic monitar (no synchronization)
  - (Other identity-sensitive operations (e.g, weak references) are also disabled
  - Must extend Object ora value-capable abstract class
    - Some restrictions on value-capable abstract classes

- Value objects are objects, referred to by object references like any other
  - But we want to be able to optimize away the physical form of the reference
- Because they lack identity, value objects are freely copyable, which
enables many optimizations
  - Can be transparently shredded into scalars and reconstructed into objects
again, with no loss of information
  - Most on-stack usage never allocates anything
  - Calling convention is highly optimized, passing fields in registers
  - Allows immutable objects to be as fast as mutable ones!

# Example: Scalarization In Local Usage

In the example below the accumulator state (`total`) will be routinely *hoisted into registers*, **no objects will
actually be allocated**.
As a result, we have the safety of immutability with the performance of mutability!

```java
value record ImmutableAccumulator(int total) {
  ImmutableAccumulator() { this(0); }

  ImmutableAccumulator plus(int n) {
    return new ImmutableAccumulator(total + n);
  }
}

ImmutableAccumulator acc = new ImmutableAccumulator();
for (String s : strings)
acc = acc.plus(s.length());
```
# Example: Scalarization In Calling Convention 

When a value object is passed between methods, instead of passing a
heap pointer, an optimized calling convention passes the scalarized fields. Both for parameters and for return values.

```java
value record Point(double x, double y) { }

Point unitVector(double angle) {
return new Point(Math.cos(angle), Math.sin(angle));
}

Point p = unitVector(Math.PI); // no Point object allocation, sin and cos returned in registers
````

# JEP 401 (Value Classes)

- Valhalla will deliver in phases
  - The first JEP delivers value classes only
  - Literally one syntactic change: the value modifier on classes (records too)
  - Migrate some JDK classes (Integer, Optional, LocalDate, etc) to be value classes
- Focus is on semantic foundations and basic optimizations
  - Won’t get full flattening and density optimizations yet, but will get
    - Scalarization of value parameters/returns in calling convention, locals
    - Heap flattening of <64 bit value objects (need a bit for null)
- Relatively little new complexity to absorb
  - Basically, “Does this class need identity? No? Then disavow it.”
- Early access builds are available at https://jdk.java.net/valhalla

# Performance Gains

The Valhalla team did run a huge set of performance tests and come up with fantastic results:

- Dramatic speedups (and reduced allocation) on operations on arrays of null-
restricted value objects vs identity objects
  - Gains come from scalarization, increased locality, reduced allocation
  - Comparing arrays of null-restricted value objects with arrays of identity objects
- Complex matrix multiplication: Throughput 6x, allocation 1000x less
- Summing array of pairs of ints (with a mutable accumulator)
  - 3.5x faster using a for-loop
  - 12x faster using a stream of indexes
- Summing array of pairs of ints (with an immutable accumulator)
  - 8.2x faster
  - Speed of mutability with the safety of immutability!

# Summary

After almost 10 years of development the Valhalla team will deliver a stream of small features, which 
have a small impact on the programming model, but a huge one on the Java compiler, JVM. 
As a result the JVM can highly optimize the code leading to huge performance gains.

There will also be improvements in Generics - unification and reification.

Phased delivery plan (4 JEPs so far):

- [Value Classes and Objects - JEP 401](https://openjdk.org/jeps/401)  is already in Early Access
- [Null-Restricted and Nullable Types](https://openjdk.org/jeps/8303099)
- [Null-Restricted Value Class Types](https://openjdk.org/jeps/8316779)
- [Enhanced Primitive Boxing - JEP 402](https://openjdk.org/jeps/402)

---
# References

- [Talk By Brian Goetz at Devoxx](https://www.youtube.com/watch?v=eL1yyTwu4hc)  
- [Value Classes and Objects - JEP 401](https://openjdk.org/jeps/401)  
- [Null-Restricted and Nullable Types](https://openjdk.org/jeps/8303099)  
- [Null-Restricted Value Class Types](https://openjdk.org/jeps/8316779)  
- [Enhanced Primitive Boxing - JEP 402](https://openjdk.org/jeps/402)  
- [Flexible Constructor Bodies - JEP 482](https://openjdk.org/jeps/482)