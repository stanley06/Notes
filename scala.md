## Implicits

## Polymorphism
- subtyping polymorphism
    - using `extends` keyword
    - polymorphism decision is made at run time
- parametric polymorphism
    - use type parameter
    - no polymorphism decision to make
- ad-hoc polymorphism
    - based on `implicit` classes/conversions
    - polymorphism decision is made at compile time


```scala
trait Appendable[A] {
    def append(a: A): A
}

class AppendableInt(i: Int) extends Appendable[Int] {
    override def append(a: Int) = i + a
}

class AppendableString(s: String) extends Appendable[String] {
    override def append(a: String) = s.concat(a)
}

implicit def toAppendInt(i: Int) = new AppendableInt(i)
implicit def toAppendString(s: String) = new AppendableString(s)

def appendItems[A](a: A, b: A)(implicit ev: A => Appendable[A]) = a append b

val res1 = appendItems(1,2)
val res2 = appendItems("2", "3")
```



## Type Classes (a.k.a Functional Programming Design Pattern)
- easy to extend libraries (add implementation for new types/overwrite implementation for existing types) without touching existing code

```scala
trait Appendable[A] {
    def append(a: A, b: A) A
}

object Appendable {
    implicit val appendableInt = new Appendable[Int] {
        override def append(a: Int, b: Int) = a + b
    }
    implicit val appendableString = new Appendable[String] {
        override def append(a: String, b: String) = a.concat(b)
    }
    // overwrite implementation for existing type
    implicit val appendableInt2 = new Appendable[Int] {
        override def append(a: Int, b: Int) = a*b
    }

    def appendEvidence[A](a: A, b: A)(implicit ev: Appendable[A]) = ev.append(a,b)

    def appendContextBound[A: Appendable](a: A, b: A) = implicitly(A).append(a,b)

    def apply[A](implicit appendable: Appendable[A]): Appendable[A] = appendable
}

```
