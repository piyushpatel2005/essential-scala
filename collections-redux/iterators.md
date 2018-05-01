## Iterators and Views

Iterators and views are two parts of the collection library that don't find much use outside of a few special cases.

### Iterators

Scala's iterators are like Java's iterators. You can use them to walk through the elements of a collection, but only once. Iterators have `hasNext` and `next` methods, with the obvious semantics. Otherwise they behave like sequences, though they don't inherit from `Seq`.

Iterators don't find a great deal of use in Scala. Two primary use cases are operating on collections that are too large to fit in memory or in particularly high performance code.


### Views

When performing a sequence of transformations on a collection, a number of intermediate collections will be constructed. For example, in the below example two intermediate collections will be created by the first and second call to `map`.

```scala
Seq(1, 2, 3).map(_ * 2).map(_ + 4).map(_.toString)
// res0: Seq[String] = List(6, 8, 10)
```

It is as if we'd written

```scala
val intermediate1 = Seq(1, 2, 3).map(_ * 2)
val intermediate2 = intermediate1.map(_ + 4)
val result = intermediate2.map(_.toString)
```

These intermediate collections are not strictly necessary. We could instead do the full sequence of transformations on an element-by-element basis. Views allows this. We create a view by calling the `view` method on any collection. Any traversals of a view are only applied when the `force` method is called.

```scala
val view = Seq(1, 2, 3).view.map(_ * 2).map(_ + 4).map(_.toString)
// view: scala.collection.SeqView[String,Seq[_]] = SeqViewMMM(...)

view.force
// res1: Seq[String] = List(6, 8, 10)
```

Note that when a view is forced the original type is retained.

For very large collections of items with many stages of transformations a view can be worthwhile. For modest sizes views are usually slower than creating the intermediate data structures.
