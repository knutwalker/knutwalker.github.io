---
layout: post
title: Fun with scalatests PropertyChecks
tags:
- scala
---

I just want to drop off some things that might be useful with [ScalaTest](http://www.scalatest.org/ "ScalaTest") and [ScalaCheck](http://scalacheck.org/ "ScalaCheck").

I prefer using [ScalaTest `with PropertyChecks`][1] over [ScalaCheck Properties][2], mainly because I can easily use `Matcher`s within a property.
Here are a few things I discovered while working with these two toolkits (tbc):

* * *

## Version compatibility

The latest version of ScalaTest, `2.0` does not work with the latest version of ScalaCheck, `1.11.1`.
You have to use `1.10.1` of ScalaCheck:

```scala
libraryDependencies ++= Seq(
  "org.scalatest"  %% "scalatest"  % "2.0"    % "test",
  "org.scalacheck" %% "scalacheck" % "1.10.1" % "test"
)
```

* * *

## No Shrinking

[Shrinking][3] is an interesting concept of ScalaCheck, that will try to find the minimal failing test case for a property.
However, shrinking ignores restrictions of your generator, e.g. shrinking a `Gen.choose(1, 10)` might try to falsify the property with values such as `0`, and `-1`.
If these values don't lie within your domain, this will definitely lead to an error, but one that shadows the actual implementation error.

Take this example:

```scala
val m = Map(
  1 -> 1, // wrong
  2 -> 4, // correct
  3 -> 6  // correct
)

forAll(Gen.chooseNum(1, 3)) {
  n => m(n) should be (n * 2)
}
```

This should fail with `Message: 1 was not equal to 2 ... Occurred when passed generated values (arg0 = 1)`
But instead it fails with `Message: key not found: 0 ... Occurred when passed generated values (arg0 = 0 // 1 shrink)`

ScalaCheck has a `Prop.forAllNoShrink` to disable shrinking, but ScalaTest has no equivalent.
Fortunately, Shrinking can be configured as a TypeClass `Shrink[T]`. The interface is easy enough:

```scala
sealed abstract class Shrink[T] {
  def shrink(x: T): Stream[T]
}

object Shrink {
  def apply[T](s: T => Stream[T]): Shrink[T] = new Shrink[T] {
    override def shrink(x: T) = s(x)
  }
}
```


The created `Stream[T]` is used to provide all values that should be tried during the shrinking process.
To disable shrinking, one just has to return an empty `Stream[T]`. There is a default `Shrink.shrinkAny`, which does just that.

We just need to bring that implicitly into scope as a `Shrink[Int]` and the test should fail with the proper error message:

```scala
import org.scalacheck.Shrink
implicit val noShrink: Shrink[Int] = Shrink.shrinkAny

val m = Map(
  1 -> 1, // wrong
  2 -> 4, // correct
  3 -> 6  // correct
)

forAll(Gen.chooseNum(1, 3)) { n =>
  m(n) should be (n * 2)    // Message: 1 was not equal to 2 ... Occurred when passed generated values (arg0 = 1)
}
```

Cool, this works!

Allegedly, this is [fixed](https://github.com/rickynils/scalacheck/issues/8) with `1.11.0`, but this version is not compatible with ScalaTest (as might the fix, i haven't looked into it).

* * *

## Determinacy

Sometimes, you want to replay a test with the exact same generated values, have it run deterministically with a specific seed value.
This can be done per-generator or per property test.
The crux is to replace the random generator with a custom one and the only way to do so is with a `Gen.parameterized`

```scala
def deterministicNumberGen: Gen[Int] = {
  val r = new java.util.Random(100L)
  Gen.parameterized { params =>
    // params.copy to use the custom Random Number Generator
    Gen.value(params.copy(rng = r).choose(0, 100)).map(_.toInt)
  }
}

var l1 = List.empty[Int]
var l2 = List.empty[Int]

forAll(deterministicNumberGen) { n => l1 = n :: l1 }
forAll(deterministicNumberGen) { n => l2 = n :: l2 }

l1 shouldBe l2

println(l1 take 10 mkString ", ")   // => 55, 65, 92, 52, 16, 12, 0, 24, 21, 76
```

This is effectively the same as a `Gen.choose(0, 100)`, only that its seed value is fixed to 100.
Using a custom generator like this requires you to basically rewrite all other existing generator, a task you probably don't want to do.

TypeClasses to the rescue! Most (if not all) generator eventually use `Gen.choose[T]` to generate their values and `Gen.choose[T]` uses the TypeClass `Choose[T]` to actually generate the random values.
So, one just has to bring a `Choose[T]` into the implicit scope, that uses a custom RNG.

```scala
import org.scalacheck.Choose
implicit val chooseInt: Choose[Int] = new Choose[Int] {
  def choose(low: Int, high: Int) = {
    if (low > high) Gen.fail
    else {
      val r = new java.util.Random(100L)
      Gen.parameterized { params =>
        // params.copy to use the custom Random Number Generator
        Gen.value(params.copy(rng = r).choose(low, high)).map(_.toInt)
      }
    }
  }
}

var l1 = List.empty[Int]
var l2 = List.empty[Int]

forAll(Gen.choose(0, 100)) { n => l1 = n :: l1 }
forAll(Gen.choose(0, 100)) { n => l2 = n :: l2 }

l1 shouldBe l2

println(l1 take 10 mkString ", ")   // => 55, 65, 92, 52, 16, 12, 0, 24, 21, 76
```

Now, you only have to follow [ScalaChecks `Choose[T]`](https://github.com/rickynils/scalacheck/blob/1.10.1/src/main/scala/org/scalacheck/Gen.scala#L24) and implement this for each of the remaining `Choose[T]`s and you can run deterministic tests with all other generators.

**Note**: This probably only work with `1.10.x`, it seems `1.11.x` take a slightly different approach. But then again, I haven't looked into it yet.

* * *


  [1]: http://www.scalatest.org/user_guide/generator_driven_property_checks "Generator Driven Property Checks"
  [2]: https://github.com/rickynils/scalacheck/wiki/User-Guide#properties "ScalaCheck Properties"
  [3]: https://github.com/rickynils/scalacheck/wiki/User-Guide#test-case-minimisation "Shrinking"
