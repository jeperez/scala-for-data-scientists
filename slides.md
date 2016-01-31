<div style="border-radius: 10px; background: #EEEEEE; padding: 20px; text-align: center; font-size: 1.5em">
  <big><b>Introduction to Scala </b></big> </br>
  </br>
  Chris McKinlay <br/>
  <code>chris@datascience.com</code> <br/>
  <br/>

</div>

---

#John Backus
![](img/john_backus.jpg)

"Conventional programming languages are growing
ever more enormous, but not stronger. Inherent defects
at the most basic level cause them to be both fat and
weak: their primitive word-at-a-time style of programming
inherited from their common ancestor--the von
Neumann computer, their close coupling of semantics to
state transitions, their inability to effectively use powerful
combining forms for building new programs from existing ones,
and their lack of useful mathematical properties for reasoning
about programs..."

-  [Can Programming Be Liberated from the von Neumann Style?](http://dl.acm.org/citation.cfm?id=359579), 1978 Turing Award Lecture

.notes: Decouple semantics from state transitions. Use powerful primitives.

---

#The von Neumann Bottleneck

We need other techniques for defining high-level abstractions such
as collections, polynomials, geometric shapes, strings, documents.
Ideally: Develop theories of collections, shapes, strings, …

 * *Reduce* syntactical noise
 * Provide useful types that solve *many classes* of problems
 * *Add* type-safety with minimal "extra work"

---

#Language Elements

---

#Lexical Scoping

    !scala
    val x = 0
    def f(y: Int) = y + 1
    val result = {
      val x = f(3)
      x * x
    } + x

.notes: QUESTION: what is the value of x? y is a bound variable, x is free.

---

#Never use `return` ([why?](https://tpolecat.github.io/2014/05/09/return.html))

    !scala
    def max(x: Int, y: Int) = {
      if (x>y) x
      else y
    }

.notes:  the return value is by default the last line. Scala has a `return` keyword, but you should never use it.

___

#Type Declaration and Inference

    !scala
    scala> def plusOne(x: Int) = x + 1
    plusOne: (x: Int)Int

.notes: Scala infers the return type. This can make for very concise code. Also Python-like

___

#Recursion

    !scala
    def factorial(n: Int): Int = {
      if (n == 0) 1
      else n * factorial(n-1)
    }

.notes: with recursive functions we have to define the return type otherwise the type inference algorithm would get stuck in an infinite loop.  

---

#Function Literals

    !scala
    scala> (1 to 3).map((arg: Int) => factorial(arg))
    res0: immutable.IndexedSeq[Int] = Vector(1, 2, 6)

.notes:  Scala's type system is very rich. We'll talk a lot more about Scala's collection library in the next weeks. It's data structures are much more powerful that Python's. IndexedSeq is a Trait (like an interface in Java)

---

#Function Literals

    !scala
    scala> (1 to 3).map(arg => factorial(arg))
    res1: immutable.IndexedSeq[Int] = Vector(1, 2, 6)
    scala> (1 to 3).map(factorial _ )
    res2: immutable.IndexedSeq[Int] = Vector(1, 2, 6)
    scala> (1 to 3).map(factorial)
    res3: immutable.IndexedSeq[Int] = Vector(1, 2, 6)
    scala> (1 to 3) map factorial
    res4: immutable.IndexedSeq[Int] = Vector(1, 2, 6)

.notes: Just like lambdas in Python. Scala's type system infers the Int arg and return types. If the literal consists of a single statement (the return) and a single argument then you can leave them out. These are called partially applied functions and we will return to them next class. Scala's type system infers the Int arg and return types. Method dots are optional.

---

#Procedures vs Processes

    factorial(6)
    6 * factorial(5)
    6 * 5 * factorial(4)
    6 * 5 * 4 * factorial(3)
    6 * 5 * 4 * 3 * factorial(2)
    6 * 5 * 4 * 3 * 2 * factorial(1)
    6 * 5 * 4 * 3 * 2 * 1 * factorial(0)
    6 * 5 * 4 * 3 * 2 * 1 * 1
    720

.notes: Let's look at the computational process that results from calling factorial. Odersky discusses the same thing in the context of rewriting rules.

---

    !scala
    scala> factorial(100000)
    java.lang.StackOverflowError

---

#Tail Recursion

    !scala
    def factorial2(n: Int): Int = {
      def factIter(n: Int, total: Int): Int = {
        if (n == 0) total
        else factIter(n-1, n*total)
      }
      factIter(n,1)
    }
    (0 to 5).map(factorial2)

---

#Iterative Processes

    factorial2(6)
    factIter(6,1)
    factIter(5,6)
    factIter(4,30)
    factIter(3,120)
    factIter(2,360)
    factIter(1,720)
    factIter(0,720)
    720


---

    !scala
    def fibonacci(n: Int): Int = {
      if (n == 0 || n == 1) 1
      else fibonacci(n - 1) + fibonacci(n - 2)
    }

.notes: QUESTION: what is the aysmptotic computational complexity of fibonacci(n)?

---

    fibonacci(4)
    fibonacci(3)
    fibonacci(2)
    fibonacci(1)
    1
    fibonacci(0)
    1
    fibonacci(1)
    1
    fibonacci(2)
    fibonacci(1)
    1   
    fibonacci(0)
    1

---

![](img/fib-process.png)

.notes: Notice that the entire computation of (fib 3) -- almost half the work -- is duplicated. Process is O(theta^n)

---

    !scala
    (0 to 10) map fibonacci
    (0 to 30) map fibonacci
    (0 to 50) map fibonacci

---

    !scala
    def fibonacci2(n: Int): Int = {
      def fibIter(n: Int, a: Int, b: Int): Int = {
        if (n==0) a
        else fibIter(n-1, b, a+b)
      }
      fibIter(n,1,1)
    }
    (0 to 50) map fibonacci2

---

![](img/fib-mat.png)

---

#Avoiding the von Neumann Bottleneck

---

Avoid conceptualizing data structures word-by-word.

* imperative state  
* loop-based logic
* referential-opacity

---

#Step 1: use var sparingly

    !scala
    def printArgs1(args: Array[String]): Unit = {
      var i = 0
      while (i < args.length) {
        println(args(i))
        i += 1
      }
    }

    def printArgs2(args: Array[String]): Unit = {
      for (arg <- args)
        println(arg)
    }

.notes: for (arg <- args) is Scala's equivalent of a list comprehension

---

#Step 2: use loops sparingly

    !scala
    def printArgs2(args: Array[String]): Unit = {
      for (arg <- args)
        println(arg)
    }

    def printArgs3(args: Array[String]): Unit = {
      args.map(println)
    }

.notes: even simpler to pass println to map

---

#Step 3: use side effects sparingly

    !scala
    def printArgs3(args: Array[String]): Unit = {
      args.map(println)
    }

    def formatArgs(args: Array[String]): String = args.mkString("\n")
    println(formatArgs(Array("nine", "ten")))

.notes: We've isolated the non-referentially transparent code to one line. You can tell b/c the return type is not Unit. We can now refactor the code without fear of breaking things.

---

#Imperative QuickSort

    !scala
    def swap(i: Int, j: Int): Unit =  {
      val t = xs(i); xs(i) = xs(j); xs(j) = t
    }

.notes: QUESTION: What will happen if I run this in the interpreter? xs is in the closure of swap, but the compiler won't be able to find it.

---

    !scala
    def quickSort(xs: Array[Int]): Unit = {
      def sortHelper(l: Int, r: Int): Unit =  {
        val pivot = xs((l + r) / 2)
        var i = l; var j = r
        while (i <= j) {
          while (xs(i) < pivot) i += 1
          while (xs(j) > pivot) j -= 1
          if (i <= j) swap(i, j); i += 1; j -= 1  
        }
        if (l < j) sort1(l, j)
        if (j < r) sort1(i, r)
      }
      sortHelper(0, xs.length - 1)
    }

---

#Functional QuickSort

    !scala
    def quickSort2(xs: Array[Int]): Array[Int] = {
      if (xs.length <= 1) xs
      else {
        val pivot = xs(xs.length / 2)
        Array.concat(
          quickSort2(xs filter (pivot >)),
          xs filter (pivot ==),
          quickSort2(xs filter (pivot <)))
      }
    }

.notes: The functional program captures the essence of the quicksort algorithm in a concise way. But where the imperative implementation operates in place by modifying the argument array, the functional implementation returns a new sorted array and leaves the ar- gument array unchanged. The functional implementation makes it look like Scala is a language that’s specialized for functional operations on arrays. In fact, it is not; all of the operations used in the example are simple library methods of a sequence class Seq[T] which is part of the standard Scala library, and which itself is implemented in Scala. Because arrays are instances of Seq all sequence methods are available for them.The functional implementation thus requires more tran- sient memory than the imperative one. QUESTION: what is going on with xs filter (pivot >) ?

---

#John Backus
![](img/john_backus.jpg)

"...An alternative functional style of programming is
founded on the use of combining forms for creating
programs. Functional programs deal with structured
data, are often non-repetitive and non-recursive, are hierarchically
constructed, and do not require the complex machinery of procedure
declarations to become generally applicable. Combining
forms can use high level programs to build still higher
level ones in a style not possible in conventional languages."

-  [Can Programming Be Liberated from the von Neumann Style?](http://dl.acm.org/citation.cfm?id=359579), 1978 Turing Award Lecture

---

#Challenge Question

Write a pair of functions to print out Pascal's Triangle: one function to generate the numbers and one to format the string.

    !scala
    scala> println(showTriangle(10))
    1
    1 1
    1 2 1
    1 3 3 1
    1 4 6 4 1
    1 5 10 10 5 1
    1 6 15 20 15 6 1
    1 7 21 35 35 21 7 1

.notes: No side effects!

---

#A Bit of History

![](img/history.png)

[https://medium.com/@markobonaci/the-history-of-hadoop-68984a11704](https://medium.com/@markobonaci/the-history-of-hadoop-68984a11704)

---

#OpenMP (1997)
![](img/OpenMP.jpg)

.notes: started w/ Backus' Fortran 1.0 in 1997. parallel processing only done on expensive supercomputers. complex and not robust to node failures.

---

#MapReduce (2004)
![](img/mapreduce.png)

.notes: combined FP primitives with world-class cluster computing know-how. the simple MR framework (read Backus) meant that all of a sudden it became relatively simple and reliable to compute with large arrays of cheap nodes, any of which might fail. these guys are legends. they are also behind tensorFlow.

---

#Hadoop (2006)
![](img/hadoop-ecosystem.png)

.notes: open source MapReduce. became a top level apache project in 2006. batched processing. static pageRank anecdote. by 2006 google had already moved on to streaming.

---

#Kafka and Spark (2012)
![](img/kafka-spark.png)

.notes: Both Scala projects. Came out of LinkedIn and Berekley's AmpLabs respectively. Framework and language have converged. Scala Collections API, Spark API, Scalding API, Kafka API are almost identical.

---

#Back to Functional QuickSort

    !scala
    def quickSort2(xs: Array[Int]): Array[Int] = {
      if (xs.length <= 1) xs
      else {
        val pivot = xs(xs.length / 2)
        Array.concat(
          quickSort2(xs filter (pivot >)),
          xs filter (pivot ==),
          quickSort2(xs filter (pivot <)))
      }
    }

.notes: The functional program captures the essence of the quicksort algorithm in a concise way. But where the imperative implementation operates in place by modifying the argument array, the functional implementation returns a new sorted array and leaves the ar- gument array unchanged. The functional implementation makes it look like Scala is a language that’s specialized for functional operations on arrays. In fact, it is not; all of the operations used in the example are simple library methods of a sequence class Seq[T] which is part of the standard Scala library, and which itself is implemented in Scala. Because arrays are instances of Seq all sequence methods are available for them.The functional implementation thus requires more tran- sient memory than the imperative one. QUESTION: what is going on with xs filter (pivot >) ?

---

The `filter` method of an object of type `Array[T]` thus has the signature:

    !scala
    def filter(p: T => Boolean): Array[T]

.notes: In particular, there is the method filter which takes as argument a predicate func- tion. This predicate function must map array elements to boolean values. The result of filter is an array consisting of all the elements of the original array for which the given predicate function is true. Here,T => Boolean is the type of functions that take an element of type t and return a Boolean. Functions like filter that take another function as argument or return one as result are called higher-order functions.

---

    !scala
    scala> val foo = (x: Int) => x % 2 == 0
    foo: Int => Boolean = <function1>
    scala> numbers.filter(_ % 2 == 0)
    res0: List[Int] = List(2, 4, 6, 8, 10, 12)    

.notes: very idiomatic in Scala to use _ for throwaway parameters. Easier on the working memory.

---

    !scala
    val foo = _ + _

.notes: QUESTION: is this an error? why? the compiler needs more type information to infer missing parameters.

---


    scala> val foo = _ + _
    error: missing parameter types
    val foo = (_: Int) + (_: Int)
    foo: (Int, Int) => Int = <function2>

.notes: QUESTION: what is <function2>?

---

#What is `function2`?
[http://www.scala-lang.org/api/rc2/scala/Function2.html](http://www.scala-lang.org/api/rc2/scala/Function2.html)

---

#Partially Applied Functions

    !scala
    def sum(a: Int, b: Int, c: Int) = a + b + c
    val foobar = sum(1, 2, _: Int)

.notes: I told you last time we'd get to these. QUESTION: what is the type of foobar?

---

#Partially Applied Functions

    !scala
    scala> val foobar = sum(1, 2, _: Int)
    foobar: Int => Int = <function1>
    scala> foobar(3)
    res0: Int = 6

.notes: you can also partially apply a function to none of its arguments

---

#Partially Applied Functions

    !scala
    scala> println(sum)
    error: value sum is not a member of Unit
    scala> println(sum _ )
    <function1>



.notes: This is an example of the design trade-offs of Scala and classical functional languages such as Haskell or ML. In these languages, partially applied functions are considered the normal case. Furthermore, these languages have a fairly strict static type system that will usually highlight every error with partial applications that you can make. Scala bears a much closer relation to imperative languages such as Java, where a method that’s not applied to all its arguments is considered an error. Scala normally requires you to specify function arguments that are left out explicitly, even if the indication is as simple as a ‘_’. Scala allows you to leave off even the _ only when a function type is expected.


---

    !scala
    def sum(f: Int => Int, a: Int, b: Int): Int = {
      if (a > b) 0
      else f(a) + sum(f, a + 1, b)
    }

---

#What is the return type here?

    !scala
    def sum(f: Int => Int, a: Int, b: Int): Int = {
      if (a > b) 0
      else f(a) + sum(f, a + 1, b)
    }
    sum(_ * 2, _: Int, _: Int)

---

    !scala
    scala> sum(_ * 2, _: Int, _: Int)
    res0: (Int, Int) => Int = <function2>

---

    !scala
    def sumDoubles(a: Int, b: Int): Int =
      sum(_ * 2, a, b)
    def sumQuintuples(a: Int, b: Int): Int =
      sum(_ * 5, a, b)

.notes: but this isn't DRY, I'm carrying a and b around everywhere

---

#Currying

    !scala
    def s2(f: Int => Double): (Int, Int) => Double = {
      def sumF(a: Int, b: Int): Double =
        if (a > b) 0 else f(a) + sumF(a + 1, b)
      sumF
    }  
    s2(x => x * x)(0, 10)

.notes: the function sum is applied to the squaring function (x => x * x). The resulting function is then applied to the second argument list: (0, 10).

---

#Currying

    !scala
    def s3(f: Int => Double)(a: Int, b: Int): Double =
      if (a > b) 0 else f(a) + sum3(f)(a + 1, b)

---

#What is the return value here?

    def foo(x: Int): Double = 1/factorial(x).toDouble
    s3(foo)(0,10)

---

    !scala
    scala> s3(foo)(0,10)
    res0: Double = 2.7182818

---

#Pascal's Triangle

    !scala
    def pascal(r: Int)(c: Int): Int = {
      if (r == 0 || c == 0 || r == c) 1
      else pascal(r-1)(c-1) + pascal(r-1)(c)
    }

    def showTriangle(rows: Int) = {
      val lines = for {
        r <- 0 until rows
      } yield (0 to r) map pascal(r) mkString " "
      "\n" + (lines mkString "\n")
    }
    println(showTriangle(10))

---

    !scala
    scala> pascal(4)(_)
    res8: Int => Int = <function1>

---

# Thanks!
