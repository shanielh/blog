---
title: "Covariance, Contravariance and Culture in Scala"
description: "Subtyping rules are applied instantly in our brain. We know that we can pass every instance of Bar or instance of class that is derived from Bar to this method, and it can return an instance of Buzz…"
date: "2017-09-10T19:10:06.597Z"
categories: 
  - Scala
  - Culture
  - Programming

published: true
canonicalLink: https://medium.com/@shanielh/covariance-contravariance-and-culture-in-scala-d0317fd02e19
---

When we are reading a method signature, for instance:

```
def foo(bar: Bar): Buzz
```

Subtyping rules are applied instantly in our brain. We know that we can pass every instance of Bar or instance of class that is derived from Bar to this method, and it can return an instance of Buzz or of any class that is derived from Buzz.

Covariance (+) and Contravariance (-) in Programming Languages is a way to define if a type (e.g. Generic Type) can be replaced with a base / derived type respectively.

Let’s have a look at the `Iterator[A]` trait:

```
trait Iterator[+A] {
  def hasNext(): Boolean
  def next(): A
}
```

The `Iterator` trait returns the type `A` and it’s covariant. That means that we can convert `Iterator[A]` to `Iterator[B]` whenever `B` is a base class of `A`. Another example:

```
trait Writer[-A] {
  def write(value: A): Unit
}
```

The `Writer` trait receives the type `A` and it’s contravariant. That means that we can convert `Writer[A]` to `Writer[B]` whenever `B` is a derived class of `A`.

**Small Tip:** I use to get confused between the two terms. In Scala it’s easier to remember the sign that we should use: `+` means “more” types (base classes, broader definition allowed) and `—` means “less” types (derived classes, narrower definition allowed).

#### Using Covariance and Contravariance to Save a bit of memory

This week I code reviewed a pull request at Upsolver. That pull request had a piece of code that looked like this :

```
trait Foo[A] extends AutoClosable {
  def foo(value: A): Unit
}

case class NopFoo[A] extends Foo[A] { ... }

case class DefaultFoo[A] extends Foo[A] { ... }

object Foo {
  def apply[A](...): Foo[A] = {
    if (...) DefaultFoo[A]() else NopFoo[A]()
  }
}
```

Everything looks fine, Right?

Let’s have a look on the one of the most influencing types in Scala, `Option[A]` :

```
sealed abstract class Option[+A] extends Product with Serializable {
  ...
}

final case class Some[+A](x: A) extends Option[A] {
  def isEmpty = false
  def get = x
}

case object None extends Option[Nothing] {
  def isEmpty = true
  def get = throw new NoSuchElementException("None.get")
}
```

Hey! `None` is a case _object_ that implements `Option[Nothing]`. That means that every time that we use `None`, we are not creating a new instance of it. Since in Scala, `Nothing` inherits from all types (including `Null`) and `Option[+A]` is covariant and `None` implements `Option[Nothing]`, we can use `None` as option of any `A`.

Getting back to the code example, this is how the code looks after my review:

```
trait Foo[-A] extends AutoClosable {
  def foo(value: A): Unit
}

case object NopFoo extends Foo[Any] { ... }

case class DefaultFoo[A] extends Foo[A] { ... }

object Foo {
  def apply[A](...): Foo[A] = {
    if (...) DefaultFoo[A]() else NopFoo
  }
}
```

We applied the same “optimization” that Scala uses for `None`, execpt using Contravariance instead of Covariance because we’re getting the type `A` as parameter instead of returning it.

### Is this a Premature Optimization?

In [Upsolver](http://www.upsolver.com) we believe that when we’re creating something, we should do it once and do it well. Nobody wants to reopen packages that have been written, tested and reviewed. That’s why we bring our A game for every pull request.

This so called “optimization” might be small. Some even might call it “premature”. I would say that premature optimization is taking steps which impact the maintainability of the code in order to improve run time. In this case, the code maintainability is not affected by this change, which turns this into an opportunity to learn how to correctly use cases like this in Scala and propogate that knowledge through our organization.

For me it’s about the culture of writing the best code we can, and learning new things every day.