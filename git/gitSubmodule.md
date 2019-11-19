### 什么是子模块 

子模块是在主项目中，使用其他项目作为主项目的一个附属目录

###  为什么使用子模块 

解耦、提高协作度 

### 什么时候使用子模块 

当协作双方共同使用一份文件时（例如协作双方使用protobuf时） 

### 怎么使用子模块 

#### 1、添加子模块 

当我们想在项目中使用子模块时，使用git submodule add （URL）添加 

```shell
git submodule add https://github.com/letsgo/proto.git 
```

默认情况下，上条命令会在执行路径下，创建一个与仓库名称相同的目录作为子模块目录，但是你想自己创建子模块的目录，则可以在添加子模块命令后增加对应目录名称，例如： 

```shell
git submodule add https://github.com/letsgo/proto.git protos
```

* 当commit时，proto记录的模式时160000，这是git的一种特殊模式，本质上意味着你是将此次提交记作一项目录记录的，而非将它记录成一个子目录或者一个文件 

#### 2、克隆已有子模块的项目 

当你在克隆含有子模块的项目时，默认会将子模块的目录下载至本地，但子模块对应的目录内容为空 

```shell
$git clone https://hi.hxxbaby.com/letsgo/reviewers.git 

$cd reviewers 

$cd protos/proto 

$ls 

$ 
```

所以你需要使用`git submodule init`，来初始化本地配置文件（根据.gitsubmodule），接下来使用`git submodule update`，从子模块对应的仓库拉取数据，拉取子模块在最后一次提交时，commitId对应的版本 

在gitlab中显示如下 

![image-20190920175110471](/images/image-20190920175110471.png)



还有另外一种方式，即在`git clone`时，加`—recursive`选项，它会自动初始化子模块和更新子模块 

![image-20190920175031071](/images/image-20190920175031071.png)

#### 3、在有子模块中的项目中工作 

##### 拉取上游修改 

* 查看子模块差异 

1.使用`git diff --submodule`，可以查看子模块的差异，如果你不想每次都输入`--submodule`，可以使用命令`git config --global diff.submodule log`设置子模块全局变量 

2.当子模块更新时，使用`git status`命令，会显示子模块有新的commit 

![image-20190920175246523](/images/image-20190920175246523.png)

如果设置了status.submodulesummary，使用`git status`则会显示子模块的更新摘要 

设置status.submodulesummary使用命令`git config status.submodulesummary 1` 

再次使用`git status` 

![image-20190920175302502](/images/image-20190920175302502.png)

* 拉取上游修改 

1.简单的操作 

每次需要更新时，进入到子模块对应的目录中，使用`git pull`命令，拉取当前分支最新的修改 

2.精细的操作 

使用命令`git submodule update --remote` 

> 此命令默认拉取master分支的最新commit，如果想拉取指定分支，请使用`git config -f .gitmodules submodule.<子模块名称>.branch <指定的分支>`，在使用上述更新命令，就可以拉取指定的分支代码 

##### 在子模块上工作 

与我们使用主项目时，操作一致，先拉取基类分支最新的commit，在根据基类分支checkout出一个新的分支，在新分支上进行修改，随后提交 

当我们在本地修改时，别人已提交时，可使用`git submodule update --remote --merge` 或者 `git submodule update --remote --rebase` 

> 当我们运行`git submodule update`从子模块仓库中抓取修改时，Git 将会获得这些改动并更新子目录中的文件，但是会将子仓库留在一个称作“游离的HEAD”的状态。这意味着没有本地工作分支（例如 “master” ）跟踪改动。所以你做的任何改动都不会被跟踪 

> 为了将子模块设置得更容易进入并修改，你需要做两件事。首先，进入每个子模块并检出其相应的工作分支。接着，若你做了更改就需要告诉 Git它该做什么，然后运行`git submodule update --remote`来从上游拉取新工作。你可以选择将它们合并到你的本地工作中，也可以尝试将你的工作变基到新的更改上 

##### 发布子模块的改动 

1.简单的操作 

进入到子模块目录中，使用常用push手段提交 

2.精细的操作 

当我们在主项目、子模块都有修改时，只提交了主项目的修改，子模块因被我们所忽略所以没有提交，为了不让这种情况发生，git可以设置在推送主项目时，检查子模块是否已push，`git push`接受flag `--recurse-submodules` <check>或 <on-demand> 

option <check> ：会检查，当子模块有修改时，并且没有push，则主项目push失败 

option <on-demand> ： 会先将子模块推送，后将主项目推送，如子模块推送失败，则主项目推送也失败 

##### 子模块合并修改 

![image-20190920175344887](/images/image-20190920175344887.png)



![image-20190920175456072](/images/image-20190920175456072.png)



![image-20190920175515214](/images/image-20190920175515214.png)



##### 子模块技巧 

1.foreach 

当主项目拥有众多子模块时，此时需要添加一个功能，而此功能需要多个子模块支持时，为每一个子模块checkout对应的分支是很痛苦的，而使用foreach就能很轻松的帮助我们在多个子模块项目中，使用命令 

git submodule foreach 'git stash' 

git submodule foreach 'git checkout -b feature/new' 

git diff; git submodule foreach 'git diff' 

2.别名 

如果在主项目中使用了大量的子模块，以下一些命令十分有用（git别名参见参考） 

```shell
git config alias.sdiff '!'"git diff && git submodule foreach 'git diff'" 

git config alias.spush 'push --recurse-submodules=on-demand' 

git config alias.supdate 'submodule update --remote --merge' 
```



##### 子模块的问题 

1.创建的子模块目录，依旧存在 

当创建一个新分支，在其中添加一个子模块，之后切换到没有该子模块的分支上时，你仍然会有一个还未跟踪的子模块目录 

![image-20190920175628943](/images/image-20190920175628943.png)

想要移除此目录也很方便，但是当你移除之后，再回到拥有子模块的分支时，需要你用`git submodule update --init`来重新建立和填充 

![image-20190920175651876](/images/image-20190920175651876.png)

2.在已存在的目录基础上，新建子模块 

在已存在的目录智商，新建子模块，会得到以下错误，如果使用rm -rf protoexist，也会得到如下问题 

![image-20190920175711453](/images/image-20190920175711453.png)

解决上处问题，首先需要取消暂存protoexist目录，才可添加子模块 

![image-20190920175722559](/images/image-20190920175722559.png)

* 另外一个问题 

![image-20190920175735779](/images/image-20190920175735779.png)

> 参考链接 

> git子模块 ：https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97 

> git别名参考：https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-Git-%E5%88%AB%E5%90%8D#r_git_aliases 