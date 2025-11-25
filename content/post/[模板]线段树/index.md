+++
date = '2025-11-18T08:09:32+08:00'
draft = true
title = '[模板]线段树'
+++

# 线段树

## 线段树的学习——用于维护区间信息( logn )

基于二叉树建树 

用以下方法定义父节点与子节点的关系:

```
    #define lc p << 1
    #define rc p << 1 | 1
```

设定$w$为需要维护的数组
定义：

```
struct  node {
    int l, r, sum;
}tr [N * 4];
```

进行建树用$tr$来表示树建完后的状态



占用比一般在0.9左右，所以我们一般开线段树都是四倍的空间

### 建树

```
void build(int p, int l,int r) {
    tr [p] = { l,r,w [l] };
    if (l == r)return;
    int mid = l + r >> 1;
    build(lc, l, mid);
    build(rc, mid + 1, r);
    tr [p].sum = tr [lc].sum + tr [rc].sum;
}
```

### 点修改

### 复杂度为O(logn)

```
void update(int p, int x, int k) {//点修改
    if (tr[p].l == x && tr[p].r == x) {
        tr[p].sum += k;
        return;//叶子修改
    }
    int mid = tr[p].l + tr[p].r >> 1;
    if (x <= mid)update(lc, x, k);
    if (x > mid)update(rc, x, k);
    tr[p].sum = tr[lc].sum + tr[rc].sum;
}
```

从根节点进入，递归查找叶子节点，修改叶子节点的值，从下到上更新统计值

### 区间查询

运用的是拆分和拼凑的思想

从根节点进入，递归进行以下操作:

1. 若查询区间 [l, r] 完全覆盖`当前节点`区间 ，立即回溯，并返回该节点的sum值
2. 若左子节点与[l, r]有重叠，则递归访问左子树
3. 若右子节点与[l, r]有重叠，则递归访问右子树

```
int query(int p, int x, int y) {//区间查询
    if (x <= tr[p].l && tr[p].r <= y)
        return tr[p].sum;
    int mid = tr[p].l + tr[p].r >> 1;
    int sum = 0;
    if (x <= mid)sum += query(lc, x, y);
    if (y > mid)sum += query(rc, x, y);
    return sum;
}
```

## 线段树模板

### 区间求和

然后加上一些对懒的学习，可以封装一下做到下面这样的结构体封装：

```
struct SGT{
  #define lc u<<1
  #define rc u<<1|1
  struct sgt{ int l,r,sum,add; }tr[N*4];

  void pushup(int u){ //上传
    tr[u].sum=tr[lc].sum+tr[rc].sum;
  }
  void pushdown(int u){ //下传
    if(tr[u].add){
      tr[lc].sum+=tr[u].add*(tr[lc].r-tr[lc].l+1),
      tr[rc].sum+=tr[u].add*(tr[rc].r-tr[rc].l+1),
      tr[lc].add+=tr[u].add,
      tr[rc].add+=tr[u].add,
      tr[u].add=0;      
    }
  }
  void build(int u,int l,int r){ //建树
    tr[u]={l,r,w[l],0};
    if(l==r) return;
    int m=l+r>>1;
    build(lc,l,m);
    build(rc,m+1,r);
    pushup(u);
  }
  void change(int u,int x,int y,int k){ //区修
    if(x>tr[u].r || y<tr[u].l) return;
    if(x<=tr[u].l && tr[u].r<=y){
      tr[u].sum+=(tr[u].r-tr[u].l+1)*k;
      tr[u].add+=k;
      return;
    }
    pushdown(u);
    change(lc,x,y,k); 
    change(rc,x,y,k);
    pushup(u);
  }
  int query(int u,int x,int y){ //区查
    if(x>tr[u].r || y<tr[u].l) return 0;
    if(x<=tr[u].l && tr[u].r<=y) return tr[u].sum;
    pushdown(u);
    return query(lc,x,y)+query(rc,x,y);
  }
}S;
```

### 多功能线段树

然后在下面放一个多功能线段树便于其他变换的线段树理解

```
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

struct SegmentTree {
#define lc (u << 1)      // 左儿子索引
#define rc (u << 1 | 1)  // 右儿子索引

    struct Node {
        int l, r;           // 节点管理的区间 [l, r]
        long long sum;      // 区间和
        int min_val;        // 区间最小值
        int max_val;        // 区间最大值
        int add;            // 懒惰标记，表示本区间所有元素需要加的值
        // 构造函数，初始化节点，懒惰标记初始为0
        Node(int l = 0, int r = 0, long long sum = 0, int min_val = 0, int max_val = 0, int add = 0)
            : l(l), r(r), sum(sum), min_val(min_val), max_val(max_val), add(add) {}
    };

    vector<Node> tr; // 线段树数组
    vector<int>& w;  // 对原始数组的引用

    // 构造函数：接收原始数组，初始化线段树
    SegmentTree(vector<int>& nums) : w(nums) {
        int n = w.size();
        tr.resize(n * 4); // 分配4倍空间
        build(1, 0, n - 1);
    }

    // 向上更新父节点信息（合并左右子树信息）
    void pushup(int u) {
        tr[u].sum = tr[lc].sum + tr[rc].sum;
        tr[u].min_val = min(tr[lc].min_val, tr[rc].min_val);
        tr[u].max_val = max(tr[lc].max_val, tr[rc].max_val);
    }

    // 向下传递懒惰标记
    void pushdown(int u) {
        if (tr[u].add != 0) { // 如果存在懒惰标记
            int len_left = tr[lc].r - tr[lc].l + 1;  // 左子树区间长度
            int len_right = tr[rc].r - tr[rc].l + 1; // 右子树区间长度

            // 更新左子树：值加上标记*区间长度，最值加上标记，懒惰标记累加
            tr[lc].sum += (long long)tr[u].add * len_left;
            tr[lc].min_val += tr[u].add;
            tr[lc].max_val += tr[u].add;
            tr[lc].add += tr[u].add;

            // 更新右子树
            tr[rc].sum += (long long)tr[u].add * len_right;
            tr[rc].min_val += tr[u].add;
            tr[rc].max_val += tr[u].add;
            tr[rc].add += tr[u].add;

            tr[u].add = 0; // 清空当前节点的懒惰标记
        }
    }

    // 建树
    void build(int u, int l, int r) {
        if (l == r) {
            tr[u] = Node(l, r, w[l], w[l], w[l], 0); // 叶子节点
            return;
        }
        tr[u].l = l;
        tr[u].r = r;
        int mid = (l + r) >> 1;
        build(lc, l, mid);
        build(rc, mid + 1, r);
        pushup(u); // 非叶子节点，构建完后向上更新信息
    }

    // 区间修改：[x, y] 内的每个元素加上 k
    void update(int u, int x, int y, int k) {
        if (x > tr[u].r || y < tr[u].l) return; // 无交集

        if (x <= tr[u].l && tr[u].r <= y) { // 当前区间被完全覆盖
            int len = tr[u].r - tr[u].l + 1;
            tr[u].sum += (long long)k * len; // 更新区间和
            tr[u].min_val += k;              // 更新最小值
            tr[u].max_val += k;              // 更新最大值
            tr[u].add += k;                  // 打上懒惰标记
            return;
        }

        pushdown(u); // 下传懒惰标记
        int mid = (tr[u].l + tr[u].r) >> 1;
        if (x <= mid) update(lc, x, y, k);
        if (y > mid) update(rc, x, y, k);
        pushup(u); // 递归后更新当前节点
    }

    // 查询区间信息（和、最小值、最大值）
    // 使用引用参数返回结果，避免创建临时结构体[1](@ref)
    void query(int u, int x, int y, long long& sum, int& min_val, int& max_val) {
        if (x > tr[u].r || y < tr[u].l) { // 无交集，返回不影响结果的值
            sum = 0;
            min_val = INT_MAX;
            max_val = INT_MIN;
            return;
        }

        if (x <= tr[u].l && tr[u].r <= y) { // 当前区间被完全包含
            sum = tr[u].sum;
            min_val = tr[u].min_val;
            max_val = tr[u].max_val;
            return;
        }

        pushdown(u); // 下传懒惰标记

        int mid = (tr[u].l + tr[u].r) >> 1;
        long long sum_l = 0, sum_r = 0;
        int min_l = INT_MAX, min_r = INT_MAX;
        int max_l = INT_MIN, max_r = INT_MIN;

        if (x <= mid) {
            query(lc, x, y, sum_l, min_l, max_l); // 查询左子树
        }
        if (y > mid) {
            query(rc, x, y, sum_r, min_r, max_r); // 查询右子树
        }

        // 合并左右子树结果[1](@ref)
        sum = sum_l + sum_r;
        min_val = min(min_l, min_r);
        max_val = max(max_l, max_r);
    }

    // --- 对外的简化接口 ---
    void update_range(int l, int r, int k) {
        update(1, l, r, k);
    }

    long long query_sum(int l, int r) {
        long long sum;
        int min_val, max_val; // 临时变量，无需使用
        query(1, l, r, sum, min_val, max_val);
        return sum;
    }

    int query_min(int l, int r) {
        long long sum; // 临时变量，无需使用
        int min_val, max_val;
        query(1, l, r, sum, min_val, max_val);
        return min_val;
    }

    int query_max(int l, int r) {
        long long sum; // 临时变量，无需使用
        int min_val, max_val;
        query(1, l, r, sum, min_val, max_val);
        return max_val;
    }
};
```

然后下面是学习线段树可以练习的题
