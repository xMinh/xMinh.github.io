---

layout: post
title: '蒟蒻的Link Cut Tree'
subtitle: '然而啥都不会'
date: 2018-04-09
categories: OI
cover: 'https://github.com/xMinh/xMinh.github.io/blob/master/pic/background/Other/%E9%9D%9E%E5%8A%A8%E6%BC%AB/t0136a882573eb91501.jpg?raw=true'
tags: 动态树

---

# 蒟蒻的Link Cut Tree

这里就不讲解太详细了，因为我觉得这个大佬的博客挺好的

[传送门](http://www.cnblogs.com/flashhu/p/8324551.html)

不过蒟蒻笔者的代码还是比较规范的，可以赏脸来瞧一瞧。

个人对link cut tree的理解就是一棵动态的树(森林)，你可以任意增边、删边、甚至像树剖一样维护路径上的信息，还可以有换根等等骚操作。

实现的方法当然就是我们的Splay了，我们Splay构建辅助树，当然构建要用到树剖的思想——一条**重链**就是一棵辅助树，辅助树按照在原树中的深度维护。那轻边怎么办？就由子节点所在的辅助树的根连向父节点就行了。但是这里要注意一点，就是“认父不认子”，在连接辅助树的轻边中，子节点储存父亲是谁，方便后面Access，但是父节点是不用储存**这个**子节点的信息的——注意，只是“这个子节点”，也就是说对于同一棵辅助树中的子节点还是要保存的。这样就能区分出每一棵辅助树了。

这里说一下一些link cut tree的基本操作，但是要先说一下对于splay部分的改动。

### Rotate

```javascript
void Rotate(int x) {
	int fa=f[x],gr=f[fa],kind=Get(x);	
	if (!Isroot(fa)) son[gr][son[gr][1]==fa]=x;
	son[fa][kind]=son[x][kind^1];
	if (son[x][kind^1]) f[son[x][kind^1]]=fa;
	son[x][kind^1]=fa;
	f[fa]=x;f[x]=gr;
	Pushup(fa);
	Pushup(x);
}
```

大家可以看到与普通的Rotate还是没有太大差异的，但是有一点就是当fa不为根时的特判要放在前面。因为我们link cut tree中用的是is_root的判定函数(下面会有)，如果先操作再判定就会出现差异，所以要区别于普通Splay的写法。

### Splay

```javascript
void Splay(int x) {
	q[++top]=x;
	for (int i=x;!Isroot(i);i=f[i]) q[++top]=f[i];
	while (top) Pushdown(q[top--]);
	for (int fa;!Isroot(x);Rotate(x))
		if (!Isroot(fa=f[x])) 
			Rotate(Get(fa)==Get(x)?fa:x);
}
```

link cut tree中的Splay要先从根节点一路Pushdown一下更新下来，至于为什么……好吧我也背的板子，但是感性理解一下，先这么写着吧。

### Access

这个我文章顶部那个传送门里面解析过程解析的已经比较详细了，建议大家去看一下。

这个函数的作用就是把根节点到指定节点的一条路径变成一条重链，也就是打通一条指定节点到根节点的路径，当然这条路径的顶部是根节点，底部是指定节点。

```javascript
void Access(int x) {
	for (int t=0;x;t=x,x=f[x]) {
		Splay(x);
		son[x][1]=t;
		Pushup(x);
	}
}
```

### Is root

判断当前节点是不是辅助树的根，运用了“认父不认子”的原理。

```javascript
bool Isroot(int x) {return son[f[x]][0]!=x && son[f[x]][1]!=x;}
```

### Make root

换根。当然是换原树的，而不是辅助树的。当你想操作一条路径的时候，就要先把这条路上的一个点变成根，然后再Access。原理就是先把指定节点与根节点Access，再把它转到根节点，然后再把它的左子树转到右子树上，就可以了。

```javascript
void Makeroot(int x) {
	Access(x);
	Splay(x);
	rev[x]^=1;
	swap(son[x][0],son[x][1]);
} 
```

### Find root

找根。当然是找指定节点在原树中的根(不要忘了link cut tree可能是森林)，而不是辅助树的。主要用于判断两点连通性。先把一个节点和根联通，再把它转到根节点上，之后一直往左儿子找就能找到根了。

```javascript
int Findroot(int x) {
	Access(x);
	Splay(x);
	while (son[x][0]) x=son[x][0];
	return x;
}
```

### Link

连接两个不联通的点。这里不是专指直接联通，间接联通也算联通。先Make root一下再用Find root判断，但要注意，这里不需要Push up(“认父不认子”原理)。

```javascript
void Link(int x,int y) {
	Makeroot(x);
	if (Findroot(y)==x) return;
	f[x]=y;
}
```

### Split

这个不是普通的Splay里面那个Split。这里的操作就是上文提到的“操作一条路径”，先把路径上的一个点设成根，然后再把另一个点与之打通，但要注意再打通后要用Splay更新一下路径信息。由于Access的性质，这样就得到了操作路径。

```javascript
void Split(int x,int y) {
	Makeroot(x);
	Access(y);
	Splay(y);
}
```

### Cut

这个可以来看一下。切断原树上的一条边，当然如果题目不能保证切边合法的话我们要特判一下。

怎么特判呢？这里传送门的大佬给出了三个特判，但[wgl大佬](https://a-failure.github.io/)觉得一个就够了

我们先Split一下，如果两者相连的话，此时x应该正好在y的左儿子上，而且x应没有右儿子。这样特判一下再切断就好了。

```javascript
void Cut(int x,int y) {
	Split(x,y);
	if (son[y][0]==x && !son[x][1]) {
		f[x]=son[y][0]=0;
		Pushup(y);
	}
}
```

### Modify

单点修改操作。这里解释一下在修改之前为什么要把它转上去——因为如果不转上去的话就要修改一堆点，这样只需要修改它自己就可以了。

```javascript
void Modify(int x,int y) {
	Access(x);
	Splay(x);
	a[x]=y;
	Pushup(x);
}
```

## Link Cut Tree例题

### [luogu3690 LCT模板](https://www.luogu.org/problemnew/show/P3690)

就是我上文提到的那些操作。

```javascript
#include<iostream>
#include<cstdio>
#define maxn (300000 + 10)
using namespace std;
int son[maxn][2],f[maxn],sum[maxn],a[maxn],rev[maxn],q[maxn];
int n,m,top;

int read() {
    char c=getchar();
    int r=0,f=1;
    while (c<'0' || c>'9') {
        if (c=='-') f=-1;
        c=getchar();
    }
    while (c>='0' && c<='9') {
        r=(r*10)+(c^48);
        c=getchar();
    }
    return r*f;
}

int Get(int x) {return son[f[x]][1]==x;}
bool Isroot(int x) {return son[f[x]][0]!=x && son[f[x]][1]!=x;}

void Pushup(int x) {
	int l=son[x][0],r=son[x][1];
	sum[x]=sum[l]^sum[r]^a[x];
}

void Pushdown(int x) {
	int l=son[x][0],r=son[x][1];
	if (rev[x] && x) {
		if (l) {
			rev[l]^=1;
			swap(son[l][0],son[l][1]);
		}
		if (r) {
			rev[r]^=1;
			swap(son[r][0],son[r][1]);
		}
		rev[x]=0;
	}
}

void Rotate(int x) {
	int fa=f[x],gr=f[fa],kind=Get(x);	
	if (!Isroot(fa)) son[gr][son[gr][1]==fa]=x;
	son[fa][kind]=son[x][kind^1];
	if (son[x][kind^1]) f[son[x][kind^1]]=fa;
	son[x][kind^1]=fa;
	f[fa]=x;f[x]=gr;
	Pushup(fa);
	Pushup(x);
}

void Splay(int x) {
	q[++top]=x;
	for (int i=x;!Isroot(i);i=f[i]) q[++top]=f[i];
	while (top) Pushdown(q[top--]);
	for (int fa;!Isroot(x);Rotate(x))
		if (!Isroot(fa=f[x])) 
			Rotate(Get(fa)==Get(x)?fa:x);
}

void Access(int x) {
	for (int t=0;x;t=x,x=f[x]) {
		Splay(x);
		son[x][1]=t;
		Pushup(x);
	}
}

void Makeroot(int x) {
	Access(x);
	Splay(x);
	rev[x]^=1;
	swap(son[x][0],son[x][1]);
} 

int Findroot(int x) {
	Access(x);
	Splay(x);
	while (son[x][0]) x=son[x][0];
	return x;
}

void Link(int x,int y) {
	Makeroot(x);
	if (Findroot(y)==x) return;
	f[x]=y;
}

void Split(int x,int y) {
	Makeroot(x);
	Access(y);
	Splay(y);
}

void Cut(int x,int y) {
	Split(x,y);
	if (son[y][0]==x && !son[x][1]) {
		f[x]=son[y][0]=0;
		Pushup(y);
	}
}

void Query(int x,int y) {
	Split(x,y);
	printf("%d\n",sum[y]);
}

void Modify(int x,int y) {
	Access(x);
	Splay(x);
	a[x]=y;
	Pushup(x);
}

int main() {
	n=read(); m=read();
	for (int i=1;i<=n;i++) a[i]=read(),sum[i]=a[i];
	for (int i=1;i<=m;i++) {
		int opt=read(),x=read(),y=read();
		if (opt==0) Query(x,y);
		if (opt==1) Link(x,y);
		if (opt==2) Cut(x,y);
		if (opt==3) Modify(x,y);
	}
}
```

### [luogu1501 TreeⅡ](https://www.luogu.org/problemnew/show/P1501)

动态树上的线段树模板2

有两个地方需要注意：

1.因为动态树的区间大小不一定，所以要记录size，下传加标记的时候sum[l/r]+=add[x]*size[l/r];

2.不要小瞧51061这个数。如果我们的size是51060，add也是51060，你就会惊奇的发现，爆int了！

```javascript
#include<iostream>
#include<cstdio>
#define maxn (100000 + 10)
#define mod 51061
#define ll long long 
using namespace std;
int n,m,top;
int f[maxn],son[maxn][2],q[maxn],rev[maxn],size[maxn];
ll sum[maxn],add[maxn],mul[maxn],v[maxn];

int read() {
    char c=getchar(); int r=0,f=1;
    while (c<'0' || c>'9') {
        if (c=='-') f=-1;
        c=getchar();
    }
    while (c>='0' && c<='9') {
        r=(r*10)+(c^48);
        c=getchar();
    }
    return r*f;
}

int Get(int x) {return son[f[x]][1]==x;}
bool Isroot(int x) {return son[f[x]][0]!=x && son[f[x]][1]!=x;}
void swap(int &x,int &y) {int t=x;x=y;y=t;} 

void Pushup(int x) {
    int l=son[x][0],r=son[x][1];
    sum[x]=(sum[l]+sum[r]+v[x])%mod;
    size[x]=size[l]+size[r]+1;
}

void Pushdown(int x) {
    int l=son[x][0],r=son[x][1];	
    if (mul[x]!=1) {
        if (l) {
            mul[l]=(mul[l]*mul[x])%mod;
            add[l]=(add[l]*mul[x])%mod;
            v[l]=(v[l]*mul[x])%mod;
            sum[l]=(sum[l]*mul[x])%mod;
        }
        if (r) {
            mul[r]=(mul[r]*mul[x])%mod;
            add[r]=(add[r]*mul[x])%mod;
            v[r]=(v[r]*mul[x])%mod;
            sum[r]=(sum[r]*mul[x])%mod;
        }
        mul[x]=1;
    }
    if (add[x]) {
        if (l) {
            add[l]=(add[l]+add[x])%mod;
            v[l]=(v[l]+add[x])%mod;
            sum[l]=(sum[l]+add[x]*size[l])%mod;
        }
        if (r) {
            add[r]=(add[r]+add[x])%mod;
            v[r]=(v[r]+add[x])%mod;
            sum[r]=(sum[r]+add[x]*size[r])%mod;
        }
        add[x]=0;
    }
    if (rev[x]) {
        if (l) {
            rev[l]^=1;
            swap(son[l][0],son[l][1]);
        }
        if (r) {
            rev[r]^=1;
            swap(son[r][0],son[r][1]);
        }
        rev[x]=0;
    }
}

void Rotate(int x) {
    int fa=f[x],gr=f[fa],kind=Get(x);
    if (!Isroot(fa)) son[gr][son[gr][1]==fa]=x;
    son[fa][kind]=son[x][kind^1];
    if (son[x][kind^1]) f[son[x][kind^1]]=fa;
    son[x][kind^1]=fa;
    f[fa]=x;f[x]=gr;
    Pushup(fa);
    Pushup(x);
}

void Splay(int x) {
    q[++top]=x;
    for (int i=x;!Isroot(i);i=f[i]) q[++top]=f[i];
    while (top) Pushdown(q[top--]);
    for (int fa;!Isroot(x);Rotate(x)) 
        if (!Isroot(fa=f[x]))
            Rotate(Get(fa)==Get(x)?fa:x);
}

void Access(int x) {
    for (int t=0;x;t=x,x=f[x]) {
        Splay(x);
        son[x][1]=t;
        Pushup(x);
    }
}

void Makeroot(int x) {
    Access(x);
    Splay(x);
    rev[x]^=1;
    swap(son[x][0],son[x][1]);
}

void Split(int x,int y) {
    Makeroot(x);
    Access(y);
    Splay(y);
}

void Link(int x,int y) {
    Makeroot(x);
    f[x]=y;
}

void Cut(int x,int y) {
    Split(x,y);
    f[x]=son[y][0]=0;
    Pushup(y);
}

void Add(int x,int y,int z) {
    Split(x,y);
    v[y]=(v[y]+z)%mod;
    sum[y]=(sum[y]+z*size[y])%mod;
    add[y]=(add[y]+z)%mod;
}

void Mul(int x,int y,int z) {
    Split(x,y);
    v[y]=(v[y]*z)%mod;
    sum[y]=(sum[y]*z)%mod;
    add[y]=(add[y]*z)%mod;
    mul[y]=(mul[y]*z)%mod;
}

int main() {
    n=read(); m=read();
    for (int i=1;i<=n;i++) v[i]=sum[i]=size[i]=mul[i]=1;
    for (int i=1;i<=n-1;i++) {
        int x=read(),y=read();
        Link(x,y);
    }
    for (int i=1;i<=m;i++) {
        char opt=getchar();int x=read(),y=read(),z;
        if (opt=='+') {
            z=read();
            Add(x,y,z);
        }
        if (opt=='-') {
            int u=read(),v=read();
            Cut(x,y);Link(u,v);
        }
        if (opt=='*') {
            z=read();
            Mul(x,y,z);
        }
        if (opt=='/') {
            Split(x,y);
            printf("%lld\n",sum[y]%mod);
        }
    }
}
```

