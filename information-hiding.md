# 信息隐藏 

上篇的 Queue 的实现在效率上来说还是比较高效的，但是却给类的使用者暴露一些不必要的实现细节，比如 Queue 的构造函数使用了两个 List 对象，其中一个还是个倒序的列表。本篇介绍如何隐藏这些不必要的信息。

在 Java 中，你可以使用私有的构造函数来隐藏构造函数，在 Scala 中的主构造器缺省包含在类定义中，但你还是可以使用private来修改其访问属性。  

例如：

```
class Queue[T] private(
  private val leading:List[T],
  private val trailing:List[T]
 )
```

在类名和参数之间的 private 修饰符表明该构造器是私有的，只能在类或其伙伴对象之中使用。而类本身还是可以公开访问的：

```
scala> new Queue(List(1,2),List(3))
<console>:9: error: constructor Queue in class Queue cannot be accessed in object $iw
              new Queue(List(1,2),List(3))
              ^
```

由于主构造器变成私有的，因此我们需要另外的方式来构造 Queue 的实例，一种方法是使用辅助构造器，比如如下的辅助构造函数：

```
def this() =this(Nil,Nil)
```

```
def this(elem: T*) = this (elems.toList,Nil)
```

注意，参数类 T* 代表的不定长参数类型。

另外一种方法是使用 Factory 方法来构造一个队列。 一个比较简洁的方法是使用伙伴对象 ，并定义 apply 方法，比如：

```
object Queue {
   def apply[T](xs: T*) = new Queue[T](xs.toList,Nil)
}
```

我们使用 apply 定义了构造 Queue 的 factory 方法，因此调用时可以使用 Queue(1,2,3)的形式来构造一个实例。 Queue(1,2,3)实际为 Queue.apply(1,2,3)调用，对调用者来说看起来好像定义了一个可以全局访问的 factory 构造方法。而其实对于 Scala 来说，所有的方法都必须包含着某个类或对象中。

除了使用私有方法来隐含实现细节外，还有一种方法可以实现信息的隐藏。我们可以使用 trait 定义类的接口，而把实现细节全部隐藏起来（使用私有类），代码实现如下：

```
trait Queue[T]{
  def head: T
  def tail: Queue[T]
  def enqueue(x:T): Queue[T]
}
object Queue {
  def apply[T](xs: T*):Queue[T] = new QueueImpl[T](xs.toList, Nil)
  private class QueueImpl[T](
        private val leading: List[T],
        private val trailing: List[T]
      ) extends Queue[T]{
    private def mirror =
      if (leading.isEmpty)
        new QueueImpl(trailing.reverse, Nil)
      else
        this
    def head = mirror.leading.head
    def tail = {
      val q = mirror
      new QueueImpl(q.leading.tail, q.trailing)
    }
    def enqueue(x: T) =
      new QueueImpl(leading, x :: trailing)
  }
}
```

这个实现定义一个 Public 的 Trait 方法，而隐藏了所有的实现细节。
