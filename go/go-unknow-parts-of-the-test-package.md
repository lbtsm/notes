## [译]Go test中鲜为人知的功能



文本翻译自：https://medium.com/@blanchon.vincent/go-unknown-parts-of-the-test-package-df8988b2ef7f



# 序



`go test`命令也许是开发者们在Go中经常使用的命令了，然而，在使用中你可能不了解`go test`一些有意思的细节，下面就让我们开始学习它（go test）吧！



# 跳过缓存的惯用方式



在Go中，当你在命令行中连续敲下两次test命令，当test第一次尝试运行并通过时，你可以看到只有一次实际的运行，实际上，`go test`在test文件没有修改时，会使用缓存系统运行test，接下来，就让我们用math包尝试运行一下吧：

```
root@91bb4e4ab781:/usr/local/go/src# go test ./math/
ok     math   0.007s
root@91bb4e4ab781:/usr/local/go/src# go test ./math/
ok     math   (cached)
```

在运行`go test `时，go会检查test文件是否有被修改，也会检查环境变量和运行时所携带的命令参数，当更新了环境变量或添加了命令行参数，则不使用缓存

```
go test ./math/ -v
[...]
=== RUN   ExampleRoundToEven
--- PASS: ExampleRoundToEven (0.00s)
PASS
ok   math 0.007s
```

添加`-v`参数，在运行一次test，如图，此次test没有使用缓存，关于test的缓存，Go使用hash算法去比较运行的文件内容、环境变量和运行时所携带的命令行参数，一旦经过计算，则缓存到`$GOCACHE`对应文件夹下，在UNIX则默认缓存到`$CDG_CACHE_HOME`或者`$HOME/.cache`文件夹下，因此清除缓存则需要清除对应文件夹下的内容。



关于运行时所携带的`flags`，Go官方文档已作出了说明，并不是所有`flag`都会被缓存：

> 运行缓存匹配的规则是，包括运行相同的test二进制文件，命令行参数完全来自一组“可缓存”的参数，包括（-cpu、-list、-parallel、-run、-short和-v），如不想使用测试缓存，下次运行test时，所携带的`falg`则与缓存中的的`flag`不相同即可，而跳过test缓存的惯用方式则是显式的使用`-count=1`



从`count`定义数字运行test以来，`-count=1`显式地说明了test只运行一次，不会多也不会少，这使得它成为跳过test缓存的惯用方式的完美候选者。



我们应该注意Go1.12之前，我们可以设置环境变量`GOCACHE=off`来避开测试缓存

`go test math/`

当我们运行test时，Go将运行指定包下的每一个包，Go处理测试报名的方式为我们提供了更多的测试策略。



# 白盒测试和黑盒测试



黑盒测试由你的代码组成并且没有任何关于内部结构的知识（首字母小写的），只有可导入的函数和可获得的结构（首字母大写的），而白盒测试允许我们通过私有的方法实现内部的测试。Go原生都支持两种。这里有一个简单的程序示例，我们将为其编写白盒测试和黑盒测试去观察它们各自的优势。

```go

package deck

import (
	"errors"
	"math/rand"
)

var Empty = errors.New("Empty deck")

type Deck struct {
	cards []uint8
	shuffled bool
}

func NewDeck(numbers uint8) *Deck {
	cards := make([]uint8, 0, numbers)
	for i := uint8(0); i < numbers; i++ {
		cards = append(cards, i+1)
	}

	d := Deck{ cards: cards }

	return &d
}

func (d *Deck) Draw() (card uint8, err error) {
	if !d.shuffled {
		d.shuffle()
	}

	if len(d.cards) == 0 {
		return 0, Empty
	}
	card, d.cards = d.cards[0], d.cards[1:]

	return card, nil
}

func (d *Deck) shuffle() {
	rand.Shuffle(len(d.cards), func(i, j int) {
		d.cards[i], d.cards[j] = d.cards[j], d.cards[i]
	})
	d.shuffled = true
}
```

这段代码支持洗牌并且允许用户从中抽取一张牌。黑盒测试确保我们创建一副牌并且能让我们抽牌直到最后。（译者注：请注意下段代码的package，与我们实际中编写的测试包的区别）

```go
package deck_test

import (
	"github.com/blanchonvincent/test-package/card"
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestDeckCanDrawCards(t *testing.T) {
	var num uint8 = 10

	d := deck.NewDeck(num)
	for i := uint8(0); i < num; i++ {
		_, err := d.Draw()
		assert.Nil(t, err)
	}
	_, err := d.Draw()
	assert.Equal(t, err, deck.Empty)
}
```

编写正确的黑盒测试的唯一条件是在导入的包名后添加_test后缀，它被视为一个不同的包，因此它无法访问测试目标包中未导出（首字母小写）的函数。Go原生地支持这种黑盒测试，并且编译器不会计较在相同路径下拥有不同的报名。



白盒测试将确保我们在第一次抽牌前，进行一次洗牌。

```go
package deck

import (
	"fmt"
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestDeckShouldBeShuffledOnce(t *testing.T) {
	var num uint8 = 5

	d := NewDeck(num)
	assert.Equal(t, len(d.cards), int(num))
	assert.Equal(t, d.shuffled, false, "Deck should init as not shuffled")
	orderBefore := fmt.Sprint(d.cards)

	d.shuffle()
	assert.Equal(t, d.shuffled, true, "Deck has not been marked as shuffled")
	orderAfter := fmt.Sprint(d.cards)

	assert.NotEqual(t, orderBefore, orderAfter, "Deck once shuffled should have new card order")
}
```

这个测试使用了（与目标测试的包）相同的包名，因此可以访问私有函数（成员）



然而，白盒测试拥有一个约束，在黑盒测试中，测试你的公共方法，并不理会包内部的实现，因此我们可以自由的变更和提高内部的实现而不破坏任何测试，而白盒测试则与内部实现紧紧地绑在了一起，改善内部实现则会中断测试。



接下来，让我们走进test包的另一个特性，基准测试。



# 根据你的test运行基准测试



自Go1.12以来，允许你使用`-benchtime=1x`，`-benchtime=10x`等等，去运行你想运行的基准测试时间，`-benchtime=1x`标记对你的测试套件很有用，因为它至少允许你至少运行你的基准测试，来检查是否由于你上次修改而遭到破坏。

在Go1.12之前，我们能够使用`-benchtime=1ns`在1ns之后停止基准测试中的循环来实现相同的结果，自1ns是最小的时间单位，所以它只会运行一次基准测试。基准测试允许你获取运行时的指标，例如一次运行的时间、使用内存或者在堆中分配的内存，自Go1.13出版之后，你可以获取的指标远不止如此



# 报告自定义的指标



Go1.13自带了`reportMetric`方法允许我们获取自定义的指标，让我们在先前的例子上，修改代码，在抓第一张牌之前，随机洗牌1-20次，在这里我们编写一个报告洗牌次数与基准测试次数之间比例的例子



```go
package deck

import (
	"testing"
)

func BenchmarkGC(b *testing.B) {
	b.ReportAllocs()
	shuffled := 0

	for i := 0; i < b.N; i++ {
		d := NewDeck(100)
		_, _ = d.Draw()
		shuffled += int(d.shuffled)
	}

	b.ReportMetric(float64(shuffled)/float64(b.N), "shuffle/op")
}
```

然后生成结果

```
BenchmarkDeckWithRandomShuffle-8       88666        12389 ns/op            5.15 shuffle/op        144 B/op         2 allocs/op
PASS
ok     1.529s
```

我们可以观察这个结果，Go1.13版本带来了有一个变化，不会再根据B.N循环而是使用真实的数字，CL允许在基准测试中减少来自外部的干扰，例如GC，尤其是在一个很小的墙时间（译者注：wall time这个词翻译的不是很好，有大牛走路路过，可以帮忙解答下吗？）内基准测试也可以跑得更快。

























