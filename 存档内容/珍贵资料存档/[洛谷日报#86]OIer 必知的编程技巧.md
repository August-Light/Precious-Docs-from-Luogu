Studying Father's luogu blog <https://www.luogu.com.cn/blog/StudyingFather/some-coding-tips-for-oiers>

本文将给各位介绍一些无论是平时训练或者参加比赛时都比较有用的编程技巧，它们可以让我们的程序可读性更强，方便我们的调试，有些技巧甚至可以帮助我们获得更多的分数。

PS：本文已经同步发表于我的博客，欢迎各位 dalao 前来围观。（[戳这里](https://studyingfather.com/archives/511)）

## 重载运算符

重载运算符是通过对运算符的重新定义，使得其支持特定数据类型的运算操作。

有的时候，我们构造了一种新的数据类型（高精度数，矩阵），当然可以将数据类型的运算操作用函数的形式写出来。但采用重载运算符的方式，可以让程序更为自然。

当然，重载运算符在语法上也有一些要求：

1. 重载运算符的一般形式为`返回值类型 operator 运算符(参数,...)`
2. 在结构体内部重载运算符时，括号内的参数会少一个（事实上，另外一个参数是 this 指针，即指向当前参数的指针~~（这个不重要啦，你只需要知道就好）~~），也就是说，两元运算符只需要 1 个参数。（在结构体外部定义时，两元运算符还是需要 2 个参数）

其他要求就和普通函数的要求差不多了。

举一个简单的例子：

```cpp
#include <stdio.h>
struct pair_num//一个二元组类型
{
 int x,y;
 pair_num operator +(pair_num a)const //不加const会CE
 {
  pair_num res;
  res.x=x+a.x;//x事实上是this.x
  res.y=y+a.y;
  return res;
 }
 pair_num operator -(pair_num a)const
 {
  pair_num res;
  res.x=x-a.x;
  res.y=y-a.y;
  return res;
 }
 bool operator <(pair_num a)const //sort,set等数据结构需要使用小于号
 {
  return x<a.x||(x==a.x&&y<a.y);
 }
}a,b,res;
pair_num operator *(pair_num a,pair_num b)//在结构体外部定义时，不要加const
{
 pair_num res;
 res.x=a.x*b.x;
 res.y=a.y*b.y;
 return res;
}
int main()
{
 scanf("%d%d",&a.x,&a.y);
 scanf("%d%d",&b.x,&b.y);
 res=a+b;
 printf("%d %d\n",res.x,res.y);
 res=a-b;
 printf("%d %d\n",res.x,res.y);
 res=a*b;
 printf("%d %d\n",res.x,res.y);
 return 0;
}
```

## namespace 的用法

在 C++ 程序中，我们往往会加上 `using namespace std;` 这一行语句，那么它是干嘛的呢？

事实上，C++ 引入了一个叫做名字空间（namespace）的机制。引入这个机制的目的，是为了解决程序中变量/函数名重复的问题。

举个栗子：我写了一个 `solve()` 函数，另外一个人也写了一个 `solve()` 函数，如果把这两个函数放在一个程序里面，就会出现函数重名的问题。（当然，如果函数的参数列表不同（即参数个数和参数类型不同），是可以重载函数的，但仅仅是返回值不同时，不能重载函数）这时，我可以把我的 `solve()` 函数放在 `namespace code1` 中，把另一个人的 `solve()` 放在 `namespace code2` 当中，然后，通过 `code1::solve();` 语句就可以调用我写的 `solve()` 函数， `code2::solve()` 可以调用另外一个人的 `solve()` 函数。

写成代码大概是这样的：

```cpp
//P4774 NOI2018 屠龙勇士
//根据自己在同步赛上写的骗分代码改编而成
//总共拿了40分
#include <iostream>
#include <algorithm>
#include <cmath>
#include <cstring>
using namespace std;
long long n,m,a[100005],p[100005],aw[100005],atk[100005];
namespace one_game
{
 //其实namespace里也可以声明变量
 void solve()
 {
  for(int y=0;;y++)
   if((a[1]+p[1]*y)%atk[1]==0)
   {
    cout<<(a[1]+p[1]*y)/atk[1]<<endl;
    return;
   }
 }
}
namespace p_1
{
 void solve()
 {
  if(atk[1]==1)//solve 1-2
  {
   sort(a+1,a+n+1);
   cout<<a[n]<<endl;
   return;
  }
  else if(m==1)//solve 3-4
  {
   long long k=atk[1],kt=ceil(a[1]*1.0/k);
   for(int i=2;i<=n;i++)
    k=aw[i-1],kt=max(kt,(long long)ceil(a[i]*1.0/k));
   cout<<k<<endl;
  }
 }
}
int main()
{
 int T;
 cin>>T;
 while(T--)
 {
  memset(a,0,sizeof(a));
  memset(p,0,sizeof(p));
  memset(aw,0,sizeof(aw));
  memset(atk,0,sizeof(atk));
  cin>>n>>m;
  for(int i=1;i<=n;i++)
   cin>>a[i];
  for(int i=1;i<=n;i++)
   cin>>p[i];
  for(int i=1;i<=n;i++)
   cin>>aw[i];
  for(int i=1;i<=m;i++)
   cin>>atk[i];
  if(n==1&&m==1)one_game::solve();//solve 8-13
  else if(p[1]==1)p_1::solve();//solve 1-4 or 14-15
  else cout<<-1<<endl;
 }
 return 0;
}
```

事实上，C++标准库把所有的函数都放在了 `namespace std` 当中，所以我们调用这些函数的时候，应该采用 `std::xxx` 的形式来调用标准库中的函数。当然，在加入 `using namespace std;` 一行后，就可以省略 `std::` 了，~~我们就可以偷懒了。~~

需要注意的是，将 `std` 命名空间全部引入，会增大标准库里的标识符与自己定义的标识符发生冲突，而导致编译失败的风险。

当然，我们在比赛的时候，可以通过使用 namespace 来使我们的程序结构更加规整。比较常见的用法就是针对每个子任务（诸如上面的屠龙勇士），各写一个 namespace，在其中定义我们解决该子任务所需要的变量与函数，这样各个子任务间互不干扰，方便了我们这些蒟蒻们骗分，也降低了正解爆炸的分数损失。

## 善用标识符进行调试

我们在本地测试的时候，往往要加入一些调试语句。要提交到 OJ 的时候，就要把他们全部删除，有些麻烦。

我们可以通过定义标识符的方式来进行本地调试。

大致的程序框架是这样的：

```cpp
#define DEBUG
#ifdef DEBUG
 //do something
#endif
// or
#ifndef DEBUG
 //do something
#endif
```

`#ifdef` 会检查程序中是否有通过 `#define` 定义的对应标识符，如果有定义，就会执行下面的内容，`#ifndef` 恰恰相反，会在没有定义相应标识符的情况下执行后面的语句。

我们提交程序的时候，只需要将 `#define DEBUG` 一行注释掉即可。

当然，我们也可以不在程序中定义标识符，而是通过 `-DDEBUG` 的编译选项在编译的时候定义 `DEBUG` 标识符。这样就可以在提交的时候不用修改程序了。

PS：不少 OJ（包括洛谷，poj，Codeforces 等主流 OJ）都开启了 `-DONLINE_JUDGE` 这一编译选项，enjoy it!

## 对拍

有的时候我们写了一份代码，但是不知道它是不是正确的。这时候就可以用对拍的方法来进行检验或调试。

什么是对拍呢？具体而言，就是通过对比两个程序的输出来检验程序的正确性。你可以将自己程序的输出与其他程序（打的暴力或者其他dalao的标程）的输出进行对比，从而判断自己的程序是否正确。

当然，我们不能自己比对两段程序的输出，所以我们需要通过批处理的方法来实现对拍的自动化。

具体而言，我们需要一个数据生成器，两个要进行对拍的程序。

每次运行一次数据生成器，将生成的数据写入输入文件，通过重定向的方法使两个程序读入数据，并将输出写入指定文件，利用 Windows 下的 `fc` 命令比对文件（Linux 下为 `diff` 命令），从而检验程序的正确性。

如果发现程序出错，可以直接利用刚刚生成的数据进行调试啦。

对拍程序的大致框架如下：

```cpp
#include <stdio.h>
#include <stdlib.h>
int main()
{
 //For Windows
 //对拍时不开文件输入输出
 //当然，这段程序也可以改写成批处理的形式
 while(1)
 {
  system("gen > test.in");//数据生成器将生成数据写入输入文件
  system("test1.exe < test.in > a.out");//获取程序1输出
  system("test2.exe < test.in > b.out");//获取程序2输出
  if(system("fc a.out b.out"))
  {
   //该行语句比对输入输出
   //fc返回0时表示输出一致，否则表示有不同处
   system("pause");//方便查看不同处
   return 0;
   //该输入数据已经存放在test.in文件中，可以直接利用进行调试
  }
 }
}
```

### 写数据生成器的几个小提示

1. 在使用 `rand()` 前，别忘了调用 `srand(time(NULL))` 来重置随机数种子。（不重置的话，每次调用 `rand()` 只会得到一套随机数）
2. `rand()` 的生成随机数范围在Windows下为 $ [0,32767] $ ，在Linux下为 $ [0,2^{31}-1] $，所以如果数据过大，最好手写一个随机数生成器。（[关于随机数生成器，可以见一位 dalao 在 CF 上的文章](http://codeforces.com/blog/entry/61587)）。

## 选择合适的编译选项

这里介绍一些常用的编译选项。他们会对我们的编程工作起到一定程度的帮助。

一般情况下，作者常用的编译选项是 `g++ a.cpp -o a -g -Wall -std=c++11`。（ `-std=c++11` 只是作者有的时候会开启 C++11 玩玩而已，~~然而 NOI/NOIP 并没有 C++11~~）

而 CCF 在 NOIp 评测时的编译选项（C++）为`g++ a.cpp -o a -lm`。

1. `-Wall`：显示所有警告信息。当我们的程序出现一些逻辑或语义上的问题时（包括未使用的变量，逻辑运算符的优先级问题，嵌套if语句的else配对问题），开启这个编译选项会显示这些警告信息，这样就可以减轻我们的调试负担。
2. `-g`：调试的时候必须打开，这样才能使用 gdb 进行调试。
3. `-Ox`：优化选项。一般能见到的优化选项有 `-O1` `-O2` `-O3`三种，其中用的最多（大多数比赛（包括 NOI）都会开启，但 NOIp 没有）的优化选项是 `-O2`。打开优化选项有时（尤其在使用 STL 时）能减少程序运行时间，但会给单步调试带来一些麻烦，有的时候也会出现负优化。（注意：在程序中加入 `#pragma GCC optimize(2)` 并不会在 NOIp 评测时开启优化）
4. `-DDEBUG`：这个在讲到用标识符进行调试的时候介绍过，是在编译阶段定义 `DEBUG` 标识符，当然这里定义的标识符也可以替换为其他的标识符，例如 `-DTEST` 会在编译阶段定义 `TEST` 标识符。

## 考场上的一些注意事项

以下内容供各位 OIer 们参考，欢迎补充。

1. 经常按 `Ctrl+S` 保存程序是一个好习惯。（提到这一点是因为我市去年提高组的时候考场突然停电，不少人没有保存的程序瞬间灰飞烟灭，还有些人存盘的位置不对，重启的时候程序也没了）
2. 打的代码最好不要删除，用 `//` 注释掉是一个更好的主意，说不定以后要用。
3. 输出 64 位整数时请记得使用 `%lld`。（当然也可以使用 cin/cout 来解决这个问题，别忘了加上 `ios::sync_with_stdio(false);` 防止 IO 耗时过久）
4. 别忘了算内存，有可能你不小心多输了一个 0 就 MLE 了。（一般情况下，256M 内存的话，`int` 数组最多可以开到 $ 5 * 10^7 $）
5. 比赛结束前最后 15 分钟一定要检查一下自己的所有代码，包括如下内容：一些调试语句是否已经被注释，源代码/输入文件/输出文件的文件名是否正确，一些不该被注释的语句（`freopen` 以及 `fclose`）是否删去注释。
6. NOIp 赛场上 STL 一定要根据实际情况（时间限制以及数据规模）谨慎使用！
7. [常见作死错误列表，各位一定要注意！](https://studyingfather.blog.luogu.org/frequent-bugs)
8. 最重要的一点！时刻记得这句话："Keep it simple and stupid."上了考场不要慌张，尽自己全力取得最好的成绩，你一定不会为此感到后悔！

------------

## 附录

参考文献：

1. 刘汝佳《算法竞赛入门经典（第2版）》
2. [cpperference.com](https://zh.cppreference.com/w/%E9%A6%96%E9%A1%B5)

特别感谢各位dalao们的支持：
* 我校 @xqmmcqs 和 @李航博 两位学长传授他们的 OI 经验，没有他们带给我的启发，就不会有本文的诞生；
* 管理员 @ComeIntoPower 辛勤的审核；
* @woshiluo 和 @BeyondLimits 两位 dalao 帮本蒟蒻审稿。

**最后，祝各位 NOIp2018 RP++！**