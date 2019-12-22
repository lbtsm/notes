代码指南风格：这些都是我工作中遇到并记录的



[TOC]



# 变量声明



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



# 检查工具



