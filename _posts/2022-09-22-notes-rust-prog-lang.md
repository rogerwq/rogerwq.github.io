---
layout: post
title: Notes on The Rust Programming Language 
date: 2022-09-22 12:35:20 +0900
# description: 
# img: 
# fig-caption: 
tags: [rust]
---


### 19.2 Advanced Trait

#### Specifying placeholder types in trait definitions with associated types

- The difference is that when using generics, ..., we must anotate the types in each implementation.

#### Default generic type parameters and operator overloading

- ... you can overload the operations and corresponding traits listed in std::ops by implementing the traits associated with the operator.
- you'll use default type parameters in two main ways:
    - to extend a type without breaking existing code
    - to allow customization in specific cases most users won't need

#### Fully qualified syntax for disambiguation: calling methods with the same name
- associated functions that are not methods don't have a self parameter.
- fully qualified syntax: <Type as Trait>::function(receiver_if_method, next_arg, ...)

#### Using supertraits to require one trait's functionality wthin another trait
- trait OutlinePrint: fmt::Display

#### Using the newtype pattern to implement external traits on external types
- ... orphan rule ... we're only allowed to implement a trait on a type if either the trait or the type are local to our crate.
- It's possible to get around this restriction using the newtype pattern.
- ... from the Haskell ...
- There is no runtime performance penalty for using this pattern, and the wrapper type is elided at compile time.
- If we wanted the new type to have every method the inner type has, implementing the ```Deref``` trait ... on the Wrapper to return the inner type would be a solution.




### 19.3 Advanced Types

#### Using the newtype pattern for type safety and abstraction

#### Creating type synonyms with type aliases
- ... to reduce repetition

#### The never type that never returns
- Functions that return never are called ```diverging functions```
- ... ```continue``` has a ! value
- The formal way of describing this behavior is that expressions of ```type !``` can be coerced into any other type.

#### Dynamically sized types and the sized trait
- ... sometime referred to as DSTs or unsized types
- a ```&str``` is two values: the address fo the str and its length
- By default, generic functions will work only on types that have a known size at compile time. However, you can sue the following special syntax to relax this restriction ... ```?Sized```
- A trait bound on ```?Sized``` means "T may or may not be Sized" and this notation overrides the default that generic types must have a known size at the compile time.
- The ```?Trait``` syntax ... is only available for ```Sized```, not any other traits.



### 19.4 advanced functions and closures

### Function pointers
- functions coerce to the type ```fn```, not to be confused with the ```Fn``` closure trait.
- The ```fn``` type is called a function pointer
- unlike closures, ```fn``` is a type rather than a trait, so we specify ```fn``` as the parameter type directly rather than declaring a generic type parameter with one of the ```Fn``` traits as a trait bound. Function pointers implement all three of the closure traits (Fn, FnMut and FnOnce).
- ..., one example of where you would want to only accept fn and not closures is when interfacing with external code that doesn't have closures: C functions can accept functions as arguments, but C doesn't have closures.
- ... enum variant ... an initializer function ... we can specify the initializer functions as arguments for methods that take closures.


### Returning closures
- closures are represented by traits, which means you can't return closures directly.
- you can't do that with closures because they don't have a concrete type that is returnable; you're not allowed to use the function pointer fn as a return type.
- ... trait object... ```Box<dyn Fn(i32) -> i32>```
