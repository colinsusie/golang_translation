# Effective Go

- [介绍](#介绍)
    - [示例](#示例)
- [格式化](#格式化)
- [注释](#注释)
- [命名](#命名)
    - [包名](#包名)
    - [Getters](#Getters)
    - [接口命名](#接口命名)
    - [MixedCaps](#MixedCaps)
- [分号](#分号)
- [控制结构](#控件结构)
    - [If](#If)
    - [重复声明和重复赋值](#重复声明和重复赋值)
    - [For](#For)
    - [Switch](#Switch)
    - [类型Switch](#类型Switch)
- [函数](#函数)
    - [多返回值](#多返回值)
    - [命名的返回结果](#命名的返回结果)
    - [Defer](#Defer)
- [数据](#数据)
    - [使用new分配](#使用new分配)
    - [构造函数和复合字面量](#构造函数和复合字面量)
    - [使用make分配](#使用make分配)
    - [Arrays](#Arrays)
    - [Slices](#Slices)
    - [二维Slices](#二维Slices)
    - [Maps](#Maps)
    - [打印](#打印)
    - [Append](#Append)
- [初始化](#初始化)
    - [常量](#常量)
    - [变量](#变量)
    - [init函数](#init函数)
- [方法](#方法)
    - [指针和值](#指针和值)
- [接口和其他类型](#接口和其他类型)
    - [接口](#接口)
    - [转换](#转换)
    - [接口转换和类型断言](#接口转换和类型断言)
    - [通用性](#通用性)
    - [接口和方法](#接口和方法)
- [空白标识符](#空白标识符)
    - [多重赋值中的空白标识符](#多重赋值中的空白标识符)
    - [未用的import和变量](#未用的import和变量)
    - [import的副作用](#import的副作用)
    - [接口检查](#接口检查)
- [内嵌](#内嵌)
- [并发](#并发)
    - [通过通信共享](#通过通信共享)
    - [Goroutines](#Goroutines)
    - [Channels](#Channels)
    - [Channels of channels](#ChannelsOfChannels)
    - [并行](#并行)
    - [泄漏的缓冲区](#泄漏的缓冲区)
- [错误](#错误)
    - [Panic](#Panic)
    - [Recover](#Recover)
- [一个Web服务器](#一个Web服务器)

## 介绍

Go是一门新语言。尽管它借鉴了现有语言的一些理念，但Go有一些不同寻常的特性使得它不同于其他语言写的程序。将C++或Java程序直译成Go通常不会得到满意的结果，因为Java程序是由Java写的，并不是Go。另一方面，从Go的视角去思考一个问题，会产生一个成功的但截然不同的程序。换句话说，要写好Go程序，理解它的特性和习惯用法是很重要的。同时了解Go即有的约定也很重要，比如命名，格式化，程序构造等等，这可以让你写的程序容易被其他Go程序员所理解。

这份文档会给出一些技巧用于编写清晰的符合习惯的Go代码。它是作为 [language specification](https://golang.org/ref/spec)，[Tour of Go](https://tour.golang.org/)，[How to Write Go Code](https://golang.org/doc/code.html) 的补充，因此建议你先阅读这些文档。

### 示例

[Go的包源码](https://golang.org/src/)不仅仅是核心库，也可以作为如何使用这门语言的示例。并且许多包还包含了可工作的，自包含的执行例子，你可以直接从[golang.org](https://golang.org/)运行它们，比如[这个](https://golang.org/pkg/strings/#example_Map)(在页面中可能需要点击"Example"展开它)。如果你有任何关于如何解决问题，或者有些东西可以怎么实现的疑问，那么库中的文档，代码和例子都可以提供相关的答案，思路和背景。

## 格式化

格式化问题最具争议，但也最无关紧要。虽说人们可以适应不同的编码风格，但如果他们不必这样做岂不是更好？，如果所有人都遵从同一种风格，那就不用在这个话题上浪费太多时间了。问题是在没有一份冗长的风格指南的情况下，如何实现这个*乌托绑*(译注：如何实现这个设想)。

在Go中我们用一种特别的方法，让机器来处理大多数格式化问题。gofmt(也可作为go fmt，它是在包级别而不是源代码级别进行的处理)会读取Go程序，然后以标准的缩进和垂直对齐来重排代码，它保留注释但必要时会重排注释。如果你想知道如何处理一些新的排版情况，那就运行gofmt；但如果结果好像不大对，那就请自己重排你的程序(或提交有关gofmt的BUG)，不必纠结于此。

举个例子，你不用花时间去对齐结构中字段的注释，gofmt会帮你处理的，如下面的声明：
```go
type T struct {
    name string // name of the object
    value int // its value
}
```

gofmt将会按列对齐：
```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

标准包中所有的代码都已经用 gmfmt 格式化过了。

还有一些关于格式化的简要细节：

- 缩进：默认使用制作符(tab)缩进(gofmt也默认使用它)，除非你认为真的有必要才使用空格。
- 行长度：Go对行的长度没有限制，不用担心打孔卡溢出(译注：不用担心超出页面限制)，如果觉得行太长，可以对它进行换行并用tab增加一个额外的缩进。
- 括号：比起C和Java，Go需要更少的圆括号：控制结构(if, for, switch)的语法不需要括号。并且操作符的优先级层次更短更清晰，所以下面句子：
```go
x<<8 + y<<16
```

就如同空格所暗示的优先级一样。

## 注释

Go提供了C风格的块注释 `/* */`和C++风格的行注释 `//`。行注释更为常用；块注释主要出现在包注释中，但在表达式或禁用大块代码时也很有用。

godoc程序(同时也是一个Web服务)会处理Go源代码并从中提取包的文档内容。在顶层声明之前的注释(中间没有换行符)会和声明一起被提取出来，作为该声明的说明文档。这些注释的风格和条理性决定了 godoc 产生的文档质量。

每一个包都应该有一段包注释(一个出现在包声明语句之前块注释)。对于有多个文件的包，包注释只需要出现在任意一个文件中。包注释应该从整体上介绍包的内容以及提供包相关的信息。它将出现在 godoc 页面的最上方，并为其后的内容建立详细的文档。
```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果包比较简单，那么包注释可以写得简要一点：

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

注释不需要进行额外的格式化(比如使用星号横条)，生成的输出甚至可能不用定宽字体来显示，所以不要依赖于空格对齐，像gofmt一样godoc会处理好这些问题。注释是未解释的纯文本，像HTML或其他如__this__这样的标注都会原样输出，所以不要使用它们。但有一个调整 godoc 会去做：就是以等宽字体显示缩进的文本，这种适用于程序片段。fmt包的包注释就利用这一点取得了很好的效果。

取决于具体的上下文，godoc 可能不会重新格式化注释，所以要尽量让它们看起来直观：使用正确的拼写，标点符号，和句子结构，折叠长行等等。

在一个包里面，任何在顶层声明之前的注释都将作为该声明的文档注释。程序中每一个可导出的名字(首字母大写)都应该有一个文档注释。

文档注释最好使用完整的句子，这样才可以实现各种自动化显示。第1个句子应该是以下方声明的名字开头的一句话摘要：
```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每一个文档注释都以所要描述的项名字开头，你就可以使用go工具的doc子命令并通过grep执行输出。假设你记不住"Compile"这个名字，而又要查找正则表达式的解析函数，那么你可以执行下面命令：
```sh
$ go doc -all regexp | grep -i parse
```

假如包里所有的文档注释都以"This function..."开头，那么grep也不能帮你记住名字了。但由于文档注释始于描述的名字，你就会看到下面的输出，这个就能帮你想起正在查找的词：
```sh
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
```

Go的声明语法允许成组的声明，一个单独的文档注释可以介绍一组相关的常量或变量。但因为是整体地声明，这样的注释常常是比较笼统的。
```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

分组还可以指示声明项之间的关系，比如一组受一个互斥量保护的变量。
```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

## 命名

在Go中命名和其他语言一样重要。它们甚至会影响语义：一个名字在包外是否可见取决于它的第一个字母是否为大写。因此，值得花一点时间讨论一下Go程序的命名规范。

### 包名

当一个包被导入时，包名将成为一个访问器用于访问包的内容。在下面声明之后：
```go
import "bytes"
```

导入包就可以使用`bytes.Buffer`。包的名字要起得好：短，简洁，易于记忆。习惯上包使用小写和单个词的名字，不需要下划线和混合大写。宁可简洁，因为每个使用你的包的人都会键入这个名字。不用担心名字冲突。包名只是用于导入的默认名，它不需要在所有代码中唯一，在极少情况下发生包名冲突，导入包可以局部地使用其他名字。任何时候混淆的情况都是极少会出现，因为导入的文件名决定了正在使用哪个包。

另一个约定是包名应该为源代码目录的基本名字，在`src/encoding/base64`目录里的包导入时写成`encoding/base64`，但名字是`base64`，而不是`encoding_base64`和`encodingBase64`。

包的导入者会通过包名来引用其内容，所以包中可导出的名字可以利用这一点来避免啰嗦。比如，在bufio中带缓冲的reader类型叫做`Reader`而不是`BufReader`，因为使用者看到的是`bufio.Reader`，一个清晰简洁的名字。并且因为导入的项总是用它们的包名来确定，所以`bufio.Reader`不会和`io.Reader`产生冲突。类似地用于新建`ring.Ring`实例的函数本来应叫`NewRing`，但由于`Ring`是该包导出的唯一类型，且该包叫`ring`，所以只要叫`New`即可，外部看到的就是`ring.New`。使用包结构可以帮助您选择好的名称。

另一个使用简短名字的例子是`once.Do`，`once.Do(setup)`读起来就很好，写成`once.DoOrWaitUntilDone(setup)`并没有任何改进，长名字不会提高可读性。一个有用的文档注释往往比长名字提供更多的价值。

## Getters

Go没有提供Getters和Setters的自动支持。你可以自己提供Getter和Setter，而且通常这样做更适合。但是把Get放到getter的名称前既不是习惯用法也很没有必要。如果你有一个字段叫`owner`，getter方法应该叫`Owner`而不是`GetOwner`。大写名字作为可导出的，使得字段和方法能够区分开来。如有必要提供Setter函数，应叫`SetOwner`。这两个名字在实践中都很易于阅读：
```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

## 接口命名

只有一个方法的接口命名，习惯上是方法名加上er后缀，或用类似的修改去构造一个代理名词：Reader,Writer, Formatter, CloseNotifier等。

`Read, Write, Close, Flush, String`等名字具有通用的签名和含义，为了避免混淆，不要给你的方法起这些名字，除非它也有相同的签名和含义。反之如果你的类型实现了一个方法，该方法和已知类型上的方法含义相同，那么请给他起相同的名字和签名。比如你的字符串转换方法应该叫`String`而不是`ToString`。

## MixedCaps

最后，Go约定对于多个单词的名字使用`MixedCaps`或`mixedCaps`，而不是下划线。

> 译注：就是驼峰式写法

## 分号

和C一样，Go的正式语法使用分号作为语句的结束，但和C不同的是这些分号不用出现在代码中。词法分析器使用一个简单的规则在扫描的时候自动插入分号，所以输入文本(译注：源码)基本上没有分号。

这个规则是：如果新行之前的最后一个符号是一个标识符(包括像int和int64这样的单词)，一个数字常量或字符串常量，或是下面的符号之一：
```go
break continue fallthrough return ++ -- ) }
```

词法分析器总是会在它们之后插入分号。这个可以简要描述为：“如果新行位于可以结束语句的符号之后，则插入一个分号。”

分号也可以在右括号前省略，所以像下面的语句：
```go
go func() { for { dst <- <-src } }()
```

不需要分号。通常Go程序只在诸如for循环有分号，为了隔开初始化，条件判断，和继续等部分。另外在一行中写多个语句也需要分号。

分号插入规则的一个影响是你不能把控制结构(if, for switch or select)的左括号写到下一行。如果你这么做，一个分号将插到左括号之前，这样将引起不想要的后果。要像下面这样写：
```go
if i < f() {
    g()
}
```
而不是像这样：
```go
if i < f()  // wrong!
{           // wrong!
    g()
}
```

> 译注：事实上编译器也不允许这么做。

## 控件结构

Go的控制结构和C相似但也有一些重要的不同：Go没有do和while循环，只有一个更通用的for；switch更加灵活；if和switch像for一样有一个可选的初始化语句；break和continue语句有一个可选的标签用于指示跳出或继续的内容。另外还有一些新的控制结构，包括类型switch和一个多路通信复用的select。控制结构的语法也稍有不同，它们没有圆括号，但正文部分必须用大括号括起来。

### If

在Go中一个简单的if语句看起来像这样：
```go
if x > 0 {
    return y
}
```

强制的大括号促使你将简单的if语句写成多行，但这不管怎么样都是一个好的风格，特别是正文中包含return或break控制语句的时候。

因为if和switch允许一个可选的初始化语句，所以用它来设置一个局部变量是一个很常见的用法：
```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在Go的库中，你会发现当一个if语句不会进入下一条语句时(即指if块结束于break, continue, goto或return)else块也会被省略：
```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

下面是一个常见的例子：代码必须防范一系列的错误条件。既然错误情况往往以return语句结束，那么最终的代码就不需要使用else语句了：
```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### 重复声明和重复赋值

题外话：上一节中的例子演示了短声明 := 如何工作。调用os.Open的声明为：
```go
f, err := os.Open(name)
```

这个语句声明了两个变量：f和err。接着几行之后调用f.Stat：
```go
d, err := f.Stat()
```

它看起来好像声明了d和err。但是请注意err出现在两个语句中。这个重复出现是合法的：err在第一个语句中是声明，而第二个语句只是重新赋值。这表示f.Stat的调用只是使用了已存在的err，并给它赋了一个新值而已。

在一个 := 声明中可以出现已经声明过的变量v，条件是：

- 这个声明和已存在的v声明处于同个作用域(如果v声明在外部的作用域，本声明则会重新创建一个变量v)。
- 在初始化时对应的值可以赋值给v，并且
- 这个声明中至少有另外一个变量是新的声明。

这个特性纯粹就是实用主义，它使我们可以方便地只使用一个err值，比如在一个长的if-else链中你会经常看到它们。

这里值得一提的是，Go函数的参数和返回值的作用域与函数体是相同的，尽管它们在语法上是在函数体的大括号之外。

### For

Go的for循环和C类似，但也不完全相同。它统一了for和while，并且没有do-while语句。for有三种形式，其中只有一种有分号：
```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

简短声明使我们很容易在循环中声明索引变量：
```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

如果你要遍历一个array, slice, string或map，或从一个channel读取值。range语句可以处理循环：
```go
for key, value := range oldMap {
    newMap[key] = value
}
```

如果你只需要第1个值(key或index)，去掉第2个值即可：
```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果你只需要第2个值(value)，可以对第1个值使用空白标识符(一个下划线)从而丢弃掉它：
```go
sum := 0
for _, value := range array {
    sum += value
}
```

空白标识符可以用在很多地方，[后面的章节](#空白标识符)会描述。

对于字符串，range做了更多的工作，它通过解析UTF-8编码将字符串分成一个个unicode码点。错误的编码会占用一个字节，并产生替代字符U+FFFD(rune是Go的专用术语，用于代表一个unicode码点，详细请看[the language specification](https://golang.org/ref/spec#Rune_literals))。下面的循环：
```go
for pos, char := range "日本\x80語" { // \x80 是一个合法的utf-8编码
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

将打印：
```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

最后，Go没有逗号操作符，并且++和--是语句而不是表达式。因此如果你想在for循环中使用多个变量，你只能使用多重赋值(这样就排除了++和--的使用)
```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Switch

Go的switch比C更通用。它的表达式不必是常量或整型。case会从上而下判断直到找到一个匹配项为止，如果switch没有表达式时它总是为true。因而可以用switch来表达if-else-if-else语句，这也是符合习惯的：
```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

case不会自动向下坠落(fall through)，可以用逗号分隔来列举情况：
```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

尽管它们在Go中不像其他类C语言那样常见，但break语句可以提前终止switch，有时候它可能还需要跳出外部的for循环，在Go中可以在循环之上写一个标签，然后break到这个标签去。下面例子展示二者的用法：
```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break  // 跳出switch
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop  // 注意是跳出for循环
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然continue也可以带一个标签，不过它只能用于循环。

作为这一节的结束，下面这个比较函数使用两个switch语句来比较两个slices：
```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### 类型Switch

switch也可以用于发现接口变量的动态类型。这个类型switch使用类型断言的语法：关键字type在圆括号中。如果在switch表达式中声明一个变量，这个变量将在每个从句中拥有相应的类型。这种情况下重用变量名是可以的，它实际上是在每个case下声明一个同名但不同类型的新变量。
```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

## 函数

### 多返回值

Go中有一个与众不同的特性是函数可以返回多个值。这种形式可以用来改进C程序中的一些笨拙的习惯用法：返回错误值(例子-1表示EOF)，并修改传递地址的参数。

在C中，write错误由一个负数表示，错误码隐藏在某个易丢失的地方。而在Go中，Write可以返回一个计数和一个错误：“是的，你写了一些字节，但不是全部，因为你填满了设备”。os包中的文件写方法是这样的：
```go
func (file *File) Write(b []byte) (n int, err error)
```

正如文档所述，它返回已写入的字节数，当n != len(b)时还会返回一个非空的error。这是一种通用的风格；可以看[错误处理](#错误)章节获得更多的示例。

这种方式避免了通过传递指针参数来模拟引用参数的需要。下面是一个简单的函数，它从一个byte slice获得一个数字并返回这个数字和下一个位置。

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

你可以使用它扫描slice b中的数字，像这样：
```go
for i := 0; i < len(b); {
    x, i = nextInt(b, i)
    fmt.Println(x)
}
```

### 命名的返回结果

可以为Go函数中的返回值(或叫结果参数)起一个名字，并且可以像普通变量一样使用它们，它们看起来就像是传进来的参数一样。当函数开始执行时这些结果参数会初始化为相关类型的零值。如果一个函数执行return语句时没有带任何返回值，这些已命名的结果参数就会作为返回值。

命名并不是强制的，但它可以使代码更简短清晰：它们就是文档。如果我们对nextInt的返回结果命名，就能很明显看出哪个int代表的什么：
```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名的结果参数已经初始化并且绑定到无参数的返回值，它们可使程序即简单又清晰。下面io.ReadFull就是一个很好的例子：
```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer

Go的defer语句会在函数返回之前，调度一个函数(延迟函数)立即运行。这是一种独特但很有用的方法，比如用于处理不管函数从哪个路径返回，资源必须释放这种情况。典型的例子是解锁一个互斥锁或者关闭一个文件。
```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

延迟调用一个函数(如上面的f.Close)有两个优点。首先它确保你不会忘记关闭文件； 其次它的关闭紧跟着打开，这比把它放在函数结尾更清晰。

延迟函数(如果函数是一个方法则包含一个接受者)的参数会在defer执行的时候求值，而不是在延迟函数被调用时求值。除了避免在延迟函数执行时改变变量值之外，还意味着单个延迟调用可以延迟多个函数的执行。下面是一个简单的例子：
```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

延迟函数是后进先出的顺序，所以上面的代码在函数返回时打印的是4 3 2 1 0。一个更实际的例子是通过程序跟踪函数执行，我们可以像这样编写两个简单的跟踪例程：
```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

其实我们还可以做得更好，利用延迟函数的参数是在defer执行时求值的这个特点。tracing例程可以返回untracing需要的参数。像下面的例子：
```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

打印：
```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯于其他语言块级的资源管理方式的程序员来说，defer似乎很奇怪。但它最有趣、最强大的应用恰恰来自这样一个事实: 它不是基于块的，而是基于函数的。在[Panic](#Panic)和[Recover](#Recover)章节中，我们会看到另一个例子。

## 数据

### 使用new分配

Go有两个分配原语：即内置函数new和make。它们做的事情不大一样，应用的类型也不一样，却很容易混淆，但其规则其实很简单。让我们先来说说new。它是一个用于分配内存的内置函数，不同于其他语言的new，它不初始化内存，它仅仅使之设为零值。也就是说new(T)会分配一个类型为T的零存储(内存为零值)并返回它的地址：也就是一个类型为*T的值。用Go的术语讲，它返回一个指针，指向新分配的类型为T的零值。

由于new返回的内存是清零的，在设计数据结构时每种类型的零值无须进一步的初始化就能被使用。这表示该数据结构的使用者可以使用new创建一个实例然后马上让它工作。比如bytes.Buffer的文档指出：“Buffer的零值是一个可用的空Buffer”。同样的sync.Mutex没有显式的构建函数或Init方法，一个sync.Mutex的零值即定义为一个未锁的互斥量。“零值”属性具有传递性，考虑下面类型的声明：
```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

SyncedBuffer类型的值只要在分配或者声明之后，即可马上使用。在下面的代码片段中p和v都能正常工作，并不需要进一步的处理：
```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### 构造函数和复合字面量

有时候零值还不够好，还需要一个构建函数，像下面来自os包的例子：
```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

上面的代码有些啰嗦，我们可以使用复合字面量简化它，复合字面量是一个表达式，它在每次求值时创建一个新实例。
```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

注意和C不同的是返回一个局部变量的地址是完全没问题的。和变量关联的存储在函数返回后仍然有效。实际上每次带着复合字面量的地址求值时，都会分配一个新实例，所以我们可以合并上面两个行为：
```go
return &File{fd, name, nil, 0}
```

复合字面量里的字段需要依次排列并且必须全部出现。但是通过显式的将元素标记为field:value对，字段的初始化就可以任意顺序，而且对于缺失的字段会赋予零值。因而我们可以这样写：
```go
return &File{fd: fd, name: name}
```

极端情况下如果一个复合字面量没有包含任何字段，那它会创建这个类型的零值，所以new(File)和&File{}是等价的。

复合字面量也可以用于创建arrays, slices和maps，其字段名根据创建的类型可能是索引或键值。在下面这些例子中，无论Enone、Eio和Einval的值是什么，只要他们是不同的，初始化都可以成功：
```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

> 译注：实际上对于array和slice来说，Enone, Eio这些必须是非负的整型常量才行。

### 使用make分配

回到分配这个话题上来，内置函数make(T, args)和new(T)的用途不同。它仅可以创建slices, maps和channels，并且它返回一个已初始化(不是零值)的类型为T的值(不是*T)。有此区别的原因是这三个类型背后的数据结构在使用之前必须初始化。比如一个slice，它包含了三个成员：一个指向内部数组的指针，以及一个长度和一个容量。在这些成员初始化之前，slice为nil。对于slices, maps和channels来说make会初始化它们内部的数据结构。比如：
```go
make([]int, 10, 100)
```

这一句分配了一个具有100个int的数组，然后创建一个slice，长度为10，容量为100，它指向内部数组的前面10个元素。(创建一个slice时，容量参数可以忽略；具体请看[slices](#Slices)章节)。相比之下，new([]int)返回一个指针，指向新分配的零值slice结构，也就是一个指向nil的slice指针。

通过下面例子可以阐明new和make的区别：
```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

请记住make只能用于maps, slices和channels，并且不是返回一个指针。要获得指针，请使用new或显式地取变量的地址。

### Arrays

数组在规划内存的详细布局时很有用，有时候可以帮助避免过多的分配内存。但最主要的是它是slice的基础(下节的主题)，为了给后面的主题打好基础，下面是关于数组的一些要点。

Go的数组和C的数组主要的区别是，在Go中：

- 数组是一个值，将一个数组赋值给另一个，将拷贝所有的元素。
- 特别是如果你传递一个数组给函数，函数接收到的是数组的拷贝，而不是一个指向它的指针。
- 数组的大小是数组类型的一部分，所以[10]int 和 [20]int 是不同的类型。

数组是值的属性虽然很有用但也代价昂贵；如果你想要有和C类似的行为而且还要保证效率，那么可以传递数组的指针：
```c
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

但这种风格不是Go的习惯用法，你应该使用slice。

### Slices

Slices通过对数组的封装，为数据序列提供更通用、更强大、更方便的接口。除了像变换矩阵那样有明确维数的情况之外，大部分数组的编程都是使用slices来完成(而不是简单的arrays)。

Slices引用着底层的数组，如果你将一个slice赋值给另一个，它们都会引用同一个数组。如果一个函数带着slice参数，改变slice的元素将会被调用者(caller)看到，就像传递底层数组的指针一样。下面是os包中File类型的Read方法的签名，它接受一个slice参数(而不是一个指针)和一个计数；slice的长度指定了要读的上限：
```
func (f *File) Read(buf []byte) (n int, err error)
```

这个方法返回读取的大小和一个错误值。如果想读前面32个字节到一个大的buffer中，可对buffer进行切片：
```go
n, err := f.Read(buf[0:32])
```

这样的切片是通用和高效的。实际上如果先把效率放在一边，下面的代码也可以读前面32个字节到buffer中。
```go
var n int
var err error
for i := 0; i < 32; i++ {
    nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
    n += nbytes
    if nbytes == 0 || e != nil {
        err = e
        break
    }
}
```

只要切片不超出底层数组的限制，它的长度就可以被改变。切片的容量通过内置函数cap获得，容量确定了slice的最大长度。下面是一个增加数据到切片的函数，如果数据超出容量，切片会被重新分配，函数返回增加后的切片。对一个nil slice调用len和cap是合法的并且会返回0，下面函数就利用了这一点：
```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

尽管Append可以修改slice里面的元素，但slice本身是通过值传递的(它是一个拥有指针，长度和容量的运行时结构)，所以必须在函数最后返回新的slice。

追加slice的功能是如此有用，它是从内置函数append抓取出来的。要理解append函数的设计，我们还需要一些额外的信息，后面我们再回过头来看这个函数。

### 二维Slices

Go的数组和切片是一维的。要创建一个等价的2D数组或切片，需要定义一个数组的数组，或切片的切片，像这样：
```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

因为切片长度可变，所以有可能里面的每个切片都拥有不同的长度。像这个LinesOfText的例子：每一行都有一个独立的长度：
```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时候需要分配一个2D的切片，例如处理像素的扫描行。有两种方法可以实现它，一种是独立的分配每一个切片；另一种是分配一个大的数组然后将各个切片指向其中。使用哪一种方法取决于你的应用。如果切片长度可能会增缩，那么应该独立的分配切片避免当前行覆盖掉下一行。否则，通过单次分配来构造对象会更高效。下面是这两种方法的参考。第一种：
```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

第二种是一次分配，然后切成多行：
```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Maps

Maps是一个方便且强大的内置数据结构，它关联了key和Value。Key可以是任意有定义等式运算符的类型，比如整型，浮点数和复数，数字串，指针和接口(只要动态类型支持等式)，结构和数组。切片不能作为Key，因为它们没有定义等式。maps会持有内部数据结构的引用，如果你传递一个map给一个函数，而该函数改变了map的内容，那么这个改变对于调用者是也可见的。

Maps可以使用冒号隔开键值对的复合字面量来构造，所以在初始时它们很容易构造：
```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

对map的赋值和获取从语法上看很像Arrays/Slices，只不过它的索引不局限于整型。
```go
offset := timeZone["EST"]
```

如果试图用一个不存在的Key去获取关联的值，Maps会返回值所对应类型的零值。比如Map的Value是整型，那么查找一个不存在的key将会返回0。可以通过将map的值设为bool型来实现集合类型。将一个值加入集合就是将map的相关项设为true，判断值是否在集合中就是简单的索引Map：
```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时候你想区分是项不存在还是值本来就为零值，可以用多重赋值来实现：
```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

这个用法被称为`comma ok`。在这上面的例子中，如果tz存在，seconds会得到期望的值，且ok将是true；如果不存在，seconds会被设为零值，且ok将是false。下面的if语句将该用法和一个错误报告放在一起：
```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

如果你只是只想知道项是否存在而不关心值是什么，可以使用空白标识符：
```go
_, present := timeZone[tz]
```

要删除map中的项可以使用delete内置函数，它的参数是一个map和要删除的key。即使key不存在也不会出错，所以delete函数是安全的：
```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### 打印

Go使用的格式化打印类似于C的printf函数族，但相比之下更丰富和更通用。这些函数在fmt包中，如： fmt.Printf, fmt.Fprintf, fmt.Sprintf等等。格式化字符串的函数(Sprintf等等)会返回一个最终的字符串而不是将结果填充在一个传入的Buffer中。

对应于每个Printf, Fprintf 和 Sprintf，还有另一组函数如Print 和 Println， 这些函数你不需要提供格式化字符串，它会对传进的参数使用默认的格式。Println会在参数之间插入空格且在输出之前加上换行符，而Print则只会加上空格。下面的例子将产生相同的输出：
```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

fmt.Fprint第1个参数接受任何实现io.Writer接口的对象。比如像os.Stdout和os.Stderr就是实现该接口的对象。

从下面开始就和C不大一样了。首先像%d这样的数字格式化并不指示参数的符号和大小，函数会根据参数的类型决定打印的结果：
```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印：
```go
18446744073709551615 ffffffffffffffff; -1 -1
```

如果你只是想要默认的输出，可以使用通用的%v(表示打印“值”)；Print和Println就是使用这个来打印结果的。并且该格式可以打印任意值，甚至像arrays, slices, structs和maps都可以。下面是打印上一节定义的时区map的语句：
```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

这会输出：
```go
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于maps， Printf等函数会按对key按字母排序进行打印。

当要打印一个结构时，使用%+v会连同字段的名字和值一起打印。%#v则会按Go的语法打印值的详细信息：
```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

将打印：
```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

使用%q会在打印string或[]type时自动加上双引号，使用%#q则会加上反引号(%q也可以用于integer和rune，会加上一个单引号并打印unicode字符)。%x可以打印strings, byte arrays和byte slices以及integers类型，它会输出一长串十六进制，如果x前加一个空格(% x)，那么它会在每个字节间插入空格。

另一个实用的格式是%T，它会打印值对应的类型：
```go
fmt.Printf("%T\n", timeZone)
```

会打印：
```go
map[string]int
```

如果你想控制自定义类型的打印格式，只要为这个类型加一个方法：String() string。比如对于类型T：
```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

会输出：
```
7/-2.35/"abc\tdef"
```

(如果你想像打印\*T的值一样打印T的值，那么String方法的接受者就必须是一个T值；上面例子使用T指针是因为这样更高效也更符合结构的习惯用法。请看下面章节[指针和值](#指针和值)获得更多信息)

有一个细节我们要注意一下：构造String方法时，要小心Sprintf内部会再次调用你的String方法，这样会造成死循环。比如Sprintf尝试直接打印接收者时，String方法又会被调用到：
```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

要修正这个也很容易：将参数强转成基本字符串类型即可：
```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

在[初始化](#初始化)章节中，我们将看到另一种避免死循环的技术。

另一个打印用法是将一个打印例程的参数直接传给另一个打印例程。Printf可以接受任意的类型的实参因为它使用interface{}作为可变形参：
```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

在Printf里面，v相当于一个[]interface{}类型的变量，但如果将它传给另一个可变参数的函数，它的作用就类似于常规的参数列表。下面是Println函数的实现，它将参数直接传给fmt.Sprintln：
```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

在v后面写...会告诉编译器将v看作一个参数列表传递。否则如果直接传v的话，v就会被当成是一个slice参数。

可以查看fmt包的文档以获得更多信息。

顺便提下，可变参数...后面可以是一个具体的类型，比如下面的Min函数，它会选择int列表中最小的值返回：
```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append

现在我们可以来说说上面遗留的append内置函数的设计思路了。append的签名和我们上面的自定义Append函数不一样，它是这样的：
```go
func append(slice []T, elements ...T) []T
```

T只是任意类型的一个占位符。你自己是不能写一个参数类型由调用者决定的的函数的，这就是为什么append是内置函数，它需要编译器的支持才能实现。

append做的事情是将elements追加到slice后面并返回新的slice。这个结果slice必须要返回的，原因就像我们写的自定义Append一样，其内部的数组可能会被改变。下面的例子：
```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

会打印[1 2 3 4 5 6]，所以append有点像Printf一样可以收集任意数量的参数。

如果我们想像上面那个Append一样在slice后追加一个slice呢？方法是使用...将参数展开即可，就像上面的Output做的那样。下面片段产生和上面一样的输出：
```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

如果没有...将不能编译通过，因为y不是一个int类型。

## 初始化

尽管从表面上看，和C/C++的初始化没有太大的不同，但Go的初始化更强大。初始化期间可以构造复杂的结构，并且可以正确处理初始化对象(即使在不同的包)之间的顺序问题。

### 常量

常量在Go中就是不变量：它们只会在编译期间创建(即使它们可以像本地变量一样定义在函数中)；并且只能是数字，字符，字符串和布尔值。因为编译期这个限制，定义常量的表达式必须是能够在编译期间求值的常量表达式。比如：1<<3 是一个常量表达式，而math.Sin(math.Pi/4)不是，因为函数调用只能运行期执行。

在GO中，可以使用iota枚举器来创建枚举常量。iota可以是表达式的一部分且能够隐式的重复，所以它很容易构造出一系列的复杂值：
```go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

> 译注：上面iota会一直递增，表达式就变成1 << (10 * 1), 1 << (10 * 2)。。。

由于可以给自定义类型附加方法(比如String)，这使得任意值在打印时可以格式化自己。尽管你会发现这个功能主要应用在结构上，但像浮点数这样的标量类型也很有用，像下面的ByteSize：
```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式YB打印为1.00YB，而ByteSize(1e13)打印为0.09TB。

这里ByteSize的String方法使用Sprintf是安全的(不会死循环)，因为它用%f调用Sprintf，这不是一个字符串格式化：Sprintf只会在想要一个字符串时才会调用String方法，而%f是想要一个浮点数。

### 变量

变量可以像常量那样初始化，但初始化器可以是一个在运行期才求值的表达式：
```go
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### init函数

最后，每个源代码文件可以自己定义一个的无参数init函数，用于设置一些必要的状态(实际上每个代码文件可以有多个init函数)。init函数只会在包中所有变量(最外层的变量)求值完毕之后才调用，而这些变量的求值会在所有import进来的包初始化完之后才进行。

init函数一个常见用途是在程序真正执行之前检查或修复程序的状态。
```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## 方法

### 指针和值

就像我们在ByteSize看到那样，可以为任何命名的类型定义方法(除了指针和接口)；接收者不一定是结构。

在上面讨论slices的时候，我们写了一个Append函数。我们可以把它定义为slice的一个方法。要做到这一点，首先定义一个命名的类型，然后使该方法的接收者是该类型的值：
```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

这仍然要求方法返回更新后的slice，但我们可以消除这个不便，只要重新定义方法使接收者为ByteSice的指针类型，这样就能改写调用者的slice:
```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

事实上我们还可以做得更好，如果把我们的函数修改得像标准的Writer方法一样：
```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

这样*ByteSlice就能满足标准的io.Writer接口，我们就可以使用Fprintf向它打印内容：
```go
var b ByteSlice
fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们需要传递ByteSlice的地址，因为只有*ByteSlice满足io.Writer。方法接收者是指针还是值的区别是：值方法可以被值变量和指针变量调用，但指针方法只能被指针变量调用。

之所以有这个规则是因为指针方法可以修改接收者；如果一个值可以调用指针方法，方法接收到的是值的一个副本，这样对这个接收者的任何修改都会被丢弃掉。所以Go不充许出现这个错误。不过，有一个方便的例外，当值为可寻址时，Go在值调用指针方法时，会自动对值插入寻址操作符(&)。在我们的例子中，b是可寻地址的，所以可以这样调用b.Write。编译器将会重写为(&b).Write。

顺便提下在一个字节切片上使用Write的思路是bytes.Buffer实现的核心。

## 接口和其他类型

### 接口

在Go中接口提供了一个指定对象行为的方法：如果某些东西能做这个，那么它就能在这里使用。我们已经看过几个简单的例子；自定义打印器可以通过String方法实现；而Fprintf使用Write方法可以把格式化字符串输出到任何对象。只带有一两个方法的接口在Go中很常见，并且其名通常来自它的方法。比如io.Writer就是那些实现了Writer方法的对象的接口。

一个类型可以实现多个接口。比如，一个列表如果实现了sort.Interface接口就可以被sort包里的例程排序；sort.Interface包含了Len(), Less(i, j int) bool, 和 Swap(i, j int)方法。其次，它也可以有一个自定义的格式化(通过实现fmt.Stringer接口的String方法)。

在下面例子中，Sequence 同时满足这两个条件。
```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### 转换

Sequence的String方法重复做了Sprint已经为slice做过的事情(它的复杂度还是O(N2)，非常糟糕)。如果我们在调用Sprint之前先把Sequence转换成[]int，就能分担掉这些工作量(并提高性能)。
```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

这个方法也是另一个从String方法安全调用Sprintf的例子。因为两个类型(Sequence和[]int)是相同的(如果我们忽略类型名)，所以他们的转换是合法的。这个转换不会创建一个新值，它只是临时地把现有值当成新的类型。（另外像整型转换成浮点数这些会创建新值)

在Go程序中转换表达式的类型以便访问不同的方法集是很常见的。我们可以使用已存在的类型sort.IntSlice去精减整个例子：
```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

>译注，精减后不需要再实现sort.Interface接口了。

现在Sequence不用实现多个接口(排序和打印)，我们将数据项转换成多种类型(Sequence, sort.Interface和[]int)，每次转换都完成掉一部分工作。这在实践中不多见，却很有效。

### 接口转换和类型断言

[类型Switch](#类型Switch)是转换的一种形式：它们接受一个接口，对于switch中的每个case，它将接口转换成这个case指定的类型。下面是fmt.Printf如何使用类型switch将一个值转成字符串的简化版。如果值本来就是字符串，那它就是我们想要的字符串；否则如果值有定义String方法，那么会调用它得到我们想要的结果。
```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

第一个case转换成一个具体的值；第二个case将inteface转换成另一个interface。对混合类型来说这种方式非常好。

如果我们只关注其中一个类型要怎么做呢？如果我们知道这个值包含一个字符串，我们只想提取它呢? 可以使用只有一个case的类型switch，也可以使用类型断言(type assert)。类型断言接受一个接口值，并从中提取指定类型的值。它的语法借用了类型switch开头的语句，但使用一个显式的类型而不是type关键字：
```go
value.(typeName)
```

其结果是一个typeName类型的新值。typeName必须是由接口引用的具体类型，或是值能转换到的第二个接口类型。要从值中提取字符串，我们可以写：
```go
str := value.(string)
```

如果value不能转换成字符串，程序就会崩溃于一个运行时错误。为了防止这种情况发生，可以使用"comma, ok"的写法安全地测试值是否是一个字符串。
```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

如果类型断言失败，str仍然会存在且还是一个string类型，但它将是一个零值，一个空字符串。

下例有一个if-else语句，它等价于本节开头的那个类型switch:
```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### 通用性

如果一个类型的存在仅仅是为了实现一个接口，并且永远不会导出这个接口之外的其他方法，那么这个类型本身也没有必要导出。只导出接口就可以使我们只关注其行为而非其实现。并且具有不同属性的其他实现可以反映原始类型的行为。同时它也避免了为拥有共同方法的每个实例上编写重复文档的需要。

在这种情况下，构造函数应该返回一个接口值而不是实现的类型。比如在哈唏库里面，crc32.NewIEEE和adler32.New都返回hash.Hash32接口。如果程序中将CRC-32算法替换为Adler-32算法，只需要修改构造函数就可以了，后面的代码不会因更换算法而受到影响。

同样的方式允许将多种加密包中的流加密算法(streaming cipher algorithms)与它们链接在一起的分组加密( block ciphers )隔离开。在crypto/cipher包中的Block接口定义了分组加密的行为：为单个数据块加密。然后类似于bufio包，实现该接口的加密包可以用于构造流加密，流加密由Stream接口表示，流加密并不用知道分组加密的细节。

crypto/cipher的接口像这样：
```go
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

下面是CTR流(counter mode stream)的定义，它将分组加密转换为流加密，注意分组加密的细节被抽象掉了：
```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

NewCRT不仅适用于一种特定的加密算法和数据源，也适用于任何Block接口的和任何Stream接口的实现，因为它返回的是接口值。要替换CRT加密的为其他加密算法只是一个局限的修改。调用构造函数的地方需要修改，但周围的代码只知道Stream接口，所以它们不会察觉到不同。

### 接口和方法

几乎任何类型都可以附加方法，所以几乎任何类型可以满足一个接口。像http包，它定义了Handler接口，任何实现了Handler的对象都可以处理HTTP请求。
```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

ResponseWriter本身就是一个接口，它提供了响应客户端所需要的方法的访问，这些方法包含标准的Write方法，所以一个http.ResponseWriter可以用于所有io.Writer可以用的地方。Request是一个经过解析的客户端请求的结构。

为简单起见，我们忽略掉POST方法并认为请求总是用GET方法。这种简化并不会影响Handler的设置。下面是一个简单但完整的Handler实现，用于统计访问页面的次数。
```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

(如上所说http.ResponseWriter有一个Write方法，所以Fprintf可以将内容输出给它)，下面是如何将Counter附加到URL树节点的做法
```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但何必把Counter作为一个结构？其实一个整型就满足要求了。(方法接收者必须是一个指针，这样增加计数才能生效)：
```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

当访问页面时，如果你的程序有一些内部状态也需要被通知该怎么办？可将channel绑定到页面请求上：
```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想在/args页面上显示服务器程序启动时的系统参数要怎么做？很容易写一个函数打印这些参数：
```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

如何将这个变成HTTP服务？我们可以将ArgServer作为某种类型的方法，但还有另一个更清晰的方式：既然可以为任何类型(除了指针和接口)定义方法，那么我们可以为一个函数写一个方法。Http包里包含了一这样的代码：
```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

HandlerFunc是一个带ServeHTTP方法的类型，所以这个类型的值可以处理HTTP请求。看看这个方法的实现：接收者是一个函数f，方法会调用这个函数f。这个做法看起来有点奇怪，但是它与接收者是一个channel然后方法发送数据给channel这个做法并没有什么区别。

要将ArgServer转为一个HTTP服务，首先要修改它的函数签名：
```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

ArgServer现在具有和HandlerFunc一样的签名了，因此它可以转换成HandlerFunc然后调用它的方法。就像我们将Sequence转换为IntSlice然后调用IntSlice.Sort一样。现在代码比较简单了：
```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当有人访问/args页面时，安装在这个页面上的处理器有ArgServer值和HandlerFunc类型。HTTP服务器将调用HandlerFunc类型的ServeHTTP方法，ArgServer作为接收者，方法转而去调用ArgServer(经由HandlerFunc.ServeHTTP里面的f(w, req))。最终系统参数就将显示到页面上。

这一节中我们通过结构，整型，通道和函数制作了一个HTTP服务器。所有这些都是因为接口只是方法的集合，而(几乎)任何类型都可以定义方法。

## 空白标识符

在[for_range循环](#For)和[maps](#Maps)中我们已经多次提到空白标识符了。可以使用任何类型的任意值分配或声明空白标识符，并无害地丢弃该值。它有点像写到unix的/dev/null文件去：它代表一个作为变量占位符的只写(write-only)值，这个值从来不会用到。它还有很多用途我们没有看到。

### 多重赋值中的空白标识符

在for range循环中使用空白标识符是通用情况中的一种特殊情况，通用情况是：多重分配。

如果一个赋值需要多个值在左边，但其中一个值在程序中不会用到，那么在左边使用空白标识符就可以避免创建一个无用的变量，并且很清楚的表示这个值会被丢弃。比如要调用一个返回值和错误的函数，如果我们只关注错误，那么可以使用空白标识符接收那个不会用到的值：
```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

偶尔你也会看到一些代码为了忽略错误返回值而使用空白标识符，但这是一个糟糕的做法，应该总是检查返回的错误，函数返回它们是有原因的：
```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### 未用的import和变量

导入一个没用的包或声明一个没用的变量都是错误的。导入无用包会使程序膨胀和拖慢编译；而一个变量初始化后却未使用，至少是在浪费计算，并且可能暗示了一个大的BUG。然而一个程序正在开发时，未用的导入和变量经常发生，删除它们只是为了让编译通过，但烦人的是后面可能又要用到它们。空白标识符提供了一个解决方法。

下面是一个写到一半的程序，它有两个未用的导入(fmt和os)一个未用的变量(fd)，所以它将不能编译通过。但目前为止可以看到代码是正确的：
```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

为了消除未用导入包的编译错误，可以用空白标识符引用导入包中的符号。类似地，将未用变量fd赋值给空白标识符也能消除编译错误。下面这个版本就可以编译通过：
```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

通常为了消除导入错误的全局声明应该在导入包语句之后，并且对声明进行注释。这样可以方便找到它们，也可以提醒你稍后对它们进行清理。

### import的副作用

像前一个例子的fmt和os，未用的导入最终应该被使用或删除掉：空白赋值将代码标识为正在进行的工作。但有时导入一个未用包是很有用的，仅因为导入有一些副作用。比如，net/http/pprof包会在init函数期间注册HTTP处理器用于提供调试信息。它有一个导出的API，但大多数客户端这只是需要处理器的注册，然后可以通过网页去访问这些调试信息。为了仅使用导入包的副作用，可以将包重命名为空白标识符：
```go
import _ "net/http/pprof"
```

这种方式的导入清楚的表明导入这个包只是为了它的副作用，因为没有其他可能性去使用这个包：它没有名字(如果给包命名，而我们又没有使用包，那编译器不会编译通过)。

> 所谓副作用，说白了就是要使导入包的init函数得到调用。

### 接口检查

在上面讨论接口时我们看到的，一个类型不用显式地声明它实现了某个接口。相反，一个类型实现了一个接口仅是因为它实现了接口的方法。实践中大多数接口转换是静态的，因而会在编译期检查。比如，将\*os.File传递给一个期望io.Reader的函数将编译不通过，除非\*os.File 实现了io.Reader接口的方法。

有一些接口检查也会发生在运行期。一个例子是在encoding/json包中定义了一个 Marshaler接口。当JSON编码器接收到一个实现了该接口的值时，编码器不会调用标准的转换，转而调用该值的marshaling方法将值转换为JSON。编码器在运行期使用类型断言来判断值是否实现了接口：
```go
m, ok := val.(json.Marshaler)
```

如果仅仅为了判断类型是否实现了接口而并不使用该接口，可以使用空白标识符忽略掉类型断言的值：
```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

如果一个类型--比如json.RawMessage--需要一个自定义的JSON显示，它就必须实现json.Marshaler接口。但这里没有静态转换能让编译自动去检查它是否实现了该接口。如果类型不小心不满足该接口，JSON编码器虽然还可以工作，但它就没法使用自定义的实现了。为了保证自定义实现的正确性，可以在包中使用空白标识符作一个全局声明：
```go
var _ json.Marshaler = (*RawMessage)(nil)
```

这个声明中，赋值会引起\*RawMessage到Marshaler的转换，这要求\*RawMessage实现Marshaler接口，在编译期间会检查这个属性。如果json.Marshaler接口修改了，这个包将不能编译，然后我们就知道需要修改\*RawMessage。

上面空白标识符的出现表明此声明仅用于类型检查，而不用于创建变量。不要对满足接口的每种类型作此操作。通常这种声明只有在代码中不存在静态转换的情况下才使用，这种情况是很少见的。

## 内嵌

Go没有提供典型的类型驱动的子类化概念，但它通过在结构或接口内嵌类型，也能提供一部分这种能力。

接口内嵌非常简单。之前我们有提过io.Reader和io.Writer接口，它们的定义是这样的：
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

io包也导出了另外几个接口，特定对象可以实现这些接口的方法。比如，有一个io.ReadWriter接口包含了Read和Write。我们当然可以让io.ReadWriter列出这两个方法，但更简单的做法是让两个接口内嵌到新接口中，像这样：
```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

就像看到的那样：一个ReadWriter可以做Reader和Writer做的事情，它是两个内嵌接口的并集(这两个接口的方法集必须不相交)。只有接口可以内嵌到接口中。

同样的基本思路也适用于结构，但其意义更深远。bufio包有两个结构类型：bufio.Reader和 bufio.Writer，每一个都实现了io包中相应的接口(io.Reader和io.Writer)。bufio还实现了一个带缓冲的reader/writer，它是通过将reader和writer结构内嵌到另一个结构来实现的：结构内直接列出两个类型而不给他们字段名：
```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

内嵌元素是指向结构的指针，在使用之前他们需要先初始化以指向可用的结构。ReadWriter当然也可以像这样写：
```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但这样的话为了满足io的接口，我们还需要提供转发方法，像这样：
```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接内嵌结构，我们就可以避免这种流水账。内嵌类型的方法是“免费提供”的，这个意思是 bufio.ReadWriter不仅仅拥有bufio.Reader和bufio.Writer的方法，同时还满足了三个接口：io.Reader, io.Writer, 和 io.ReadWriter。

这是内嵌类型和子类化的一个重要区别。当我们内嵌一个类型时，这个类型的方法也变成外部类型的方法，但当方法被调用时，接收者还是内部类型而不是外部类型。在我们的例子中，当bufio.ReadWiter的Read方法被调用时，它实际上和上面写的转发方法的效果是一样的：接收者是ReadWriter的reader字段，而不是ReadWriter自己。

下面例子显示Job拥有一个内嵌字段和一个普通字段：
```go
type Job struct {
    Command string
    *log.Logger
}
```

Job类型现在有了*log.Logger的Print, Printf, Println等方法。我们也可以给Logger一个字段名，但其实并不需要这样做。一旦初始化完我们就可以使用Job打印日志：
```go
job.Println("starting now...")
```

Logger是Job的一个字段，我们可以在Job的构造函数中按正常方式初始化它，像这样：
```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或者写一个复合字面量：
```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用内嵌字段，该字段的类型名(不用带包名)即可作为字段名。在这里，如果我们需要在一个Job类型的变量上访问*log.Logger，可以写成job.Logger。这种写法的一个用途是重定义Logger的方法：
```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

内嵌类型会引入名字冲突的问题，但解决它的方法也很简单：首先，外部结构的字段或方法X会隐藏掉内嵌结构的同名字段或方法，如果log.Logger也包含了一个字段或方法叫Command，那Job的Command会覆盖它。

其次，如果同一个名字出现在相同的内嵌级别中，这通常会引起一个错误：比如假设Job有一个正常字段叫Logger，此时再去内嵌log.Logger到Job就会发生错误。然而，若重名永远不会在类型定义之外的地方使用，那就不会出错。这种限定能在外部修改内嵌类型时提供某种保护。因此就算添加的字段和另一个子类型的字段相冲突，只要这两个字段永远不会被使用，那就没有问题。

## 并发

### 通过通信共享

并发编程是一个很大的主题，这里只提及一些特定于go的亮点。

在许多环境中，因为要实现正确的访问共享变量这个微妙的需求，并发编程变得很困难。Go鼓励使用另一种方法：共享值通过channels来传递，而不会让各个执行线程主动共享(译注：即不通过线程的同步设施来共享)。任意时间只一个协程可以访问这个共享值。这样的设计避免了数据竞争的发生。为了鼓励这种思考方式，我们把它简化为一句口号:

    不要通过共享内存通信，取而代之，通过通信来共享内存

这个方法可能有点极端，比如引用计数可以通过在一个整型变量的上下放一个互斥量来实现。但作为一个高层次的方法，使用channels去控制访问会更容易写出清晰正确的程序。

我们可以通过一个运行在单CPU上的典型的单线程程序来思考这个模型。它不需要同步原语。 现在再运行一个这样的实例，它也同样不用同步原语。现在让它们通信起来，如果通信本身就是一个同步器，它们也仍然不需要其他的同步设施。比如unix管道就非常适合这个模型。尽管Go的并发方法来源于Hoare的Communicating Sequential Processes (CSP)，但它也可以看作是一个类型安全的Unix管道的泛化。

### Goroutines

它们被称为goroutines，是因为现有的术语——线程、协程、进程等等——无法准确表达它的含义。一个goroutine有一个简单的模型：它是在同一个地址空间中和其他goroutine一起并发执行的函数。它是轻量的，只有一点成本花费在栈空间的分配上。栈一开始很小，所以它们很节省，后面需要的时候再从堆分配(或释放)空间。

goroutines会多路复用到多个OS线程上，因此如果其中一个阻塞，比如在等待I/O时，其他协程会继续运行。它们的设计隐藏了很多创建线程和管理的复杂性。

在一个函数或方法前加上go关键字，会使它运行在一个新的goroutine上。当函数调用执行完成时，goroutine会悄悄的退出。(这个效果类似于Unix shell的&符号，用于在后台运行命令)
```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

在一个协程调用中使用函数字面量非常方便：
```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

在Go中，函数字面量是一个闭包：它确保函数所引用的变量在函数为活动状态时仍然存在。

上面这个示例不太实用，因为函数没有通知其完成的方式，为此我们需要通道。

### Channels

像maps一样，channels通过make分配，且其结果值会引用到底层的数据结构。make函数有一个可选的整型参数，它设置channels的缓冲大小。默认是0，此时channels是一个无缓冲或同步的通道。
```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

无缓冲的channels会通信时会同步交换数据，它确保(两个goroutines)计算处于已知状态。

这里有很多使用channels的好习惯。我们先从下一个开始，在前一节我们在后台运行了一个排序。一个channel可以让启动goroutine(译注：就是运行排序协程的那个协程)等待排序结束。
```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

接收者总是阻塞着直到有数据可接收。如果channel是无缓冲的，发送者会阻塞直到接收者接收数据。如果channel是有缓冲的，发送者会在数据成功拷贝进缓冲之后就返回；如果缓冲满了，那就会阻塞等待直到接收者接收数据，使缓冲区有空位出来。

一个带缓冲的channel可以像信号量那样使用，例如限制吞吐量。在下面例子中进来的请求传递给handle，handle会发送一个值给channel，然后处理请求，接着又从channel接收一个值为下一个handle准备好“信号量”。channel缓冲的容量会限制同时调用process的数量：
```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```

一旦MaxOutstanding个处理器在执行process, 后面的处理器尝试向一个满的channel缓冲发送数据就会阻塞住，直到前面的处理器完成了处理并从缓冲中取出数据，使缓冲变成未满。

这个设计有一个问题，试想：Serve为每个进来的请求创建一个新的协程，但同时只有最多MaxOutstanding个可以运行。如果进来的请求过快的话，最后会导致程序消耗尽有限的资源。我们可以修改Serve通过控制协程的创建来解决这一问题。下面是一个显而易见的解决方案，但请注意里面有一个BUG，随后我们会修正它：
```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

这个BUG是在go的for循环里，循环变量会在每次迭代中重用，所以req会被所有协程共享。这不是我们想要的效果。我们需要确保req对每个协程都是唯一的。下面是一种解决方法：在协程中将req作为参数传递给闭包：
```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

请比较一下这两个版本以了解声明和运行闭包的一些差异。另一个解决方法是创建一个新的同名变量：
```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

这个写法看起来很奇怪：
```go
req := req
```

但在Go中这是合法的也是惯用的。你得到一个同名的但是全新的变量，这样会在内部隐藏循环变量，但因此变量对每个协程就变成唯一的。

我们回到编写服务器的通用问题上，另一个好的管理资源的办法是启动固定数量的协程，它们都从请求channel上读取数据。协程的数量限定了同时调用process的数量。Serve函数也会接受一个通知它退出的channel；启动完那些协程之后，Serve会阻塞在接收这个channel上。
```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### ChannelsOfChannels

channel有一个非常重要的属性是它是一个first-class值，它可以像其他任何类型一个分配和传递。这个特性常用于实现安全的并行的多路分解。

前一节的例子中，handle是一个理想化的请求处理器，我们没有定义它处理的类型。假设这个类型包含了一个用于应答的channel，每个客户端可以为答案提供自己的路径。下面是Request的定义：
```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

客户端提供一个函数和参数，同时里面还有一个channel用于接收答案。
```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在服务器这边，处理器只须作如下修改：
```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

要让它变得更实用还需要做很多工作。但这是一个限流、并行、无阻塞的RPC系统框架，而且里面看不到任何互斥量。

### 并行

另一个例子是多核的并行计算，如果一个计算能分离成多个独立执行的片段，它就能并行化，一个channel用来通知每一个片段的完成。

假设我们要对一个向量的项上执行费时的操作，并且每一个项的操作都是独立的，如下面的例子：
```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

我们可以在循环中启动多个独立的处理片段，每个CPU一个。它们可以以任意顺序完成。我们只需要在启动完所有的协程之后，通过提取完通道的数据来统计完成信号。

```go
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

我们可以向runtime查询CPU的数量，而不是通过一个常量去定义numCPU。这个函数[runtime.NumCPU](https://golang.org/pkg/runtime#NumCPU)返回机器中CPU核的数量，所以我们可以这样写：
```go
var numCPU = runtime.NumCPU()
```

还有一个函数[runtime.GOMAXPROCS](https://golang.org/pkg/runtime#GOMAXPROCS)，它报告(或设置)Go程序可以同时运行的用户定义的核心数。它默认是runtime.NumCPU返回的值，但是也可以通过起一个类似名字的Shell变量，或者直接调用这个函数来改写它。如果runtime.GOMAXPROCS传入的值是0，则意思是取值。因此若我们想满足用户的资源请求，我们应该这样写：
```go
var numCPU = runtime.GOMAXPROCS(0)
```

请一定不要搞混并发和并行的含义，并发是将程序构造为独立执行的组件，并行是在多个CPU同时执行运算以提高效率。尽管Go的并发特性使得一些问题很容易被构造成并行运算，但Go是一个并发语言，而不是并行，且并不是所有的并行问题都适用于Go的模型。关于并发和并行的区别，请参阅[这篇博客](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)

### 泄漏的缓冲区

并发编程的工具甚至可以使得非并发的思想更易于表达。下面是从一个RPC包中抽象出来的例子。客户端协程循环从一些源(比如网络)接收数据。为了避免频繁分配和释放缓冲区，它要维护一个freelist，这个freelist使用一个带缓冲的channels来表示。如果通道为空，则分配一个新的Buffer。一旦消息Buffer准备好了，它就通过serverChan发送给服务器。
```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

服务器循环从客户端接收消息，处理它，然后将buffer返回给freelist。
```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

客户端尝试从freelist接收缓冲；如果没有就分配一个新的。服务器处理完会把b压回给freelist，但如果freelist满了的话buffer会被丢弃，随后被GC回收掉。(select语句中的default块会在其他case不满足时执行，这使得select不会阻塞)，这个实现仅用几行代码就构建了一个无泄漏的freelist，它依赖于带缓冲的channel和GC来进行管理。

## 错误

库函数必须向调用者返回某些错误提示。如早前所说，Go的多返回值很容易做到除了返回正常值之外再返回一个详细的错误描述。使用这个特性去提供细节错误信息是一个好的风格。比如，我们会看到os.Open失败时不仅返回一个nil，还会返回一个错误值描述哪里出错了。

按照惯例，错误为类型error，这是一个简单的内置接口：
```go
type error interface {
    Error() string
}
```

库作者可以自由的用各种丰富的底层模型来实现这个接口，使得它不仅可以看到错误信息，也可以提供一些上下文信息。如前所述，os.Open除了返回*os.File，也会返回一个错误值。如果文件成功打开，错误值是一个nil。但如果出现问题，错误值将是一个os.PathError。
```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

PathError的Error方法会产生像这样的字符串:
```sh
open /etc/passwx: no such file or directory
```

像这样的错误会包含有问题的文件名，相关的操作，以及触发的操作系统错误。既使打印信息和引起这个问题的调用相距很远。它也比单纯的“"no such file or directory"提供了更多的信息。

在可行的情况下，错误字符串应该标识它们的来源，例如使用前缀来命名产生错误的操作或包。比如，在image包中，一个字符串用于表示因未知格式而出现解码错误应该是："image: unknown format"。

调用者如果关注更详细的错误，它可使用类型switch或类型断言查找具体的错误并提取更多细节。对于PathErrors它可能包含了检查内部字段Err以进行错误恢复：
```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

上面第二个if语句是一个类型断言。如果断言失败，ok将为false，e将为nil。如果成功，ok将会是true，并且e将是一个*os.PathError类型。然后我们就可以用它来检查关于这个错误的更多信息。

### Panic

向调用者报告错误的常用做法是返回一个额外的error值。Read方法就是一个例子，它返回一个字节计数和一个error。但如果错误是不可恢复的呢？有时程序根本无法继续执行。

为此有一个内置函数panic，它会创建一个运行时错误，并停止程序的运行。这个函数接受一个参数，这个参数可以是任意值，但通常是一个字符串，用于在程序挂之前打印出来。调用painc也是一种提醒有些不可能的事情发生了的方式，比如退出一个死循环：
```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

上面仅仅是一个例子，但实际的库应该避免panic。如果问题可以被掩盖或绕过，让程序继续运行总是好过直接让程序停止。一个例外是在初始化的时候，如果库真的不能设置好自己，那它可能就有理由调用panic：
```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### Recover

panic的调用可能包含隐含的运行时错误，像slice索引超出边界，或类型断言失败。它会立即终止函数的运行，并开始展开协程的栈，延迟函数会在展开过程中执行。如果展开到达协程栈的顶端，程序就挂了。然而通过使用内置函数recover可以让你重新获得对协程的控制并且继续正常的执行。

revover的调用会停止栈展开并返回传给panic的参数。因为在展开时只有延迟函数里的代码可以执行，所以recover只有在延迟函数里才有用。

recover的一种应用是关闭错误的协程而不会杀死其他协程。
```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

这个例子里如果do(work)产生panic，错误将会输出并且该协程会退出，但不会干扰到其他协程。不需要在延迟闭包中执行任何其他的操作，recover会处理好这一切。

由于只有在延迟函数中调用recover才会返回非nil值。所以延迟代码中可以调用那些本身使用了panic和recover的库例程而不会失败。比如safelyDo中的延迟函数可以在recover之前调用日志函数，且日志代码将不受panicking状态的影响。

有了恢复模式，do函数(或者它调用的任何函数)可以通过简单地调用painc彻底摆脱各种错误情况。我们可以使用这个思路来简化复杂程序的错误处理。让我们看一个regexp包的理想化版本，它会调用painc并传递一个本地的错误类型来报告解析错误。下面是Error类型，error方法，以及Compile函数的定义：
```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果doParse产生panic，recover块会将函数返回值设为nil(延迟函数可以修改命名的返回值)。然后，在对err的赋值时断言e是不是本地类型Error，以此检查问题是不是解析错误。如果它不是解析错误，断言将失败，并引发一个运行时错误使得栈继续被展开，就好像没被中断过一样。这个检查意味着如果错误不符合期待，比如索引越界，那么代码仍然会失败。

有了错误处理，error方法(因为这个方法绑定了一个类型，所以它和内置函数error同名也没什么问题)可以很容易地报告解析错误，而不用担心需要手动解析展开的栈。
```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

尽管这个模式很有用，它也应该仅用于包内部。Parse将其内部的panic调用转换为error值，它并不会把panic暴露给外部。这是一个值得遵守的良好规则。

顺便说一句，如果实际错误发生，重新引起panic的用法会改变panic的错误值。但是原来的和新的失败都将显示在崩溃报告中。所以产生错误的根源仍然可以被看到。因此这种重新引起panic的方法通常都能满足需求(毕竟是崩溃了)。但如果你只想显示原始的错误，那你可以多写一点代码把其他的错误过滤掉，并重新调用panic传入原来的错误。这里就将这个练习留给读者了。

## 一个Web服务器

让我们用一个完整的Go程序来完成本篇，一个Web服务器。这实际是一种`web re-server`(译注：可以叫中转服务器，通过调用其他服务器的功能提供服务)。Google在`chart.apis.google.com`上提供了一个服务，可以用来把格式化数据转换成表格和图形。但是它很难交互的使用，因为你需要将数据作为一个query放入到URL中。这里的程序为一种数据形式提供了更友好的接口：给出一小段文本，它会调用`chart server`去产生QR码。QR码(二维码)是一个对文本进行编码的盒子矩阵，这个图像可以用手机摄像头捕获并解析，比如解析为一个URL。这可以解决你在手机的小键盘上输入URL的麻烦。

下面是一个完整的程序，随后会对其进行解释：
```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```

main之前的部分很容易理解，通过一个flag为服务器设置默认的端口。模板变量templ是真正有趣的地方。它构建了一个模板，这个模板将由服务器执行并显示到页面。

main函数解析flag，并使用我们上面介绍的机制：将函数QR绑定到服务器的根路径上。接着http.ListenAndServe用于启动服务器，并在服务器运行时一直阻塞着。

QR仅仅接收带有表单数据的请求，并对名为s的表单数据执行模板。

模板包html/template非常强大，这个程序只用到它的皮毛。本质上它通过替换传递给templ.Execute的数据项的元素(在本例中是表单值)，来即时重写一段HTML文本。在模板文本（templateStr）中，双括号分隔的部分表示模板动作。仅当当前数据项(表示为.)不为空时，从{{if.}}到{{end}}的片段才会执行。也就是说如果字符串为空，模板的这部分会忽略掉。

模板中的两个{{.}}表示将数据(即查询字符串)加到模板并显示在网页上。HTML模板包会自动对文本进行转义，所以文本可以安全地显示。

剩下的模板字符串只是网页加载时要显示HTML文本。如果觉得这里的说明太过简略，请参阅模板包的文档获得更全面的解释。

这就是结果：用少数代码和一些数据驱动的HTML文本就完成了一个有用的Web服务器。Go确实强大到可以用几行代码完成很多事情。

