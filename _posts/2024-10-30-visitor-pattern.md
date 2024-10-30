---
title: "Design Pattern: Visitor"
last_modified_at: 2024-10-21T16:20:02-05:00
categories:
  - Blog
  - Design Patterns
tags:
  - design pattern
  - Java
classes: wide  
---

# Visitor Design Pattern

The **Visitor pattern** is one of the most popular design patterns introduced in the famous book [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns).

The pattern is getting mentioned very often in the context of Data Oriented Programming.

## The Problem

Imagine that we've have a family of classes and wanna associate a set of behavior with each one. The usual way to implement this requirement in object oriented languages is to create the classes and put the methods in them. If we want to add a new class to that family, then we create it and implement all methods in this new class. No existing class needs to be changed. But if we need to add a new behavior then we've to change every single class. This is not nice.

The functional languages take an another path. They don't have the notion of a class with methods. Types and functions are totally distinct. To implement an operation for a number of types, you define a function. In the body of that function, you use `pattern matching` to implement the operation for each type. That way adding a new operation is very easy - just implement a new function to handle each type in one place. But adding a new type gets hard. In that case you've to change every operation.

As we see adding a new class/type to a family of classes is very easy in object oriented languages, whereas adding a new method to a family of classes is hard.

That's where the `Visitor pattern`comes into play.

The `Visitor pattern` is kind of approximating the functional style within OOP languages.






