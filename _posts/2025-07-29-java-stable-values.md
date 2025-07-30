---
title: "Java Stable Values: Defer Initialization Of Immutable Fields"
last_modified_at: 2025-07-19T11:00:00-00:00
categories:
  - Blog
tags:
  - Java
classes: wide
---

Occasionally, you want to **initialize an immutable attribute after class construction**, because the attribute is not needed immediately. Its initialization might be time-consuming and resource-intensive, so you prefer to pay the cost only if the attribute is actually used.

In a non-trivial application with many such attributes, this approach can **significantly improve Java application startup time**. This concept is commonly called *lazy initialization*, but **Java stable values take it further**. They guarantee that the initialization code runs **at most once**, even in multi-threaded scenarios, and that the resulting object is eligible for **JVM constant folding**. This means that after the attribute is initialized, the JIT compiler may replace subsequent calls to the stable value with the constant resulting object.

Common workarounds to implement lazy initialization look like this:

```java
class MyClass {

  private PreciousResource resource = null;

  PreciousResource getResource() {
    if (resource == null) {
      resource = createPreciousResource();
    }
    return resource;
  }
  //...
}
```
However, this approach has several issues:

1. You must always call the getter method `getResource()`. Forgetting it and accessing the field directly risks a `NullPointerException`.
2. It is **not thread-safe**. You must apply additional techniques like [double-checked locking](https://en.wikipedia.org/wiki/Double-checked_locking) to ensure thread safety.
3. The JVM cannot perform optimizations such as constant folding on this pattern.

## Stable Values - Java 25 Preview

Java 25 introduces a superior alternative to lazy initialization by **decoupling the creation and initialization of stable values**.

Here are multiple ways to rewrite the above code with stable values:

```java

  // First approach
  private final StableValue<PreciousResource> resource = StableValue.of();

  PreciousResource getResource() {
    return resource.orElseSet(() -> createPreciousResource());
  }

  // More convenient approach combining declaration and initialization code
  private final Supplier<PreciousResource> resource = StableValue.supplier(() -> createPreciousResource());

  PreciousResource getResource() {
    return resource.get();
  }
```

The Stable Values feature in Java 25 offers many helper classes and utility methods to simplify working with deferred immutable values. For example, you can have a list whose entries are **initialized lazily when first accessed** and then constant folded by the JVM. Similar implementations exist for maps, sets, and more. For details, see the references below.

## References

[https://openjdk.org/jeps/502](JEP 502: Stable Values (Preview))
