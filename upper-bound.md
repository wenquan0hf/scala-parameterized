# 类型上界 #
本篇介绍类型上界，我们使用合并排序算法来给人名排序，这里先定义一个 Person 类，它派生于 Ordered Trait，定义如下：

```
class Person(val firstName:String, val lastName:String) 
	extends Ordered[Person]{
	def compare(that:Person) ={
		val lastNameComparison=
			lastName.compareToIngnoreCase(that.lastName)
		if(lastNameComparison!=0)
			lastNameComparison
		else
			firstName.compareToIngnoreCase(that.firstName)
	}
	override def toString= firstName + " " + lastName
}
```

我们先测试一下这个类对象之间的比较关系，注意 Ordered Trait 定义了对象之间的<，>，>=，<=关系。

```
scala> val robert=new Person("Robert","Jones")
robert: Person = Robert Jones
```

```
scala> val sally = new Person("Sally","Smith")
sally: Person = Sally Smith
```

```
scala> robert < sally
res1: Boolean = true
```

```
scala> james == james1
res2: Boolean = false
```

我们定义 merge sort 算法如下：

```
def orderedMergeSort[T <: Ordered[T]] (xs: List[T]):List[T] ={
    def merge(xs:List[T],ys:List[T]):List[T] =
      (xs ,ys ) match {
        case (Nil, _) => ys
        case (_,Nil) => xs
        case (x:: xs1,y :: ys1 ) =>
          if (x < y) x:: merge(xs1,ys)
          else y :: merge( xs,ys1)
      }
    val n = xs.length /2
    if(n==0) xs
    else {
      val (ys, zs)= xs splitAt n
      merge(orderedMergeSort(ys),orderedMergeSort(zs))
    }
  }
```

这个函数要求输入的参数的类型需要派生于 Ordered trait，此时你需要使用类型上界，类型上界使用 <:，如本例中的 T <: Ordered[T] ，它代表类型 T 的上界是 Ordered[T]，也就是说传入的参数类型必须是类型 Ordered[T]的子类。

我们之前定义的 List[Person] 满足这个条件。 比如：

```
scala> val people = List (
     |     new Person("Larry","Wall"),
     |     new Person("Anders","Hejlsberg"),
     |     new Person("Guido","van Rossum"),
     |     new Person("Alan","Kay"),
     |     new Person("Yukihiro","Matsumoto")
     | 
     |   )
people: List[Person] = List(Larry Wall, Anders Hejlsberg, Guido van Rossum, Alan Kay, Yukihiro Matsumoto)
```

```
scala> val sortedPeople=orderedMergeSort(people)
sortedPeople: List[Person] = List(Anders Hejlsberg, Alan Kay, Yukihiro Matsumoto, Guido van Rossum, Larry Wall
```