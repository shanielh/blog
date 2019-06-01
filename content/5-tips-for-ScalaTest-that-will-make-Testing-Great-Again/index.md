---
title: "5 tips for ScalaTest that will make Testing Great Again"
description: "I was exercising coding by writing a library that basically does “retry” mechanisms based on the great zmanio/atmos library which unfortunately isn’t maintained any more and not published for scala…"
date: "2017-05-06T15:18:13.825Z"
categories: 
  - Testing
  - Scala
  - Scalatest
  - Coding
  - Programming

published: true
canonicalLink: https://medium.com/@shanielh/5-tips-for-scalatest-that-will-make-testing-great-again-58190df1df88
---

I was exercising coding by writing a library that basically does “retry” mechanisms based on the great [zmanio/atmos](https://github.com/zmanio/atmos/) library which unfortunately isn’t maintained any more and not published for scala 2.12.

I started writing down tests after writing some basic traits and implementations and this time I challenged myself to write the tests as good as I can.

This is what I learned from the process, let’s start.

![](./asset-1.jpeg)

### Test Cases

According to ScalaTest documentation, you can use`TableDrivenPropertyCheck`only with `PropSpec`, but there is another way to create “Test Cases” easily with almost any other test style:

Embed placeholder 0.34881372850240777

And the run results are:

```
[info] CalculatorSpec:
[info] add
[info] - should return 2 for 1 and 1
[info] add
[info] - should return 4294967294 for 2147483647 and 2147483647
```

As you can see, the “add” word is appears twice in the log. I really tried to do something about it, but the best option I found was to change the spec style to `FunSuite`:

Embed placeholder 0.15936923845114515

Now the log looks better:

```
[info] CalculatorSpec:
[info] - add should return 2 for 1 and 1
[info] - add should return 4294967294 for 2147483647 and 2147483647
```

### TypeCheckedTripleEquals

ScalaTest supports the famous `===` operator. For those of you who haven’t learned JavaScript, this operator asserts equality for the value and type. It’s really useful when you change the return type of a method and you assert on the method result - in that case the compilation will fail.

Using it is simple, Just mix-in (or import) the `org.scalactic.TypeCheckedTripleEquals` trait and use it :

Embed placeholder 0.6395150432620695

This code above will not compile:

```
[error] .../Examples.scala:8: types Long and Int do not adhere to the type constraint selected for the === and !== operators; the missing implicit parameter is of type org.scalactic.CanEqual[Long,Int]
[error]     target should === (5)
```

To make it compile, You have to change line 8 to :

```
target should === (5L)
```

### Abstract Base Spec

You can make `abstract` specs to reuse the same code in tests that test the same trait. You may think that this is a use case for [Behavior Driven Tests](http://www.scalatest.org/user_guide/sharing_tests). But our implementations don’t behave the same, so we want different cases for every implementation. I’ll demonstrate one way to do it.

Lets assume we have a trait which calculates delay based on the number of failed attempts:

```
trait Backoff {
  def calculate(attempts: Int): FiniteDuration
}
```

We can have many implementations for this trait: `NoBackoff`, `ConstantBackoff`, `LinearBackoff`, `ExponentialBackoff, FibonacciBackoff`, `RandomBackoff` and many more.

We can fall off to repeat ourselves when writing the tests. Instead, let’s implement a `BackoffSpec` which is abstract and derive from it for every implementation :

Embed placeholder 0.45944740253727345

You can see that I used the first tip to create many test cases for every `Spec`, check out the log created from the run of those tests:

```
[info] ConstantBackoffSpec:
[info] - calculate should return 1 second for 1 attempts
[info] - calculate should return 1 second for 2 attempts
[info] NoBackoffSpec:
[info] - calculate should return 0 days for 1 attempts
[info] - calculate should return 0 days for 2 attempts
```

Nice, and the best part — ScalaTest won’t run the abstract spec since it’s abstract!

### Custom Matchers

ScalaTest allows us to write custom matchers for specific types and it allows us to make the test more readable, especially when testing domain objects.

This feature is [documented very well](http://www.scalatest.org/user_guide/using_matchers#usingCustomMatchers). And their example is great:

```
file should endWithExtension ("txt")
```

is much more readable than :

```
file.getName should endWith (".txt")
```

### Bonus Tip: Continuous Testing with SBT

My favorite way to work on tests is to split screen into two: One part for IntelliJ IDEA (or your favorite IDE) and the other part for terminal running SBT with a special command running in it.

-   According to SBT documentation, prefixing command with a tilde sign will make it watch the source code and re-run on every change (see: [Triggered Execution](http://www.scala-sbt.org/0.13/docs/Triggered-Execution.html))
-   According to ScalaTest documentation, “To run only tests affected by your latest code changes, either in main or test, you can use `test-quick`" (see: [Using ScalaTest with SBT](http://www.scalatest.org/user_guide/using_scalatest_with_sbt))

When running `~test-quick` we’re getting the best of all worlds: SBT will watch for changes in our source code and ScalaTest will run only the tests that are affected by the changes made.

Running tests using this method lets you code without interruptions to your workflow, and the confidence knowing you hadn’t broken anything.

#### To Summarize

Writing automated tests is part of our job as software developers and these should not be considered as second class citizens in our code base.

Once you start treating your tests code the same as “run time” code, it’ll be far more challenging and fun to code it.

Enjoy testing and please respond if you have other ways to make “Testing Great Again!”