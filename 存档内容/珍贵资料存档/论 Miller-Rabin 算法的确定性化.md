It's LUNATIC time! <https://www.luogu.com.cn/blog/wangrx/miller-rabin>

## 前言

在刚开始学习 Miller-Rabin 的时候，我对算法本身的随机性感到相当程度的不满。

然后，百度百科上的一段话便吸引了我的注意。

> 卡内基梅隆大学的计算机系教授 Gary Lee Miller 首先提出了基于广义黎曼猜想的确定性算法，  
> 由于广义黎曼猜想并没有被证明，其后由以色列耶路撒冷希伯来大学的 Michael O. Rabin 教授作出修改，  
> 提出了不依赖于该假设的随机化算法。

于是便有了几个问题：
- 这个确定性算法到底是什么？
- 既然算法被随机化仅仅是因为广义黎曼猜想未被证明，    
  那么是否意味着它至少在 OI 范围内（$2^{64}$ 以内） 是正确的，可以用于 OI？
- 是否有简洁明快的方法，在 OI 范围内实现 **确定性** 判素？

相信不少初学者都有与我一样的问题，  
这篇博文将介绍 Miller Rabin 算法及其确定性化，解决这几个疑问。

前置芝士：[乘法逆元](https://www.luogu.com.cn/problem/P5431)

## 费马素性检验

素性检验最暴力的方法是 $\Theta(\sqrt{n})$ 的，为了应付 $2^{64}$ 以内的数据，我们需要一个更迅速的方法来判定素数。 

具体的，我们有费马小定理：
> 若 $p$ 为素数，对于所有的 $1\le a<p$，有 $a^{p-1}\equiv 1\pmod{p}$ 。

考虑它的逆否命题：
> 若存在 $1\le a<p$，使得 $a^{p-1}\not\equiv 1\pmod{p}$，则 $p$ 一定不是素数。

那我们可以在 $[1,p-1)$ 内随便选取几个数，快速幂来检验 $p$ 是否是素数。

- 若存在快速幂的结果不是 $1$，则 $p$ 一定不是素数。
- 否则，$p$ 大概率是一个素数。

这一方法称为 **费马素性检验** 。

## 二次探测定理与 Miller Rabin 素性检验

遗憾的是，费马素性检验存在一个问题：  
有的合数 $p$ 也满足 $a^{p-1}\equiv 1\pmod{p} \quad(1\le a<p)$，  
这样的数被称为卡迈克尔数（Carmichael Number），又称为费马伪素数。  
例如，$561=3\times 11\times 17$ 就是一个卡迈克尔数。  
在 $10^9$ 以内，这样的数有 $646$ 个，显然 $2^{64}$ 以内不能通过打表来满足素性检测的要求。

这迫使我们去优化费马素性检验。具体地，我们有二次探测定理：

> 若 $p$ 为奇素数，则 $x^2\equiv 1\pmod{p}$ 的解为 $x\equiv\pm 1\pmod{p}$。

若 $p$ 是奇数，显然 $p-1$ 是偶数，可以这样优化算法的正确性：
- 设选取的底数为 $a$，初始化指数 $d=p-1$。
- 快速幂判定 $a^d\bmod{p}$ 是否为 $1$，不是 $1$ 则 $p$ 为合数。
- 否则，重复 $d\leftarrow \dfrac{d}{2}$ 直到 $d$ 为奇数或 $a^d\not\equiv 1\pmod{p}$
- 最后，若 $a^d\not\equiv\pm 1\pmod{p}$，则 $p$ 为合数，否则 $p$ 大概率是个素数。

```cpp
typedef unsigned long long ull;
typedef unsigned int word;
bool check(const word a,const ull p){
	ull d=p-1,get=pow(a,d,p);
	if(get!=1) return 1;//特判 d=p-1 的情况
	while((d&1)^1)
		if(d>>=1,(get=pow(a,d,p))==p-1) return 0;//先 d/=2，再计算快速幂
		else if(get!=1) return 1;
	return 0;
}
```

初学者写 Miller-Rabin 时可能会犯的错误：
- 没有特判 $2^{p-1}\equiv -1\pmod{p}$ 的情况。
- 先计算 $a^d\bmod{p}$，再 $d\leftarrow \dfrac{d}{2}$，导致忽略了 $d$ 为奇数的情况。

若随机选取 $k$ 个底数，Miller-Rabin 的错误率为 $4^{-k}$。  
也就是说，选取 $20\sim 40$ 个底数，正确率是可以接受的。

## 固定底数与确定性检验

Miller-Rabin 的时间复杂度为 $\Theta(k\log^2 p)$，实际是用于获取高达 $2^{1024}$ 的 “工业” 素数的。  
在 OI 的 $2^{64}$ 范围内，我们可以将其确定性化。

> 如果我们假设广义 Riemann 猜想（GRH）成立，那么证明 $n$ 是素数就只需要验证  
> “ $n$ 可以通过以 $2,3,4,\cdots \lfloor 2(\log n)^2\rfloor$ 为底的 Rabin-Miller 测试 ”了。  
> 这个算法，如果能严格证明的话，运行时间是 $O((\log n)^5)$。  
> ——摘自知乎回答[如何编程判断一个数是否是质数？](https://www.zhihu.com/question/308322307)

$n=2^{64}$ 时，$\lfloor 2(\log n)^2\rfloor=8192$，  
即使 GRH 成立，改造后算法的运行时间已经不可接受了。  

但是，就算 GRH 不成立，通过选取几个固定的底数，  
我们依然能在一定的范围内实现确定性判素。

- 对于 $2^{32}$ 以内的判素，选取 $2,7,61$ 三个底数即可。
- 对于小于 $2^{64}$ 以内的判素，选取 $2,325,9375,28178,450775,9780504,1795265022$ 七个底数即可。
- 如果是考场上，选取 $2,3,5,7,11,13,17,19,23,29,31,37$   
  （也就是前十二个质数）作为底数即可，它适用于 $2^{78}$ 以内的判素。  
  对于选取前 $n$ 个质数作为底数的适用范围，详见 [OEIS](https://oeis.org/A014233)  

由此我们得到了最终的固定判素的代码（需要 C++ 11）：
```cpp
typedef unsigned long long ull;
typedef unsigned char byte;
const byte test[]={2,3,5,7,11,13,17,19,23,29,31,37};
bool miller_rabin(const ull p){
	if(p>40){
    	for(word a:test)
			if(check(a,p)) return 0;
        return 1;
    }
    for(word a:test)
		if(p==a) return 1;
	return 0;
}
```
## 参考资料
- 知乎回答[如何编程判断一个数是否是质数？](https://www.zhihu.com/question/308322307)
- [维基百科](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test#Deterministic_variants_of_the_test)
- [Deterministic variants of the Miller-Rabin primality test](https://miller-rabin.appspot.com)


希望借我这篇博文，确定性判素的 miller-rabin 能在 OI 中普及。  
望大家学习时不要人云亦云、浅尝辄止。