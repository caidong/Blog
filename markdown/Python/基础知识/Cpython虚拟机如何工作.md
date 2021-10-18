---
title: Cpython 工作机制
date: 2021-08-15 23:12:43
tags:
- 笔记
categories: 
- Python
---

 [博客地址](https://b.top1.pub/)

#### 介绍

你有没有想过当你运行你的一个程序时，Python做了什么？

```
python script.py
```

这篇文章开启一个寻找这个问题答案的系列文章。我们将深入了解CPython的内部结构，这是最受欢迎的Python实现。通过这样做，我们将在更深层次上理解语言本身。这是这个系列的主要目标。如果你熟悉Python 和 无障碍的阅读C，但没有太多CPython源码的经验，这那么你很有可能会发现这篇文章很有趣。

##### 什么是CPython以及为什么每个人都想要学习它

让我们先陈述下大家都知道的事实。CPython是一个用C写的Python解析器。它是Python中的一个实现，还有着PyPy、Jython、IronPython和一些其他的实现。Cpython 最突出的是一个原创的、最多维护和最受欢迎的一个。

CPython实现了Python,但什么是Python?最简单的回答-Python是一个编程语言。当同样的问题被正确提出时，答案就会变得更加微妙：Python定义是什么？Python不像C语言，没有正式的规范，最接近的 是 Python 语言参考https://docs.python.org/3.9/reference/index.html，下面是它的开头语：

		*当我尽量精确的可能，除了语法和词法之外，我选择使用英语而不是正式的规范。这使得文档更容易被一般人理解，但是会留下歧义的空间。因此如果你来自火星并试图仅从这个文档中重新实现python,你可能不得不猜测，事实上你最终可能会实现完全不同的语言。如果你正在使用Python并想知道关于该语言特定领域的确切规则是什么，那么你绝对可以在这找到他们*。

所以Python不仅仅由它的语言参考定义。说Python是由其参考实现CPython定义的也是错误的，因为有些实现细节不属于该语言的一部分。依赖于引用计数的垃圾收集器就是一个例子。由于没有单一的真实来源，我们可以说Python部分由语言参考定义，部分由其主要实现CPython定义。

这样的推理可能看起来很迂腐，但我认为澄清我们将要研究的主题的关键作用至关重要。不过，你可能仍想知道，为什么我们应该研究它。除了单纯的好奇之外，我想还有下面一些原因：

	- 对语言更加全面和深入的理解。如果了解Python的实现细节，则更容易掌握它的某些特性。
	- 实践中的实现细节。对象是如何存储的，垃圾回收器是如何工作的，当你想要理解语言的适用性和限制性，预估性能和检测效率低下时，如何协调多线程是关键重要的主题。
	- CPython 提供了允许扩展Python和C语言中嵌入Python的Python/C接口。为了搞笑的使用这个API程序猿需要很好地理解Cpython 是如何工作的。

##### 理解CPython工作原理需要什么

	CPython被设计成易维护。新手能当然可以期望能够阅读源码和理解它在做什么。然而，它可能需要一些时间。写这个系列文章就是希望你能缩短它。

###### 如何推出这个系列

	我选择从顶到下的方式。这个部分我们将探索CPython的核心概念虚拟机。接下来，我们将看到CPython编译一个程序到VM中执行。然后，我们将熟悉源码和进一步通过执行程序，是学习主要部分解释器的方法。最终地，我们将选出语言的部分，一步一步的看他们是如何实现的。这是我的一个初略想法，不仅不是一个严格计划。
	
	提示： 这篇文章我参考的是CPython3.9。一些实现细节将在CPython的升级中改变。我将尝试保持追踪重要的改变并更新笔记。

###### 大方向
	一个可执行的Python程序大致包含三阶段：
	1.初始化
	2. 编译

 3. 解释

    在初始化阶段，CPython初始化Python运行必要的数据结构。它也准备了如内置数据类型，配置和加载内置模块，设置导入系统以及一些其他事情。这是一个非常重要的阶段，由于服务性质，经常被CPython探索者所忽略。

    接下来是编译阶段，CPython是一个解释器，不是一个感觉上的编译器，他不产生机器码。解释器，然而，经常在执行之前将源码翻译成一些中间表示。CPython也是如此。这个翻译阶段和典型的编译阶段做的一样：转换源码和构建AST(抽象语法树)，从AST生成字节码，甚至执行一些字节码优化。

    在看下一阶段之前，我们需要理解什么是字节码，字节码是一些列指令。每个指令包含连个字节：一个操作码和一个参数。像下面这个例子：

    ```python
    def g(x)
    	return x+3
    ```

    CPython翻译函数体g(x)成下面序列字节码：[124, 0, 100, 1, 23, 0, 83, 0],如果我们运行标准模块dis,去拆解它，我们将得到下面：

    ```
    $ python -m dis example1.py
    ...
    2           0 LOAD_FAST            0 (x)
                2 LOAD_CONST           1 (3)
                4 BINARY_ADD
                6 RETURN_VALUE
    ```

    这个LOAD_FAST操作码对应字节码124和参数0。LOAD_CONST操作码对应字节100和参数1.BINARY_ADD 和RETURN_VALUE 指令总是分别编码成（23,0）和（83,0）它们都不需要参数。

    CPython 的核心是一个可执行字节码的虚拟机。从前面的例子，你可能猜测到他们是如何运行的。CPython 虚拟机是基于栈的。这意味着它执行指令使用栈存储和取回数据。LOAD_FAST指令推入一个本地变量到栈中。LOAD_CONST 推入一个常量。BINART_ADD 从栈中弹出两个对象，使他们相加并返回结果，最后，RETURN_VALUE 弹出在栈任何值，并返回结果给调用者。

    字节码执行发生巨大的评估循环，该循环在指令执行时运行。它将停止产生值或发生错误。

    如此简短的概述引发了许多问题：

    - 操作码LOAD_FAST 和 LOAD_CONST 操作码是很忙意思？他们是索引？他们索引什么？

    - VM是否在堆栈上放置值或对对象的引用？

    - CPython如何知道x是局部变量？

    - 如果参数太大而无法放入单个字节怎么办？

    - 两个数字相加和连接两个字符串一样吗？如果是，那么VM如何区分两种操作？

  为了回答这些和其他有趣的问题，我们需要理解CPython VM的核心概念

##### 代码对象，函数对象，片段

##### 代码对象

我们看简单的函数字节码像什么。但是典型的Python程序是更复杂的、VM如何执行包含函数定义的模块，并进行函数的调用？

思考下面的程序：

```
def f(x):
    return x + 1

print(f(1))
```

它的字节码是什么样的？要回答这个问题我们先要分析下程序做了什么。它定义了一个函数f(),调用f()并传递参数1和打印返回结果，不管函数f()做了什么，它都不是字节码模块的一部分。我们能够通过运行反编译程序来验证自己。

```bash
$ python -m dis example2.py

1           0 LOAD_CONST               0 (<code object f at 0x10bffd1e0, file "example.py", line 1>)
            2 LOAD_CONST               1 ('f')
            4 MAKE_FUNCTION            0
            6 STORE_NAME               0 (f)

4           8 LOAD_NAME                1 (print)
           10 LOAD_NAME                0 (f)
           12 LOAD_CONST               2 (1)
           14 CALL_FUNCTION            1
           16 CALL_FUNCTION            1
           18 POP_TOP
           20 LOAD_CONST               3 (None)
           22 RETURN_VALUE
...
```

在第一行我们通过从称代码对象的东西创建函数来定义函数f()并绑定名称f。我们没有看到返回递增参数的函数f()的字节码。

作为单个单元（如模块活函数体）执行的代码段成为代码块。CPtyhon在称为代码对象的结构中存储有关代码块功能的信息。它包含字节码和像在块中的变量名称列表之类的东西。运行模块或调用函数意味着开始评估相应的代码对象。

###### 函数对象

然而，函数不仅仅是代码对象。它必须包含等额外的信息，例如函数名、文档字符串、默认参数在封闭作用域中定义的变量值 。这些信息和代码对象一起存储在函数对象中。MAKE_FUNCTION 指令被常常用来创建它。在CPython源码中函数对象结构的定义前面有如下注释：

*函数对象和代码对象不应相互混淆：*

*函数对象通过执行 def 语句创建的，他们的__code__属性中引用了一个代码对象，这是一个纯粹的语法对象*即只不过是一些源代码行的编译版本。每个源代码“片段”有一个代码对象，但每个代码对象可以被零个或多个函数对象引用，这仅取决于到目前为止源代码中的“def”语句执行了多少次。

几个函数对象怎么会引用一个代码对象呢？下面是一个例子：

```python
def  make_add_x ( x ): 
    def  add_x ( y ): 
        return  x  +  y 
    return  add_x

add_4  =  make_add_x ( 4 ) 
add_5  =  make_add_x ( 5 )
```

`make_add_x()`函数的字节码包含`MAKE_FUNCTION`指令。函数`add_4()`和`add_5()`是使用相同的代码对象作为参数调用此指令的结果。但是有一个不同的论点—— 的值`x`。每个函数都通过[单元变量](https://tenthousandmeters.com/blog/python-behind-the-scenes-5-how-variables-are-implemented-in-cpython/)机制获得自己的功能，该机制允许我们创建像`add_4()`和这样的闭包`add_5()`。

在我们进入下一个概念之前，先看一下代码和函数对象的定义，以更好地了解它们是什么。

```c
struct  PyCodeObject  { 
    PyObject_HEAD 
    int  co_argcount ;             /* #arguments, 除了 *args */ 
    int  co_posonlyargcount ;      /* #positional only arguments */ 
    int  co_kwonlyargcount ;       /* #keyword 只有参数 */ 
    int  co_nlocals ;              /* #局部变量*/ 
    int  co_stacksize ;            /* #entries 需要评估堆栈 */ 
    int  co_flags ;                /* CO_...，见下文 */ 
    int  co_firstlineno ;          /* 第一个源代码行号 */ 
    PyObject  * co_code ;          /* 指令操作码 */ 
    PyObject  * co_consts ;         /* 列表（使用的常量）*/ 
    PyObject  * co_names ;          /* 字符串列表（使用的名称）*/ 
    PyObject  * co_varnames ;       /* 字符串元组（局部变量名）*/ 
    PyObject  * co_freevars ;       /* 字符串元组（自由变量名）*/ 
    PyObject  * co_cellvars ;       /* 字符串元组（单元变量名）*/

    Py_ssize_t  * co_cell2arg ;     /* 映射作为参数的单元格变量。*/ 
    PyObject  * co_filename ;       /* unicode（从哪里加载它）*/ 
    PyObject  * co_name ;           /* unicode (name, for reference) */ 
        /* ... 更多成员 ... */ 
};
typedef  struct  { 
    PyObject_HEAD 
    PyObject  * func_code ;         /* 一个代码对象，__code__ 属性 */ 
    PyObject  * func_globals ;      /* 字典（其他映射不会做）*/ 
    PyObject  * func_defaults ;     /* NULL 或元组 */ 
    PyObject  * func_kwdefaults ;   /* NULL 或 dict */ 
    PyObject  * func_closure ;      /* NULL 或单元格对象的元组 */ 
    PyObject  * func_doc ;          /* __doc__ 属性，可以是任何东西 */ 
    PyObject  *功能名称；        /* __name__ 属性，一个字符串对象 */ 
    PyObject  * func_dict ;         /* __dict__ 属性，一个 dict 或 NULL */ 
    PyObject  * func_weakreflist ;  /* 弱引用列表 */ 
    PyObject  * func_module ;       /* __module__ 属性，可以是任何东西 */ 
    PyObject  * func_annotations ;  /* 注释、字典或 NULL */ 
    PyObject  * func_qualname ;     /* 限定名 */ 
    vectorcallfunc  vectorcall ; 
}  PyFunctionObject ;
```

##### 帧对象

当 VM 执行一个代码对象时，它必须跟踪变量的值和不断变化的值堆栈。它还需要记住它在哪里停止执行当前代码对象以执行另一个以及返回的位置。CPython 将此信息存储在框架对象中，或者简单地存储在框架中。框架提供了可以执行代码对象的状态。由于我们越来越习惯使用源代码，因此我也将框架对象的定义留在这里：

```c
struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;      /* previous frame, or NULL */
    PyCodeObject *f_code;       /* code segment */
    PyObject *f_builtins;       /* builtin symbol table (PyDictObject) */
    PyObject *f_globals;        /* global symbol table (PyDictObject) */
    PyObject *f_locals;         /* local symbol table (any mapping) */
    PyObject **f_valuestack;    /* points after the last local */

    PyObject **f_stacktop;          /* Next free slot in f_valuestack.  ... */
    PyObject *f_trace;          /* Trace function */
    char f_trace_lines;         /* Emit per-line trace events? */
    char f_trace_opcodes;       /* Emit per-opcode trace events? */

    /* Borrowed reference to a generator, or NULL */
    PyObject *f_gen;

    int f_lasti;                /* Last instruction if called */
    /* ... */
    int f_lineno;               /* Current line number */
    int f_iblock;               /* index in f_blockstack */
    char f_executing;           /* whether the frame is still executing */
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];  /* locals+stack, dynamically sized */
};
```

创建第一帧以执行模块的代码对象。每当需要执行另一个代码对象时，CPython 都会创建一个新框架。每帧都有对前一帧的引用。因此，帧形成帧堆栈，也称为调用堆栈，当前帧位于顶部。当一个函数被调用时，一个新的帧被压入堆栈。从当前执行的帧返回时，CPython 通过记住其最后处理的指令来继续执行前一帧。从某种意义上说，CPython VM 除了构造和执行帧之外什么都不做。然而，正如我们很快就会看到的，这个总结，委婉地说，隐藏了一些细节。

###### 线程、解释器、运行时

我们已经看过三个重要的概念：

- 代码对象
- 一个函数对象；和
- 一个框架对象。

CPython 还有三个：

- 线程状态
- 翻译状态；和
- 运行时状态。

#### 线程状态

线程状态是一种数据结构，包含线程特定的数据，包括调用堆栈、异常状态和调试设置。它不应与操作系统线程混淆。不过，它们之间的联系很紧密。考虑使用标准[`threading`](https://docs.python.org/3/library/threading.html)模块在单独的线程中运行函数时会发生什么：

```
from threading import Thread

def f():
    """Perform an I/O-bound task"""
    pass

t = Thread(target=f)
t.start()
t.join()
```

`t.start()`实际上通过调用 OS 函数（`pthread_create()`在类 UNIX 系统和`_beginthreadex()`Windows 上）创建一个新的 OS 线程。新创建的线程从`_thread`负责调用目标的模块调用函数。该函数不仅接收目标和目标的参数，还接收要在新 OS 线程中使用的新线程状态。一个操作系统线程以其自己的线程状态进入求值循环，因此它总是在手边。

我们可能还记得著名的 GIL（全局解释器锁），它可以防止多个线程同时处于评估循环中。这样做的主要原因是在不引入更细粒度的锁的情况下保护 CPython 的状态免受损坏。[Python/C API 参考](https://docs.python.org/3.9/c-api/index.html)清楚地解释了 GIL：

> Python 解释器不是完全线程安全的。为了支持多线程 Python 程序，有一个全局锁，称为全局解释器锁或 GIL，必须由当前线程持有才能安全地访问 Python 对象。如果没有锁，即使是最简单的操作也可能导致多线程程序出现问题：例如，当两个线程同时增加同一个对象的引用计数时，引用计数可能最终只会增加一次而不是两次。

要管理多个线程，需要有比线程状态更高级的数据结构。

#### 解释器和运行时状态

事实上，有两种状态：解释器状态和运行时状态。对两者的需求似乎并不明显。然而，任何程序的执行都至少有一个实例，这是有充分理由的。

解释器状态是一组线程以及特定于该组的数据。线程共享诸如加载模块 ( `sys.modules`)、内置函数 ( `builtins.__dict__`) 和导入系统 ( `importlib`) 之类的东西。

运行时状态是一个全局变量。它存储特定于进程的数据。这包括 CPython 的状态（例如它是否已初始化？）和 GIL 机制。

通常，一个进程的所有线程都属于同一个解释器。然而，在极少数情况下，人们可能想要创建一个子解释器来隔离一组线程。[mod_wsgi](https://modwsgi.readthedocs.io/en/develop/user-guides/processes-and-threading.html#python-sub-interpreters)，它使用不同的解释器来运行 WSGI 应用程序，就是一个例子。隔离最明显的效果是每组线程都获得了包括 在内的所有模块的自己版本`__main__`，这是一个全局命名空间。

CPython 没有提供一种简单的方法来创建类似于`threading`模块的新解释器。此功能仅通过 Python/C API 支持，但这[可能](https://www.python.org/dev/peps/pep-0554/)有一天[会改变](https://www.python.org/dev/peps/pep-0554/)。

### 架构总结

让我们快速总结一下 CPython 的架构，看看一切是如何组合在一起的。解释器可以看作是一个分层结构。下面总结了层是什么：

1. 运行时：进程的全局状态；这包括 GIL 和内存分配机制。
2. 解释器：一组线程和它们共享的一些数据，例如导入的模块。
3. 线程：特定于单个操作系统线程的数据；这包括调用堆栈。
4. Frame：调用栈的一个元素；一个框架包含一个代码对象并提供一个状态来执行它。
5. 评估循环：执行框架对象的地方。

这些层由我们已经看到的相应数据结构表示。但在某些情况下，它们并不等效。比如内存分配的机制就是使用全局变量来实现的。它不是运行时状态的一部分，但肯定是运行时层的一部分。

### 结论

在这一部分中，我们概述了`python`执行 Python 程序的作用。我们已经看到它分三个阶段工作：

1. 初始化 CPython
2. 将源代码编译为模块的代码对象
3. 执行代码对象的字节码。

负责字节码执行的解释器部分称为虚拟机。CPython VM 有几个特别重要的概念：代码对象、框架对象、线程状态、解释器状态和运行时。这些数据结构构成了 CPython 架构的核心。

我们还有很多东西没涵盖。我们避免深入研究源代码。初始化和编译阶段完全超出了我们的范围。相反，我们从虚拟机的广泛概述开始。这样，我想，我们可以更好地看到每个阶段的责任。现在我们知道 CPython 将源代码编译成什么——代码对象。[下次](https://tenthousandmeters.com/blog/python-behind-the-scenes-2-how-the-cpython-compiler-works/)我们将看到它是如何做到的。