yurzhang 的博客 <https://www.luogu.com.cn/blog/yurzhang/awesome-sanitizers>

C++ 太复杂了！即使编写的时候非常小心，也不能完全杜绝内存安全问题的出现。就算是算法竞赛时编写的百余行简短代码，也时不时会有数组越界、变量未初始化、未定义行为等问题困扰我们。

在算法竞赛短短几个小时中，花费时间检查这种与算法本身无关的错误，无疑是浪费时间。那有没有什么工具能帮助我们检查这类错误呢？

Sanitizers 是由 Google 发起的一套开源工具集，包含了数个帮助开发者查找 C++ 代码中疑难杂症的工具。其中有些工具，在算法竞赛领域也十分实用。

## AddressSanitizer

[AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) 简称 ASan，可以帮助我们检查代码中的内存使用问题。

它从 LLVM 3.1 起被集成到 Clang 中，从 GCC 4.8 起被集成到 GCC 中。你可以通过在编译参数中添加 `-fsanitize=address` 来使用它。

### 解引用悬垂指针

来看一段显然存在问题的代码：

```cpp
int main() {
  char *x = new char[10];
  delete[] x;
  return x[5];
}
```

我们申请了一段长度为 10 的 `char` 数组，随后释放了它，这时我们再取出数组的第 5 个元素——然而这个位置的内存已经被我们释放掉了！

像这种使用已经被释放的内存的问题，我们称之为**解引用悬垂指针**。让我们试试 ASan 能不能帮我们发现这个问题吧！

首先尝试编译这份代码：

```bash
g++ main.cpp -o main -g -fsanitize=address
```

代码顺利编译，看上去什么都没有发生？不要着急，我们试试执行编译出的程序：

```bash
./main
```

程序崩溃了！同时给出了很长一段错误信息：

```plain
=================================================================
==19311==ERROR: AddressSanitizer: heap-use-after-free on address 0x602000000015 at pc 0x562f45c941de bp 0x7ffc3b87b640 sp 0x7ffc3b87b630
READ of size 1 at 0x602000000015 thread T0
    #0 0x562f45c941dd in main /home/yur/Workspace/Codes/main.cpp:4
    #1 0x7ff979dca28f  (/usr/lib/libc.so.6+0x2328f)
    #2 0x7ff979dca349 in __libc_start_main (/usr/lib/libc.so.6+0x23349)
    #3 0x562f45c940a4 in _start ../sysdeps/x86_64/start.S:115

0x602000000015 is located 5 bytes inside of 10-byte region [0x602000000010,0x60200000001a)
freed by thread T0 here:
    #0 0x7ff97a38e35a in operator delete[](void*) /usr/src/debug/gcc/libsanitizer/asan/asan_new_delete.cpp:155
    #1 0x562f45c941a1 in main /home/yur/Workspace/Codes/main.cpp:3
    #2 0x7ff979dca28f  (/usr/lib/libc.so.6+0x2328f)

previously allocated by thread T0 here:
    #0 0x7ff97a38d7f2 in operator new[](unsigned long) /usr/src/debug/gcc/libsanitizer/asan/asan_new_delete.cpp:98
    #1 0x562f45c9418a in main /home/yur/Workspace/Codes/main.cpp:2
    #2 0x7ff979dca28f  (/usr/lib/libc.so.6+0x2328f)

SUMMARY: AddressSanitizer: heap-use-after-free /home/yur/Workspace/Codes/main.cpp:4 in main
Shadow bytes around the buggy address:
  0x0c047fff7fb0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fc0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fd0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fe0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7ff0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c047fff8000: fa fa[fd]fd fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8010: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8020: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8030: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8040: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8050: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==19311==ABORTING
```

通过这段错误信息，我们可以看到 ASan 告诉我们，在 `main.cpp` 的第 4 行出现了一个 `heap-use-after-free` 错误，也就是我们在释放了某块堆内存后又使用了它。

不仅如此，它还告诉我们这块内存是在 `main.cpp` 的第 2 行申请的，在 `main.cpp` 的第 3 行被释放掉，并且打印出了这三个地方的调用栈！

也就是说，ASan 很好地完成了它的工作，但是是在运行时而不是编译时。这听上去有些令人沮丧，但这已经是最好的结果了，因为很多运行时错误只有在特定的输入下才会出现，在编译时发现它们是不可能的！

换一个角度想想，利用 ASan，我们可以在遇到运行时错误的时候，快速定位到错误的原因，以及发生错误的位置，这已经极大地方便调试了！

### 全局数组越界

看到这里，你可能会想：好吧 ASan 确实是个很棒的工具，可是在算法竞赛中我们真的很少很少会手动分配堆内存，事实上如果不使用 STL，我的程序可能根本不会用到任何堆内存，那么 ASan 对我来说不就没有用处了吗？

我们来看一段更贴近算法竞赛的，显然存在问题的代码：

```cpp
#include <iostream>

constexpr int N = 100010;

int n, a[N];

int main() {
  for (int i = 0; i < N; ++ i)
    a[i] = i;

  std::cin >> n;
  std::cout << a[n] << std::endl;
  return 0;
}
```

我申请了一个长为 $N$ 的全局数组——这种做法在算法竞赛中很常见，然后我将数组中每个位置的值初始化为它的下标。

接着我们输入了一个 $n$，并输出下标为 $n$ 的位置的值。这太糟糕了！当输入的 $n$ 足够大的时候，这段代码显然会出现数组越界！

让我们再看看 ASan 的表现：

![global-buffer-overflow](https://yurzhang.com/usr/uploads/2022/11/2709697251.gif)

不出所料，ASan 也能检测到这个错误，它告诉我们在 `main.cpp` 的第 12 行出现了一个 `global-buffer-overflow` 错误：

```plain
=================================================================
==65354==ERROR: AddressSanitizer: global-buffer-overflow on address 0x558afdb55f68 at pc 0x558afdaf12e3 bp 0x7fff39a9d580 sp 0x7fff39a9d570
READ of size 4 at 0x558afdb55f68 thread T0
    #0 0x558afdaf12e2 in main /home/yur/Workspace/Codes/main.cpp:12
    #1 0x7fc18ed9c28f  (/usr/lib/libc.so.6+0x2328f)
    #2 0x7fc18ed9c349 in __libc_start_main (/usr/lib/libc.so.6+0x23349)
    #3 0x558afdaf1124 in _start ../sysdeps/x86_64/start.S:115

0x558afdb55f68 is located 0 bytes to the right of global variable 'a' defined in 'main.cpp:5:8' (0x558afdaf44c0) of size 400040
SUMMARY: AddressSanitizer: global-buffer-overflow /home/yur/Workspace/Codes/main.cpp:12 in main
Shadow bytes around the buggy address:
  0x0ab1dfb62b90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62ba0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62bb0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62bc0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62bd0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0ab1dfb62be0: 00 00 00 00 00 00 00 00 00 00 00 00 00[f9]f9 f9
  0x0ab1dfb62bf0: f9 f9 f9 f9 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62c00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62c10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62c20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab1dfb62c30: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==65354==ABORTING
```

### 以及…

除此之外，ASan 还能检测堆栈上数组的越界、使用离开作用域时被析构的对象、初始化顺序导致的 bug、内存泄漏等疑难杂症。由于它们在算法竞赛中出现得较少，这里就不多赘述。

值得一提的是，既然 ASan 是在运行时进行的检测，那它就必然会对程序的效率产生一定影响。经过[测试](https://github.com/google/sanitizers/wiki/AddressSanitizerPerformanceNumbers)，开启 ASan 后的程序效率平均只有原先的一半左右。这个损失会根据平台和程序的不同而有所不同，但无论如何，请记得在进行**重视效率**的测试（如测试你的程序执行大样例时是否会超时）时关掉 ASan 以保证测试结果准确。

## UndefinedBehaviorSanitizer

[UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) 简称 UBSan，可以帮助我们检查代码中的未定义行为。

它从 LLVM 3.3 起被集成到 Clang 中，从 GCC 4.9 起被集成到 GCC 中。你可以通过在编译参数中添加 `-fsanitize=undefined` 来使用它。

### 按位位移溢出

```cpp
int main() {
  int x = 42;
  x <<= 27;
  return 0;
}
```

上面的代码，按位左移的时候会溢出 `int` 的上界，这是未定义行为。如果你使用了 UBSan，它会在运行时报错：

```plain
main.cpp:3:5: runtime error: left shift of 42 by 27 places cannot be represented in type 'int'
```

### 数值溢出

```cpp
using i64 = long long;

constexpr int mod = 998244353;

int main() {
  int x = 114514;
  int y = 1919810;

  x = ((i64)x * x + y * y) % mod;
  return 0;
}
```

上面的代码，`y * y` 的部分会溢出 `int` 的上界，这是未定义行为。如果你使用了 UBSan，它会在运行时报错：

```plain
main.cpp:9:23: runtime error: signed integer overflow: 1919810 * 1919810 cannot be represented in type 'int'
```

### 使用空指针

```cpp
#include <iostream>

struct Node {
  Node *left, *right;
  int value;

  Node(int _value):
    left(nullptr),
    right(nullptr),
    value(_value) { }

  ~Node() {
    delete left;
    delete right;
  }
};

void print(Node *now) {
  if (now->left != nullptr)
    print(now->left);

  printf("%d ", now->value);

  if (now->right != nullptr)
    print(now->right);
}

int main() {
  Node *root = new Node(5);

  root->left = new Node(3);
  root->right = new Node(8);

  root->left->right = new Node(4);

  print(root);

  delete root;

  print(nullptr);
  return 0;
}
```

上面的代码是使用指针实现的简单的二叉树，其中 `print` 函数存在一个致命的问题：如果传入的参数本身是空指针，它就会在 `now->left` 这一步对空指针取左儿子，这是未定义行为。如果你使用了 UBSan，它会在运行时报错：

```plain
main.cpp:19:12: runtime error: member access within null pointer of type 'struct Node'
```

如果希望 UBSan 打印出错位置的调用栈，可以在执行程序时加上 `UBSAN_OPTIONS=print_stacktrace=1` 环境变量：

![stacktrace](https://yurzhang.com/usr/uploads/2022/11/1177270905.gif)

### 除此之外…

UBSan 还能检测十数种未定义行为，并且每种未定义行为是否需要被检测可以通过编译参数自由控制。与 ASan 相同的是，UBSan 也会影响程序运行效率，在实际使用时请务必注意！

## ThreadSanitizer

[ThreadSanitizer](https://github.com/google/sanitizers/wiki#threadsanitizer) 简称 TSan，可以帮助我们检查多线程代码中的数据竞争。

算法竞赛不涉及多线程程序，因此这里不讨论 TSan 相关内容，有兴趣可以自行了解。

## MemorySanitizer

[MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html) 简称 MSan，可以帮助我们检查对未初始化的值的使用。

它从 LLVM 4.0 起被集成到 Clang 中。而不幸的是，GCC 并没有集成这个工具。你只能通过在 Clang 中使用 `-fsanitize=memory` 参数来使用它。

```cpp
#include <iostream>

// please ignore unused `argv` :)
int main(int argc, char *argv[]) {
  int a[10];
  for (int i = 2; i < 10; ++ i)
    a[i] = i;

  int x = a[argc];
  int y = x * 2;

  if (y == 0) {
    std::cout << "Hello" << std::endl;
  } else {
    std::cout << "MSan" << std::endl;
  }
  return 0;
}
```

上面的代码中，当我们通过 `./main` 执行程序时，`argc` 的值将会为 $1$，然而我们没有初始化 `a[1]`，MSan 将会在运行时报错：

```plain
==95226==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x55a96ddb3c96 in main /home/yur/Workspace/Codes/main.cpp:12:7
    #1 0x7f1b4f9d628f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #2 0x7f1b4f9d6349 in __libc_start_main (/usr/lib/libc.so.6+0x23349) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #3 0x55a96dd221d4 in _start /build/glibc/src/glibc/csu/../sysdeps/x86_64/start.S:115

SUMMARY: MemorySanitizer: use-of-uninitialized-value /home/yur/Workspace/Codes/main.cpp:12:7 in main
Exiting
```

可以发现，MSan 直到第 12 行 `if (y == 0)` 处才报错，这是因为 MSan **不会**在读取未初始化值时**立即**报错，而是会等到这个值影响程序执行时才报错！在此之前，它会跟踪并记录这个未初始化值产生的影响。

如果一个未初始化值经过很远的传播后才引发 MSan 报错，你可能得花很大力气才能找到这个值是在哪里产生的。这时你可以选择在编译时加上 `-fsanitize-memory-track-origins` 参数，这样 MSan 报错时会从最初的未初始化值开始追踪，打印它的每一次移动，直到来到报错的位置：

```plain
==96812==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x559164b57fe0 in main /home/yur/Workspace/Codes/main.cpp:12:7
    #1 0x7fd4af63f28f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #2 0x7fd4af63f349 in __libc_start_main (/usr/lib/libc.so.6+0x23349) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #3 0x559164ac61d4 in _start /build/glibc/src/glibc/csu/../sysdeps/x86_64/start.S:115

  Uninitialized value was stored to memory at
    #0 0x559164b57f5d in main /home/yur/Workspace/Codes/main.cpp:10:7
    #1 0x7fd4af63f28f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)

  Uninitialized value was stored to memory at
    #0 0x559164b57e95 in main /home/yur/Workspace/Codes/main.cpp:9:7
    #1 0x7fd4af63f28f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)

  Uninitialized value was created by an allocation of 'a' in the stack frame of function 'main'
    #0 0x559164b57920 in main /home/yur/Workspace/Codes/main.cpp:4

SUMMARY: MemorySanitizer: use-of-uninitialized-value /home/yur/Workspace/Codes/main.cpp:12:7 in main
Exiting
```

## Work with sanitizers

上述例子中的所有代码，在开启 GCC 最高级别警告的情况下都不会产生任何警告（最后一个例子的 `unused parameter` 除外），它们几乎只能在运行时才能发现问题，而这也就是 Sanitizers 的职责所在。

如果你在调试的时候遇到了恼人的「段错误」（运行时错误），而你又没有头绪，不妨试试 Sanitizers 吧。只需要在编译时加上 `-fsanitize=address,undefined` 参数，然后重新执行样例，兴许就能发现问题了。

> 才疏学浅，如有纰漏，欢迎指出！
