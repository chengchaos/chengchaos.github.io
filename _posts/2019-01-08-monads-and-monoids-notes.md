---
title: Monads 和 Monoids 学习笔记
key: 20190108
tags: scala, fp, monads, monoids
---

找到一个非常好的 Scala 的学习博客：[写点什么](http://hongjiang.info/)。

以下文字转载了博主的[我所理解的monad](http://hongjiang.info/understand-monad-0/)系列。这是我找到的最容易理解的 Monads 的文章，感谢博主。


<!--more-->

## 〇，我所理解的monad

> [http://hongjiang.info/understand-monad-0/](http://hongjiang.info/understand-monad-0/)

以前在别人的一篇 blog 里看到过有这样一句话，大意是：关于 monad，几乎每个在学习函数式编程中接触到这个模式的，都会写一篇博客描述他的理解。而且不同的人对 monad 的理解有所不同(暗讽 monad 的复杂)。

我也不例外，大约是 3 年前刚接触 scala 时，无意搜到了 james 的《[monads are elephants](http://james-iry.blogspot.jp/2007/09/monads-are-elephants-part-1.html)》，一下子把我给难住了。scala 在刚开始学习时，如果只把它当作 java 的替代者来用，继续用 OO 的范式的话，并不难。但在继续深入它的类型和函数式特征之后，发现有一篇广袤的世界是之前不曾看到的，可以以一种新的视角来思考编程的本质。而 scala 在 OO 与”新世界”之间架起一座桥(Bridge to Terabithia)，通过这座桥的一个代价是，抛弃原先认为编程理所当然应该这样的观念。

在新的世界同样有设计模式，不过它们抽象的角度不同。理解起来有些困难，最大的原因是缺乏对这个新世界的认识，具体的说 scala 里最显著困难是它的类型系统，当类型的抽象度变高之后(犹如从二维世界到三维世界)，它的含义和使用场景也会复杂很多。而 monad 正是基于高阶类型做的抽象。所以在了解 monad 前一定先要了解 scala 中高阶类型的概念，这块可以参考我 blog 中的 scala 类型系统系列。

通常 monad 也是一个标识，判断程序员对函数式世界的了解程度，它背后隐藏着很多的概念，并且很多来自数论(范畴领域)，对于非科班出身的我来说，总是心存敬畏而避而远之，但我们如果去掉这些理论描述，仅从程序的实现来看，还是比较容易的，只要把这些概念一点点分解开的话。

在 james 的这篇 [A Brief, Incomplete, and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.co.at/2009/05/brief-incomplete-and-mostly-wrong.html)，(译文参考这里:[程序语言简史(伪)](https://www.soimort.org/posts/160/) )，有段关于monad的描述很有意思：

> Haskell 由于使用了 Monad 这种较费解的概念来控制副作用而遭到了一些批评意见。Wadler试图平息这些质疑，他解释说：“一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已，这有什么难以理解的？”

我们就从这句话说起，先理解是“幺半群”，别被这个字面术语吓到，其实用程序表达起来很简单。



## 一，半群(semigroup)与幺半群(monoid)

> [http://hongjiang.info/semigroup-and-monoid/](http://hongjiang.info/semigroup-and-monoid/)

google 到数学里定义的群(group): `G` 为非空集合，如果在G上定义的二元运算 `*`，满足

```
（1）封闭性（Closure）：对于任意 a，b∈G，有 a*b∈G
（2）结合律（Associativity）：对于任意 a，b，c∈G，有（a*b）*c=a*（b*c）
（3）幺元 （Identity）：存在幺元 e，使得对于任意 a∈G，e*a=a*e=a
（4）逆元：对于任意 a∈G，存在逆元 a^-1，使得 a^-1*a=a*a^-1=e
```

则称`（G，*）`是群，简称 G 是群。

如果仅满足封闭性和结合律，则称G是一个半群（`Semigroup`）；如果仅满足封闭性、结合律并且有幺元，则称G是一个含幺半群（`Monoid`）。

相比公式还是用代码表达更容易理解，下面表示一个半群(semigroup):

```scala
trait SemiGroup[T] {
    def append(a: T, b: T): T
}
```

特质 SemiGroup，定义了一个二元操作的方法 `append`，可以对半群内的任意 2 个元素结合，且返回值仍属于该半群。

我们看具体的实现，一个Int类型的半群实例：

```scala
object IntSemiGroup extends SemiGroup[Int] {
    def append(a: Int, b: Int) = a + b
}


// 对2个元素结合
val r = IntSemiGroup.append(1, 2)

```


现在在半群的基础上，再增加一个幺元(Identity，也翻译为单位元)，吐槽一下，幺元这个中文不知道最早谁起的，Identity 能表达的意义(同一、恒等)翻译到中文后完全消失了。

```scala
trait Monoid[T] extends SemiGroup[T] {
    // 定义单位元
    def zero: T
}
```


上面定义了一个幺半群，继承自半群，增加了一个单位元方法，为了容易理解，我们用 zero 表示，半群里的任何元素 a 与 zero 结合，结果仍是 a 本身。

构造一个Int类型的幺半群实例：

```scala
object IntMonoid extends Monoid[Int] {
    // 二元操作
    def append(a: Int, b: Int) = a + b
    // 单位元
    def zero = 0
}
```


构造一个 `String` 类型的幺半群实例：

```scala
object StringMonoid extends Monoid[String] {
    def append(a: String, b: String) = a + b
    def zero = ""
}
```

再构造一个复杂点的 `List[T]` 的幺半群工厂方法：

```scala
def listMonoid[T] = {
    new Monoid[List[T]] { 
        def zero = Nil
        def append(a: List[T], b: List[T]) = a ++ b 
    }
}
```

OK，现在我们已经了解了幺半群是什么样了，但它有什么用？


## 二，fold 与 monoid

> [http://hongjiang.info/fold-and-monoid/](http://hongjiang.info/fold-and-monoid/)


在 Scala 中的核心数据结构 `List` 中，定义了 `fold` 操作，实际上这些方法是定义在 scala 集合库的最顶层特质 `GenTraversableOnce` 中的：

List中的左折叠(借用wiki上的图)：

![""](http://hongjiang.info/wp-content/uploads/2013/12/foldLeft.jpg)

```scala
def foldLeft[B](z: B)(op: (B, A) => B): B
```

从图中可以看到，左折叠是用一个初始元素 z 从 `List` 的左边第一个元素开始操作，一直到对所有的元素都操作完。

现在我们对一个List进行累加操作：

```scala
scala> List("A","B","C").foldLeft("")(_+_)
res5: String = ABC
```


上面 `foldLeft` 传入的两个参数空字符串，以及二元操作函数 `_+_`  不正好符合字符串 `monoid` 的定义吗？

```scala
object StringMonoid extends Monoid[String] {
    def append(a: String, b: String) = a + b
    def zero = ""
}
```


用 `StringMonoid` 来代入：

```scala
scala> List("A","B","C").foldLeft(StringMonoid.zero)(StringMonoid.append)
res7: String = ABC
```


现在我们对 List 定义一个累加其元素的方法：

```scala
scala> def acc[T](list: List[T], m: Monoid[T]) = {
    list.foldLeft(m.zero)(m.append)
}
```

再进一步，把第二个参数改为隐式参数

```scala
scala> def acc[T](list: List[T])(implicit m: Monoid[T]) = { 
    list.foldLeft(m.zero)(m.append) 
}
```

现在 Monoid 成了一个 type class，我们还可以再简化写法，用上下文绑定：

```scala
scala> def acc[T: Monoid](list: List[T]) = {
    val m = implicitly[Monoid[T]]
    list.foldLeft(m.zero)(m.append) 
}
```

如果我们在上下文提供了对应隐式值，就等于对List有了这种累加的能力：

```scala
scala> implicit val intMonoid = new Monoid[Int] { 
            def append(a: Int, b: Int) = a + b
            def zero = 0 
        }


scala> implicit val strMonoid = new Monoid[String] { 
            def append(a: String, b: String) = a + b
            def zero = ""
        }

scala> acc(List(1,2,3))
res10: Int = 6

scala> acc(List("A","B","C"))
res11: String = ABC
```

现在我们把 Monoid 看成基于二元操作(且提供单位元)的计算能力的抽象，不过仅仅是 `fold` 操作的话，还看不出它有什么威力。Monoid/SemiGroup 中的结合律(associativity)特性才是它的威力所在，这个特性使得并行运算变得容易。


## 三，半群(semigroup)与并行运算

> [http://hongjiang.info/semigroup-and-parallel/](http://hongjiang.info/semigroup-and-parallel/)

因为半群里的“结合律”特性，使得我们可以对一些任务拆分采用并行处理，只要这些任务的结果类型符合“结合律”(即没有先后依赖)。让我们看一个单词统计的例子，阿里中间件团队前几个月有过一次编程比赛就是统计单词频度，参考：[Coding4Fun第三期活动总结](http://jm-blog.aliapp.com/?p=3093)

编程比赛主要是考察技巧，现在我们看看现实世界中用 scala 怎么解决这个问题，如果数据是巨大的，无疑要采用 `map-reduce` 的思路，如果我们不依赖 hadoop 之类的系统来解决，最简单的方式就是 `actor` 处理，最后再汇总结果。这篇 blog 不讨论数据的统计处理，看看最后如何把这些结果合并


```scala
type Result = Map[String,Int]

object combiner extends SemiGroup[Result] {

    def append(x: Result, y: Result): Result = {
        val x0 = x.withDefaultValue(0)
        val y0 = y.withDefaultValue(0)
        val keys = x.keys.toSet.union(y.keys.toSet)
        keys.map{ k => (k -> (x0(k) + y0(k))) }.toMap
    } // 不考虑效率
}
```

现在假设不同的 `actor` 分别返回 `r1`, `r2`

```scala
val r1 = Map("hello" -> 1, "world" -> 2)
val r2 = Map("hello" -> 2, "ok" -> 5)

val counts = combiner.append(r1, r2)
```

如果有其他 `actor` 再返回结果的话，只要继续合并下去：

```scala
combiner.append(r3, counts)
```

twitter 开源的 algebird，就是基于 semigroup/monoid 的。不过我们不必沿着这条路深入下去，monad 并不是为了并发而发明的，只是它正好是一个半群(幺半群)，半群的结合律符合并行运算(由半群演化出来的迹幺半群和历史幺半群是进程演算和并行计算的基础)。这是另一个方向，不继续涉及下去，我们后续回到 monad 的其他特征上。


## 四，函子(functor)是什么

> [http://hongjiang.info/understand-monad-4-what-is-functor/](http://hongjiang.info/understand-monad-4-what-is-functor/)


大致介绍了幺半群(monoid)后，我们重新回顾最初引用 Wadler(haskell 委员会成员，把 monad 引入 haskell 的家伙)的那句话：

> 一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已 

现在我们来解读这句话中包含的另一个概念：自函子(Endofunctor)，不过我们先需要一些铺垫：


### 首先，什么是函子(Functor)？

乍一看名字，以为函子(functor)对函数(function)是一种封装，实际没有关系，尽管他们都是表示映射，但两者针对的目标不一样。

函数表达的映射关系在类型上体现在特定类型(proper type)之间的映射，举例来说：

```scala
// Int => String
scala> def foo(i:Int): String = i.toString

// List[Int] => List[String]
scala> def bar(l:List[Int]): List[String] = l.map(_.toString)

// List[T] => Set[T]
scala> def baz[T](l:List[T]): Set[T] = l.toSet
```


而函子，则是体现在高阶类型(确切的说是范畴，可把范畴简单的看成高阶类型)之间的映射(关于高阶类型参考: [scala类型系统：24) 理解 higher-kinded-type](http://hongjiang.info/scala-higher-kinded-type/))，听上去还是不够直观，函子这个术语是来自群论(范畴论)里的概念，表示的是范畴之间的映射，那范畴又与类型之间是什么关系？


### 把范畴看做一组类型的集合

假设这里有两个范畴：范畴 C1 里面有类型 `String` 和类型 `Int`；范畴 C2 里面有 `List[String]` 和 `List[Int]`

![""](http://hongjiang.info/wp-content/uploads/2013/12/functor-1.jpg)


### 函子表示范畴之间的映射

从上图例子来看，这两个范畴之间有映射关系，即在 C1 里的 `Int` 对应在 C2 里的 `List[Int]`，C1 里的 `String` 对应 C2 里的 `List[String]`，在 C1 里存在 `Int->String` 的关系态射(术语是 `morphism`，我们可理解为函数)，在 C2 里也存在 `List[Int]->List[String]` 的关系态射。

换句话说，如果一个范畴内部的所有元素可以映射为另一个范畴的元素，且元素间的关系也可以映射为另一个范畴元素间关系，则认为这两个范畴之间存在映射。所谓函子就是表示两个范畴的映射。


### 怎么用代码来描述函子？

从上图的例子，我们已经清楚了 `functor` 的含义，即它包含两个层面的映射：

```
1) 将 C1 中的类型 T 映射为 C2 中的 List[T] :  T => List[T]
2) 将 C1 中的函数 f 映射为 C2 中的 函数 fm :  (A => B) => (List[A] => List[B])
```

要满足这两点，我们需要一个类型构造器

```scala
trait Functor[F[_]] {

    def typeMap[A]: F[A]

    def funcMap[A,B](f: A=>B): F[A]=>F[B] 
}
```


我们现在可以把这个定义再简化一些，类型的映射方法可以不用，并把它作为一个 `type class`：

```scala
trait Functor[F[_]] {
    def map[A,B](fa: F[A], f: A=>B): F[B]
}
```


现在我们自定义一个 `My[_]` 的类型构造器，测试一下这个 `type class`:

```scala
scala> case class My[T](e:T)

scala> def testMap[A,B, M <: My[A]](m:M, f: A=>B)(implicit functor:Functor[My]) = {
 |          functor.map(m,f)
 |      }

scala> implicit object MyFunctor extends Functor[My] {
 |          def map[A,B](fa: My[A], f:A=>B) = My(f(fa.e))
 |      }


// 对 My[Int], 应用函数 Int=>String 得到 My[String]
scala> testMap(My(200), (x:Int)=>x+"ok")
res9: My[String] = My(200ok)

```


不过大多数库中对 `functor` 的支持，都不是通过 `type class` 模式来做的，而是直接在类型构造器的定义中实现了 `map` 方法：

```scala
scala> case class My[A](e:A) {
     |     def map[B](f: A=>B): My[B] = My(f(e))
     | }

scala> My(200).map(_.toString)
res10: My[String] = My(200)
```


这样相当于显式的让 `My` 同时具备了对类型和函数的映射 `(A->My[A]，A=>B -> My[A]=>My[B]`；在 haskell 里把这两个行为也叫提升(lift)，相当于把类型和函数放到容器里)，所以我们也可以说一个带有 `map` 方法的类型构造器，就是一个函子。


### 范畴与高阶类型

我们再来思考一下，如果忽略范畴中的关系(函数)，范畴其实就是对特定类型的抽象，即高阶类型(first-order-type 或 higher-kinded-type，也就是类型构造器)，那么对于上面例子中的”范畴 C2″，它的所有类型都是 `List[T]` 的特定类型，这个范畴就可以抽象为 `List` 高阶类型。那对于”范畴 C1″呢？它又怎么抽象？其实，”范畴 C1″的抽象类型可以看做是一个 `Identity` 类型构造器，它与任何参数类型作用构造出的类型就是参数类型：

```scala
scala> type Id[T] = T
```

是不是很像单位元的概念？在 shapeless 里已经提起过

这么看的话，如果范畴也可以用类型(高阶)来表达，那岂不是只用普通函数就可以描述它们之间的映射了？别急，先试试，方法里是不支持类型构造器做参数的：

```scala
scala> def foo(cat: Id) = print(cat)
    <console>:18: error: type Id takes type parameters
```


方法中只能使用特定类型(proper type)做参数。


## 五，自函子(Endofunctor)是什么

> [http://hongjiang.info/understand-monad-5-what-is-endofunctor/](http://hongjiang.info/understand-monad-5-what-is-endofunctor/)


经过前面一篇对函子(Functor)的铺垫，我们现在可以看看什么是自函子(Endofunctor)了，从范畴的定义看很简单：

> 自函子就是一个将范畴映射到自身的函子 (A functor that maps a category to itself)

这句话看起来简单，但有个问题，如何区分自函子与 `Identity` 函子？让我们先从简单的“自函数”来看。

### 自函数(Endofunction)

自函数是把一个类型映射到自身类型，比如 `Int=>Int`, `String=>String` 等

注意自函数与 `Identity` 函数的差异，`Identity` 函数是什么也不做，传入什么参数返回什么参数，它属于自函数的一种特例；自函数是入参和出参的类型一致，比如 `(x:Int) => x * 2` 或 `(x:Int) => x * 3` 都属于自函数：

!["http://hongjiang.info/wp-content/uploads/2013/12/endo-function.jpg"](http://hongjiang.info/wp-content/uploads/2013/12/endo-function.jpg)


### 自函子(Endofunctor)

自函子映射的结果是自身，下图是一个简单的情况:

!["http://hongjiang.info/wp-content/uploads/2013/12/endo-functor-22.jpg"](http://hongjiang.info/wp-content/uploads/2013/12/endo-functor-22.jpg)


假设这个自函子为 `F`，则对于 `F[Int]` 作用的结果仍是 `Int`，对于函数 `f: Int=>String` 映射的结果 `F[f]` 也仍是函数 `f`，所以这个自函子实际是一个 `Identity` 函子(自函子的一种特例)，即对范畴中的元素和关系不做任何改变。

那怎么描述出一个非 `Identity` 的自函子呢？在介绍范畴在程序上的用法的资料里通常都用 haskell 来举例，把 haskell 里的所有类型和函数都放到一个范畴里，取名叫 `Hask`，那么对于这个 `Hask` 的范畴，它看上去像是这样的：

!["http://hongjiang.info/wp-content/uploads/2013/12/hask-cat.jpg"](http://hongjiang.info/wp-content/uploads/2013/12/hask-cat.jpg)

先来解释一下（画这个图的时候做了简化），`A` , `B` 代表普通类型如 `String`, `Int`, `Boolean` 等，这些(有限的)普通类型是一组类型集合，还有一组类型集合是衍生类型(即由类型构造器与类型参数组成的)，这是一个无限集合(可以无限衍生下去)。这样范畴 `Hask`就涵盖了 haskell 中所有的类型。

对于范畴 `Hask` 来说，如果有一个函子 `F`，对里面的元素映射后，其结果仍属于 `Hask`，比如我们用 `List` 这个函子：

```haskell
List[A], List[List[A]], List[List[List[A]]]...
```


发现这些映射的结果也是属于 `Hask` 范畴(子集)，所以这是一个自函子，实际上在 `Hask` 范畴上的所有函子都是自函子。

我们仔细观察这个 `Hask` 范畴的结构，发现它实际是一个 `fractal` 结构，所谓 `fractal` (分形)，是个很神奇的结构，在自然界也大量存在：

!["http://hongjiang.info/wp-content/uploads/2013/12/fern.jpg"](http://hongjiang.info/wp-content/uploads/2013/12/fern.jpg)


如上面的这片叶子，它的每一簇分支，形状上与整体的形状是完全一样的，即局部与整体是一致的结构(并且局部可以再分解下去)

这种结构在函数式语言里也是很常用的，最典型的如 `List` 结构，由 `head` 和 `tail` 两部分组合而成，而每个 `tail` 也是一个 `List` 结构，可以递归的分解下去。



## 六，从组合子(combinator)说起

> [http://hongjiang.info/understand-monad-6-combinator/](http://hongjiang.info/understand-monad-6-combinator/)



从同事的反馈了解到前边几篇 monad 的介绍说的有些抽象。现在还没有提出 monad，如果把 monoid 与 endofunctor 结合起来，恐怕更是抽象。还是回到用大家更能体会的场景来描述，等对 monad 有了基本的印象之后，最后再试图把 monad 在现实世界的表现与范畴论里的形式结合起来。

我想到一个比较好的方式是从行为的组合来说 monad，因为这个层面的例子容易接受，最近在用的 spray-routing 框架，它的核心 `Directive` (指令)就是一个行为组合子 monad

最初知道组合子(combinator)这个单词是从《PIS》书中专门有一章讲“连接符解析”(combinator parsing)，当时没有理解 combinator 的概念，那章也是偏重说 parser 的；后来在很多资料里看到把 monad 也当成 combinator 对待，一直没有悟清楚各种 combinator 是不是一回事，直到看到了 ajoo 的系列文章(推荐一下)：

```
论面向组合子程序设计方法: 
http://www.blogjava.net/ajoo/category/6968.html

论面向组合子程序设计方法 之一 创世纪
论面向组合子程序设计方法 之二 失乐园
论面向组合子程序设计方法 之三 失乐园之补充
论面向组合子程序设计方法 之四 燃烧的荆棘
论面向组合子程序设计方法 之五 新约
论面向组合子程序设计方法 之六 oracle
论面向组合子程序设计方法 之七 重构
论面向组合子程序设计方法 之八 monad
论面向组合子程序设计方法 之九 南无阿弥陀佛
论面向组合子程序设计方法 之十 还是重构
论面向组合子程序设计方法 之十一 微步毂纹生
```


ajoo 是个牛人，早期在 javaeye 有看到过他实现了一套基于 jvm 的 haskell，取名“jaskell”。那也是好些年前了，在 javaeye 围观他与 Trustno1 等人对函数式编程的讨论，云里雾里。现在感觉稍微能跟这些大牛们有一些对话，却找不到这样的圈子了。引用 ajoo 的话:

>
> 组合子，英文叫 combinator，是函数式编程里面的重要思想。如果说 OO 是归纳法（分析归纳需求，然后根据需求分解问题，解决问题），那么 “面向组合子”就是“演绎法”。通过定义最基本的原子操作，定义基本的组合规则，然后把这些原子以各种方法组合起来。
>


引用另外一位函数式领域的大师的说法：

>
> A combinator is a function which builds program fragments from program fragments
>


这句话有一些“自相似性”，可以把 combinator 先当成一种“胶水”，更具体一些，把 scala 的集合库里提供的一些函数看成 combinator:

```
map
foreach
filter
fold/reduce
zip/partition
flatten/flatMap
```

而 monad 正是一个通用的 combinator，也是一种设计模式，但它有很多面，就我后续的举例来说，先把 monad 看成一个“行为”的组合子。体现在代码上：

```scala
class M[A](value: A) {  

    // 1) 把普通类型 B 构造为 M[B]
    //  因为 M 定义为 class 并提供了构造方法，可以通过 new 关键字来构造，该工厂方法可以省略
    def unit[B] (value : B) = new M(value)  

    // 2) 不是必须的
    def map[B](f: A => B) : M[B] = flatMap {x => unit(f(x))}  

    // 3) 必须，核心方法
    def flatMap[B](f: A => M[B]) : M[B] = ...  
} 
```

一个 monad 内部除了一个工厂方法 unit(注意，与 `Unit` 类型没有关系，unit 这里表示“装箱”，源自 haskell 里的叫法)，还包含了两个 combinator 方法，`map `和 `flatMap`，这两个组合子中 `flatMap` 是不可少的，`map` 可以通过 `flatMap` 来实现。

unit 方法不一定会定义在 monad 类内部，很多情况下会在伴生对象中通过工厂方法来实现。

这里忽略了 monad 里的[法则](http://hongjiang.info/monads-are-elephants-part3-chinese/)，重点先理解 M 封装了什么？以及内部的 `map` 与 `flatMap` 的含义是什么。


## 七，把monad看做行为的组合子

> [http://hongjiang.info/understand-monad-7-action-combinator/](http://hongjiang.info/understand-monad-7-action-combinator/)

先从最简单的情况开始，我们用 monad 封装一段执行逻辑，然后提供一个 map 组合子，可以组合后续行为：

```scala
// 一个不完善的 monad
class M[A](inner: => A) {

    // 执行逻辑
    def apply() = inner

    // 组合当前行为与后续行为，返回一个新的 monad
    def map[B](next: A=>B): M[B] = new M(next(inner))
}
```


用上面的例子模拟几个行为的组合：

```scala
scala> val first = new M({println("first"); "111"})
first: M[String] = M@745b171c

scala> val second = first.map(x => {println("second"); x.toInt})
second: M[Int] = M@155f28dc

scala> val third = second.map(x => {println("third"); x + 200})
third: M[Int] = M@b345419

scala> third()
first
second
third
res0: Int = 311
```


看上去对行为的组合也不过如此，其实在 `Function1` 这个类里已经提供了对只有一个参数的函数的组合：

```scala
scala> class A; class B; class C

scala> val f1: A=>B = (a:A) => new B

scala> val f2: B=>C = (b:B) => new C

scala> val f3 = f1 andThen f2
f3: A => C = <function1>
```

`Function1` 里的 `andThen` 方法可以把前面函数的结果交给后边的函数处理，`A=>B` 与 `B=>C` 组成函数 `A=>C`

这样看上去我们当前实现的 monad 和 `Function1` 有些相似，封装了一段行为，提供方法将下一个行为组合成新的行为(可以不断的组合下去)，在 java 里我们可以对比 Command 模式和 Composite 模式。


### 真实世界的”行为”(Action),普通函数的问题

#### 1) 结果的不确定性，行为链的校验问题

这个不完善的 monad 已经可以做一些事情，但有个很显著的问题，只能组合”普通函数(plain)”，所谓普通函数是指结果类型就是我们想要的数据类型。

比如 `f: A=>B` 这个函数可以表示一个行为，这个行为最终得到的数据类型是B，但如果这个行为遇到异常，或者返回为 `null`，这时我们的组合过程会如何？

这意味着我们必须在组合过程中判断每个函数的结果是否合法，只有合法的情况下，才能与下一步组合。你可能会想把这些判断放在组合逻辑里不就得了吗？

```scala
def map[B](next: A=>B): M[B] = { 
    val result = inner  // 执行了当前行为
    if (result != null) {
        new M(next(result))
    } else {
        null.asInstanceOf[M[B]]
    }
}
```


上面的情况不是我们希望的，因为要判断当前行为(inner)的结果是否符合传递给下一个行为，只有执行当前行为才能拿到结果。而我们希望组合子做的是先把所有行为组合起来，最后再一并执行。

看来只能要求 `next` 函数在实现的时候做了异常检测，但即使 `next` 函数做了判断，也不一定完美；因为行为已经先被组合好了，执行的时候，链上的每个函数仍会被调用。假设组合了多个函数，执行时中间的一个函数即使为 `null`，仍会传递给下一个执行（下一个也必须也对参数检测），其实这个时候已经没有必要传递下去了

在 scala 里对这种结果不确定的情况，提供了一个专门的 `Option`，它有两个子类：`Some` 和 `None`，其中 `Some` 表示结果是一个正常值，可以通过模式匹配来获取到这个值，而 `None` 则表示为空的情况。

如果我们 `Option` 来表示我们前面的行为，可以写为： `f: A=>Option[B]`，即结果被一个富类型给包装起来，它表示结果可能有值，也可能为空。

#### 2) 副作用的问题

另外一个真实世界中无法用函数式完美解决的问题是 IO 操作，因为 IO 无论如何总要伴随状态的产生或变化(也就是副作用)

一段常见的举例片段：

```scala
def read(state: State) = {
    // 返回下一状态和读取到的字符串
    (state.nextState, readLine)
}

def write(state: State, str: String) = {
    //返回下一状态，和字符串处理的结果
    //这里是 print 返回 为Unit 类型，所以最终返回一个 State 和 Unit 的二元组
    (state.nextState, println(str))
}
```

这两个函数类型为 `State => (State, String)` 和 `(State,String) => (State, Unit) `在入参和出参里都伴随 State，是一种”不纯粹”的函数，因为每次都执行状态都不一样，即使两次的字符串是一样的，状态也是不同的。这是一种典型的“非引用透明”问题，这样的函数无法满足结合率，函数的组合性无法保障。

要实现组合特性，需要把函数转换为符合“引用透明”的特性，我们可以通过柯里化来把两个函数转化为高阶函数：

```scala
def read = 
    (state: State) => (state.nextState, readLine)

def write(str: String) = 
    (state: State) => (state.nextState, println(str))
```


现在这两个函数相当于只是构造了行为，要执行它们需要后续传入“状态”才行；等于把执行推迟了；同时这两个函数现在符合“引用透明”特性了。

再进一步，为了把 State 在构造行为的时候给隐藏起来，我们继续重构：

```scala
class IOAction[A](expression: =>A) extends Function1[State, (State,A)] { 

    def apply(s:State) = (state.next, expression) 
}

def read = new IOAction[String](readLine)

def write(str: String) = new IOAction[Unit](println(str))
```

现在我们可以把 read 看做是一个 `() => IOAction[String]` 的函数，write 则是： `String => IOAction[Unit]` 的函数。


#### 用一个”富类型”表示行为的结果

我们看到现实世界的行为，除了可以用 `A=>B` 这样的 plain function 描述，还有 `A=>Option[B]` 或 `A=>IOAction[B]` 这种结果是一个富类型的函数来描述。我们把后两种统一描述为：

```scala
A => Result[B]
```

当我们要组合一个这种形式的行为时，不能再使用 `map`，而是 `flatMap` 组合子。

实际上，我们一开始就提到过 `map` 并不是必须的方法，`flatMap` 才是，可以把 `map` 看做只是一种特例。把行为都用统一形式 `A => Result[B]` 来描述，对于普通的 `A=>B` 也可以转为 `A=>Result[B]` 

现在我们看几个 `flatMap` 的例子，先看一个 `Option` 的实际例子：

```scala
scala> Some({println("111"); new A}).
        flatMap(a => {println("222");Some(new B)}).
        flatMap(b => {println("333"); None}).
        flatMap(c => {println("444"); Some(new D)})
111
222
333
res14: Option[D] = None
```

上面的例子看到，在组合第三步的时候，得到 `None`，后边的第四步 `c => {println("444"); Some(new D)}` 没有被执行。这是因为 `None` 与后续的行为结合时，会忽略后续的行为。




If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
