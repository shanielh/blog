---
title: "Scala Code Reading: esclient / Magnet Pattern"
description: "Disclaimer: I started writing this post long before ES client supported the REST endpoint that I’m currently using. This post is more about the Magnet Pattern in Scala. Weeks ago I looked for…"
date: "2017-05-01T07:22:43.572Z"
categories: 
  - Scala
  - Programming
  - Type Classes
  - Code Reading
  - Code

published: true
canonicalLink: https://medium.com/@shanielh/scala-code-reading-esclient-magnet-pattern-ae761479b22a
---

First post in Medium 🎊 .

_Disclaimer_: I started writing this post long before ES client supported the REST endpoint that I’m currently using. This post is more about the Magnet Pattern in Scala.

Weeks ago I looked for ElasticSearch client library in Scala / Java for something I was working on. Since I’m working in Scala, I opened the [list of Scala libraries in ElasticSearch documentation](https://www.elastic.co/guide/en/elasticsearch/client/community/current/index.html#scala) and I ran over the list.

I looked specifically for a thin client that will work against the HTTP REST endpoint of ElasticSearch. The default Java client joins the cluster and acts as a node. The software I’m working on should be as much as stateless as it can be, so I couldn’t use it (too many threads and state).

I ran across this library called [esclient](https://github.com/scalastuff/esclient) that contains two source files. It isn’t maintained but it gave me idea that maybe I should write a code that looks the same in my source since I’m going to run a small set of commands against ES.

#### ESClient.scala

This file contains the facade to ElasticSearch, and [it’s pretty simple](https://github.com/scalastuff/esclient/blob/master/src/main/scala/org/scalastuff/esclient/ESClient.scala). The only bad thing here, is that this client does not use the REST endpoint, it uses the Java client to query ES and thus, joins the cluster as a node.

The one cool thing about it, is that it uses the Magnet pattern, which is a Type class that ties two other types together. If you don’t know what Type Classes are, you really should [read about it](http://danielwestheide.com/blog/2013/02/06/the-neophytes-guide-to-scala-part-12-type-classes.html).

The magnet pattern in this case ties request type with response type. So when you use the client, you won’t be able to request A and expect B.

#### [ActionMagnet.scala](https://github.com/scalastuff/esclient/blob/master/src/main/scala/org/scalastuff/esclient/ActionMagnet.scala)

This file contains all the implicit values for ActionMagnets that can be used with the ESClient.

How does it work?

Embed placeholder 0.6170308924983643

You can see that the function has three type parameters, and it has one parameter of type `Action[Request, Response, RequestBuilder, Client]`. Those types are all from the underlying Java ElasticSearch library. You can see that the author used the strong type system of the underlying library to be able to make things “safe”.

The compiler won’t let you to call `ESClient.execute` with `Request` and `Response` that doesn’t have matching `ActionMagnet` for them. The implicit `ActionMagnet`s are found in the Companion Object for the `ActionMagnet` trait ([Read more about implicit resolution in Scala](http://docs.scala-lang.org/tutorials/FAQ/finding-implicits.html)).

If there are actions that you want to call and the library doesn’t implement, you can implement your own `ActionMagnet`. Too bad that the factory methods are private.

#### Conclusion

Reading code is a great way to learn from other people experience. Reading this library took me about 10 minutes and I learned from it two things:

-   Libraries should follow the [Open/Closed principle](https://en.wikipedia.org/wiki/Open/closed_principle) to lower their maintenance and extend their usage.
-   Type Classes in Scala are great for _compile time_ safety. The Magnet Pattern creates another level of safety by tying two types (or more) that shouldn’t be tied together in the first place (in our case: Data Transfer Objects).