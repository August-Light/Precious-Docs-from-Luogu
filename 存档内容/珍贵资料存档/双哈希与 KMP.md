摘自 <https://www.luogu.com.cn/training/276497>

### 双哈希的参考实现

以下代码采用了双哈希，可以计算一个字符串 $S$ 某个子串的哈希值：

```cpp
const int MAXN= 1e6 + 3;
typedef unsigned int       u32;
typedef unsigned long long u64;
const u32 BAS1 = 13331, MOD1 = 1e9 + 7;
const u32 BAS2 =   131, MOD2 = 1e9 + 9;
int H1[MAXN], P1[MAXN];
int H2[MAXN], P2[MAXN];
u64 get(int l, int r){
    int h1 = (H1[r] - 1ull * H1[l - 1] * P1[r - l + 1] % MOD1 + MOD1) % MOD1;
    int h2 = (H2[r] - 1ull * H2[l - 1] * P2[r - l + 1] % MOD2 + MOD2) % MOD2;
    return 1ull * h1 << 32 | h2;
}
void init(int n, char S[]){
    P1[0] = P2[0] = 1;
    for(int i = 1;i <= n;++ i)
        H1[i] = (1ull * H1[i - 1] * BAS1 + S[i]) % MOD1,
        H2[i] = (1ull * H2[i - 1] * BAS2 + S[i]) % MOD2,
        P1[i] = 1ull * P1[i - 1] * BAS1 % MOD1,
        P2[i] = 1ull * P2[i - 1] * BAS2 % MOD2;
}
```

- 通过 `init(n, S[])` 进行初始化；
- 通过 `get(l, r)` 查询 $S[l\cdots r]$ 这段子串的哈希值。

实现细节：

- $\mathrm{MOD_1}$ 和 $\mathrm{MOD_2}$ 的取值不宜太大，不然可能会有溢出风险。可以取在 $10^9$ 数量级；
- 代码本质上是用 $\lang\mathrm{BASE}_1,\mathrm{MOD_1}\rang$ 和 $\lang\mathrm{BASE}_2,\mathrm{MOD_2}\rang$ 对字符串分别进行一次哈希，接着在 `get` 函数内将结果合并。

双哈希在查询次数 $q=10^6$ 的情况下，碰撞概率约为 $\dfrac{q}{\mathrm{MOD_1}\times \mathrm{MOD_2}}\approx 10^{-12}$，可以忽略不计。

### 关于 KMP 部分的补充解释

考虑到这部分理解难度确实较大，于是在这里再写一下解释。

首先我们定义了前缀数组 $\pi$，$\pi_i$ 表示 $S$ 的前缀 $S[1\cdots i]$ 的最长 border 的长度。换言之，$S[1\cdots i]$ 的最长 border 就是 $S[1\cdots \pi_i]$。

关于 $\pi$ 的求法，可以这样总结：

- 记 $A$ 为前缀 $S[1\cdots i]$ 的 border 长度集合，$A=\{a_1,a_2,\cdots,a_p\}$，也就是 $S[1\cdots i]$ 的所有 border 为 $S[1\cdots a_1],S[1\cdots a_2],\cdots,S[1\cdots a_p]$。为了方便，我们约定 $a_1>a_2>\cdots>a_p$；
- 记 $B$ 为前缀 $S[1\cdots i-1]$ 的 border 长度集合，$B=\{b_1,b_2,\cdots,b_q\}$，也就是 $S[1\cdots i-1]$ 的所有 border 为 $S[1\cdots b_1],S[1\cdots b_2],\cdots,S[1\cdots b_q]$。为了方便，我们约定 $b_1>b_2>\cdots>b_q$。

我们在课上已经证明了如下两个重要结论：

- 如果 $x\in A$ 且 $x>0$，那么必须要有 $(x-1)\in B$。这条结论的证明可以用下图说明：
![](https://cdn.luogu.com.cn/upload/image_hosting/18zpo4mv.png)

- $\pi_{a_1}=a_2,\pi_{a_2}=a_3,\cdots$ 以及 $\pi_{b_1}=b_2,\pi_{b_2}=b_3,\cdots$，这是利用一个字符串的次长 border 一定是它最长 border 的最长 border 来证明。大家可以画图理解。

接着来解释代码（下图代码计算了字符串 $S$ 的 $\pi$ 数组，存到了 $\text{N}$ 里）：

```cpp
    scanf("%s", S + 1), n = strlen(S + 1);
    for(int i = 2;i <= n;++ i){
        int p = N[i - 1];
        while(S[p + 1] != S[i] && p != 0) p = N[p];
        if(S[p + 1] == S[i]) N[i] = p + 1;
    }
```

- 首先，$\pi_1=0$。可以不用计算。
- 接着令 $p\gets\pi_{i-1}$，对应上文我们给出的集合 $A,B$ 的定义，也就是 $p\gets b_1$。
- 因为如果有 $x\in A$ 就一定要有 $(x-1)\in B$，而 $p\in B$，于是我们检查 $p+1$ 是否在 $A$ 里。检查方式依然可以参考刚刚给出的图片，只需要比较 $S_{p+1}$ 位置是否与 $S_i$ 位置相同即可。
- 如果不满足条件，则 $p\gets \pi_p$，对应着上文所述 border 的第二条结论。一开始 $p\gets b_1$，那么 $p\gets \pi_{b_1}$ 变成了 $b_2$，$p\gets \pi_{b_2}$ 变成了 $b_3$，以此类推。
- 有一个特殊情况就是 $p$ 变成了 $0$。因为 $\pi_0=0$，为了防止无限循环应该直接跳出。但此时 $A$ 中可能有长度为 $1$ 的 border，也可能没有非空 border。于是需要检查 $S_{p+1}$ 和 $S_i$ 的关系。这一步（`if(S[p + 1] == S[i]) N[i] = p + 1;`）可以展开如下：
```cpp
if(p == 0){
  if(S[1] == S[i]) N[i] = 1; else N[i] = 0;
} else {
  N[i] = p + 1;
}
```

接下来是关于复杂度的说明。

根据第一条结论可以发现，$\pi_i\le \pi_{i-1}+1$。那么每次 $i$ 增加的时候 $\pi_i$ 最多增加 $1$。考虑那个 while 循环的时间复杂度。记 $\pi_i$ 在 $\pi_{i-1}$ 的基础上减少的值为 $d$，产生的循环次数一定不会大于 $d$。同时从概念上 $\pi_i$ 的值都是自然数。

也就是说，$+1$ 的次数是 $\mathcal O(n)$ 级别的，于是 $-1$ 的次数一定也是 $\mathcal O(n)$ 的。也就是 while 循环的总次数是 $\mathcal O(n)$ 次。于是总复杂度就是 $\mathcal O(n)$。