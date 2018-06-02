---

layout: post
title: 'KMP+AC自动机学习笔记'
subtitle: '开局一个Aufun 算法全靠抄'
date: 2018-06-02
categories: OI
cover: 'https://raw.githubusercontent.com/xMinh/xMinh.github.io/master/pic/backgrounds/3068befa-45dd-491c-a506-66b86a5c65f4.jpg'
tags: KMP AC自动机

---

# KMP+AC自动机 学习笔记

省选之前写的，忘了发了……

## KMP

### 讲解

推一个Blog 写的特别好 真的不骗人 老实人不忽悠 一看就明白 博主人超好  

**[Cansult](https://www.cansult.ga/%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0_%20KMP.html)**

以及Asia觉得讲的特别好的**[Martix 67神犇](http://www.matrix67.com/blog/archives/115)**

这种算法的作用在O(n+m)的复杂度内完成对两个字符串的匹配，并能求出每一个匹配的位置。

结合Cansult的Blog看懂代码应该不难。

### 代码

```javascript
#include<iostream>
#include<cstdio>
#include<cstring>
#define N (1000000 + 10)
using namespace std;
char s[N],t[N];
int n,m,p[N];

int main() {
    cin>>s+1;
    cin>>t+1;
    n=strlen(s+1);
    m=strlen(t+1);
    int j=0;
    p[0]=p[1]=0;
    for (int i=2;i<=m;i++) {
        while (j && t[i]!=t[j+1]) j=p[j];
        if (t[i]==t[j+1]) j++;
        p[i]=j;
    }
    j=0;
    for (int i=1;i<=n;i++) {
        while (j && s[i]!=t[j+1]) j=p[j];
        if (s[i]==t[j+1]) j++;
        if (j==m) {
            printf("%d\n",i-m+1);
            j=t[j];
        }
    }
    for (int i=1;i<=m;i++) printf("%d ",p[i]);	
}
```

## AC自动机

### 讲解

强烈推荐HN省队巨佬**[yyb](http://www.cnblogs.com/cjyyb/p/7196308.html)**的Blog

你们可能还想要AC自动机题单？

**[这里是史上最大的Faker 千古神犇REfun](http://www.cnblogs.com/refun/tag/AC%E8%87%AA%E5%8A%A8%E6%9C%BA/)**

还有我真™不是托儿…… 

~~我只是懒得打字~~

### 实现

#### Build Trie

```javascript
void Trie(string s) {
    int len=s.length(),x=0;
    for (int i=0;i<s.length();i++) {
        if (!son[x][s[i]-'a']) 
        son[x][s[i]-'a']=++id;
        x=son[x][s[i]-'a'];
    }
    Ed[x]++;
}
```

建Trie树的过程，没啥好说的吧……

#### Get Fail

```javascript
void Get_Fail() {
    for (int i=0;i<26;i++) 
        if (son[0][i]) {
            fail[son[0][i]]=0;
            q.push(son[0][i]);
        }
    while (!q.empty()) {
        int x=q.front();
        q.pop();
        for (int i=0;i<26;i++) 
            if (son[x][i]) {
                fail[son[x][i]]=son[fail[x]][i];
                q.push(son[x][i]);
            }
            else son[x][i]=son[fail[x]][i];
    }
}
```

先处理直连根节点的，然后广搜建Fail指针，把Trie树补成Trie图。

伟大的神犇REfun曾经说过：“Fail指针指向的是当前节点所代表的串的最长后缀。”所以我们在处理当前串的时候可以一块把它的Fail以及Fail的Fail等一块处理掉。

然后把Trie树补成Trie图就是为了匹配的时候省时，这厢匹配不上就跳到它的最长后缀的这个点上继续匹配。当然这样就有了一些奇怪的性质，后面(可能)会讲到。

#### Query

```javascript
int Query(string s) {
    int len=s.length(),x=0,ans=0;
    for (int i=0;i<len;i++) {
        x=son[x][s[i]-'a'];
        for (int j=x;j && Ed[j]!=-1;j=fail[j]) {
            ans+=Ed[j];
            Ed[j]=-1;
        }
    }
    return ans;
}
```

这个唯一要说的就是跳Fail的那句了，这里写的这个Query是统计某个模式串是否在文本串中出现，所以每个点最多统计一次就行了，统计完给Ed打个标记，下次用Fail上跳的时候不走这个点了。但是如果是统计某个串出现的次数，那就不要打标记了，因为要统计多次，这个应该都懂……

### 代码

#### [是否出现？](https://www.luogu.org/problemnew/show/P3808)

```javascript
#include<iostream>
#include<cstdio>
#include<cctype>
#include<queue>
#define N (1000000 + 10)
using namespace std;
int n,id,son[N][26],Ed[N],fail[N];
string st;

int read() {
    char c=getchar();
    int r=0,f=1;
    while (!isdigit(c)) {
        if (c=='-') f=-1;
        c=getchar();
    }
    while (isdigit(c)) {
        r=(r*10)+(c^48);
        c=getchar();
    }
    return r*f;
}

void Trie(string s) {
    int len=s.length(),x=0;
    for (int i=0;i<s.length();i++) {
        if (!son[x][s[i]-'a']) 
        son[x][s[i]-'a']=++id;
        x=son[x][s[i]-'a'];
    }
    Ed[x]++;
}

queue<int> q;

void Get_Fail() {
    for (int i=0;i<26;i++) 
        if (son[0][i]) {
            fail[son[0][i]]=0;
            q.push(son[0][i]);
        }
    while (!q.empty()) {
        int x=q.front();
        q.pop();
        for (int i=0;i<26;i++) 
            if (son[x][i]) {
                fail[son[x][i]]=son[fail[x]][i];
                q.push(son[x][i]);
            }
            else son[x][i]=son[fail[x]][i];
    }
}

int Query(string s) {
    int len=s.length(),x=0,ans=0;
    for (int i=0;i<len;i++) {
        x=son[x][s[i]-'a'];
        for (int j=x;j && Ed[j]!=-1;j=fail[j]) {
            ans+=Ed[j];
            Ed[j]=-1;
        }
    }
    return ans;
}

int main() {
    n=read();
    for (int i=1;i<=n;i++) {
        cin>>st;
        Trie(st);
    }
    Get_Fail();
    cin>>st;
    printf("%d",Query(st));
}
```

#### [出现几次？](https://www.luogu.org/problemnew/show/P3796)

```javascript
#include<iostream>
#include<cstdio>
#include<string>
#include<cstring>
#include<cctype>
#include<queue> 
#define N (150 + 1)
#define Maxn (10500 + 10)
using namespace std;
string st[N],S;
int n,id,son[Maxn][26],fail[Maxn],Ed[Maxn],ans[Maxn],pos[N];

int read() {
    char c=getchar();
    int r=0,f=1;
    while (!isdigit(c)) {
        if (c=='-') f=-1;
        c=getchar();
    }
    while (isdigit(c)) {
        r=(r*10)+(c^48);
        c=getchar();
    }
    return r*f;
}

void Clear(int x) {
    memset(son[x],0,sizeof(son[x]));
    fail[x]=0; Ed[x]=0; ans[x]=0;
}

int Trie(string s) { 
    int len=s.length(),x=0;
    for (int i=0;i<len;i++) {
        if (!son[x][s[i]-'a']) {
            son[x][s[i]-'a']=++id; 
            Clear(id);
        }
        x=son[x][s[i]-'a'];
    }	
    Ed[x]++;
    return x; 
}

queue<int> q;

void Get_Fail() {
    for (int i=0;i<26;i++) 
        if (son[0][i]) q.push(son[0][i]);
    while (!q.empty()) {
        int x=q.front(); q.pop();
        for (int i=0;i<26;i++) 
            if (son[x][i]) {
                fail[son[x][i]]=son[fail[x]][i];
                q.push(son[x][i]);
            }
            else son[x][i]=son[fail[x]][i];
    }
}

void Query(string s) {
    int len=s.length(),x=0;
    for (int i=0;i<len;i++) {
        x=son[x][s[i]-'a'];
        for (int j=x;j;j=fail[j]) 
            ans[j]+=Ed[j];
    }
}

int main() {
    n=read();
    while (n) {
        id=0;Clear(0);
        for (int i=1;i<=n;i++) {
            cin>>st[i];
            pos[i]=Trie(st[i]);
        }
        Get_Fail();
        cin>>S;
        Query(S);
        int Max=-1;
        for (int i=1;i<=n;i++)
            Max=max(Max,ans[pos[i]]);
        printf("%d\n",Max);
        for (int i=1;i<=n;i++) 
            if (ans[pos[i]]==Max)
                cout<<st[i]<<endl;
        n=read();
    }
}
```

### PPT

和组里几位神仙搞了一个互助社……

然而让我第一个讲题？WTF？

**[蒟蒻的PPT](http://note.youdao.com/noteshare?id=d9a4f8948a27417d49d261f99f512e9e)**

