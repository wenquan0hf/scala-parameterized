# 概述和例子说明 #
本专题介绍 Scala 的参数化类型，在介绍的过程中同时演示了信息隐藏的技术，这里我们通过实现一个纯函数化实现的队列来举例说明。

参数化类型帮助你实现通用的类型和 Trait。比如通用的集合类 Set，该通用集合类可以通过制定类型参数 T，Set[T]，它通过实例化参数类型，可以定义 Set[String]，Set[Int]等等。

此外，一般来说  String 是 AnyRef，但在其它一些语言中，Set[String]并一定是 Set[AnyRef]的子类。在 Scala 中可以通过制定参数化类型之间的继承属性的 Variance 特性，来说明该参数化类型中使用不同的参数类型后的类型之间是否也存在继承关系。

首先我们定义一些我们要实现的这个简单的对象的一些基本需求：  
这个函数化队列要求支持下面三个基本操作：  
head 返回队列的首元素    
tail 返回队列的除首元素之外的其余元素（也是一个队列）  
enqueue 把新元素添加到队列尾部后返回一个新队列。  

和一般队列实现不同的是，函数化队列实现时不改变队列的内容，当需要修改队列时构造一个新队列。

如何构造一个效率高的队列呢？也就是 head，tail，enqueue所花费的时间不应当随队列的大小而改变，一个简单的实现可以使用 List 类来实现，但 enqueue 操作的时间和队列的长度成正比，这里给出一个使用两个 List 对象的队列实现，具体算法不解释了（本专题侧重点不在算法本身）可以实现一个高效的队列操作。

```
class Queue[T](
  private val leading:List[T],
  private val trailing:List[T]
 ){
  private def mirror =
    if(leading.isEmpty)
      new Queue(trailing.reverse,Nil)
    else
       this
  def head=mirror.leading.head
  def tail={
    val q= mirror
      new Queue(q.leading.tail,q.trailing)
  }
  def enqueue(x:T)=
    new Queue(leading,x::trailing)
}
```

使用这个实现，进行一些基本的队列操作：

```
scala> val q = new Queue(List(1,2,3),Nil)
q: Queue[Int] = Queue@24cc40b6
```

```
scala> val q1 = q enqueue 4
q1: Queue[Int] = Queue@67825189
```

```
scala> q
res0: Queue[Int] = Queue@24cc40b6
```

```
scala> val p = q1 head
p: Int = 1
```

```
scala> q1
res1: Queue[Int] = Queue@67825189
```

这个实现满足功能需求，但我们希望可以实现如下的形式：

```
scala> val q = Queue(1,2,3)
q: Queue[Int] = Queue(1,2,3)
```

```
scala> val q1 = q enqueue 4
q1: Queue[Int] = Queue(1,2,3,4)
```

```
scala> q
q: Queue[Int] = Queue(1,2,3)
```

在之后的文章我们逐步的优化这个实现。
