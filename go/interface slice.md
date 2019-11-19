## [译]*InterfaceSlice*



文本翻译自：https://github.com/golang/go/wiki/InterfaceSlice



# 引言



在Go中你可以指定任何类型给`interface{}`，所以人们常常会编写以下的代码

```go
var dataSlice []int = foo()
var interfaceSlice []interface{} = dataSlice
```

并且得到一个错误

```go
cannot use dataSlice (type []int) as type []interface { } in assignment
```

由此引出一个问题，为什么我可以指定任何类型给`interface{}`，但是为什么不能指定slice给 `[]interface{}`



# 为什么？

主要有两个原因：

第一个原因是 类型为`[]interface`的变量不是`interface{}`类型，而是一个恰好元素类型为`interface{}`的slice，但是在这里，也可以说是想要表达的意义是很清晰明了的了。

好吧，是吗？

一个类型为`[]interface`的变量，在编译时，拥有一个特殊的内存布局。

每个`interface{}`都拥有两个属性（一个属性是它所包含的类型，另一个属性则是包含的数据或指向数据的指针），因此，长度为N，类型为`[]interface{}`的slice在编译，会开辟N*2大小的块。

这就不同于相同长度下，类型为[]MyType的slice在编译时，开辟的N*sizeof(MyType)大小的块

从数据背后看出它们的不同，这也就是为什么不能指定[]MyType给`[]interface{}`



# 如何替代上述代码？



这取决于你想要做的事情。

如果您想要一个任意数组类型的容器，并且您计划在执行任何下标操作之前更改回原始类型，则可以使用 `interface{}`。代码将是通用的（如果不是编译时类型安全的时候）并且速度快。

如果你真的想要一个`[]interface{}`因为你将在转换之前想做下标操作，或者你正在使用特定的接口类型而你想使用它的方法，那你就不得不复制切片。

```go
var dataSlice []int = foo()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
	interfaceSlice[i] = d
}
```



