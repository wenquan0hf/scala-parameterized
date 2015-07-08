# 类型下界 #
我们还是使用前面的 Queue 的例子，我们知道，我们不能使用+T（协变关系）来定义 Queue，然而，我们可以通过给 equeue 方法本身提供一个类型参数使之一般化。

```
class Queue[+T] ( private val leading: List[T],
	private val trailing: List[T] {
	def enqueue[U >: T ](x: U) =
		new Queue[U](leading, x:: trailing)
}
```

在这个新定义中，使用了一个新的类型参数 U， 语法结构 U >: T ，定义 T 为 U 的下界，因此U必须是 T 的一个父类。 enqueue 的返回类型也变成 Queue[U]，而不是之前的 enqueue[T]。

举例来说，比如说一个 Fruit 类定义了两个子类 Apple 和 Orange， 使用这个新的 enqueue 定义，可以把一个 Orange 对象添加到一个 Queue[Apple]队列中，其返回结果为一 个Queue[Fruit]类型。
