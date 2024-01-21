Tanarichi Kagiri <https://www.luogu.com.cn/blog/ternarytree/hu-kao-chang-ce-shuo-zai-jian-c-te-shu-ku-han-shuo-yu-guan-jian-ci-zhi>

# Part 0: 前言

某个人在 CSP-S 2022 的考场上取得了 $55$ 分三等的优异成绩，这一切还要从 Linux 说起。

为什么要写这篇文章？因为考场上，很多机子无法提供 Linux 操作系统或者虚拟机环境，在把某些 Linux 特有函数作为变量后，本地很多可以通过编译，而 CCF 评测机是在 Linux 系统下运行的，也就导致了代码的编译错误（CE）。

若无特殊说明，本文采用洛谷评测测试 Linux 环境下的代码。语言为 C++14 (GCC 9)。

目前整理到的量不算很多，欢迎投稿。

# Part 1: 库函数与关键词

- ## Windows 下编译不出现错误，但是 Linux 下编译错误的

  - ## bzero, bcopy, bcmp

    之所以把这玩意放在第一个，首先就是开头实实在在是笔者本人的经历。

    考场上笔者使用了一个名叫 `bzero` 的数组，本地 Windows Dev C++ 开启 `-Wall -Wextra` 后仍然无警告通过编译。而在洛谷机子或者官方评测机会导致 CE 发生。
    
    世界名画：
    
    ![](https://cdn.luogu.com.cn/upload/image_hosting/gv2rez9b.png)

    本地 vscode 会给出警告，但是成功过编：

    ![](https://cdn.luogu.com.cn/upload/image_hosting/7d7qkjv6.png)

    bzero 函数是 Linux 系统下特有的函数。作用是将一个字符串前 $n$ 个字符设为 $0$（即清空）。

    与之类似的还有 bcopy, bcmp。不再赘述。

  - ## index

    有时候要表示下标，有人就会采用 index 作为变量名。在 windows 下这个变量名看起来人畜无害，只会抛出一个小小的警告。但是经过[洛谷编译器实测](https://www.luogu.com.cn/record/102535576)，发现 Linux 下 index 会导致 CE。

    如果要表示下标，建议的替代变量名：ind, pos 等。
    
  - ## ffs

    还是 CSP-S 2022 T2。某佬在程序中使用了 ffs 作为变量名，在 windows 系统下仅会给出警告，喜提 CE。
    


- ## Windows 下编译会抛出错误的

  - ## y0, y1, yn, j0, j1, jn

    这几个是比较老生常谈的库函数了，在 Windows 系统下大概率可以产生编译错误信息，但是不排除某些特殊情况下被坑。

    ![](https://cdn.luogu.com.cn/upload/image_hosting/vruoy3mq.png)

    主要来说，注意给坐标取名的时候注意不要使用即可。
    
   - ## time

     很多人在设随机数种子的时候都会写 `srand(time(0))`，但是还是有人会把 time 当变量名导致 CE，警钟敲烂。
    
  - ## nextafter

    返回 $x$ 与 $y$ 之间，与 $x$ 邻近的可表示的浮点数。
    
  - ## signal
  
    函数 signal 用于捕获信号，可指定信号处理的方式。在 windows 下，若采用 signal 作为全局变量名会导致重名 CE。
    
  - ## ws
  
    和 skipws 相同，感觉没什么用的鸭子。
    
  - ## tm
  
    ~~为了让你言语文明真是用心良苦~~
    
    是一个结构体。如果在代码中使用，容易导致编译错误。

  - ## next, prev

    典。机房同学之前链式向前星写 next 爆零。

    作用是返回迭代器的上一个，下一个。
    
  - ## plus, minus, multiplies, divides, div, modulus, negate, greater, less, bit_and, bit_or, bit_xor, bit_not 
  
    运算相关的尽量谨慎。一不小心就会和 cmath 或者其他库里的函数撞上。

- ## Windows 和 Linux 下不出现错误，但是不排除特定版本编译器出现错误的

  - ## pipe

    与 ws 相同的地方是，这个变量名取决于编译器的版本。洛谷 Linux 下不会导致 CE，但是不建议使用。


  - ## sbrk

    ~~不知道会不会有把后缀数组里面的 rk 写成 sbrk 然后爆零的可怜蛋~~

    sbrk 能够修改程序 BSS 段的大小。百度百科对 sbrk 也有进行介绍，但是本地和洛谷 Linux 均不出现问题。


    
- ## 因 C++ 不同版本差异导致 CE 的

  - ## size
  
    博特说，size 在 C++17 下编译会 CE，而 C++14 以下不会。
    
    亲测，博特说的没错。
    
    [C++14 过编记录](https://www.luogu.com.cn/record/102540888)
    
    [C++17 CE 记录](https://www.luogu.com.cn/record/102540878)
    
  - ## gcd
  
    gcd 在 C++14 以前是没有这个函数的，想要用这个函数必须使用 C++17 以上版本，否则还是得乖乖用 __gcd。

# Part 2: 其他技巧

- ## 不使用 namespace::std

  大部分被重名的函数，都被包含于标准库 std 下。如果不适用 std 则可以很好的避免大部分重名问题。同时，可以通过 `using std::cin`，`using std::cout` 等来获得等价效果。

- ## 使用自己的 namespace

  举个例子：
  
  ![](https://cdn.luogu.com.cn/upload/image_hosting/otx2tdfl.png)
  
  （疯狂叠 buff.jpg）
  
  在图中我加上了大部分 Part 1 中所述的变量名，完全没有警告和错误。这有效地说明，在自己的 namespace 中编写代码是很安全的方式。
  
  
- ## 使用 struct 封装。

  和 namespace 同理，一个结构体内部的成员名称即使重名也不会 CE。例如：
  
  ![](https://cdn.luogu.com.cn/upload/image_hosting/eu9nybv2.png)
  
  可以正常通过编译，0 warning and 0 error.
  
- ## 用大写 + 小写的方式来命名

  大部分情况下，C++ 内置函数的名字组成方式都是小写字母，数字加下划线。可以看到，上述介绍的变量名全部都是小写字母与数字的组合。所以，用大小写混用的方式可以极大概率避免重名的发生。
  
- ## 多翻阅 C++ 官方中文文档

  很有效的方法，也是根本措施。
  
  网址：[cppreference](https://zh.cppreference.com/w/%E9%A6%96%E9%A1%B5)