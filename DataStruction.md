---
title: Data Structure
tags: Computer
categories: Computer
top: false
---

- 用低劣的水平描述数据结构的东西，后续考研还要细学
- 目前主要是要考试，大体过一遍，如果你质疑我的文章，那一定是我错了，我会忽略一些专业术语，更偏向于自己的理解思考
- <对于学过数据结构的人可能有一定用处吧>

------

# 前置

- 一堆概念
  - 数据：客观事物的符号描述
  - 数据元素：数据的基本单位
  - 数据项：组成数据元素的、有独立含义的、不可分割的最小单位
  - 数据对象：性质相同的数据元素的集合
- 逻辑结构
  - 线性与非线性
  - 线性、树、图、集合
- 存储结构
- 时间复杂度、空间复杂度

# 线性表

#### 普通

#### 单链表

#### 循环链表

#### 双向链表

# 栈和队列

#### 栈（先进后出）

- 不想说了，没完没了

#### 队列（先进先出）

- 普通队列
- 循环队列
- 双向队列（两个端点都能入队、出队）
- 优先队列（队列元素维持有序，算竞常用）

# 字符串

### KMP算法(字符串匹配，t向s匹配)

- 书上的代码便于理解，但是不能跑，下标有问题
- 这个难点就在一个求next数组吧。所谓next，就是每个字符的最长相同前后缀。至于代码上注意把握是在递归求解即可

#### 结构体声明

```c++
#define MAXSIZE 1024
typedef struct{
    char ch[MAXSIZE+1];		//字符串
    int length;				//长度
}String;
```

#### 求解next数组

- 优化前

```c++
void Get_Next(String t, int ne[])
{
    ne[0] = -1;
    int i = 0, j = -1;
    while(i < t.length){
        if(j == -1 || t.ch[i] == t.ch[j]){
            i++, j++;
            ne[i] = j;		//相等，直接在之前的基础上加一
        }else{ 
            j = ne[j];      //不相等，向前递归
        }
    }
}
```

- 优化后（优化是优化掉了 t 串 ne[j] 位置的元素与 t[j] 相同的情况，这种情况的匹配一否全否。所以会让ne[j]递归到 t[ne[j]] != t[j]）

```c++
void Get_Next(String t, int nextval[])
{
    nextval[0] = -1;
    int i = 0, j = -1;
    while(i < t.length){
        //相同前后缀，进入判断
        if(j == -1 || t.ch[i] == t.ch[j]){
            i++, j++;						//修改位置
            if(t.ch[i] != t.ch[j])
                nextval[i] = j;				//不相等，直接取值             
            else
                nextval[i] = nextval[j];    //相等，无意义，再次向前求解
        }else
            j = nextval[j];      //非相同前后缀，向前递归
    }
}
```

#### KMP

```c++
int KMP(String s, String t)
{
    int nextval[MAXSIZE+1];
    Get_Next(t, nextval);		             //求解nextval
    int i = 0, j = 0;
    while(i < s.length && j < t.length){
        if(j == - 1 || s.ch[i] == t.ch[j]){  //相同，可匹配
            i++, j++;
        }else							     //不可匹配，向前追溯
            j = nextval[j];        
    }
    //输出匹配位置
    if(j >= t.length)
        return i - t.length + 1;	//输出实际位置长度，非数组下标
    return 0;
}
```

# 树

### 一堆概念

- 写写有些什么吧
  - 结点
    - 父母结点、祖先结点、左右孩子结点、兄弟结点、子孙结点、前后辈结点
  - 结点的度、结点的层次、树的高度
  - 有序树、无序树
  - 二叉树、满二叉树、完全二叉树、线索二叉树
  - 森林

### 二叉树的存储

#### 顺序存储

- 将二叉树按照完全二叉树层序依次编号，自上而下，自左向右。按这个编号顺序放进一维容器就可
- 二叉树中的没有的结点，编号后按值为0放进容器
- 这样存储，若结点下标为k，则其左儿子下标为2k，右儿子为2k+1

#### 链式存储

```c++
struct Node{
    Data data;	            //数据
	int *lchild, *rchild;	//指向左儿子和右儿子
    int *parent				//指向父节点
}
```

### 二叉树的遍历

#### 先序遍历

- 根左右，就是从根结点开始跑图^_^

```c++
//递归是万能的
void front(Node x){
    cout <<x->data <<" ";	//输出根
	front(x->lchild);		//递归左儿子
    front(x->rchild);		//递归右儿子
}
//下面那俩写成递归代码和这个差不多
```

#### 中序遍历

- 左根右，看成果树自然脱落，然后顺序捡果子

#### 后序遍历

- 左右根，好像没什么可描述的思路。

#### 层序遍历

- 顾名思义，从根结点开始，一层一层输出就完事

#### 树的遍历

- 这个就直接合并了，没有中序遍历，其他三种都是有的

### 森林

- 把一堆树放一起

#### 森林遍历

- 先序遍历
  - 先找森林中第一棵树的根结点
  - 先序遍历第一棵树，把剩余的树看成新的森林
  - 重复上面的操作
- 中序遍历
  - 先找森林中第一棵树的`子树森林`（也就是第一棵树中除根结点以外的部分）
  - 后序遍历第一棵树，把剩余的树看成新的森林
  - 重复上面的操作

### 二叉树、树、森林的转换

- 树变二叉树
  1. 在所有的兄弟之间连线
  2. 只保留父节点与第一个儿子结点的连线，去掉其他父子结点之间的连线
- 森林变二叉树
  1. 对每一棵树执行树变二叉树的操作
  2. 将森林中所有的树的根结点连起来
- 二叉树变森林
  1. 对于一个结点x，他是父结点的左孩子，在父节点和结点x的右孩子、以及其右孩子的右孩子…之间连线
  2. 去掉所有结点与右孩子的连线

### 哈夫曼树

- 最优树，带权路径最短，也就是所有结点的深度与权值积之和最小

### 哈夫曼编码

- 在哈夫曼树上按照`左0右1`的规则从根节点走到目标节点形成的数字串

# 图

### 一堆概念

- 有向图、无向图、混合图
- 点、权值
- 出度、入度
- 边、弧
- 强连通（有向图）
  - 两个顶点强连通：两个顶点至少存在一条互相可达路径
  - 强连通图：任意两个顶点强连通
  - 强连通分量：非强连通图中的最大强连通子图
- 连通、回路（环）、稀疏图、稠密图
- 零图：仅有孤立结点
- 平凡图：只有一个孤立结点
- 多重图：有多重边的图。非多重图为线图
- 简单图：无多重边和自回环的图
- 完全图：任意两个不同结点之间都有边的简单图
  - 记为Kn
  - 无向完全图的边数为n(n-1)/2
- k度正则图：在无向图G=<V，E>中，每个结点的度都是k
- 点割集：有一个连通图和点的集合，如果在图上把所有集合中的点删去，这个图就不是连通图，而这个集合中的点少一个，它还是连通图，这个集合就叫点割集
- 割点：存在两个点u、w，使得uw之间的每一条通路都必须经过点v，那么v就是割点。意思也就是没了v他就不是个连通图
- 结点连通度：在一个非完全图的连通图里，变为不连通图所需删去结点的最小数目
- 边割集、割边、边连通度
- 结点连通度𝜅(G) ≤ 边连通度𝜆(G) ≤ 最小度δ(G)

### 存储

```
有些东西，凭直觉走就行了
```

|            | 邻接表                       | 邻接矩阵             | 十字链表 | 邻接多重表 |
| ---------- | ---------------------------- | -------------------- | -------- | ---------- |
| 空间复杂度 | 有向图o(V+E2)，无向图o(V+E)  | o(V2)                | o(V+E)   | o(V+E)     |
| 适用范围   | 稀疏图                       | 稠密图               | 有向图   | 无向图     |
| 缺点       | 点和边难删除，一找就需要遍历 | 删点需要删除整行整列 |          |            |

#### 邻接表

- 就像一个二维向量？
- 先把每个点都摘出来是第一维。然后对于每个点，又把所有跟它有关的边都存一遍是第二维

#### 逆邻接表（有向图）

- 针对于有向图，邻接表的第二维放的是由某个点指向的所有边（其实应该叫弧）
- 逆邻接表，就是把第二维给倒一下，存所有指向某个点的边
- 比如点v，就存a->v，b->v，c->v…

#### 邻接矩阵

- 正常理解就行了
- 第n行第m列等于1表示：n->m = 1
- 具体数字，那就看什么图了，可以表示权值，也可以表示是否有边

#### 十字链表（有向图）

- 基于邻接表做改进，每个结构里多了两个指针，一个指向弧头相同的结构，另一个指向弧尾相同的结构

```c++
//这玩意只是为了更加形象，不代表就这么简单
struct Node{
    int u;		//弧尾
    int v;		//弧头
    int value;	//权值
    struct Node *hlink;    //指向下一个弧头相同的Node结构
    struct Node *tlink;    //指向下一个弧尾相同的Node结构
}
```

#### 邻接多重表（无向图）

- 跟十字链表大同小异，无非是没有了指向（弧变成了边）

```c++
struct Node{
    int u;		//边的一个结点
    int v;		//边的另一个结点
    int value;	//权值
    struct Node *hlink;    //指向下一个含u的Node结构
    struct Node *tlink;    //指向下一个含v的Node结构
}
```

这俩表示方式也就相当于是：

一条边两个点，分别把跟两个点有关的东西全部串在了一起

### 搜索

#### 深度优先搜索（DFS）

- 一条道走到黑，走完回去再走另一条

#### 广度优先搜索（BFS）

- 同深度的路同时试探，多点开花

### 拓扑排序

#### 作用对象

- 无环有向图

#### 具体步骤

1. 找一个无前驱(入度为0)的点存入拓扑序，或者找到后直接输出
2. 将该点以及所有以该点作为起点的边删掉
3. 重复1、2操作，直到所有的点都存入拓扑序中或者找不到入度为0的点

#### 分析

1. 根据操作就可以分析出带环的图无法进行拓扑排序，因为一个环中不存在入度为0的点
2. 因为一个图在拓扑排序前，或者拓扑排序开始后，都有可能存在许多个入度为0的点，此时就可以选择不同的点继续进行拓扑排序，所以拓扑排序的解不是唯一的

#### 应用范围

1. 检测一个图是否含有环
2. 求解AOE图的关键路径
3. 在有承接关系的工程中计算最优的项目处理顺序

#### 实现

```c++
#define MAXN 1000
//邻接矩阵定义
int egde[MAXN+1][MAXN+1];
int n, res[MAXN+1];
//记录每个点的入度
int in_cnt[MAXN+1];
queue<int>q;
bool Topo_sort()
{
    //初始化入度
    memset(in_cnt, 0, sizeof(in_cnt));
    //存度
	for(int i = 1; i <= n; i++){
	    for(int j = 1; j <= n; j++){
	        //指向边不存在或者指向自己
	        if(i == j || edge[i][j] == -1) continue;
            in_cnt[j]++;
	    }
	}
	int cnt = 0; //记录拓扑序
	//拓扑排序
	for(int i = 1; i <= n; i++){
	    //将入度为0的点放入队列
	    if(in_cnt[i] == 0) q.push(i);
	}
 
	while(!q.empty())
	{
	    int temp = q.front();  q.pop();
	    res[++cnt] = temp;
	    //删除以其为起点的边
	    for(int i = 1; i <= n; i++){
	        //边不存在
	        if(edge[temp][i] == -1) continue;
	        //指向的点的入度减一
	        in_cnt[i]--;
	        if(in_cnt == 0) q.push(i);
	    }
	}
	//得到的拓扑序中的点的个数小于图中点的个数
	//说明拓扑排序无法完成，可能存在自环
    if(cnt != n) return false;
    else return true;
}
```

### 最小生成树-prim

1. 将图分成两个部分，在树里的部分和不在树里的部分
2. 选择一条权值最小的边，且该边所连的两个点一个在树内，一个在树外
3. 将选定的边加入树内，并将该边的权值记录
4. 重复上述操作直到所有的点都被加入树中

```c++
#include <bits/stdc++.h>
using namespace std;
#define M 500000
#define N 100000
int n, m, res = 0;
//链式前向星建图
struct Node{
    int x, v, next;
}edge[M*2+1];
int head[N+1], cnt = 0, dis[N+1];
void addedge(int x, int y, int z){
    edge[++cnt].x = y;
    edge[cnt].v = z;
    edge[cnt].next = head[x];
    head[x] = cnt;
}
//边权优先队列
struct ty{
    int x, len;  
    //比大小重载
    bool operator < (const ty &a) const{
        return len > a.len;
    }
}temp;
priority_queue<ty>q;
bool vis[N+1];
 
void prim()
{
    //以初始点1更新队列
    vis[1] = true;
    for(int i = head[1]; i != -1; i = edge[i].next){
        temp.x = edge[i].x;
        dis[edge[i].x] = edge[i].v;
        temp.len = edge[i].v;
        q.push(temp);
    }
    while(!q.empty()){
        //将当前边权最小的可连接点加入树中
        //并更新最小生成树代价
        ty tmp = q.top(); q.pop();
        if(vis[tmp.x]) continue;
        vis[tmp.x] = true;
        res += tmp.len;
        //将新加入点所连接的边放进队列
        for(int i = head[tmp.x]; i != -1; i = edge[i].next){
            if(vis[edge[i].x]) continue;
            //大于等于之前所选边权的新边权无需放入
            if(dis[edge[i].x] <= edge[i].v) continue; 
            //
            tmp.x = edge[i].x;
            dis[edge[i].x] = edge[i].v;
            tmp.len = edge[i].v;
            q.push(tmp);
        }
    }
    //输出
    cout <<res;
}
 
int main(void)
{
    memset(head, -1, sizeof(head));
    memset(dis, 0x3f, sizeof(dis));
    memset(vis, false, sizeof(vis));
    
    scanf("%d%d", &n, &m);
    for(int i = 1; i <= m; i++){
        int a, b, v;
        scanf("%d%d%d", &a, &b, &v);
        addedge(a, b, v);
        addedge(b, a, v);
    }
    prim();
    return 0;
}
```

### 最小生成树-kruskal

1. 将所有的边按大小进行排序
2. 选择一条之前`未选择过的`且`权值最小的`边
3. 利用并查集判断选出的边所连接的两个点当前是否在一个集合里，如果不在，就将该边计入，并更新并查集信息
4. 重复2、3操作直到所有的点生成一颗树

```c++
#include <bits/stdc++.h>
using namespace std;
#define M 500000
#define N 100000
int n, m, res = 0;
//边结构体
struct Node{
    int x, y, v;
}edge[M+1];
//按边权值升序排列
bool cmp(Node a, Node b){
    return a.v < b.v;
}
//并查集查找父节点以及路径压缩
int fa[N+1];
int find(int x){
    if(fa[x] == x)
        return x;
    else
        return fa[x] = find(fa[x]);
}
 
void Kruskal()
{
    //初始化并查集
    for(int i = 1; i <= n; i++) fa[i] = i;
    //按边权排序
    sort(edge+1, edge+1+m, cmp);
    //依次拿边
    for(int i = 1; i <= m; i++){
        //判断选定边所连的两个点是否已经在树里
        int fx = find(edge[i].x);
        int fy = find(edge[i].y);
        if(fx == fy) continue;
        res += edge[i].v;
        fa[fx] = fy;
    }
    cout <<res;
}
int main(void)
{
    scanf("%d%d", &n, &m); 
    for(int i = 1; i <= m; i++){
        int a, b, v;
        scanf("%d%d%d", &edge[i].x, &edge[i].y, &edge[i].v);
    }
    Kruskal();
    return 0;
}
```

### 最短路

- 简单路：经过路的边都不相同
- 基本路：路出现的所有结点都不相同
- 简单回路、基本回路

|            | Floyed     | Dijkstra(优先队列优化) | SPFA(优先队列优化)         |
| ---------- | ---------- | ---------------------- | -------------------------- |
| 时间复杂度 | o(n3)      | o(n+m)(log2m)          | o(km)                      |
| 基本思想   | 动态规划   | 贪心                   | 贪心                       |
| 适用范围   | 无负环图   | 无负权图               | 无负环图                   |
| 适用范围   | 多源最短路 | 不含负权图的低复杂度解 | 含负权边时的单源最短路求解 |

- 懒得重写了，CSDN链接如下

[三种求最短路算法基本描述及实现(C++)_Woodenman杜的博客-CSDN博客_最短路算法c++](https://blog.csdn.net/qq_59700927/article/details/124547139?ops_request_misc={"request_id":"167099472316782390553941","scm":"20140713.130102334.pc_blog."}&request_id=167099472316782390553941&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-124547139-null-null.nonecase&utm_term=最短路&spm=1018.2226.3001.4450)

### 一些特殊的图

#### AOE网

- 工程图，可以用来评估工程进度
- 具体表示
  - 总的来说还是一个带权有向图
  - 只有一个入度为0的点和出度为0的点，表示工程的起点和终点，也叫入点和汇点
  - 每一条边表示一项工程，其权值可表示完成工程所需要的时间
  - 每一个结点，相当于一个锁，只有指向它的所有工程全部完成以后，它所指向的工程才能进行
  - 多个工程可以同时进行
- 一些需要关注的点
  - 每个工程的最早开始时间
  - 每个工程的最迟开始时间
  - 工程的最快完成时间（也就是从入点到汇点的最长路径）
  - 关键路径（决定完成时间的路径）

#### 欧拉图

- 欧拉回路：在一个连通图里，每个边走且只走一次的回路
- 欧拉图就是有欧拉回路的图

#### 哈密顿图

- 哈密顿回路：在一个图里，每个点走且只走一次的回路
- 哈密顿图就是有哈密顿回路的图

#### 二部图

- 将图中所有的点分成两部分，使得图中所有的边连接的两个点，都属于不同的两部分

# 查找

### 线性表查找

#### 顺序查找

- 就叫顺序查找（笑哭），线性遍历

#### 折半查找

- 也就是二分查找，一眼可懂。注意被查找的序列必须是按条件有序

```c++
//x为被查找数
while(l < r){
  int mid = (l + r) >> 2;
  if(mid >= x) r = mid;
  else l = mid + 1;
} 
```

#### 分块查找

- 将原来的表拆分成n个表（分块）
- 将每个表中权值最大的元素作为该表的关键字
- find，先通过每个表的关键字定位到正确表中，再进行查找

### 树表查找

#### 二叉排序树

- 特殊二叉树，对于每一个结点
  - 如果有左子树，左子树所有结点的权值均小于根结点
  - 如果有右子树，右子树所有结点的权值均大于根结点
- 至于查找就很简单了，从根结点一直向下比较就行，有点像二分，或者说它就是二分

#### 平衡二叉树（AVL树）

- |depth左子树 - depth右子树|<=1

#### B-树

#### B+树

### 散列表查找

# 排序

- 稳定排序：插入、冒泡、基数、归并
- 不稳定排序：希尔、选择、快排、堆

表格还是来得快哈，非常合理

| 名称         | 时间复杂度(平均)                | 空间复杂度       |
| ------------ | ------------------------------- | ---------------- |
| 选择排序     | o(n2)                           | o(1)             |
| 堆排序       | o(n2)                           | o(1)             |
| 冒泡排序     | o(n2)                           | o(1)             |
| 快速排序     | o(nlog2n)                       | o(log2n) –> o(n) |
| 插入排序     | o(n2)                           | o(1)             |
| 希尔排序     | 看希尔增量合适不合适，平均n1.3? | o(1)             |
| 基数排序     | o(d(n+rd))                      | o(n+rd)          |
| 归并排序     | o(nlog2n)                       | o(n)             |
| 计数(桶)排序 | o(n)                            | o(max-min+1)     |

### 选择排序

- 不断选择未排序序列中权值最大的数放到未排序序列末尾

```c++
void Select_sort(int a[], int n)
{
    int minx, minn;
    for(int i = 1; i < n; i++){
        //minn存放每一趟排序中的最小数，minx存放下标
        minn = a[i];  minx = i;
        for(int j = i + 1; j <= n; j++){
            if(minn > a[j]){
                minn = a[j];
                minx = j;
            } 
        }
        if(minx != i) swap(a[i],a[minx]);
    }
}
```

### 堆排序

首先明确这玩意是要顺序存储的完全二叉树，然后利用根堆的根最大（小）的性质

1. 实现一个大(小)根堆（建初堆）
2. 将根节点和末尾节点替换
3. 将除末尾节点（也就是之前根堆的根）之外的其它元素继续执行上述操作，直到排序结束

### 冒泡排序

- 通过邻近两元素的不断比较交换实现排序，共n-1趟，期间权值较大的元素不断下浮，故名冒泡

```c++
void Bubble_sort(int a[],int n)				    //n个数，从1下标开始
{
     for(int i = 1;i < n; i++){			        //n-1趟排序
     for(int j = 1; j <= n-i; j++){				//第i趟排序后有n-i个未排序的数
        if(a[j] > a[j+1]) swap(a[j], a[j+1]);	//冒泡交换
     }
     }
}
```

### 快速排序

这个像所有排序的结合体，虽然说是由冒泡改进而来。同时消除多次逆序

1. 选择一个枢轴（pivotkey）
2. 将所有权值小于pivotkey的放在左边，权值大于pivotkey的放在右边
3. 再将pivotkey归位，放在分界处
4. 对pivotkey两边的数列重复上述操作，也就是一个递归，直到排序完成

### 插入排序

拿出第一个乱序数，放到已经排好序的序列的合适位置

好像挺明了的，可以看成是一个选择排序的进化？

```c++
void insert_sort(int a[],int n)
{
    int i,j,temp;
	for(i = 2; i <= n; i++)
	{
        temp = a[i];		                         //取出要插入的数
		for(j = i; j > 1 && a[j - 1] > temp; j--)    //寻找合适的插入位置
            a[j] = a[j - 1];						 //没有找到的时候就向后推，把位置空出来
		a[j] = temp;                                 //插入数值
	}
}
```

### 希尔排序

这个东西吧，就是分组和插入排序的循环

1. 在原数组中按照一定间隔从头到尾取出一系列数形成几个新的序列
2. 对每个新的序列进行插入排序
3. 按原位置序列合并

至于这个间隔，就涉及到希尔增量，举个例子就明白了，分组不断降低。

比如对半式增量：

- 一个长度为8的数组，8/2 = 4，所以第一次就按间隔为4取数，那么形成的新序列的下标就是：<1 5>,<2 6>,<3 7>,<4 8>
- 第二次间隔为4/2 = 2，则新取出的序列为<1 3 5 7>,<2 4 6 8>
- 然后下一次就是2/2 = 1，没得分了，排序也就结束了

### 基数排序

这个不好描述，不写了~

1. 选取有效位(个十百千万)
2. 将元素按顺序放进对应数字（0-9）的队列
3. 再按0-9的顺序将每个队列的元素拿出来
4. 重复上述过程，直到排序完成（有效位的选取是从低到高的）

### 归并排序

整体就是无限二分和选数

```c++
//这里a和b数组没有声明，应该写到外面，a数组是录入数据的数组
void Merge_sort(int l, int r)
{
    //递归到了只有一个元素，直接返回
    if(l == r) return;
    //找到中点，然后两边分别再归并（二分）
    int mid = (l + r) >> 2;
    Merge_sort(1, mid);
    Merge_sort(mid + 1, r);
    //合并序列（选数）
    int p = 1, q = mid + 1;
    for(int i = 1; i <= r; i++){
        if(q > r || (p <= mid && a[p] <= a[q])) b[i] = a[p++];
        else b[i] = a[q++];
    }
    //还原到a数组里
    for(int i = 1; i <= r; i++) a[i] = b[i];
}
```

### 桶（计数）排序

- 适用于数据集中，数据范围较小且比较明确的情况

```c++
#include<bits/stdc++.h>
using namespace std;
#define MAXN 10000
int cnt[MAXN+1],n,temp;;
int main(void)
{
    //输入
    memset(cnt,0,sizeof(cnt));
    cin >>n;
    for(int i = 1; i <= n; i++){
        cin >>temp;
        cnt[temp]++; //计数
    }
    //输出
    for(int i = 0; i <= MAXN; i++){
        if(cnt[i]){ //检测数字i有没有出现过
            //出现几次输出几个i
            for(int j = 1; j <= cnt[i]; j++) cout <<i <<' ';
        }
    }
    return 0;
}
```