---

layout: post
title: 'splay瞎讲'
subtitle: '就决定是你了'
date: 2018-03-14
categories: OI
cover: 'https://github.com/xMinh/xMinh.github.io/blob/master/pic/s.jpg?raw=true'
tags: 平衡树

---

# SPLAY瞎讲

本来打算吃fgt吃到底

后来发现这个真的没人用 到时候代码调不出来也没人给我讲啊

就学了学splay

本来打算学treap但是听refun说学splay就够了？%%%

学了才发现这个是真的蛇皮

本人也是看着左右两位dalao的代码才慢慢搞懂的(%%%bigbro %%%asia)

这里瞎讲一下 主要是讲给自己听 省的自己以后打大模板的时候比较懵逼

耶

## 普通的splay

就是普通的平衡树啊

插入删除查询一条龙那种

虽然splay代码比宗法树和treap长还跑得慢

但是splay是人气炸子鸡啊 还比较社会 就不得不学一学

```javascript
#include<iostream>
#include<cstdio>
#define maxn 1000010
using namespace std;
int f[maxn],num[maxn],key[maxn],size[maxn],son[maxn][2],rt,cnt;
//快读快写  
int read(){
	char c;int r=0,f=1;
	c=getchar();
	while (c<'0' || c>'9'){
		if (c=='-') f=-1;
		c=getchar();
	}
	while (c>='0' && c<='9'){
		r=r*10+(c^48);
		c=getchar();
	}
	return r*f;
}
int po[30];
void putout(int x){
	if (!x){
		putchar(48);
		return;
	}
	int t=0;
	if (x<0) putchar('-'),x=-x;
	while (x) po[++t]=x%10,x/=10;
	while (t) putchar(po[t--]+48);
}
void clear(int x){//清空编号为x的节点 以防万一 
	f[x]=num[x]=key[x]=size[x]=son[x][1]=son[x][0]=0;
}
int get(int x){
	return son[f[x]][1]==x;
	//判断编号x是父亲的左儿子还是右儿子 
}
void pushup(int x){//更新编号x的size信息
	if (x){//编号x有可能为0 
		size[x]=num[x]+size[son[x][0]]+size[son[x][1]]; 
	}
}
void rotate(int x){//将编号x转到父亲的位置上
	int fa=f[x],gr=f[fa],kind=get(x);
	son[fa][kind]=son[x][kind^1];
	f[son[x][kind^1]]=fa;
	son[x][kind^1]=fa;
	f[fa]=x;f[x]=gr;
	//神仙操作 语言难以描述 可以画图理解 
	if (gr) son[gr][son[gr][1]==fa]=x;
	pushup(fa);pushup(x);
	//这个顺序不能反 因为已经交换 
} 
void splay(int x){//将编号x转到根
	for (int fa;fa=f[x];rotate(x)){
		if (f[fa]){
			rotate(get(x)==get(fa)?fa:x);
			//三点一线先转爹，不然先转儿子 
		}
	}
	rt=x;
}
void insert(int x){//插入数x  
	if (!rt){//如果树为空，就加一个根 
		rt=++cnt;
		num[rt]=size[rt]=1;
		f[rt]=son[rt][0]=son[rt][1]=0;
		key[rt]=x;
		return;
	}
	int now=rt,fa=0;//从根节点向下找在哪里插入 
	while (1){
		if (x==key[now]){//如果在当前点插入即可 
			++num[now];++size[now];
			pushup(fa);//更新父节点 
			splay(now);//把自己转到根节点
			return; 
		}
		fa=now;now=son[fa][key[fa]<x];
		//如果不在当前点插入，就是在当前点的子树
		//如果当前点小于x，就去右子树，不然去左子树
		if (!now){//找到位置了，但要开点 
			now=++cnt;
			num[now]=size[now]=1;
			son[now][0]=son[now][1]=0;
			f[now]=fa;
			key[now]=x;
			son[fa][key[fa]<x]=now;
			//如果满足key[fa]<x，则返回1，正好是右儿子
			//不满足返回0，就是左儿子 
			pushup(fa);
			splay(now);
			return; 
		} 
	}
}
int rnk(int x){//求数x的排名 
	int now=rt,ans=0;
	while (1){
		if (x<key[now]) now=son[now][0];
		else{
			ans+=size[son[now][0]];
			if (x==key[now]){
				splay(now);
				return ans+1; 
			}
			ans+=num[now];
			now=son[now][1];
		}
	}
}
int kth(int x){//求排名为x的是什么数 
	int now=rt;
	while (1){
		if (x<=size[son[now][0]]) now=son[now][0];
		else{
			int t=size[son[now][0]]+num[now];
			if (x<=t) return key[now];
			x-=t;
			now=son[now][1];
		}
	} 
}
int pre(){//查询根的前驱的编号 
	int now=son[rt][0];
	while (son[now][1]) now=son[now][1];
	//前驱是根的左子树中最靠右的那个
	return now; 
}
int nxt(){//查询根的后继的编号 
	int now=son[rt][1];
	while (son[now][0]) now=son[now][0];
	//后继是根的右子树中最靠左的那个
	return now; 
}
void erase(int x){//删除数x  
	rnk(x);
	//先求rank，是要找到数x的位置，然后把它旋转到根节点，方便操作
	if (num[rt]>1){//如果数x不止一个 删一个直接走人 
		--num[rt];--size[rt];
		return;
	}
	if (!son[rt][0] && !son[rt][1]){//整棵树上只有自己 
		clear(rt);rt=0;
		return;
	}
	if (!son[rt][0]){//没有左儿子 右儿子当根 清空原根 
		int oldrt=rt;
		rt=son[rt][1];f[rt]=0;clear(oldrt);
		return;
	} 
	if (!son[rt][1]){//没有右儿子 左儿子当根 清空原根 
		int oldrt=rt;
		rt=son[rt][0];f[rt]=0;clear(oldrt);
		return;
	}
	//剩下的就是最恶心的情况 一棵蓬勃生长的树 我们要删掉根节点
	int pr=pre(),oldrt=rt;
	//把前驱移到根节点上来 
	//那么原根一定是最后一个被替换的 也就变成了当前节点右儿子 
	//那么原根一定没有左儿子 因为它的前驱就是当前根
	//我们直接把原根的右儿子作为当前根的右儿子 清空原根 
	splay(pr);
	son[rt][1]=son[oldrt][1];
	f[son[oldrt][1]]=rt;
	clear(oldrt);
	pushup(rt);
} 
int main(){
	int n=read();
	for (int i=1;i<=n;i++){
		int opt=read(),x=read();
		if (opt==1) insert(x);
		if (opt==2) erase(x);
		if (opt==3) putout(rnk(x)),putchar(10);
		if (opt==4) putout(kth(x)),putchar(10);
		if (opt==5){
			insert(x),putout(key[pre()]),putchar(10),erase(x);
		}
		if (opt==6){
			insert(x),putout(key[nxt()]),putchar(10),erase(x);
		}
	}
}
```

## 文艺的splay

很骚的splay

能够做到区间翻转

```javascript
#include<iostream>
#include<cstdio>
#define big 1e9
#define maxn 100010
using namespace std;
int f[maxn],wt[maxn],size[maxn],son[maxn][2],tag[maxn],a[maxn],rt,cnt;
int read(){
	char c;int r=0,f=1;
	c=getchar();
	while (c<'0' || c>'9'){
		if (c=='-') f=-1;
		c=getchar();
	}
	while (c>='0' && c<='9'){
		r=r*10+(c^48);
		c=getchar();
	}
	return r*f;
}
int po[30];
void putout(int x){
	if (!x){
		putchar(48);
		return;
	}
	int t=0;
	if (x<0) putchar('-'),x=-x;
	while (x) po[++t]=x%10,x/=10;
	while (t) putchar(po[t--]+48);
}
int get(int x){
	return son[f[x]][1]==x;
} 
void pushup(int x){
	size[x]=size[son[x][0]]+size[son[x][1]]+1;
}
void pushdown(int x){
	if (x&&tag[x]){
		tag[son[x][0]]^=1;
		tag[son[x][1]]^=1;
		swap(son[x][0],son[x][1]);
		tag[x]=0;
	}
}
int build(int l,int r,int fa){
	if (l>r) return 0;
	int now=++cnt;
	int mid=(l+r)>>1;
	wt[now]=a[mid];//wt=weight 权值
	//我们建树维护的是下标的树 但是我们最后要输出的是权值 
	f[now]=fa;
	son[now][0]=build(l,mid-1,now);
	son[now][1]=build(mid+1,r,now);
	pushup(now);
	return now;
}
void rotate(int x){
	int fa=f[x],gr=f[fa],kind=get(x);
	pushdown(fa);pushdown(x);
	//先翻上面再翻下面 
	son[fa][kind]=son[x][kind^1];
	f[son[x][kind^1]]=fa;
	son[x][kind^1]=fa;
	f[fa]=x;f[x]=gr;
	if (gr) son[gr][son[gr][1]==fa]=x;
	pushup(fa);pushup(x);
}
void splay(int x,int goal){
	for (int fa;(fa=f[x])!=goal;rotate(x)){
		if (f[fa]!=goal)
			rotate(get(x)==get(fa)?fa:x);
	}
	if (!goal) rt=x;
}
int kth(int x){
	int now=rt;
	while (1){
		pushdown(now);
		//先翻转再查 
		if (x<=size[son[now][0]]) now=son[now][0];
		else{
			int t=size[son[now][0]]+1;
			if (x<=t) return now;
			x-=t;
			now=son[now][1];
		}
	}
}
void print(int now){
	pushdown(now);
	if (son[now][0]) print(son[now][0]);
	if (wt[now]!=-big && wt[now]!=big) putout(wt[now]),putchar(' ');
	if (son[now][1]) print(son[now][1]); 
}
int main(){
	int n=read(),m=read();
	for (int i=1;i<=n;i++) a[i+1]=i;
	a[1]=-big;a[n+2]=big;
	//设两个哨兵 方便操作 
	rt=build(1,n+2,0);
	for (int i=1;i<=m;i++){
		int l=read(),r=read();
		//对于题目给定的区间[l,r] 我们需要操作的实际上是l-1和r+1
		//加上哨兵节点 就是l和r+2 
		l=kth(l);r=kth(r+2);
		splay(l,0);splay(r,l);
		//把l旋转到根节点 把r旋转到l的右儿子
		//那么根的右儿子的左子树就是要翻转的区间 
		tag[son[son[rt][1]][0]]^=1;
	}
	print(rt);
}
```

## 二逼的splay

这玩意官方名称线段树套平衡树

领教了splay的常数巨大 涉险卡进时限

没有用fread 但是据说用了很稳

代码难度较大 思维难度较小

当然除了第二个操作 这里讲一下

```javascript
maxx //序列中出现过的最大的数
k //查询排名为k的数
mid //二分出一个数值mid
before //当前区间内比mid小的有几个数
```

这个需要从0到maxx二分，用before记录当前区间内比mid小的有几个数。但是不能中间判断(before==k-1)就退出，因为我们这里有一个很给力的样例

```javascript
4 1
4 2 2 1
2 1 4 3
```

第3大的数是2，但是如果我们刚才那么做就跑不出来。排名为3的数(k-1)为2，BUT不管我们的mid是1，2，还是4，运行出来的before都不可能为2。那应该怎么办呢？

```javascript
if (bfr<k) l=mid+1,ans=mid;
```

仔细想一下，其实只需要记录最后一个比k小的before就对了。

还有一个要注意的细节 求前驱和后继的时候 要特判一下前驱或后继为空的情况

黄学长的树套树代码风格写起来真不错！

上代码

```javascript
#include<iostream>
#include<cstdio>
#define big 2147483647
#define maxn 50010
using namespace std;
int n,ans,maxx,m,a[maxn];
int f[maxn<<5],son[maxn<<5][2],size[maxn<<5],num[maxn<<5],key[maxn<<5],cnt;
int max(int x,int y){
    return x>y?x:y;
}
int min(int x,int y){
    return x<y?x:y;
}
struct treeX{
    int rt;
    void clear(int x){
        f[x]=son[x][0]=son[x][1]=size[x]=num[x]=key[x]=0;
    }
    int get(int x){
        return son[f[x]][1]==x;
    }
    void pushup(int x){
        if (x) size[x]=num[x]+size[son[x][0]]+size[son[x][1]];
    }
    void rotate(int x){
        int fa=f[x],gr=f[fa],kind=get(x);
        son[fa][kind]=son[x][kind^1];
        f[son[x][kind^1]]=fa;
        son[x][kind^1]=fa;
        f[fa]=x;f[x]=gr;
        if (gr) son[gr][son[gr][1]==fa]=x;
        pushup(fa);pushup(x);
    }
    void splay(int x){
        for (int fa;fa=f[x];rotate(x))
            if (f[fa]) 
                rotate(get(x)==get(fa)?fa:x);
        rt=x;
    }
    void insert(int x){
        if (!rt){
            rt=++cnt;
            num[rt]=size[rt]=1;
            key[rt]=x;
            return;
        }
        int now=rt,fa=0;
        while (1){
            if (x==key[now]){
                ++num[now];++size[now];
                pushup(fa);
                splay(now);
                return;
            }
            fa=now;now=son[fa][key[fa]<x];
            if (!now){
                now=++cnt;
                f[now]=fa;
                key[now]=x;
                num[now]=size[now]=1;
                son[fa][key[fa]<x]=now;
                pushup(fa);
                splay(now);
                return;
            }
        }
    }
    int rnk(int x){
        int now=rt,ans=0;
        while (1){
            if (x<key[now]) now=son[now][0];
            else {
                ans+=size[son[now][0]];
                if (x==key[now]){
                    splay(now);
                    return ans+1;
                }
                ans+=num[now];
                now=son[now][1];
            } 
            if (!now) return ans+1;
        }
    } 
    int pre(){
        int now=son[rt][0];
        while (son[now][1]) now=son[now][1];
        return now;
    }
    int nxt(){
        int now=son[rt][1];
        while (son[now][0]) now=son[now][0];
        return now;
    }
    void erase(int x){
        rnk(x);
        if (num[rt]>1){
            --num[rt];--size[rt];return;
        }
        if (!son[rt][0] && !son[rt][1]){
            clear(rt);rt=0;return;
        }
        if (!son[rt][0]){
            int oldrt=rt;
            rt=son[rt][1];f[rt]=0;
            clear(oldrt);return;
        } 
        if (!son[rt][1]){
            int oldrt=rt;
            rt=son[rt][0];f[rt]=0;
            clear(oldrt);return;
        }
        int pr=pre(),oldrt=rt;
        splay(pr);
        son[rt][1]=son[oldrt][1];
        f[son[oldrt][1]]=rt;
        clear(oldrt);
        pushup(rt);
    }
};
struct treeY{
    treeX sg[maxn<<2];
    void change(int l,int r,int now,int goal,int x,int y){
        sg[now].insert(y);
        if (x>=0) sg[now].erase(x);
        if (l==r) return;
        int mid=l+r>>1;
        if (goal<=mid) change(l,mid,now<<1,goal,x,y);
        else change(mid+1,r,now<<1|1,goal,x,y);
    } 
    int rnk(int L,int R,int l,int r,int now,int x){
        if (L<=l && r<=R){
            return sg[now].rnk(x)-1;
        }
        int mid=(l+r)>>1,ans=0;
        if (L<=mid) ans+=rnk(L,R,l,mid,now<<1,x);
        if (R>mid) ans+=rnk(L,R,mid+1,r,now<<1|1,x);
        return ans;
    }
    int lb(int L,int R,int k){
        int l=0,r=maxx,ans,bfr;
        while (l<=r){
            int mid=l+r>>1;
            bfr=rnk(L,R,1,n,1,mid);
            if (bfr<k) l=mid+1,ans=mid;
            else r=mid-1; 
        }
        return ans;
    }
    int pre(int L,int R,int l,int r,int now,int x){
        if (L<=l && r<=R){
            sg[now].insert(x);
            int last=sg[now].pre();
            ans=last?key[last]:-big;
            sg[now].erase(x);
            return ans;
        }
        int mid=(l+r)>>1,ans=-big;
        if (L<=mid) ans=max(ans,pre(L,R,l,mid,now<<1,x));
        if (R>mid) ans=max(ans,pre(L,R,mid+1,r,now<<1|1,x));
        return ans;
    }
    int nxt(int L,int R,int l,int r,int now,int x){
        if (L<=l && r<=R){
            sg[now].insert(x);
            int next=sg[now].nxt();
            ans=next?key[next]:big;
            sg[now].erase(x);
            return ans;
        }
        int mid=(l+r)>>1,ans=big;
        if (L<=mid) ans=min(ans,nxt(L,R,l,mid,now<<1,x));
        if (R>mid) ans=min(ans,nxt(L,R,mid+1,r,now<<1|1,x));
        return ans;
    }
}T;
int read(){
    char c;int r=0,f=1;
    c=getchar();
    while (c<'0' || c>'9'){
        if (c=='-') f=-1;
        c=getchar();
    }
    while (c>='0' && c<='9'){
        r=(r<<3)+(r<<1)+(c^48);
        c=getchar();
    }
    return r*f;
}
int po[30];
void putout(int x){
    if (!x){
        putchar(48);
        return;
    }
    if (x<0) putchar('-'),x=-x;
    int t=0;
    while (x) po[++t]=x%10,x/=10;
    while (t) putchar(po[t--]+48);
}
int main(){
    n=read();m=read();
    for (int i=1;i<=n;i++){
        a[i]=read();maxx=max(maxx,a[i]);
        T.change(1,n,1,i,-1,a[i]); 
    }
    for (int i=1;i<=m;i++){
        int opt=read(),l,r,k,pos;
        if (opt==1){
            l=read();r=read();k=read();
            putout(T.rnk(l,r,1,n,1,k)+1),putchar(10); 
        }
        if (opt==2){
            l=read();r=read();k=read();
            putout(T.lb(l,r,k)),putchar(10); 
        }
        if (opt==3){
            pos=read();k=read();
            T.change(1,n,1,pos,a[pos],k);
            a[pos]=k;maxx=max(maxx,k);
        }
        if (opt==4){
            l=read();r=read();k=read();
            putout(T.pre(l,r,1,n,1,k)),putchar(10);
        }
        if (opt==5){
            l=read();r=read();k=read();
            putout(T.nxt(l,r,1,n,1,k)),putchar(10);
        } 
    }
}
```

