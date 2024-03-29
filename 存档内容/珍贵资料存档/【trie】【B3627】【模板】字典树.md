一扶苏一 的博客 <https://www.luogu.com.cn/blog/fusu2333/solution-b3627>

## 【trie】【B3627】【模板】字典树

## Analysis

考虑对模式串建立 trie 树，每个节点维护一个值 cnt，在每个模式串插入结束以后，从字典树上对应的结点令 cnt 加一。则每次查询相当于在 trie 树上匹配到的结点的子树中 cnt 的和。

于是在建立好 trie 树以后做一次 dfs，求出子树 cnt 和，就可以 $O(1)$ 回答查询了。

## Code

```cpp
#include <iostream>
#include <unordered_map>


struct Node {
  int cnt;
  std::unordered_map<char, Node*> ch;

  Node() : cnt(0) {};

  void dfs() {
    for (auto [x, y] : ch) {
      y->dfs();
      cnt += y->cnt;
    }
  }
};
Node *rot;

int main() {
  std::ios::sync_with_stdio(false);
  std::cin.tie(0);
  int T, n, q;
  std::string s;
  for (std::cin >> T; T; --T) {
    std::cin >> n >> q;
    rot = new Node();
    for (int i = 1; i <= n; ++i) {
      std::cin >> s;
      auto u = rot;
      for (auto c : s) u = (u->ch[c] ? u->ch[c] : u->ch[c] = new Node);
      ++u->cnt;
    }
    rot->dfs();
    for (int i = 1; i <= q; ++i) {
      std::cin >> s;
      bool flag = true;
      auto u = rot;
      for (auto c : s) if (u->ch[c]) {
         u = u->ch[c];
      } else {
        flag = false; break;
      }
      std::cout << ((flag) ? u->cnt : 0) << '\n';
    }
  }
  return 0;
}
```

## gen

```py
from random import randint
import os

arr = [0, 1, 10, 100, 1000, 100000, 1]
temp = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'

for id in range(1, 7):
  inname = "trie" + str(id) + '.in'
  ansname = 'trie' + str(id) + '.ans'
  f = open(inname, 'w', newline='\n')
  t = arr[id]
  f.write(str(t) + '\n')
  if id != 6:
    for tt in range(t):
      n = 100000 // t
      m = n
      last = 500000 // t
      text = []
      f.write(str(n) + " " + str(m) + '\n')
      for i in range(m - 1):
        s = str()
        lst = randint(1, (last // (m - i + 1)))
        for j in range(lst):
          s += temp[randint(0, len(temp) - 1)]
        text.append(s)
        last -= lst
      s = str()
      for i in range(last):
        s += temp[randint(0, len(temp) - 1)]
      text.append(s)
      last = 1500000 // t
      for i in range(n - 1):
        j = randint(0, len(text) - 1)
        lst = randint(1, (last // (m - i + 1)))
        s = text[j]
        for k in range(lst):
           s += temp[randint(0, len(temp) - 1)]
        f.write(s + '\n')
        last -= lst
      s = str()
      for i in range(last):
        s += temp[randint(0, len(temp) - 1)]
      f.write(s + '\n')
      for i in range(m):
        f.write(text[i] + '\n')
  else:
    f.write("100000 100000\n")
    lst = 1900000
    for i in range (100000):
      last = randint(0, lst // (100000 - i) - 1)
      s = 'a'
      for j in range(last):
        s += temp[randint(0, len(temp) - 1)]
      f.write(s + '\n')
      lst -= last
    for i in range(100000):
      f.write('a\n')
  f.flush()
  os.system('std.exe < ' + inname + ' > ' + ansname)
```