---
title: "Traversing Object Graph with Shapeless (and Why You Should Write the Same Code Twice)"
description: "I’ve been writing DNS server in Scala for weeks now, on and off. Just my luck, my laptop died before I pushed the git repository to the remote server. Weighting my options, I decided it’d be easier…"
date: "2018-09-16T16:04:41.056Z"
categories: 
  - Scala
  - Coding
  - Generic Programming

published: true
canonicalLink: https://medium.com/@shanielh/traversing-object-graph-with-shapeless-and-why-you-should-write-the-same-code-twice-96fc09bc5be9
---

I’ve been writing DNS server in Scala for weeks now, on and off. Just my luck, my laptop died before I pushed the git repository to the remote server. Weighting my options, I decided it’d be easier to rewrite the whole thing than to restore the data; and since now I knew what I was facing — I could do things better.

One of the things I had to implement as part of creating a DNS server is to figure out how to serialize or deserialize the DNS packets sent over the network, and specifically each and every one of 79 existing record types¹.

In the old code, I had manually implemented serialize and deserialize methods for each record type and for parts of the data sent over the wire. This time, I thought, why not do it better?

Before reading this you should know how implicit resolution works. I recommend [Implicit Design Patterns in Scala by Li Haoyi](http://www.lihaoyi.com/post/ImplicitDesignPatternsinScala.html) and [The neophyte’s Guide to Scala Part 12: Type Classes](https://danielwestheide.com/blog/2013/02/06/the-neophytes-guide-to-scala-part-12-type-classes.html).

#### Heterogeneous Lists

I first encountered shapeless / “Generic programming for Scala” a few weeks ago, but I didn’t have time to figure out what they’re talking about. But writing the code again, I found the time — and was glad I did.

Let’s start by defining a heterogeneous list as a linked list that can contain different data types, and the data type of each item in the list can be identified by the compiler.

Embed placeholder 0.1660360601113413

The first line imports a bunch of stuff from Shapeless — we’re going to use this in the examples that follow. in the third line I’ve created a heterogeneous list that contains integer and string. As you can see, the compiler identifies each element’s type: `::[Int, ::[String, HNil]]`. You can also see that in the next line, the compiler understands the type of the expression `“Hello” :: a`.

#### Converting Case Classes to Heterogeneous Lists vice versa

Shapeless provides a way to convert case classes to heterogeneous vice versa:

Embed placeholder 0.9270141330568082

#### Traversing Heterogeneous Lists using Implicits

In my case, all I wanted is to serialize and deserialize objects, but let’s take it down a notch and write code that counts the number of objects I have in an Object Graph. The structure of our program is going to be :

Embed placeholder 0.07087642806082872

The result should be 6. Recall that we support only case classes and for each type (/ trait) which isn’t a case class that we want to support, we have to implement `ObjectCounter[A]`. Luckily, we can let the users implement it for their own types.

Let’s start by implementing `ObjectCounter[A]` for heterogeneous lists. The following implicit methods are added to `object ObjectCounter`, since we want the compiler to find them easily:

Embed placeholder 0.5191156805242336

Our code already works with heterogeneous lists! The compiler is smart enough to choose the `hListCounter` method over the `objectCounter` method when the object is `HList`.

To make our initial example code work, we have to use another trick called low priority default implicit². To do this, we’ll create a trait with implicits that will have a lower priority and then extend `object ObjectCounter` with that trait. The compiler will then assign the implicits in the trait a lower priority:

Embed placeholder 0.844837306268647

I included the `objectCounter` in the low priority implicits trait because I want `caseClassCounter` to have priority over it. Otherwise, the compiler would throw an `ambiguous implicit values` error.

Another thing I had to do is to use `shapeless.Lazy` in the `caseClassCounter` implicits. Without this, the compiler might throw `diverging implicit expansion` error.

Finally, I notified the compiler what is the type of heterogeneous list returns for the case class counter, which allows it to find an object counter for that type of list. I used `Generic.Aux`, which is what `Generic.apply[T]` returns.

Go ahead and run the example code, If you did everything right, `6` will be printed to the screen.

#### Conclusions

Using Shapeless you can write functions that don’t care about the underlying object type, which explains the library’s tagline — Generic programming for Scala.

Detaching emotionally from the code you write is a great way to learn and evolve your coding skills. I wish I had time to write each piece of code twice or more.

---

¹ [Wikipedia: List of DNS record types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)

² [Between Zero & Hero — Scala Tips and Tricks for the intermediate Scala developer (Slide 32)](https://speakerdeck.com/agemooij/between-zero-and-hero-scala-tips-and-tricks-for-the-intermediate-scala-developer?slide=32)