## [译] Go Code Review Comments



文本翻译自：https://github.com/golang/go/wiki/CodeReviewComments



这个页面收集了在审查Go代码期间的常见的建议，因此，一个详细的解释可以通过速记来解释。这是一大堆常见的错误，而不是全面的风格指南。



你可以查看此作为补充[[Effective Go](https://golang.org/doc/effective_go.html)]。



* Go fmt

  对你的代码运行[gofmt](https://golang.org/cmd/gofmt/)以自动修复大多数机械式风格问题。几乎所有的go代码都可以使用gofmt。本文档的其余部分将讨论非机械式风格。

  另一种方法是使用[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)，`go fmt`的超集，它会根据需要额外添加（和删除）import。



* Comment Sentences

  从[commentary](https://golang.org/doc/effective_go.html#commentary.)看到，注释文档声明应该是一个完整的句子，即使这看起来很多余。注释应该以所描述事物的名称开头，并以句号结尾。这种方法使得在go doc中可以提取出不错的格式化文档。



* Contexts



* Copying

  在Go中，从其他包中复制结构体时，一定要小心注意。

  一般来说，如果它的方法与指针类型关联，则不复制类型T的值，*T。



* Declaring Empty Slice

  当生命一个空数组时，推荐使用

  ```go
  var a []string
  ```

  而不是

  ```go
  t := []string{}
  ```

  前者声明了一个nil的slice，而后者声明了一个non-nil的slice，但是长度为0，这两种形式在功能上是相等的 — 不管是len和cap都是0，而nil slice则是推荐使用的。

  请注意，在一些情况下，使用non-nil但是长度为0的slice更推荐，例如使用JSON编码时（[]string被编码成 null，而[]string{}被编码成[]）

  当设计接口时，请避免nil slice 和non-nil zero-length slice，这可能会导致微妙的编程错误。

  更多关于nil的讨论，请查看Francesc Campoy的演讲 [Understanding Nil](https://www.youtube.com/watch?v=ynoY2xz-F8s).



* Crypto Rand



* Doc Comment



* Dont Panic



* Error Strings



* Examples



* Goroutine Lifetimes



* Handle Errors



* Imports



* ImportBlank



* Import Dot



* In-Band Errors



* Indent Error Flow



* Initialisms



* Interfaces



* Line length



* Mixed Caps

  See [MixedCaps](https://golang.org/doc/effective_go.html#mixed-caps)，Go语言推荐shying驼峰，而不是蛇形小写。例如对于不需要被导出的变量，使用maxLength而不是MaxLength或者MAX_LENGTH



* Named Result Parameters



* Naked Returns



* Package Comments



* Package Names



* Pass Values



* Receiver Names



* Receiver Type



* Synchronous Functions



* Useful Test Failures



* Variable Names

























