## Operators

Recall that is Scala we can use any method as an operator. In particular

```scala
a b c
```

is equivalent to

```scala
a.b(c)
```

This simple feature allows us to create a more natural notation in many domains. A straightforward example is representing mathematical notation for extended numeric types. Here we implement a simple vector type with element-wise addition and multiplication.

```scala
class Vector(val data: Array[Double]) {
  def +(other: Vector):Vector =
    new Vector(
      data.zip(other.data).map {
        case (v1, v2) => v1 + v2
      }
    )

  def *(other: Vector): Vector =
    new Vector(
      data.zip(other.data).map {
        case (v1, v2) => v1 * v2
      }
    )
}

object Vector {
  def apply(xs: Double*): Vector = new Vector(xs.toArray)
}

scala> val vector = Vector(1.0, 2.0) + Vector(3.0, 4.0)
vector: Vector = Vector@58bead4f

scala> vector.data
res6: Array[Double] = Array(4.0, 6.0)
```

As a bonus we also defined a companion object with an `apply` constructor. This shows how we use variable arguments in Scala.

### Exercise

Add an element-wise exponentiation operator `**` to `Vector`, which raises each element to the given power. Use `scala.math.pow` for the per-element computation. Return a `Vector` with the results.

## Special Methods

Scala makes heavy use of convention in method names to reduce boilerplate syntax. We've already seen some examples of this:

- Use of single-argument methods as infix operators
- argumentless methods for field accessor syntax
- `apply` for function application syntax `foo(...)`
- `update` for function application update `foo(...) = bar`
- `unapply` for extending pattern matching

There are some more special method names that we either haven't yet seen or have only mentioned in passing:

- `foo_=` methods for assignment syntax
- `unary_foo` methods where `foo` can be `+`, `-`, `~`, or `!`.


### Custom Setters

The first allows us to write custom setters. For example, imagine a bank account class where we want to log to an audit trail whenever we change the balance. We can implement this as follows:

```scala
object Account {
  private var _balance = 0
  def balance = _balance
  def balance_=(newBalance: Int):Unit = {
    println("AUDIT TRAIL: Changed balance to "+newBalance)
    _balance = newBalance
  }
}

scala> Account.balance
res0: Int = 0

scala> Account.balance = 100
AUDIT TRAIL: Changed balance to 100
Account.balance: Int = 100

scala> Account.balance
res1: Int = 100
```

Note how we can write `Account.balance = 100` and the method `balance_=` is called. These setter methods can only be used in tandem with a getter. If a getter is not defined we get a runtime error.

```scala
object NoGetter {
  private var _balance = 0
  def balance_=(newBalance: Int):Unit = {
    println("AUDIT TRAIL: Changed balance to "+newBalance)
    _balance = newBalance
  }
}

scala> NoGetter.balance = 100
<console>:11: error: value balance is not a member of object NoGetter
val $ires1 = NoGetter.balance
                      ^
<console>:8: error: value balance is not a member of object NoGetter
       NoGetter.balance = 100
                ^
```

Note that whenever we write a `val` under the covers it is implemented use a getter and setter.


### Unary Methods

The unary methods allow us to define prefix operators that can be used with our objects. We can only define `+`, `-`, `~`, or `!`. Let's add a `-` prefix operator to our Vector:

```scala
class Vector(val data: Array[Double]) {
  def +(other: Vector):Vector =
    new Vector(
      data.zip(other.data).map {
        case (v1, v2) => v1 + v2
      }
    )

  def *(other: Vector): Vector =
    new Vector(
      data.zip(other.data).map {
        case (v1, v2) => v1 * v2
      }
    )
  def unary_- : Vector =
    new Vector(
      data.map (elt => -elt)
    )
}

object Vector {
  def apply(xs: Double*): Vector = new Vector(xs.toArray)
}

scala> Vector(1.0, 2.0, 3.0).unary_-.data
res6: Array[Double] = Array(-1.0, -2.0, -3.0)

scala> (- Vector(1.0, 2.0, 3.0)).data
res7: Array[Double] = Array(-1.0, -2.0, -3.0)
```


### Exercise

Extend our `Vector` implementation to support

- an `apply` method providing element access.
- an `update` method supporting element mutation.


## Operator Associativity

Operator associativity is the final wrinkle in Scala's support for operators. Any methods that ends with a colon (`:`) is right-associative. That means `a op: b` is equivalent to `b.op:(a)` rather than `a.op:(b)`. We've seen this used extensively in the collection classes. For example, `+:` is the prepend operation on `Seq`.

## Final Words

Operator syntax, and particularly symbolic operators, are a powerful way of creating a concise syntax to express domain specific operations. However it can also be a great way to obscure your code. When creating symbolic operators you must consider the costs and benefits. Generally, if an operator has a well known meaning (e.g. the arthimetic operators) or is commonly known within a specific domain *and* dealing with the domain is a major part of the program you're writing then symbolic operators can be a good approach. In other cases caution is advised.
