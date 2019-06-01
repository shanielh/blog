---
title: "Writing your first Scala script using Ammonite"
description: "Ammonite, a package of libraries for Scala scripting is finally out. Its website states:"
date: "2019-06-01T05:31:31.792Z"
categories: []
published: false
---

Ammonite, a package of libraries for Scala scripting is finally out. Its website states: 

> Ammonite lets you use the [Scala language](https://www.scala-lang.org/) for scripting purposes: in the [REPL](http://ammonite.io/#Ammonite-REPL), as [scripts](http://ammonite.io/#ScalaScripts), as a [library to use in existing projects](http://ammonite.io/#Ammonite-Ops), or as a standalone [systems shell](http://ammonite.io/#Ammonite-Shell).

I was really excited and found the perfect reason to use it when my manager told me that we need to send an email every hour to one of our clients with some operational KPIs. We’re running the script over TeamCity as a scheduled build that runs every hour to that specific client request.

Before we start, [Install Ammonite](http://ammonite.io/#Ammonite-REPL) on your computer by running: 

```
$ sudo curl -L -o /usr/local/bin/amm https://git.io/vQEhd && sudo chmod +x /usr/local/bin/amm && amm
```

### Creating our first Scala Script

Let’s save our first script file in a “script.sc” file : 

```
println("Welcome to Scala Scripting!")
```

Now head to the directory in which you saved the file and run in the terminal

```
amm script.sc
```

The output should be “Welcome to Scala Scripting!”. Let’s take it up a notch and use some Java Dependencies: 

```
import java.time.Instant;

println("Welcome to Scala Scripting!")
println("The time is " + Instant.now())
```

Using Ivy Dependencies works as well: 

```
import $ivy.`org.backuity::ansi-interpolator:1.1.0`, org.backuity.ansi.AnsiFormatter.FormattedHelper

println(ansi"Welcome to %bold{Scala Scripting}!")
```

Awesome, we have the JVM force with us. 

**note**: You can also import other script file `import $file.FileName` (without the .sc extension).

### @main Methods

Ammonite lets us define in our scripts multiple main methods using the `@main` annotation.

```
@main
def main(repeats: Int, string: String): Unit = {
  for (i <- 1 to repeats) {
    println(string.format(i));
  }
}
```

Running the script with `amm script.sc` will yield

```
Missing arguments: (--repeats: Int, --string: String)

Arguments provided did not match expected signature:

main
  --repeats  Int
  --string   String
```

Running again with `amm script.sc --repeats 5 --string "%d. Hello World"` will yield: 

```
1. Hello World
2. Hello World
3. Hello World
4. Hello World
5. Hello World
```

As we expected. You can have more than one `@main` method as long as they have different names and you will explicitly have to say which one to run.