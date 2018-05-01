## Creating Type Classes

In the previous sections we saw how to create and use type class instances. Now we're going to explore creating our own type classes.

### Elements of Type Classes

There are four components of the type class pattern:

- the actual type class itself;
- the type class instances;
- interfaces using implicit parameters; and
- interfaces using enrichment and implicit parameters.

We have already seen type class instances and talked briefly about implicit parameters. Here we will look at defining our own type class, and in the following section we will look at the two styles of interface.

### Creating a Type Class

Let's start with an example---converting data to HTML. This is a fundamental operation in any web application, and it would be great to be able to provide a `toHtml` method across the board in our application.

One implementation strategy is to create a trait we `extend` wherever we want this functionality:

```scala
trait HtmlWriteable {
  def toHtml: String
}

final case class Person(name: String, email: String) extends HtmlWriteable {
  def toHtml = s"<span>$name &lt;$email&gt;</span>"
}
```

```scala
Person("John", "john@example.com").toHtml
// res1: String = <span>John &lt;john@example.com&gt;</span>
```

This solution has a number of drawbacks. First, we are restricted to having just one way of rendering a `Person`. If we want to list people on our company homepage, for example, it is unlikely we will want to list everybody's email addresses without obfuscation. For logged in users, however, we probably want the convenience of direct email links. Second, this pattern can only be applied to classes that we have written ourselves. If we want to render a `java.util.Date` to HTML, for example, we will have to write some other form of library function.

Polymorphism has failed us, so perhaps we should try pattern matching instead? We could write something like




```scala
object HtmlWriter {
  def write(in: Any): String =
    in match {
      case Person(name, email) => ???
      case d: Date => ???
      case _ => throw new Exception(s"Can't render ${in} to HTML")
    }
}
```

This implementation has its own issues. We have lost type safety because there is no useful supertype that covers just the elements we want to render and no more. We can't have more than one implementation of rendering for a given type. We also have to modify this code whenever we want to render a new type.

We can overcome all of these problems by moving our HTML rendering to an adapter class:

```scala
trait HtmlWriter[A] {
  def write(in: A): String
}

object PersonWriter extends HtmlWriter[Person] {
  def write(person: Person) = s"<span>${person.name} &lt;${person.email}&gt;</span>"
}
```

```scala
PersonWriter.write(Person("John", "john@example.com"))
