---
title: "Design Pattern: Visitor"
last_modified_at: 2024-10-31T16:20:02-05:00
categories:
  - Blog
  - Design Patterns
tags:
  - design pattern
  - visitor pattern
  - Java
classes: wide  
---

# Visitor Design Pattern

> Source Code: [GitHub](https://github.com/tokalak/visitor-pattern-in-java)

The **Visitor pattern** is one of the most popular design patterns introduced in the famous book [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns).

The pattern is getting mentioned very often in the context of Data Oriented Programming.

## The Problem

Imagine that we've have a family of classes and wanna associate a set of behavior with each one. The usual way to implement this requirement in object oriented languages is to create the classes and put the methods in them. If we want to add a new class to that family, then we create it and implement all methods in this new class. No existing class needs to be changed. But if we need to add a new behavior then we've to change every single class. This is not nice.

The functional languages take an another path. They don't have the notion of a class with methods. Types and functions are totally distinct. To implement an operation for a number of types, you define a function. In the body of that function, you use `pattern matching` to implement the operation for each type. That way adding a new operation is very easy - just implement a new function to handle each type in one place. But adding a new type gets hard. In that case you've to change every operation.

As we see adding a new class/type to a family of classes is very easy in object oriented languages, whereas adding a new method to a family of classes is hard.

That's where the `Visitor pattern`comes into play.

The `Visitor pattern` is kind of approximating the functional style within OOP languages.

## The Example

Imagine having a bunch of types with some methods. 
To keep the example simple, we limit oursevles to a few types and methods.

### The Types

```java
abstract class Car {
  abstract void accept(CarVisitor visitor);
}

class TraditionalCar extends Car {
  @Override
  void accept(CarVisitor visitor){
    visitor.visitTraditionalCar(this);
  }
}

class ElectricCar extends Car {
  @Override
  void accept(CarVisitor visitor){
   visitor.visitElectricCar(this); 
  }
}
```

### The Visitor Interface

```java
interface CarVisitor {
  
  void visitTraditionalCar(TraditionalCar traditional);

  void visitElectricCar(ElectricCar electric);
}
```
Please note that each subtype of car calls the corresponding method in the visitor interface.

### Operations

Each operation that can be performed on cars is a new class that implements the interface `CarVisitor`.

Implementing a `fuel()` operation for all types of cars would look like:

```java
class FuelVisitor implements CarVisitor {

  @Override
  public void visitTraditionalCar(TraditionalCar traditional) {
    System.out.println("Fueling a traditional car with gasoline.");
  }

  @Override
  public void visitElectricCar(ElectricCar electric) {
    System.out.println("Fueling an electric car.");
  }
}

```

### Bring All Together

```java

public class Demo {
  public static void main(String[] args){
    var fuel = new FuelVisitor();
    
    new TraditionalCar().accept(fuel)
    new ElectricCar().accept(fuel);
  }
}

```

Running the demo lead to following output in the console:

```bash
Fueling a traditional car with gasoline.
Fueling an electric car.
```

## Wrap-Up

We've seen that creating a new method operating on a family of types without changing the types is possible with the Visitor pattern.
But we had to create a bunch of classes to accomplish the task. There must be a better way.  
That's where Data Oriented Programming comes into play.

## Data Oriented Programming

* TO BE CONTINUED *

## References

- [Visitor pattern](https://en.wikipedia.org/wiki/Visitor_pattern)
- [Data Oriented Programming in Java](https://www.infoq.com/articles/data-oriented-programming-java)
- [Source Code](https://github.com/tokalak/visitor-pattern-in-java)






