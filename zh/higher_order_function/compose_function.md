# 函数组合 function composition

通过前面介绍的高阶函数,  map, fold 以及柯里化, 其实已经见识到什么是函数组合了.
如之前例子中的 map 就是 由 fold 函数与 reverse 函数组合出来的.

这就是函数式的思想, 不断地用已有函数, 来组合出新的函数.

![](http://turner.faculty.swau.edu/mathematics/materialslibrary/images/composition.jpg)

如图就是函数组合，来自[Catgory Theory](https://en.wikipedia.org/wiki/Category_theory)（Funtor 也是从这来的，后面会讲到）, 既然从 A到B 有对应的映射f，B到 C有对应的映射g， 那么`(g.f)(x)`也就是 `f` 与 `g`
的组合`g(f(x))`就是 A到 C 的映射。上一章实现的 map 函数就相当于 `reverse.fold`.

### compose

我们可以用 Eweda 非常方便的 compose 方法来组合函数
```js
var gf = E.compose(f, g)
```

说到了函数组合, 柯里化, 我想现在终于可以解释清楚为什么在这里选用 Eweda/Ramda 而不是 Underscore 了.

举过例子🌰 如果我现在想要 tasks 列表中所有属性为` completed`为` true` 的元素, 并按照` id` 排序.

underscore 里会这样写:
```js
_(tasks)
    .chain()
    .filter( task => task.completed===true)
    .sortBy( task => task.id).value();
```

这种方式怎么看都不是函数式, 而是以对象/容器为中心的串联，有些像 jquery 对象的链式调用,
或者我们可以写的函数式一些, 如
```js
_.sortBy(_.filter(tasks, task => task.completed===true), task => task.id)
```

恩恩, 看起来不错嘛, 但是有谁是这么用 underscore的呢. 一般都会只见过
链式调用才是 underscore 的标准写法。

来对比一下用 Eweda/Ramda 解决的过程 :
```js
compose(sortBy(task=>task.id), filter(task=>task.completed===true))(tasks)
```
好像没什么区别啊? 不就是用了 compose 吗?

区别大了这, 看见` tasks` 是最后当参数传给`E.compose()`的吗? 而不是写死在filter 的参数中. 这意味着在接到需要处理的数据前, 我已经组合好一个新的函数在等待数据, 而不是把数据混杂在中间, 或是保持在一个中间对象中. 而 underscore 的写法导致这一长串`_.sortBy(_.filter())`其实根本无法重用。

好吧如果你还看不出来这样做的好处. 那么来如果我有一个包含几组 tasks的列表 groupedTasks, 我要按类型选出 completed 为 true 并按 id 排序. 如我现在数据是这个：
```json
groupedTasks = [
  [{completed:false, id:1},{completed:true, id:2}],
  [{completed:false, id:4},{completed:true, id:3}]
]
```

underscore:
```js
_.map(groupedTasks,
   tasks => _.sortBy(_.filter(tasks, task => task.completed===true), task => task.id))

```
看见我们又把`_.sortBy(_.filter())`这一长串原封不动的拷贝到了 map 里。
因为underscore 一开始就要消费数据，使得很难重用，除非在套在另一个函数里：
```js
function completedAndSorted(tasks){
  return _.sortBy(_.filter(tasks, task => task.completed===true), task => task.id))
}
_.map(groupedTasks, completedAndSorted)
```
只有这样才能重用已有的一些函数。或者虽然 underscore 也有`_.compose` 方法，但是
几乎所有 underscore 的方法都是先消费数据（也就是第一个参数是数据），使得很难放到
`compose` 方法中，不信可以尝试把 filter 和 sortBy 搁进去，反正我是做不到。

来看看真正的函数组合
```js
var completedAndSorted = compose(sortBy(task=>task.id),
                                 filter(task=>task.completed===true))
map(completedAndSorted, groupedTasks)
```
看出来思想完全不一样了吧.

由于 Eweda/Ramda 的函数都是自动柯里化,而且数据总是最后一个参数, 因此可以随意组合, 最终将需要处理的数据扔给组合好的函数就好了. 这才是函数式的思想. 先写好一个公式，在把数据扔给
公式。而不是算好一部分再把结果给另一个公式。

![](http://www.moebiusnoodles.com/s/wp-content/uploads/2012/09/ThreeFunctionMachines.jpg)

而 underscore 要么是以对象保持中间数据, 用 chaining 的方式对目标应用各种函数（书上会写这是Flow-Base programming，但我觉得其实是 Monad，会在下一章中介绍）, 要么用函数嵌套函数, 将目标一层层传递下去.

###pipe

类似 compose, eweda/ramda 还有一个方法叫 pipe, pipe 的函数执行方向刚好与 compose 相反. 比如 `pipe(f, g)`, `f`会先执行, 然后结果传给` g`, 是不是让你想起了 bash 的 pipe
```
find / | grep porno
```
实际上就是`pipe(find, grep(porno))(/)`

没错,他们都是一个意思. 而且这个函数执行的方向更适合人脑编译(可读)一些.

如果你已经习惯 underscore 的这种写法
```js
_(data)
  .chain()
  .map(data1,fn1)
  .filter(data2, fn2)
  .value()
```
那么转换成 pipe 是很容易的一件事情，而且更简单明了易于重用和组合。
```js
pipe(
  map(fn1),
  filter(fn2)
)(data)
```

[完整代码](http://jsbin.com/hivaje/1/edit?js)
