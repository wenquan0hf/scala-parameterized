# Variance(类型变体)标识 #
在上篇例子中定义的 Queue 是一个 Trait，而不是一个类型，这是因为 Queue 需要一个类型参数才能构成一个类型， 也就是说你不可以直接创建一个类型为 Queue 的对象：

```
scala> def doesNotCompile(q:Queue) {}
<console>:8: error: trait Queue takes type parameters
       def doesNotCompile(q:Queue) {}
                            ^
```

实际上，Queue 允许你指明类型参数，比如 Queue[String]，Queue[Int]或 Queue[AnyRef]等，因此 Queue 也可以称为一个类型构造器，因为你可以指明一个参数类型后构造一个新的类型。

Queue 也可以称为一个通用 Trait（包含类型参数的类或 Trait称为“generic”) ，而指明了类型参数之后就不再是通用类型，而是特定的类型了，比如Queue[String]等。

类型参 数和派生结合起来之后，就会产生一些有趣的问题，比如 String 是 AnyRef 的子类，那么 Queue[String]和 Queue[AnyRef]之间会不会有什么继承关系呢？

在 Scala 中，可以有三种不同的关系，缺省情况比如 Queue[T]，Queue[String]和 Queue[AnyRef]不存在继承关系。此外 Scala 还支持两种关系： Covariance（协变关系）和 Contravariance(逆变关系），分别以
 Queue[+T]和 Queue[-T] 代表。

Covariance（协变关系）是在类型前面使用“+”号，表示如果两个类型 T，S 如果 T 是 S 的子类，那么 Queue[T]也是 Queue[S]的子类

而 Contravariable（逆变关系）则相反，如果如果两个类型 T，S 如果 T 是 S 的子类，反过来，Queue[S]是 Queue[T]的子类。

我们以一个具体的例子来说明一下，比较直观，定义三个类 

```
GrandParent ,Parent, Child ：
class GrandParent 
class Parent extends GrandParent
class Child extends Parent
```

```
class Box[+A]
class Box2[-A]
```

```
def foo(x : Box[Parent]) : Box[Parent] = identity(x)
def bar(x : Box2[Parent]) : Box2[Parent] = identity(x)
```

那么我使用下面的几种调用方法来看看+A 和-A 的不同之处：

```
scala> foo(new Box[Child]) // success
res1: Box[Parent] = Box@5da444e4
```

```
scala> foo(new Box[GrandParent]) // type error
<console>:12: error: type mismatch;
 found   : Box[GrandParent]
 required: Box[Parent]
              foo(new Box[GrandParent]) // type error
                  ^
```

```
scala> bar(new Box2[Child]) // type error
<console>:13: error: type mismatch;
 found   : Box2[Child]
 required: Box2[Parent]
              bar(new Box2[Child]) // type error
                  ^
```

```
scala> bar(new Box2[GrandParent]) // success
res4: Box2[Parent] = Box2@59615389
```

[T]，[+T]，[-T]为类型参数的三种不同的变体。使用+，-称为类型的变体标识。

在纯函数编程的世界中，很多类型存在非常明显的协变关系，但是一但出现可变的数据时，情况就发生了变化，比如我们看看下面的例子：

```
class Cell[T](init:T) {
	private[this] var current = init
	def get = current
	def set(x:T) { current = x}
}
```

如果我们假定我们定义的是 Cell[+T]而不是上面例子中的 Cell[T]（实际上编译器会在使用 Cell[+T]时报错，我们啦看看为什么？假定我们使用的是 Cell[+T]来定义，那么我们可以写如下代码：

```
val c1 = new Cell[String]("abc")
val c2: Cell[Any] = c1
c2.set(1)
val s:String = c1.get
```

这四行代码看起来都没有错（假定我们使用的是 Cell[+T]，那么 Cell[String] 是 Cell[AnyRef]的子类，因此 c2 可以使用 c1 赋值）。最后的结果我们是把整数 1 赋值给了这字符串类型，这就造成了类型不匹配，这也是为什么编译器在使用 Cell[+T]会报错：

```
<console>:10: error: covariant type T occurs in contravariant position in type T of value x
       def set(x:T) { current = x}
```

要注意的是在 Scala 中，Array[T]不是协变关系的，因此 Array[String]不是 Array[Any]的子类。因此不可以直接把 Array[String]类型的变量赋值给 Array[Any]类型的变量，例如：

```
scala> val a1=Array("abc")
a1: Array[String] = Array(abc)
```

```
scala> val a2:Array[Any] = a1
<console>:8: error: type mismatch;
 found   : Array[String]
 required: Array[Any]
Note: String <: Any, but class Array is invariant in type T.
You may wish to investigate a wildcard type such as `_ <: Any`. (SLS 3.2.10)
       val a2:Array[Any] = a1
```

但是有时需要这种赋值，Scala 允许你把特殊类型的数组强制转换成其父类型的数组，例如：

```
scala> val a2 :Array[Object] = a1.asInstanceOf[Array[Object]]
a2: Array[Object] = Array(abc)
```
