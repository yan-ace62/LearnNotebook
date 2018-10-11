# Golang: The Laws of Reflection
## Introduction
Reflection in computing is the ability of a program to examine its own structure, particularly through types; it's a form of metaprogramming. It's also a great source of confusion

在计算机中，反射是程序检查自我结构的的一种能力,这种检测通常通过类型实现。这是一种元编程的形式，也常常导致很多困扰。

In this article we attempt to clarify things by explaining how reflection works in Go. Each language's reflection model is different (and many languages don't support it at all), but this article is about Go, so for the rest of this article the word "reflection" should be taken to mean "reflection in Go".

在这篇文章中,我们尝试通过解释Go语言中反射的工作方式来说明这些概念。每种语言的反射模型都不一样（很多语言甚至步支持反射机制)，但是这篇文章是关于Go,所以下面内容中，"反射“一词总是表示Go中的反射含义。

## 类型与接口
Because reflection builds on the type system, let's start with a refresher about types in Go.

因为反射机制是构建在类型系统之上，所以让我们重新认识Go中的类型。
Go is statically typed. Every variable has a static type, that is, exactly one type known and fixed at compile time: int, float32, *MyType, []byte, and so on. If we declare

Go是一种静态类型语言，每个变量拥有一个确切的静态类型且在编译阶段是固定不变的,像int, float32, *MyType, []byte等等。如果我们声明如下：
    
    type MyInt int
    var i int
    var j MyInt

then i has type int and j has type MyInt. The variables i and j have distinct static types and, although they have the same underlying type, they cannot be assigned to one another without a conversion.

那么i拥有类型int，j拥有类型MyInt。变量i与j是两种不同静态类型，尽管它们拥有相同的底层类型。它们之间不能相互赋值,如果没有进行类型转换。

One important category of type is interface types, which represent fixed sets of methods. An interface variable can store any concrete (non-interface) value as long as that value implements the interface's methods. A well-known pair of examples is io.Reader and io.Writer, the types Reader and Writer from the io package:

接口类型是一种非常重要的类型,它表示固定方法的集合。一个接口变量可以存储任何实现接口方法的具体对象。一个被大家众所周知的例子就是来io包中的io.Reader与io.Writer接口类型。
    
    // Reader is the interface that wraps the basic Read method
    type Reader interface {
        Read(p []byte) (n int, err error)
    }

    // Writer is the interface that wraps the basic Write method.
    type Writer interface {
        Write(p []byte) (n int, err error)
    }

Any type that implements a Read (or Write) method with this signature is said to implement io.Reader (or io.Writer). For the purposes of this discussion, that means that a variable of type io.Reader can hold any value whose type has a Read method:

任何实现了Read方法的的类型都可以说实现了io.Reader（或者io.Writer)接口。从讨论的角度来看，这就是说io.Reader变量能够存储任何实现Read方法类型的值。

    var r io.Reader
    r = os.Stdin
    r = bufio.NewReader(r)
    r = new(bytes.Buffer)
    // and so on

It's important to be clear that whatever concrete value r may hold, r's type is always io.Reader: Go is statically typed and the static type of r is io.Reader.

值得说明的是，r变量无论存储了任何具体值，它的类型总是io.Reader,因为Go是一种静态类型，所以r的静态类型是io.Reader。

An extremely important example of an interface type is the empty interface:

一个非常重要也是特殊的例子是空接口类型：

    interface{}

It represents the empty set of methods and is satisfied by any value at all, since any value has zero or more methods.

它表示一个空的方法集并且能够存储任何类型的值，因为任何类型都具有0或者多个方法。

Some people say that Go's interfaces are dynamically typed, but that is misleading. They are statically typed: a variable of interface type always has the same static type, and even though at run time the value stored in the interface variable may change type, that value will always satisfy the interface.

有些人说Go的interface是一种动态类型，这是一种误导性的说法。它们还是静态类型的：interface类型的变量总是拥有同样的静态类型，即使在运行时保存在该变量中的值可能改变类型，并且这类值总需要满足interface。

We need to be precise about all this because reflection and interfaces are closely related.

我们需要明确了解这些内容，因为reflect与interface紧密相关。

## 一个接口的表示
Russ Cox has written a detailed blog post about the representation of interface values in Go. It's not necessary to repeat the full story here, but a simplified summary is in order.

Russ Cox已经写过一篇文章详细解释了Go中interface value的表示。这里没有必要重复详细的叙述那篇文章内容，而是按顺序简单的总结。

A variable of interface type stores a pair: the concrete value assigned to the variable, and that value's type descriptor. To be more precise, the value is the underlying concrete data item that implements the interface and the type describes the full type of that item. For instance, after

interface类型变量存储了一对对象：赋值给该变量的实际值，和该值的类型描述。更准确的说，值是实现该接口类型的底层实际数据项，类型是描述该数据项的全部类型。例如下面例子：

    var r io.Reader
    tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
    if err != nil {
        return nil, err
    }
    r = tty

r contains, schematically, the (value, type) pair, (tty, *os.File). Notice that the type *os.File implements methods **other than** Read; even though the interface value provides access only to the Read method, the value inside carries all the type information about that value. That's why we can do things like this:

变量r包含一对（value, type)对象，(tty, *os.File)。注意类型*os.File还实现了Read方法之外的其他方法，尽管该接口值只提供了对Read方法的访问，接口值对象内部包含了该值的所有类型信息。这也是为什么我们可以做下面的事情：

    var w io.Writer
    w = r.(io.Writer)

The expression in this assignment is a type assertion; **what it asserts is that the item inside r also implements io.Writer, and so we can assign it to w**. After the assignment, w will contain the pair (tty, *os.File). That's the same pair as was held in r. The static type of the interface determines what methods may be invoked with an interface variable, even though the concrete value inside may have a larger set of methods.

上面的赋值表达式表示一个类型断言，这个断言用于判断r内部的变量是否也实现了 io.Reader接口，如果是的话，就可以将它赋值为 w 。在赋值成功之后，w的值为一对(tty, *os.File)对象。r值也包含同样的对象。接口的静态类型决定了接口变量的那些方法可以被调用，尽管接口变量内部的实际值可能实现了许多其他方法。

Continuing, we can do this:

进一步我们还可以这样做：

    var empty interface{}
    empty = w

and our empty interface value empty will again contain that same pair, (tty, *os.File). That's handy: an empty interface can hold any value and contains all the information we could ever need about that value.

那么empty接口变量可以包含的同样的对象，(tty, *os.File)。一个interface{}的变量能够存储任意值对象，并包含我们需要与那个值有关的所有信息。

(We don't need a type assertion here because it's known statically that w satisfies the empty interface. In the example where we moved a value from a Reader to a Writer, we needed to be explicit and use a type assertion because Writer's methods are not a subset of Reader's.)

这里我们不需要进行接口类型断言操作，因为任何w对象都满足空接口类型。在上一个例子中我们将一个值从Reader接口移动到Writer接口，我们需要进行显示的接口类型断言，因为Writer接口的方法不是Reader接口方法的子集。

One important detail is that the pair inside an interface always has the form (value, concrete type) and cannot have the form (value, interface type). Interfaces do not hold interface values.

一个重要细节是一个接口变量的内部总是拥有(value, concrete type)对象，而不是拥有(value, interface type)。接口变量不能够存储接口值。

Now we're ready to reflect.

现在让我们开始讲解refelct。

## 反射第一定律
1.反射从一个接口值变成一个接口对象.

At the basic level, reflection is just a mechanism to examine the type and value pair stored inside an interface variable. To get started, there are two types we need to know about in package reflect: Type and Value. **Those two types give access to the contents of an interface variable, and two simple functions, called reflect.TypeOf and reflect.ValueOf, retrieve reflect.Type and reflect.Value pieces out of an interface value**. (Also, from the reflect.Value it's easy to get to the reflect.Type, but let's keep the Value and Type concepts separate for now.)

在基本层理解，reflect仅仅是一种机制来确定存储在接口变量中的实际类型值与类型对象。首先，我们需要知道reflect包中的两种类型：Type与Value。这两种类型提供对interface变量内部访问。reflect还提供了两个简单函数reflect.TypeOf和reflect.ValueOf，分别从interface变量获得一个reflect.Type和reflect.Value类型的变量(当然从reflect.Value可以很容易的获得reflect.Type，但是现在还是保持Value与Type概念的分开.)

Let's start with TypeOf:

让我们从TypeOf开始：

    packege main

    import (
        "fmt"
        "reflect"
    ) 

    func main() {
        var x float64 = 3.4
        fmt.Println("type: ", reflect.TypeOf(x))
    }

This program prints

程序输出结果

    type: float64

You might be wondering where the interface is here, since the program looks like it's passing the float64 variable x, not an interface value, to reflect.TypeOf. But it's there; as godoc reports, the signature of reflect.TypeOf includes an empty interface:

你可能会感到奇怪，interface出现在哪里，因为程序看起来像传递变量x(float64),而不是接口值给reflect.TypeOf。但是，根据godoc文档说明，reflect.TypeOf方法的的签名是可以接受一个空的interface:

    // TypeOf return the reflection Type of the value in the interface{}.
    func TypeOf(i interface{}) Type


































