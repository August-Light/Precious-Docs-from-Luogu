菜鸡 ut 的 blog <https://www.luogu.com.cn/blog/250637/art-is-art>

省流：这篇文章将介绍一种名为“ART 分解”的技巧，并阐释其两个不同的应用（较复杂的一种附有代码）。ART 分解这玩意有很多长得都差不多复杂度分析也都大差不差的写法，网上看到有细节差异的话十分正常，这里介绍我个人所采用的一种。

前置体系：WRM。不过不知道也不影响，反正既然不知道那平时应该都在用。

+ 什么是 ART 分解？

给定 $n$ 个点的有根树和界值 $B$。将所有子树大小不超过 $B$ 但父亲子树大小超过 $B$（根有很多种特判方案，我采用的是不将根加入分解）的点取出，容易发现所有这些点的子树两两不交。将其从原树的结构上分离，这样一个一棵大树下面挂了一堆小树的新结构就是 ART 分解。

+ ART 分解的点数上有没有什么性质？

十分显然小树至多 $B$ 个点。每个大树上的叶子原子树大小一定不少于 $B+1$，于是至多 $\lfloor\frac n{B+1}\rfloor$ 个叶子。注意大树的节点数是没有什么好的规模的。

+ ART 分解有什么用？

取 $B=\Theta(\sqrt n)$ 这就是一种奇怪的树分块，由于没有研究过不敢定论有没有什么说法。

它真正擅长的是：$B=\Theta(\log n)$，取对于不少问题，在规模较小的树上存在依赖二进制压位的加速方法，采用 ART 分解可以做一个很好的复杂度平衡。当然 Top Cluster 往往也能解决其中的一部分，这是另论了。

以下我们通过两个例子来展示 ART 分解是如何工作的。

---

1. LA

简单来说就是多次给定 $u$ 和 $k$ 查 $u$ 的 $k$ 级祖先。

由于不能保证所有读者都会长剖那一套所以简单介绍一下：对于每个节点维护其最深的儿子，这样会把树剖分成若干条链（和重剖差不多）。然后我们对每条这样剖出来的长链向上延长一倍，等于是额外维护链的一堆祖先。现在假设我们要求 $u$ 的 $k$ 级祖先，那么我们先倍增跳最高的一层跳到 $v$，那么显然有 $v$ 到 $u$ 的距离超过 $v$ 到所求祖先的距离，换句话说所求祖先一定在 $v$ 的链或其反向延长上，直接查表即可。复杂度 $O(n\log n)-\Theta(1)$，瓶颈是预处理倍增数组。

现在我们考虑把它稍微优化一下。我们注意到，实际上我们只需要解决 $u$ 是叶子的情况即可，其他情况可以简单转化过去。如此一来我们就只需要维护所有叶子的倍增数组就可以了，dfs 预处理的时候维护一个栈记祖先就好，复杂度变成了 $O(p\log n)-\Theta(1)$，其中 $p$ 是叶子数量。

那接下来就可以上 ART 了。我们发现把上面那个做法丢给大树直接秒了。而对于小树，它的大小是 $O(\log n)$，我们套娃一层，现在小树大小就只剩下 $O(\log\log n)$ 了。我们给点重标号后考虑直接暴力用一个二进制单位压下每个点的所有祖先（需要 $O(\log\log n\log\log\log n)$ 个比特，可以整），然后它就可以用位运算常数时间盘出来了。

于是我们就做完了。

---

我们考虑上面那个问题里为什么 ART 看起来非常舒适。核心原因是，复杂度瓶颈被给到了叶子数量。那如果复杂度就是只能依赖在 $n$ 上是不是就不能 ART 了呢？答案是否定的。

让我们来看看大树出问题的核心在哪里：节点太多。但它的叶子节点明明很少，这启示我们，树里有很多度为 1 的点。我们考虑，如果一个点只有一个儿子，那它和它的儿子，甚至再往下儿子，其实构成了树上的一条链，它可以和链顶链爹的边被压缩到单一条上。这需要我们写一个链上的数据结构来维护这部分的贡献。处理完之后我们的大树点数就下来了，可以正常搞了。但说起来容易做起来阴间，这个链上的数据结构需要处理长度为 $O(n)$ 的链，但我们往往并不能以这个复杂度直接完成任务，于是还要再对这条链进行序列分块（同样按 $\Theta(B)$），块内块间分开处理。以下我们将会把树分为三个部分：大树（指压链后），小树，链。

这引出了我们的第二道例题。

---

2. 树上 UFDS

简单来说就是支持两种操作：连接一个点与它的父亲，以及查询一个点沿着被连上的边能走到的最高父亲。本质是一个给定最终合并结构的并查集。

老规矩 ART，大树裸 UFDS 就是对的，记得路径压缩（不会的话可以参考代码）。由于感觉可能部分读者对并查集的复杂度不甚清晰，这里给出分析。UFDS 的复杂度在 $O(m\alpha(m,n))$，其中 $\alpha(m,n)=\min\{z|A(z,4\lceil\frac mn\rceil)\gt\log_2n\}$（没错，只是形式有关，这东西根本就不是什么简单的反函数）。换句话说这个东西只要 $m$ 稍微大一点比如 $\Omega(n\log^*n)$ 那 $\alpha$ 一整个就是真常数。于是这个东西是不可能炸到 $\Omega(n\log n+m)$ 向上的。当然，如果不放心，写个启发式合并（直接维护标号，这里就不要再并查集了）也不是不行。ART 在大树上的核心优势在于：由于链本身的连通性相当好刻画，所以其贡献会比 Top Cluster 界点连通性这个事情好刻画也好维护很多。

至于小树，我们搜出每个点的 dfs 序，初始设定所有点和父亲断边（压在一个二进制状态里）。每次连边的时候只要把点在 dfs 序上的位置所对应的状态位置为 1 即可。另外维护每个点所有祖先在二进制状态里的位置，每次查询就把两个东西按位与然后取最高位即可。

而链分块后也相当好处理：块内直接二进制压每个点和前一个点的连通性，块间裸 UFDS，即可。

很显然这几个部分拼起来听着都已经不是很阳间了，于是 ut 跑去实现了一下，文末附代码。跑得挺慢的。

---

总而言之，ART 分解好玩捏。

```cpp
class UFDS_SMALL {
 private:
  constexpr static int W = 6;
  using ull = unsigned long long;
  ull st;
  vector<int> dfn, pos;
  vector<ull> anc;
  vector<vector<int>> es;

 public:
  UFDS_SMALL() {}
  explicit UFDS_SMALL(int n) { init(n); }
  void init(int n) {
    dfn.resize(n), anc.resize(n), pos.resize(n), es.resize(n);
    st = (n == (1 << W) ? 0 : (1ull << n)) - 1;
  }
  void add_edge(int u, int v) { es[u].push_back(v), es[v].push_back(u); }
  void build(int s) {
    int tot = 0;
    anc[s] = 1;
    function<void(int, int)> dfs = [&tot, &dfs, this](int x, int fa) {
      pos[tot] = x, dfn[x] = tot++;
      for (int y : es[x])
        if (y != fa) anc[y] = anc[x] | 1ull << tot, dfs(y, x);
    };
    dfs(s, -1);
  }
  void link(int x) { st ^= 1ull << dfn[x]; }
  int findf(int x) const {
    ull s = anc[x] & st;
    return s ? pos[__lg(s)] : -1;
  }
};
class UFDS_BIG {
 private:
  vector<int> fa, msk;
  int findp(int x) {
    while (fa[x] >= 0) {
      int f = fa[fa[x]];
      if (f >= 0) fa[x] = f;
      x = fa[x];
    }
    return x;
  }

 public:
  UFDS_BIG() {}
  explicit UFDS_BIG(int n) { init(n); }
  void init(int n) {
    fa = vector<int>(n, -1), msk.reserve(n);
    for (int i = 0; i < n; ++i) msk.push_back(i);
  }
  void link(int u, int v) {
    if (fa[u = findp(u)] > fa[v = findp(v)]) swap(u, v), swap(msk[u], msk[v]);
    fa[v] += fa[u], fa[u] = v;
  }
  int findf(int x) { return msk[findp(x)]; }
};
class UFDS_LINE {
 private:
  constexpr static int W = 6, S = (1 << W) - 1;
  using ull = unsigned long long;
  vector<ull> st;
  UFDS_BIG ufds;
  static ull shr(int x) { return (++(x &= S) <= S ? (1ull << x) : 0) - 1; }

 public:
  UFDS_LINE() {}
  UFDS_LINE(int n) { init(n); }
  void init(int n) {
    int h = ((n - 1) >> W) + 1;
    st = vector<ull>(h, -1), ufds.init(h);
    st[h - 1] = shr(n - 1);
  }
  void link(int x) {
    int z = x & S, b = x >> W;
    if (!(st[b] ^= 1ull << z) && b) ufds.link(b, b - 1);
  }
  int findf(int x) {
    int b = x >> W;
    ull s = st[b] & shr(x);
    if (!s && b) s = st[b = ufds.findf(b - 1)];
    return s ? b << W | __lg(s) : -1;
  }
  int findr() { return findf((st.size() << W) - 1); }
};
class UFDS_TREE {
 private:
  constexpr static int W = 6, S = 1 << W;
  using ull = unsigned long long;
  vector<int> sz, fa;
  vector<vector<int>> es;
  vector<pair<int, int>> bel;
  vector<variant<UFDS_SMALL, UFDS_LINE>> st;
  vector<vector<int>> pos;
  vector<int> ufid, ufps;
  UFDS_BIG ufds;

 public:
  UFDS_TREE() {}
  explicit UFDS_TREE(int n) { init(n); }
  void init(int n) {
    bel.resize(n), st.resize(n), es.resize(n);
    fa.resize(n), sz.resize(n), ufid.resize(n), pos.resize(n);
  }
  void add_edge(int u, int v) { es[u].push_back(v), es[v].push_back(u); }
  void build(int s) {
    vector<int> rts;
    function<void(int)> dfs = [&rts, &s, &dfs, this](int x) {
      sz[x] = 1;
      for (int y : es[x]) erase(es[y], fa[y] = x), dfs(y), sz[x] += sz[y];
      if (x != s && sz[x] <= S) return;
      vector<int> tmp;
      tmp.reserve(es[x].size());
      for (int y : es[x])
        if (sz[y] <= S)
          rts.push_back(y);
        else
          tmp.push_back(y);
      es[x] = tmp;
    };
    dfs(s);
    for (int z : rts) {
      int tot = 0;
      function<void(int)> dfs = [&tot, &z, &dfs, this](int x) {
        bel[x] = {z, tot++}, pos[z].push_back(x);
        for (int y : es[x]) dfs(y);
      };
      dfs(z), st[z] = UFDS_SMALL(tot);
      dfs = [&z, &dfs, this](int x) {
        for (int y : es[x])
          get<UFDS_SMALL>(st[z]).add_edge(bel[x].second, bel[y].second), dfs(y);
      };
      dfs(z), get<UFDS_SMALL>(st[z]).build(0);
    }
    int tot = 0;
    dfs = [&tot, &dfs, this](int x) {
      get<UFDS_LINE>(st[x]);
      ufid[x] = tot++, ufps.push_back(x);
      for (int y : es[x]) {
        int z = y, c = 0;
        while (es[z].size() == 1u)
          bel[z] = {y, c++}, pos[y].push_back(z), z = es[z][0];
        bel[z] = {y, c++}, pos[y].push_back(z);
        st[y] = UFDS_LINE(c), es[y] = es[z], dfs(y);
      }
    };
    bel[s] = {s, 0}, st[s] = UFDS_LINE(1), pos[s] = {s};
    sz[s] = S + 1, dfs(s), ufds.init(tot);
  }
  int findf(int x) {
    auto [rt, id] = bel[x];
    if (sz[x] <= S) {
      int y = get<UFDS_SMALL>(st[rt]).findf(id);
      if (~y) return pos[rt][y];
      x = fa[rt], tie(rt, id) = bel[x];
    }
    int y = get<UFDS_LINE>(st[rt]).findf(id);
    if (~y) return pos[rt][y];
    x = ufps[ufds.findf(ufid[bel[fa[rt]].first])];
    return pos[x][get<UFDS_LINE>(st[x]).findr()];
  }
  void link(int x) {
    auto [rt, id] = bel[x];
    if (sz[x] <= S) return get<UFDS_SMALL>(st[rt]).link(id);
    get<UFDS_LINE>(st[rt]).link(id);
    if (!~get<UFDS_LINE>(st[rt]).findr())
      ufds.link(ufid[rt], ufid[bel[fa[rt]].first]);
  }
};
```