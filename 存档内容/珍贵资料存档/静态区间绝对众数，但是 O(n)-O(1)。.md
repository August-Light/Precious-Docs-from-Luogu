菜鸡 ut 的 blog <https://www.luogu.com.cn/blog/250637/mvote>

这里介绍一种理论最优复杂度的算法。

在阅读本文之前，您至少需要掌握：

+ 摩尔投票。

+ 猫树，或者其他写法的二区间合并。

我们将把算法分为求值和检验两部分依次阐述。

1. 求值

等于是要求个区间摩投出来。

序列按 $\log n$ 分块，块间猫，如此再在块内递归一层，我们接下来只需要 $O(n)-O(1)$ 地解决规模为 $O(\log\log n)$ 的问题。

我们考虑对每一个这样的块中块进行离散化（使用哈希表而不是排序来获得线性的复杂度），这样本质不同的块至多只有 $O((\log\log n)^{\log\log n})=O(2^{\log\log n\times\log\log\log n})$ 种，再考虑所有本质不同的询问，一共所有不同的状态数实际上只有 $O(2^{\log\log n\times\log\log\log n}(\log\log n)^2)$ 种，这个东西总之是亚线性的，我们全都嗯预处理出来就可以了。预处理使用摩投递推的话单次转移是常数时间，换而言之预处理的复杂度也就是这个鬼东西。至于状态的对应，我们二进制压缩一下就可以快速检索了。

总复杂度 $O(n)$。

2. 检验

实际上这部分比求值难……

首先有一个简单的结论是对于固定的 $x$ 绝对众数为 $x$ 的所有区间并集的长度是 $O(c_x)$ 的，其中 $c_x$ 表示 $x$ 的出现次数。证明就考虑一个加一减一的折线图然后顺次考虑每个峰然后分别砍掉就可以开归了。

我们考虑对于每一个并集中的区间 $[l,r]$，我们求出 $x$ 出现与否的状态在 $[l-1,r]$ 上的全局前缀和，这样所有被计算的前缀和数量也是 $O(c_x)$ 的，用哈希表存储。如此一来，对于检验 $x$ 为 $[l,r]$ 的绝对众数，我们只需要计算 $l-1$ 和 $r$ 位置上的前缀和即可。如果不存在，说明本身 $[l,r]$ 一定不是绝对众数为 $x$ 所有区间并集的子集，那么直接返回否即可。

至于这个找区间的实现……大概是我比较菜所以感觉非常阴间，我们考虑 $x$ 出现的第 $i$ 个位置为 $p_i$，那么我们维护 $2i-p_i$ 的前缀最小值和后缀最大值然后双指针对于每个位置求出“绝对众数为 $x$ 的区间最左边的 $x$ 在这个位置”的所有合法区间的并集形式，然后就可以了。这部分感觉非常抽象，建议直接看代码。

总复杂度 $O(n)$。

综上所述我们就在 $O(n)-O(1)$ 的复杂度内解决了这个问题。

```cpp
#include <bits/stdc++.h>
using namespace std;
struct custom_hash {
  static uint64_t splitmix64(uint64_t x) {
    x += 0x9e3779b97f4a7c15;
    x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
    x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
    return x ^ (x >> 31);
  }
  size_t operator()(uint64_t x) const {
    static const uint64_t FIXED_RANDOM =
        chrono::steady_clock::now().time_since_epoch().count();
    return splitmix64(x + FIXED_RANDOM);
  }
};
typedef unordered_map<int, int, custom_hash> hmap;
constexpr int N = 5e4 + 9, B = 16, H = 4, W = 2;
constexpr int S = (N >> H) + 1, Z = 1 << (H * W);
int n, m, c[N];
struct mvt {
  int x, c;
  mvt() : x(0), c(0) {}
  mvt(int x, int c) : x(x), c(c) {}
  mvt(int x) : x(x), c(1) {}
  friend mvt operator+(const mvt& a, const mvt& b) {
    if (a.x == b.x) return {a.x, a.c + b.c};
    if (a.c > b.c) return {a.x, a.c - b.c};
    return {b.x, b.c - a.c};
  }
};
template <int L, int N>
struct ct_tree {
  mvt val[N], ct[L][N];
  void build(mvt* a, int n) {
    copy_n(a, n, val);
    for (int h = 0, s = 1; s < n; ++h, s <<= 1)
      for (int i = s; i <= n; i += s << 1) {
        ct[h][i - 1] = val[i - 1];
        for (int j = i - 1; j > i - s; --j)
          ct[h][j - 1] = val[j - 1] + ct[h][j];
        if (i == n) break;
        ct[h][i] = val[i];
        for (int j = i + 1; j < i + s && j < n; ++j)
          ct[h][j] = ct[h][j - 1] + val[j];
      }
  }
  mvt query(int l, int r) const {
    if (l == r) return val[l];
    int h = __lg(l ^ r);
    return ct[h][l] + ct[h][r];
  }
};
vector<int> dsct(int* a, int n) {
  hmap tmp;
  vector<int> ret;
  for (int i = 0, t = 0; i < n; ++i) {
    int& x = tmp[a[i]];
    if (!x) ret.push_back(a[i]), x = ++t;
    a[i] = x - 1;
  }
  return ret;
}
hmap s[N];
vector<int> idv;
mvt qans[Z][H][H];
struct iiblk {
  vector<int> idv;
  int st;
  void build(int* c, int n) {
    idv = dsct(c, n), st = 0;
    for (int i = 0; i < n; ++i) st |= c[i] << (i * W);
  }
  mvt query(int l, int r) const {
    mvt ans = qans[st][l][r];
    ans.x = idv[ans.x];
    return ans;
  }
};
struct iblk {
  ct_tree<W, B> bs;
  iiblk ib[B];
  void build(int* c, int n) {
    int z = ((n - 1) >> W) + 1;
    vector<mvt> val(z);
    for (int i = 0; i < n; ++i) val[i >> W] = val[i >> W] + c[i];
    bs.build(val.data(), z);
    for (int i = 0; i < z; ++i) ib[i].build(c + (i << W), min(n - (i << W), H));
  }
  mvt query(int l, int r) const {
    int lb = l >> W, rb = r >> W;
    if (lb == rb) return ib[lb].query(l & (H - 1), r & (H - 1));
    mvt ret = ib[lb].query(l & (H - 1), H - 1) + ib[rb].query(0, r & (H - 1));
    if (lb + 1 < rb) ret = ret + bs.query(lb + 1, rb - 1);
    return ret;
  }
} ib[S];
ct_tree<B, S> ct;
void init() {
  for (int s = 0; s < Z; ++s)
    for (int l = 0; l < H; ++l) {
      mvt tmp;
      for (int r = l; r < H; ++r)
        qans[s][l][r] = tmp = tmp + ((s >> (r * W)) & (H - 1));
    }
  idv = dsct(c, n);
  vector<vector<int>> pos(n);
  for (int i = 0; i < n; ++i) pos[c[i]].push_back(i);
  for (int t = 0; t < n; ++t) {
    vector<int>& occ = pos[t];
    int k = occ.size();
    if (!k) continue;
    vector<int> val(k);
    for (int i = 0; i < k; ++i) val[i] = 2 * i - occ[i];
    basic_string<bool> pmn(k, false), smx(k, false);
    vector<int> sufmx(k);
    vector<pair<int, int>> itv, tmp;
    for (int i = 0, p = INT_MAX; i < k; ++i)
      if ((pmn[i] = val[i] < p)) p = val[i];
    for (int i = k - 1, p = INT_MIN; ~i; --i)
      if ((smx[i] = val[i] > (sufmx[i] = p))) p = val[i];
    for (int i = 0, j = 0, w = 0; i < k; ++i) {
      if (!pmn[i]) continue;
      while (j < k && (sufmx[j] >= val[i])) ++j;
      while (w < k && (w < i || !smx[w])) ++w;
      if (j < i) break;
      int l = max(i ? occ[i - 1] + 1 : 0, occ[i] - (val[w] - val[i]));
      int r =
          min(j < k - 1 ? occ[j + 1] - 1 : n - 1, occ[j] + (val[j] - val[i]));
      itv.emplace_back(l, r);
    }
    int l = -1e9, r = l - 1;
    for (auto [pl, pr] : itv)
      if (pl > r + 1) {
        if (l <= r) tmp.emplace_back(l - 1, r);
        l = pl, r = pr;
      } else
        r = max(r, pr);
    if (l <= r) tmp.emplace_back(l - 1, r);
    for (int i = 0; auto [l, r] : tmp)
      for (int j = l; j <= r; ++j) {
        while (i < k && occ[i] <= j) ++i;
        s[t].emplace(j, i);
      }
  }
  int z = ((n - 1) >> H) + 1;
  vector<mvt> val(z);
  for (int i = 0; i < n; ++i) val[i >> H] = val[i >> H] + c[i];
  ct.build(val.data(), z);
  for (int i = 0; i < z; ++i) ib[i].build(c + (i << H), min(n - (i << H), B));
}
int gsum(int p, int x) {
  auto it = s[x].find(p);
  if (it == s[x].end()) return -1;
  return it->second;
}
bool chk(int l, int r, int x) {
  int sl = gsum(l - 1, x), sr = gsum(r, x);
  if (!~sl || !~sr) return false;
  return ((sr - sl) << 1) > r - l + 1;
}
int qbase(int l, int r) {
  int lb = l >> H, rb = r >> H;
  if (lb == rb) return ib[lb].query(l & (B - 1), r & (B - 1)).x;
  mvt ret = ib[lb].query(l & (B - 1), B - 1) + ib[rb].query(0, r & (B - 1));
  if (lb + 1 < rb) ret = ret + ct.query(lb + 1, rb - 1);
  return ret.x;
}
int query(int l, int r) {
  int x = qbase(l, r);
  return chk(l, r, x) ? idv[x] : 0;
}
signed main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  cin >> n >> m;
  for (int i = 0; i < n; ++i) cin >> c[i];
  init();
  for (int l, r; m; --m) {
    cin >> l >> r;
    cout << query(--l, --r) << '\n';
  }
  return cout << flush, 0;
}
```