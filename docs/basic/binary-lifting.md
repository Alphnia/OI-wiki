author: Ir1d, ShadowsEpic, Fomalhauthmj, siger-young, MingqiHuang, Xeonacid, hsfzLZH1, orzAtalod, NachtgeistW

本页面将简要介绍倍增法。

## 简介

倍增法（英语：binary lifting），顾名思义就是翻倍。它能够使线性的处理转化为对数级的处理，大大地优化时间复杂度。

这个方法在很多算法中均有应用，其中最常用的是 RMQ 问题和求 [LCA（最近公共祖先）](../graph/lca.md)。

## RMQ 问题

参见：[RMQ 专题](../topic/rmq.md)

RMQ 是 Range Maximum/Minimum Query 的缩写，表示区间最大（最小）值。使用倍增思想解决 RMQ 问题的方法是 [ST 表](../ds/sparse-table.md)。

## 树上倍增求 LCA

参见：[最近公共祖先](../graph/lca.md)

## 例题

### 题 1

???+note "例题"
    如何用尽可能少的砝码称量出 $[0,31]$ 之间的所有重量？（只能在天平的一端放砝码）

??? note "解题思路"
    答案是使用 1 2 4 8 16 这五个砝码，可以称量出 $[0,31]$ 之间的所有重量。同样，如果要称量 $[0,127]$ 之间的所有重量，可以使用 1 2 4 8 16 32 64 这七个砝码。每次我们都选择 2 的整次幂作砝码的重量，就可以使用极少的砝码个数量出任意我们所需要的重量。
    
    为什么说是极少呢？因为如果我们要量出 $[0,1023]$ 之间的所有重量，只需要 9 个砝码，需要量出 $[0,1048575]$ 之间的所有重量，只需要 19 个。如果我们的目标重量翻倍，砝码个数只需要增加 1。这叫“对数级”的增长速度，因为砝码的所需个数与目标重量的范围的对数成正比。

### 题 2

???+note "例题"
    给出一个长度为 $n$ 的环和一个常数 $k$，每次会从第 $i$ 个点跳到第 $(i+k)\bmod n+1$ 个点，总共跳了 $m$ 次。每个点都有一个权值，记为 $a_i$，求 $m$ 次跳跃的起点的权值之和对 $10^9+7$ 取模的结果。
    
    数据范围：$1\leq n\leq 10^6$，$1\leq m\leq 10^{18}$，$1\leq k\leq n$，$0\le a_i\le 10^9$。

??? note "解题思路"
    这里显然不能暴力模拟跳 $m$ 次。因为 $m$ 最大可到 $10^{18}$ 级别，如果暴力模拟的话，时间承受不住。
    
    所以就需要进行一些预处理，提前整合一些信息，以便于在查询的时候更快得出结果。如果记录下来每一个可能的跳跃次数的结果的话，不论是时间还是空间都难以承受。
    
    那么应该如何预处理呢？看看第一道例题。有思路了吗？
    
    回到本题。我们要预处理一些信息，然后用预处理的信息尽量快的整合出答案。同时预处理的信息也不能太多。所以可以预处理出以 2 的整次幂为单位的信息，这样的话在预处理的时候只需要处理少量信息，在整合的时候也不需要大费周章。
    
    在这题上，就是我们预处理出从每个点开始跳 1、2、4、8 等等步之后的结果（所处点和点权和），然后如果要跳 13 步，只需要跳 1+4+8 步就好了。也就是说先在起始点跳 1 步，然后再在跳了之后的终点跳 4 步，再接着跳 8 步，同时统计一下预先处理好的点权和，就可以知道跳 13 步的点权和了。
    
    对于每一个点开始的 $2^i$ 步，记录一个 `go[i][x]` 表示第 $x$ 个点跳 $2^i$ 步之后的终点，而 `sum[i][x]` 表示第 $x$ 个点跳 $2^i$ 步之后能获得的点权和。预处理的时候，开两重循环，对于跳 $2^i$ 步的信息，我们可以看作是先跳了 $2^{i-1}$ 步，再跳 $2^{i-1}$ 步，因为显然有 $2^{i-1}+2^{i-1}=2^i$。即我们有 `sum[i][x] = sum[i-1][x]+sum[i-1][go[i-1][x]]`，且 `go[i][x] = go[i-1][go[i-1][x]]`。
    
    当然还有一些实现细节需要注意。为了保证统计的时候不重不漏，我们一般预处理出“左闭右开”的点权和。亦即，对于跳 1 步的情况，我们只记录该点的点权和；对于跳 2 步的情况，我们只记录该点及其下一个点的点权和。相当于总是不将终点的点权和计入 sum。这样在预处理的时候，只需要将两部分的点权和直接相加就可以了，不需要担心第一段的终点和第二段的起点会被重复计算。
    
    这题的 $m\leq 10^{18}$，虽然看似恐怖，但是实际上只需要预处理出 $65$ 以内的 $i$，就可以轻松解决，比起暴力枚举快了很多。用行话讲，这个做法的 [时间复杂度](./complexity.md) 是预处理 $\Theta(n\log m)$，查询每次 $\Theta(\log m)$。

??? note "参考代码"
    ```cpp
    #include <cstdio>
    using namespace std;
    
    const int mod = 1000000007;
    
    int modadd(int a, int b) {
      if (a + b >= mod) return a + b - mod;  // 减法代替取模，加快运算
      return a + b;
    }
    
    int vi[1000005];
    
    int go[75][1000005];  // 将数组稍微开大以避免越界，小的一维尽量定义在前面
    int sum[75][1000005];
    
    int main() {
      int n, k;
      scanf("%d%d", &n, &k);
      for (int i = 1; i <= n; ++i) {
        scanf("%d", vi + i);
      }
    
      for (int i = 1; i <= n; ++i) {
        go[0][i] = (i + k) % n + 1;
        sum[0][i] = vi[i];
      }
    
      int logn = 31 - __builtin_clz(n);  // 一个快捷的取对数的方法
      for (int i = 1; i <= logn; ++i) {
        for (int j = 1; j <= n; ++j) {
          go[i][j] = go[i - 1][go[i - 1][j]];
          sum[i][j] = modadd(sum[i - 1][j], sum[i - 1][go[i - 1][j]]);
        }
      }
    
      long long m;
      scanf("%lld", &m);
    
      int ans = 0;
      int curx = 1;
      for (int i = 0; m; ++i) {
        if (m & (1 << i)) {  // 参见位运算的相关内容，意为 m 的第 i 位是否为 1
          ans = modadd(ans, sum[i][curx]);
          curx = go[i][curx];
          m ^= 1ll << i;  // 将第 i 位置零
        }
      }
    
      printf("%d\n", ans);
    }
    ```

### 题 3

???+note "[P1084 疫情控制 ](https://www.luogu.com.cn/problem/P1084)"

​	题目大意：一棵 $n$ 个节点，以 1 为根、带边权的树，有m个军队驻扎在一些给定节点上，军队从一个节点移动到另一个节点所需时间为边权，军队可以同时移动，问最少需要多长时间才能使根到每个叶子节点的路径上都至少有一个军队（军队最终不能停在根节点）。  数据范围：$2\leq m \leq n\leq 50000$，$1\leq w\leq 10^{9}$。

??? note "解题思路"
    首先想到答案的单调性引导我们去二分答案，然后考虑如何判断确定的时间内能否控制住所有叶子节点。给定时间 $x$ ，所有军队在到达根节点之前肯定是越向上跳能管住的叶子节点越多，所以就利用倍增先让所有军队尽可能向上跳，这一步时间复杂度 $mlog(n)$ 。

​	我们发现可能有一部分军队跳上去还有多余，一部分叶子节点仍没有被覆盖到，所以我们在这一步同时记录下这些军队到节点1时的剩余时间以及它所处的子树，然后接下来就是将剩余的军队和子树用贪心法进行匹配的过程。

​	此题倍增的关键用处是在 $log(n)$ 时间内找到带边权树中的任一节点向上一段距离最终能到达的节点。

??? note "参考代码"
  ```cpp
  #include<cstdio>
  #include<iostream>
  #include<cstring>
  #include<algorithm>
  using namespace std;
  #define N 50005
  #define ll long long
  int n,m,g[N*2],tmp;
  struct la{int c,to;ll w;}e[N*2];
  inline void tu(int x,int y,ll w){e[++tmp].c=y,e[tmp].to=g[x],g[x]=tmp,e[tmp].w=w;}
  int arm[N],dp[N][22],dep[N];
  ll dis[N];
  void dfs1(int u,int f){
    dp[u][0]=f;dep[u]=dep[f]+1;
    for(int i=1;i<=20;++i){
      dp[u][i]=dp[dp[u][i-1]][i-1];
      if(!dp[u][i])break;
    }
    for(int i=g[u];i;i=e[i].to){
      int d=e[i].c;
      if(d==f)continue;
      dis[d]=dis[u]+e[i].w;
      dfs1(d,u);
    }	
  }
  int tag[N];
  int son[N],child;
  ll now[N];
  inline void tiao(int p,ll x){
    int y=p;
    for(int i=20;i>=0;--i){
      if(dis[y]-dis[dp[p][i]]<=x&&dep[dp[p][i]]>1)p=dp[p][i];
    }
    tag[p]++;
  }
  inline int fff(int p){
    for(int i=20;i>=0;--i){
      if(dep[dp[p][i]]>1)p=dp[p][i];
    }
    return p;
  }
  int in[N];
  bool dfs2(int u,int f){
    bool is=1;//是不是叶子
    bool flag=1;//
    for(int i=g[u];i;i=e[i].to){
      int d=e[i].c;
      if(d==f)continue;
      is=0;
      bool now=dfs2(d,u);
      if(!tag[d]&&!now){
        if(u==1)son[++child]=d;
        else flag=0;
      }
      else if(now&&in[d]>1)tag[d]++;
    }
    if(is&&!tag[u])return 0;
    return flag;
  }
  bool cmp(int x,int y){return dis[x]>dis[y];}
  bool no[N];
  bool check(ll x){
    memset(tag,0,sizeof(tag));int i;
    memset(no,0,sizeof(no));
    for(i=1;i<=m;++i){
      if(dis[arm[i]]>=x) tiao(arm[i],x);
      else break;
    }child=0;
    dfs2(1,0);
    for(int j=i;j<=m;++j){
      int p=fff(arm[j]);
      if(!tag[p]&&dis[p]>x-dis[arm[j]])tag[p]++,no[j]=1;
    }
    sort(son+1,son+child+1,cmp);int r=1;
    for(int j=m;j>=i;--j){//x-dis[arm[j]]从大到小
      int p=fff(arm[j]);
      if(no[j])continue;
      while(tag[son[r]]&&r<=child)r++;
      if(r>child)return 1;
      if(dis[son[r]]>x-dis[arm[j]])return 0;
      r++;
    }
    while(tag[son[r]]&&r<=child)r++;
    if(r>child)return 1;
    return 0;
  }
  int main(){	
    scanf("%d",&n);ll c;
    for(int i=1,a,b;i<n;++i)scanf("%d%d%lld",&a,&b,&c),tu(a,b,c),tu(b,a,c),in[b]++,in[a]++;
    dfs1(1,0);
    scanf("%d",&m);
    for(int i=1;i<=m;++i)scanf("%d",&arm[i]);
    sort(arm+1,arm+m+1,cmp);
    ll l=0,r=1e14;
    while(l+1<r){
      ll mid=(l+r)>>1;
      if(check(mid))r=mid;
      else l=mid;
    }
    if(check(l)){printf("%lld\n",l);}
    else if(check(r)){printf("%lld\n",r);}
    else printf("-1\n");
    return 0;
  }    
  ```
