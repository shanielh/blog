---
title: "Implementing a fluent nested API in Scala"
description: "Background"
date: "2019-06-01T05:31:24.439Z"
categories: []
published: false
---

### Background

In my workplace we use Apache Avro heavily, It’s the core foundation in our business which relies on our customers data. So we implemented a lot of internal libraries on Avro : a reader that uses a subscription model to reduce memory pressure on the JVM and a Writer that allows to place a field pointer and write to the same pointer later. 

For those who don’t know [Apache Avro](https://avro.apache.org/docs/1.8.1/index.html) already, it offers: 

> Rich data structures; A compact, fast, binary data format; A container file, to store persistent data; Remote procedure call (RPC); Simple integration with dynamic languages. Code generation is not required to read or write data files nor to use or implement RPC protocols. Code generation as an optional optimization, only worth implementing for statically typed languages.

Our writer has a mode that creates a schema for the data written dynamically as the fields are written.

But we didn’t have a library that creates a schema from code.

#### Avro Schema

Avro Schema is composed of Primitive Types (null, int, long, etc’…), arrays, maps, records and unions (some of the above). Records are composed of fields that each field has a type. 

### Creating a schema

Avro has an implementation in Java which works great. I wanted to write a fluent API for creating Avro Schema.