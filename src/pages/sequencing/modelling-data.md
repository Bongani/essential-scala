---
layout: page
title: "Modelling Data with Generic Types"
---

In this section we'll see the additional power the generic types give us when modelling data. We see that with generic types we can implement **generic sum and product types**, and also model some other useful abstractions such as **optional values**.

## Generic Product Types

Let's look at using generics to model a *product type*. Consider a method that returns two values -- for example, an `Int` and a `String`, or a `Boolean` and a `Double`:

~~~ scala
def intAndString: ??? = // ...

def booleanAndDouble: ??? = // ...
~~~

The question is what do we use as the return types? We could use a regular class without any type parameters, with our usual algebraic data type patterns, but then we would have to implement one version of the class for each combination of return types:

~~~ scala
case class IntAndString(intValue: Int, stringValue: String)

def intAndString: IntAndString = // ...

case class BooleanAndDouble(booleanValue: Boolean, doubleValue: Double)

def booleanAndDouble: BooleanAndDouble = // ...
~~~

The answer is to use generics to create a **product type** -- for example a `Pair` -- that contains the relevant data for *both* return types:

~~~ scala
def intAndString: Pair[Int, String] = // ...

def booleanAndDouble: Pair[Boolean, Double] = // ...
~~~

Generics provide a different approach to defining product types --  one that relies on aggregation as opposed to inheritance.

### Exercise: Pairs

Implement the `Pair` class from above. It should store two values -- `one` and `two` -- and be generic in both arguments. Example usage:

~~~ scala
scala> val pair = Pair[String, Int]("hi", 2)
pair: Pair[String,Int] = Pair(hi,2)

scala> pair.one
res13: String = hi

scala> pair.two
res14: Int = 2
~~~

<div class="solution">
If one type parameter is good, two type parameters are better:

~~~ scala
case class Pair[A, B](one: A, two: B)
~~~

This is just the product type pattern we have seen before, but we introduce generic types.

Note that we don't always need to specify the type parameters when we construct `Pairs`. The compiler will attempt to infer the types as usual wherever it can:

~~~ scala
scala> val pair = Pair("hi", 2)
pair: Pair[String,Int] = Pair(hi,2)
~~~
</div>

### Tuples

A *tuple* is the generalisation of a pair to any number of terms. Scala includes built-in generic tuple types with up to 22 elements, along with special syntax for creating them. With these classes we can represent any kind of *this and that* relationship between any number of terms.

The classes are called `Tuple1[A]` through to `Tuple22[A, B, C, ...]` but they can also be written in the sugared[^sugar] form `(A, B, C, ...)`. For example:

[^sugar]: The term "syntactic sugar" is used to refer to convenience syntax that is not needed but makes programming sweeter. Operator syntax is another example of syntactic sugar that Scala provides.

~~~ scala
scala> Tuple2("hi", 1) // unsugared syntax
res19: (String, Int) = (hi,1)

scala> ("hi", 1) // sugared syntax
res19: (String, Int) = (hi,1)

scala> ("hi", 1, true)
res20: (String, Int, Boolean) = (hi,1,true)
~~~

We can define methods that accept tuples as parameters using the same syntax:

~~~ scala
scala> def tuplized[A, B](in: (A, B)) = in._1
tuplized: [A, B](in: (A, B))A

scala> tuplized(("a", 1))
res21: String = a
~~~

We can also pattern match on tuples as follows:

~~~ scala
scala> (1, "a") match {
         case (a, b) => a + b
       }
res3: String = 1a
~~~

Although pattern matching is the natural way to deconstruct a tuple, each class also has a compliment of fields named `_1`, `_2` and so on:

~~~ scala
scala> val x = (1, "b", true)
x: (Int, String, Boolean) = (1,b,true)

scala> x._1
res5: Int = 1

scala> x._3
res6: Boolean = true
~~~

## Generic Sum Types

Now let's look at using generics to model a *sum type*. Again, we have previously implemented this using our algebraic data type pattern, factoring out the common aspects into a supertype. Generics allow us to abstract over this pattern, providing a, well, generic implementation.

Consider a method that, depending on the value of its parameters, returns one of two types:

~~~ scala
scala> def intOrString(input: Boolean) =
         if(input == true) 123 else "abc"
intOrString: (input: Boolean)Any
~~~

We can't simply write this method as shown above because the compiler infers the result type as `Any`. Instead we have to introduce a new type to explicitly represent the disjunction:

~~~ scala
def intOrString(input: Boolean): Sum[Int, String] =
  if(input == true) {
    Left[Int, String](123)
  } else {
    Right[Int, String]("abc")
  }
~~~

How do we implement `Sum`? We just have to use the patterns we've already seen, with the addition of generic types.

### Exercise: Generic Sum Type

Implement a trait `Sum[A, B]` with two subtypes `Left` and `Right`. Create type parameters so that `Left` and `Right` can wrap up values of two different types.

Hint: you will need to put both type parameters on all three types. Example usage:

~~~ scala
scala> Left[Int, String](1).value
res24: Int = 1

scala> Right[Int, String]("foo").value
res25: String = foo

scala> val sum: Sum[Int, String] = Right("foo")
sum: Sum[Int,String] = Right(foo)

scala> sum match {
         case Left(x) => x.toString
         case Right(x) => x
       }
res26: String = foo
~~~

<div class="solution">
The code is an adaptation of our invariant generic sum type pattern, with another type parameter:

~~~ scala
sealed trait Sum[A, B]
final case class Left[A, B](value: A) extends Sum[A, B]
final case class Right[A, B](value: B) extends Sum[A, B]
~~~

Scala's standard library has the generic sum type `Either` for two cases, but it does not have types for more cases.
</div>


## Generic Optional Values

Many expressions may sometimes produce a value and sometimes not. For example, when we look up an element in a hash table (associative array) by a key, there may not be a value there. If we're talking to a web service, that service may be down and not reply to us. If we're looking for a file, that file may have been deleted. There are a number of ways to model this situation of an optional value. We could throw an exception, or we could return `null` when a value is not available. The disadvantage of both these methods is they don't encode any information in the type system.

We generally want to write robust programs, and in Scala we try to utilise the type system to encode properties we want our programs to maintain. One common property is "correctly handle errors". If we can encode an *optional value* in the type system, the compiler will force us to consider the case where a value is not available, thus increasing the robustness of our code.

### Exercise: Maybe that Was a Mistake

Create a generic trait called `Maybe` of a generic type `A` with two subtypes, `Full` containing an `A`, and `Empty` containing no value.

<div class="solution">
We can apply our invariant generic sum type pattern and get

~~~ scala
sealed trait Maybe[A]
final case class Full[A](value: A) extends Maybe[A]
final case class Empty[A]() extends Maybe[A]
~~~
</div>

The invariant generic sum type pattern is a bit satisfying. Ideally we would like to write something like this:

~~~ scala
sealed trait Maybe[A]
final case class Full[A](value: A) extends Maybe[A]
final case object Empty extends Maybe[???]
~~~

However, objects can't have type parameters. In order to make `Empty` an object we need to provide a concrete type in the `extends Maybe` part of the definition. But what type parameter should we use? In the absence of a preference for a particular data type, we could use something like `Unit` or `Nothing`. However this leads to type errors:

~~~ scala
scala> :paste
sealed trait Maybe[A]
final case class Full[A](value: A) extends Maybe[A]
final case object Empty extends Maybe[Nothing]
^D

defined trait Maybe
defined class Full
defined module Empty

scala> val possible: Maybe[Int] = Empty
<console>:9: error: type mismatch;
 found   : Empty.type
 required: Maybe[Int]
       val possible: Maybe[Int] = Empty
~~~

The problem here is that `Empty` is a `Maybe[Nothing]` and a `Maybe[Nothing]` is not a `Maybe[Int]`. However, we now know how to overcome this problem: we can use a variance annotation, which we learned about in the last section. Covariance is the most natural variance in this case.

In the Scala standard library, this abstraction is called `Option`, with cases `Some` and `None`.

~~~ scala
sealed trait Maybe[+A]
final case class Full[A](value: A) extends Maybe[A]
final case object Empty extends Maybe[Nothing]

object division {
  def apply(num: Int, den: Int) =
    if(den == 0) Empty else Full(num / den)
}
~~~

This pattern is the most commonly used one with generic sum types.

<div class="callout callout-info">
#### Covariant Generic Sum Type Pattern

If `A` of type `T` is a `B` or `C`, and `C` is not generic, write

~~~ scala
sealed trait A[+T]
final case class B[T](t: T) extends A[T]
final case object C extends A[Nothing]
~~~
</div>

## Type Bounds

It is sometimes useful to constrain a generic type. We can do this with type bounds indicating that a generic type should be a sub- or super-type of some given types. The syntax is `A <: Type` to declare `A` must be a subtype of `Type` and `A >: Type` to declare a supertype.

For example, the following type allows us to store a `Visitor` or any subtype:

~~~ scala
case class WebAnalytics[A <: Visitor](
  visitor: A,
  pageViews: Int,
  searchTerms: List[String],
  isOrganic: Boolean
)
~~~

## Take Home Points

In this section we have used generics to model sum types, product types, and optional values using generics.

We saw the covariant generic sum type pattern, which allows us to avoid unnecessary generic type parameters.

## Exercises

#### Generics versus Traits

Sum types and product types are general concepts that allow us to model almost any kind of data structure. We have seen two methods of writing these types -- traits and generics -- when should we consider using each?

<div class="solution">
Ultimately the decision is up to us. Different teams will adopt different programming styles. However, we examine look at the properties of each approach to inform our choices:

Inheritance-based approaches -- traits and classes -- allow us to create permanent data structures with specific types and names. We can name every field and method and implement use-case-specific code in each class. Inheritance is therefore better suited to modelling significant aspects of our programs that are re-used in many areas of our codebase.

Generic data structures -- `Tuples`, `Options`, `Eithers`, and so on -- are extremely broad and general purpose. There are a wide range of predefined classes in the Scala standard library that we can use to quickly model relationships between data in our code. These classes are therefore better suited to quick, one-off pieces of data manipulation where defining our own types would introduce unnecessary verbosity to our codebase.
</div>

#### Covariance

Implement our `Sum` example using the covariant generic sum type pattern.

<div class="solution">
~~~ scala
sealed trait Sum[+A, +B]
final case class Left[A](value: A) extends Sum[A, Nothing]
final case class Right[B](value: B) extends Sum[Nothing, B]
~~~
</div>