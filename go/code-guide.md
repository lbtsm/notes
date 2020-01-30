代码指南风格：这些都是我工作中遇到并记录的



[TOC]



#  序

* 一个语言的代码规范，就像是上学时的《中小学生守则》一样，大家都知道，但是基本不怎么会去遵守

在自己写代码、帮别人code review的时候，经常会看有很多代码，虽然是同一个意思，但是使用了不同的风格去编写。故写下这篇文章来记录，并且让自己的代码趋向统一

* 另外在这里记录下go官方提供的code规范

[go effective中文版](https://go-zh.org/doc/effective_go.html)

[go effective 英文原版](https://golang.org/doc/effective_go.html)

[github go官方code review comments](https://github.com/golang/go/wiki/CodeReviewComments#Receiver_Type)



* 另外推荐读一下《重构2》 马克 大师写的第二版重构 （买书之后请一定要看 '序'，有彩蛋）



# Code Standard



* Json Declaring Empty Slice

在 Json 结构化的结构体中，如果包含数组，声明时，如果为初始化，则序列化后对应的filed为null

例如如下结构：

```go
type UserMail struct {
	Id          uint   `json:"id"`
	State       int    `json:"state"` 
}

type GetMailResponse struct {
	UserMails []UserMail `json:"user_mails"`
}
```

此声明对应输出为：

```go
resp := communication.GetMailResponse{}

// output : {"user_mails":null}
```

而另一种声明方式，则是另一种结果：

```go
resp := GetMailResponse{
		UserMails: make([]UserMail, 0),
}

// output: {"user_mails":[]}
```

如果在于前端对接时，返回的数组为空时，需要返回空数组，则需要注意此处



* Bool 判断

  在code review中，经常会有看到有人呢写下面的这种判断

  ```go
  if courseId.Bool == true {
    // do .....
  }
  ```

  而这个代码则优化（简化）为一下代码

  ```go
  if courseId.bool {
  	// do ....
  }
  // 这种代码和上述的代码的释义是一致的，但是看上去比上一种写法要简洁，优雅
  // 另外 判断为false的代码可以写成 if !courseId.bool {}
  ```

  

* 变量声明

  * var 声明方式

  ```go
  var beforeName string
  var afterName string
  ```

  ​	像上述代码，使用了多个var声明了多个相同类型的变量，则可以简化为：

  ```go
  var (
  	beforeName,afterName string
  )
  ```

  > 这个简化方式不仅可以在声明变量时使用，也同时适用于函数传参时

  

* 变量大小写

  go语言推荐使用驼峰大小写方式去编写代码，而不是像python pep8的规范那样使用下划线（蛇形）

  * const

    ```go
    const (
    	ACTION_UNKOWN                        = 0
    	ACTION_ACTIVATE                      = 1 
    	ACTION_ASSIGN                        = 2
    )
    ```

    在声明const变量时，很多人都习惯使用上面这种方式，但是这是不符合go的语言规范的。而以下声明的方式才是go推崇的

    ```go
    const (
    	ActionUnknown                        = 0
    	ActionActivate                      = 1 
    	ActionAssign                        = 2
    )
    ```

    

  * 专用名词

    在我们的开发中，会碰到类似于id、uri、http、ip、dns等等的一些专用名词，这些专用名词，不管都是大写的，还是全部小写的都是符合规范的。

    > 本人推荐使用，变量如需在外部使用，则首字母大写，其余字母则小写，例如Uri，当作函数参数时，全小写

  

* 

* 



# 检查工具



