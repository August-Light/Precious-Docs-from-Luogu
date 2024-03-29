zhy137036 的博客 <https://www.luogu.com.cn/blog/178294/fhq-treap>

FHQ-Treap 是平衡树中码量很小的一种。

- 特点
- 操作
  - 分裂
  - 合并
  - 插入
  - 删除
  - 查询排名
  - 根据排名找数
  - 前驱
  - 后继
- 复杂度
- 代码

## 特点

Treap 这个词是由 Tree 和 Heap 组合形成的，可以看出 Treap 是查找树和堆的结合，因此中文叫树堆。

每个结点除保存一个值两个子节点外，还保存一个随机权；不过不保存父结点，这就是 Treap 码量小的原因。

和其他平衡树一样，Treap 的中序遍历 值单调不减；而根据堆的性质，每个结点的权小于两个子结点的权。

Treap 分为有旋和无旋两种，而无旋 Treap 是由范浩强发明的，所以又叫 FHQ-Treap。

本文要介绍无旋 Treap。

## 操作

### 分裂

分裂操作是将一个 Treap 分成 $x,y$ 两个 Treap。其中 $x$ 中每一个结点的值都小于 $k$，而 $y$ 中每一个结点的值都大于等于 $k$。

举个例子（$k=4$）：

![](https://cdn.luogu.com.cn/upload/image_hosting/jubdyomh.png)

（上图中的数均代表值而非权或编号）

如果当前子树的根的值大于等于 $k$，就继续分裂左子树，并将分出来的右边的子树作为根节点的左子树；否则分裂右子树，并将分出来的左边的子树作为根节点的右子树。

上面这段话对应到上图就是，因为根节点的值 $5$ 大于等于 $k=4$，所以继续分裂以 $2$ 为根的子树，分出来 $\left\{1,2,3\right\}$ 和 $\left\{4\right\}$ 两个子树，并将 $\left\{4\right\}$ 作为 $5$ 的左子树；  
（继续分裂以 $2$ 为根的子树）因为 $2$ 小于 $k=4$，所以继续分裂以 $4$ 为根的子树，分出来 $\left\{3\right\}$ 和 $\left\{4\right\}$ 两个子树，并将 $\left\{3\right\}$ 作为 $2$ 的右子树。

（上面混淆了值和编号，是为了让句子简洁一些）

认真读完上面的话之后，可以看出要用递归实现。

函数 `split` 接受一个根和一个 $k$，返回分裂后的两棵子树的根。

```
struct pair{//要返回两个值，得用 pair
	int a,b;
	pair(int a_=0,int b_=0) { a=a_; b=b_; }//构造函数
};
pair split(int u,int k){//u 是根
	if(!u) return pair(0,0);//空树返回空
	if(key[u]<k){//如果值小于 k
		pair t=split(son[u][1],k);//分裂右子树
        //son[u][0]表示左子节点，son[u][1]表示右子节点
		son[u][1]=t.a;//将左边的子树作为根的右子树
		pushup(u);//重新计算子树大小
		return pair(u,t.b);//返回（结合上面的图应该可以理解）
	}else{//反之同理
		pair t=split(son[u][0],k);
		son[u][0]=t.b;
		pushup(u);
		return pair(t.a,u);
	}
}
```

### 合并

合并是将 $x,y$ 两个 Treap 合并为一个 Treap（$x$ 中的每一个结点的值都小于等于 $y$ 中每一个结点的值）

![](https://cdn.luogu.com.cn/upload/image_hosting/bheudsa4.png)

（其中绿色的数代表随机权）

比较两个树根的权，权大的作为权小的子结点，继续合并子树。

比如这张图，因为 $2>1$，所以将 $\left\{1,2,3\right\}$ 作为 $\left\{4,5,6,7,8\right\}$ 的根的左子树，继续合并 $\left\{1,2,3\right\}$ 和 $\left\{4\right\}$；  
因为 $2<3$，所以将 $\left\{4\right\}$ 作为 $\left\{1,2,3\right\}$ 的根的右子树，继续合并 $\left\{3\right\}$ 和 $\left\{4\right\}$。  
. . . . . . 

`merge` 函数接受两个参数，为两棵树的根；返回合并后的根。

```cpp
int merge(int u,int v){
	if(!u||!v) return u+v;//如果任何一棵树为空，返回另一棵树
	if(wei[u]<wei[v]){//比较随机权
		son[u][1]=merge(son[u][1],v);
        //将右子树和另一棵树合并，作为右子树
		pushup(u);//重新计算子树大小
		return u;//最终根为 u
	}else{//反之同理
		son[v][0]=merge(u,son[v][0]);
		pushup(v);
		return v;
	}
}
```

### 插入结点

比如，要插入一个值为 $k$ 的结点，怎么办呢？

先将申请一个新的结点，作为一棵树 $y$；并将原来的树分裂成 $x,z$ 两棵树。  
然后依次合并 $x,y,z$，就完成了。

```cpp
void insert(int k){
	key[++cnt]=k; wei[cnt]=rand1(); size[cnt]=1;
    //申请一个节点，作为一棵树，其大小为 1
	pair t=split(root,k);//以 k 为关键值分裂
	root=merge(merge(t.a,cnt),t.b);//依次合并，更新根
}
```

### 删除结点

删除比较巧妙，先将树分裂成 $x,y,z$ 三棵树；其中 $x$ 的每个结点的值均小于 $k$，$y$ 的每个结点的值均为 $k$，$z$ 的每个结点的值均大于 $k$。

然后目标就是要在 $y$ 中删除仅一个结点，怎么办呢？

做法是，直接合并 $y$ 的左右两棵子树，根节点就被删除掉了。

接下来的事情都知道了，依次合并 $x,y,z$。

```cpp
void eraser(int k){
	pair x,y;
	x=split(root,k);
    //将整棵树分为小于 k 的和大于等于 k 的
	y=split(x.b,k+1);
    //将大于等于 k 的分为等于 k 的和大于 k 的
	y.a=merge(son[y.a][0],son[y.a][1]);
    //直接合并 y.a 的两棵子树
	root=merge(x.a,merge(y.a,y.b));
    //依次合并起来，更新根
}
```

### 查询排名

直接分裂，并将值小于 $k$ 的树的大小加一，返回。

```cpp
int find1(int k){
	int re;
	pair t=split(root,k);//分裂
	re=size[t.a]+1;//将值小的树的结点数 +1 作为答案
	root=merge(t.a,t.b);//合并回去
	return re;//返回
}
```

### 根据排名找数

这次没法分裂合并了qwq。

![](https://cdn.luogu.com.cn/upload/image_hosting/izyszovf.png)

这次结点上的数就是结点编号了，因为结点的值和权没有用。

比如要找第 $3$ 个数，让我们从根节点开始。

首先，$1$ 号结点的左子树有 $4$ 个结点，那第 $3$ 个结点肯定在左子树了，于是去找左子树的第 $3$ 个结点；  
然后，发现 $2$ 号结点的左子树只有 $1$ 个结点，那第 $3$ 个结点肯定在右子树了，于是去找 $2$ 号结点的右子树的第 $1$ 个结点；  
接着，发现 $5$ 号结点的左子树有 $1$ 个结点，那第 $1$ 个结点肯定在左子树了，于是去找 $5$ 号结点的左子树的第 $1$ 个结点；  
最后，发现 $8$ 号结点的左子树只有 $0$ 个结点，那 $8$ 号结点自己肯定就是答案了，于是返回自己的值。

有循环和递归两种实现方式。

```cpp
int find2(int k){
	int pos=root;//当前结点初始为根
	while(pos){
		if(k==size[son[pos][0]]+1) return key[pos];
        //如果要找的正是当前结点，直接返回
		if(k<=size[son[pos][0]]) pos=son[pos][0];
        //如果要找的在左子树，当前节点转为左子节点
		else { k-=size[son[pos][0]]+1; pos=son[pos][1]; }
        //否则不仅当前结点转为右子节点，还要将 k 减去左子树大小+1
	}
}
```

```cpp
int find2(int u,int k){
	if(k==size[son[u][0]]+1) return key[u];
    //如果要找的正是当前结点，返回本节点
	if(k<=size[son[u][0]]) return find2(son[u][0],k);
    //如果要找的在左子树，返回左子树的第 k 个结点
	else return find2(son[u][1],k-size[son[u][0]]-1);
    //否则返回右子树的结点，还是要减去那个数
}
```

###  前驱

什么叫“小于 $k$，且最大的数”？不就是前面的那个数吗，所以直接查找排名比 $k$ 的排名少一的数。

```cpp
int lst(int k) { return find2(find1(k)-1); }
```

### 后继

值相同的结点可能不止一个，所以不能找后面的那个数了，但是可以将值加一，查询它的排名，并找到对应的值。

不好理解？看下面的例子：

```plain
1,1,2,2,4,4,5,5
```

要找 $2$ 的后继。  
先查询 $3$ 的排名，即小于 $3$ 的元素个数 $+1$，查询结果为 $5$。  
查询第 $5$ 个数，为 $4$。结束。

```cpp
int nxt(int k) { return find2(find1(k+1)); }
```

## 复杂度

对于一个数集，排序后就是 Treap 的中序遍历；而根肯定是随机权最小的结点，因此期望位置在正中间；所以这棵树期望平衡；所以复杂度为 $O(\log n)$。

## 完整代码

```cpp
#include<cstdio>
#define maxn 1100010
struct pair{
	int a,b;
	pair(int a_=0,int b_=0) { a=a_; b=b_; }
};
int read(){
    int ans=0; char ch=getchar();
    while(ch>'9'||ch<'0')ch=getchar();
    while(ch<='9'&&ch>='0'){
        ans=ans*10+ch-'0';
        ch=getchar();
    }
    return ans;
}
int key[maxn],wei[maxn],size[maxn],son[maxn][2];
int n,m,cnt,ans,seed=1,root,last;
int rand1() { return seed*=19260817; }
inline void pushup(int u)
	{ size[u]=size[son[u][0]]+size[son[u][1]]+1; }
pair split(int u,int k){
	if(!u) return pair(0,0);
	if(key[u]<k){
		pair t=split(son[u][1],k);
		son[u][1]=t.a;
		pushup(u);
		return pair(u,t.b);
	}else{
		pair t=split(son[u][0],k);
		son[u][0]=t.b;
		pushup(u);
		return pair(t.a,u);
	}
}
int merge(int u,int v){
	if(!u||!v) return u+v;
	if(wei[u]<wei[v]){
		son[u][1]=merge(son[u][1],v);
		pushup(u);
		return u;
	}else{
		son[v][0]=merge(u,son[v][0]);
		pushup(v);
		return v;
	}
}
void insert(int k){
	key[++cnt]=k; wei[cnt]=rand1(); size[cnt]=1;
	pair t=split(root,k);
	root=merge(merge(t.a,cnt),t.b);
}
void eraser(int k){
	pair x,y;
	x=split(root,k);
	y=split(x.b,k+1);
	y.a=merge(son[y.a][0],son[y.a][1]);
	root=merge(x.a,merge(y.a,y.b));
}
int find1(int k){
	int re;
	pair t=split(root,k);
	re=size[t.a]+1;
	root=merge(t.a,t.b);
	return re;
}
int find2(int k){
	int pos=root;
	while(pos){
		if(k==size[son[pos][0]]+1) return key[pos];
		if(k<=size[son[pos][0]]) pos=son[pos][0];
		else { k-=size[son[pos][0]]+1; pos=son[pos][1]; }
	}
}
int lst(int k) { return find2(find1(k)-1); }
int nxt(int k) { return find2(find1(k+1)); }
int main(){
	n=read(); m=read();
	for(int i=1;i<=n;i++){
		int a=read();
		insert(a);
	}
	for(int i=1;i<=m;i++){
		int o=read(),x; x=read();
		if(o==1) insert(x^last);
		if(o==2) eraser(x^last);
		if(o==3) { last=find1(x^last); ans^=last; }
		if(o==4) { last=find2(x^last); ans^=last; }
		if(o==5) { last=lst(x^last); ans^=last; }
		if(o==6) { last=nxt(x^last); ans^=last; }
	}
	printf("%d\n",ans);
	return 0;
}
```

[记录](/record/32383047)