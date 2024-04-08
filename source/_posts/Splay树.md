---
title: Splay树
tags:
  - 数据结构
  - 入门
  - Splay树
categories:
  - ACM
katex: true
abbrlink: 19905
---

Splay树，即二叉伸展树，是一种非严格平衡的二叉搜索树，由丹尼尔·斯立特Daniel Sleator 和 罗伯特·恩卓·塔扬Robert Endre Tarjan 在1985年发明（摘自百度百科）。相比与AVL、红黑树而言，Splay树不需要额外地存储平衡信息，因此显得更为简洁，代码实现也较为简单（指150行代码），同时具有很强的扩展性，常用于算法竞赛中。

至于我为什么突然想要学Splay树，因为这是Tarjan发明的……没错，就是求强连通分量、割点、桥的Tarjan算法的那个Tarjan。事情的起因是我最近刚学完强连通分量，然后查了一下Tarjan的资料，发现原来是祖师爷级别的大佬，拿过图灵奖，发明了很多图论的重要算法和数据结构。于是就对Splay树产生了兴趣，虽然这玩意已经明显超过了我的算法水平（刚学完强连通的蒟蒻），但我还是~~自不量力~~毅然决然地花了两天手搓了Splay，然后又花了一天的时间调试。
那么废话了那么多，接下来正式介绍Splay树

（以下内容和代码部分~~抄袭~~参考自[oiwiki](https://oi-wiki.org/ds/splay/)和其他人的博客、文章等）
（图片均使用开源网站[ioDraw](https://www.iodraw.com/)绘制，感谢开源者的贡献）

---

## 基本成员

Splay树需要存储的主要信息如下：

- `rt`——根节点，`ch[][2]`——左右子节点，`fa[]`——父节点；
- `cnt[]`——每个节点有几个相同的值，`sz[]`——每个节点子树的大小，`v[]`——节点的值；

然后就是变量`tot`为新插入的节点分配地址。

## 基本函数

首先是Splay树的几个基本函数：

- `get()`判断一个节点是父节点的左节点还是右节点；
- `clear()`清除一个节点的相关数据；
- `update()//网上一般叫maintain，但我用这个用习惯了……`更新（维护）一个节点的相关信息（子树大小等）；
  
```c++
bool get(int x) { return x == ch[fa[x]][1]; }
void clear(int x) { v[x] = cnt[x] = fa[x] = ch[x][0] = ch[x][1] = sz[x] = 0; }
void update(int x) { sz[x] = sz[ch[x][0]] + sz[ch[x][1]] + cnt[x]; }
```

~~然后就是Splay树最主要的操作、本文的主角——`splay`：~~
在介绍splay操作之前，我们还需要一点前置知识。

我们知道，普通的二叉搜索树容易退化成链导致效率变低，所以需要调整。但调整的同时又不能改变二叉搜索树的性质，因此显然不能直接去交换。我们需要一种新的操作——旋转。
![旋转](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/rotate.png)旋转就是把一边过多的节点往另一边“匀一下”，让整棵树更平衡。因为对搜索树来说，根节点的左子树都小于根节点，右子树都大于根节点，如果我们想要让两边的节点数更平衡的话，那就只能让根节点“易位”，换一个数来当根节点。所以旋转的本质就是改变根节点（个人认为§(*￣▽￣*)§）。同时旋转也不能改变搜索树的性质，因此两边的树的中序遍历都是1 2 3 4 5。

那么旋转操作具体要如何实现呢？让我们换一张图片来更好地说明。
![旋转](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/rotate1.png)我们的目标是把x节点旋转到y节点，其他节点都不是必须的，因此用虚线表示。
首先看看哪些节点的父节点发生了改变，首先y的父亲变成了x，然后3的父亲变成了y，但要注意的是，3节点不是必须的，也就是说，实际旋转的时候3节点可能不存在，因此需要先判定一下，然后就是x的父亲变成了z，这里z虽然也可能不存在，但如果z不存在那么z就是0，x是根节点，根节点的父亲是0这很合理，所以不需要判定。
接下来再来看看子节点的变化，子节点的变化和父节点的变化是相对应的，x的右孩子变成y，y的左孩子变成3，如果3不存在则设为0，然后z的左孩子变成x，注意判定z是否存在。
总结来说，在一次旋转操作中，我们要改变6个指针的指向，3个指向父亲的，3个指向孩子的，最后别忘了更新相应节点，具体代码实现如下：

```c++
void rotate(int x) // 旋转
{
    int cip = get(x), y = fa[x], z = fa[y];
    fa[x] = z, fa[y] = x;
    if (ch[x][cip ^ 1])
        fa[ch[x][cip ^ 1]] = y; // 改变父亲
    ch[y][cip] = ch[x][cip ^ 1];
    ch[x][cip ^ 1] = y;
    if (z)
        ch[z][y == ch[z][1]] = x; // 改变孩子
    update(y), update(x);
}
```

说完了旋转，接下来就真的正式进入到Splay树最重要的操作——`splay`。

所谓`splay`操作，就是把一个节点通过旋转移动至根节点。Splay树规定，每次访问一个节点（插入、查询等），就必须将其旋转至根节点，这涉及到Splay树的核心思想——刚被访问的节点之后仍可能被访问，因此应该让它更靠近根节点。

那么具体要如何实现呢？根据父节点是否是根节点，以及与父节点和父节点的父节点是否在同一边，分为三种情况：

- 父节点是根节点；
- 父节点不是根节点：

  - 子节点是父节点的左（右）孩子，同时父节点也是左（右）孩子；
  - 子节点是父节点的左（右）孩子，但父节点是右（左）孩子；

文字的描述可能不是很直观，直接来看图片。

首先是第一种情况：![splay](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/splay1.png)这种最简单，直接旋转一次x即可。
接下来是第二种情况：![splay](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/splay2.png)对于x与y处于同侧的情况，需要先旋转父节点y，再旋转x。
最后是第三种情况：![splay](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/splay3.png)
这种情况下，旋转两次x节点即可。

虽然听起来好像很麻烦，但其实代码很短：

```c++
void splay(int x)
{
    for (int f = fa[x]; f = fa[x], f; rotate(x))
        if (fa[f]) // 父节点为根节点只用旋转一次
            rotate(get(x) == get(f) ? f : x);
    rt = x;
}
```

当然，也许有人注意到了，之前不是说旋转操作可以把节点移动至根节点吗？那么为什么不直接一直旋转x节点呢？

这就涉及到单旋与双旋的区别，具体可以去知乎看一下这个问题[Splay 中的旋转操作用单旋与双旋的区别是什么？](https://www.zhihu.com/question/40777845)简单来说，就是双旋可以用势能分析的方法证明其均摊复杂度为$O(\log n)$，而单旋会被卡成链。至于势能分析怎么分析的，等我哪天有时间了也许会学一下（咕咕咕(/▽＼)）。

## 基本功能

在上面介绍的几个函数的帮助下，我们的Splay树终于可以成为一个真正的平衡树，去完成平衡树应有的操作了。
<(￣︶￣)↗\[GO!]

### 插入

第一个功能当然就是插入了，不能插入就始终是一棵空树，更不用说后面的操作了。
对于插入操作，首先要判断树是否为空，如果为空就直接插入根节点。否则就按照搜索树的性质往下查找，如果找到了相应节点，就把节点的计数加1，如果找不到则创建一个新节点。记得更新：

```c++
void insert(int x)
{
    if (!rt) { // 根节点为空
        v[++tot] = x, cnt[tot]++;
        rt = tot, update(tot);
        return;
    }
    int p = rt, f = 0;
    while (true) {
        if (v[p] == x) { // 存在对应节点
            cnt[p]++;
            update(p), update(f);
            splay(p);
            break;
        }
        f = p, p = ch[p][x > v[p]];
        if (!p) { // 不存在对应节点
            v[++tot] = x, cnt[tot]++;
            ch[f][x > v[f]] = tot;
            fa[tot] = f;
            update(tot), update(f);
            splay(tot);
            break;
        }
    }
}
```

### 查询排名

既然叫搜索树，那么当然有搜索的功能。如果要查询一个数`x`的排名，我们还是按照搜索树的性质向下查找。首先判断是否小于当前节点，如果小于直接搜索左子树，否则就让`ans`加上左子树的大小（之前记录的子树大小在这里排上用场了），因为我们已经可以确定左子树全部小于`x`，然后再判断是否等于当前节点，不等于则让`ans`加上当前节点的`cnt`值，然后搜索右子树，当找到的时候返回`ans + 1`，最后记得`splay`，具体代码如下：

```c++
int srank(T x) // 查询x的排名
{
    int ans = 0, p = rt;
    while (p) {
        if (x < v[p])
            p = ch[p][0];
        else if (x >= v[p]) {
            ans += sz[ch[p][0]];
            if (x == v[p]) {
                splay(p);
                return ans + 1;
            }
            ans += cnt[p];
            p = ch[p][1];
        }
    }
    return ans + 1;
}
```

### 查询第k个数

查询第k个数和查询排名的过程差不多，只是反过来而已。
首先还是先判断左子树，只不过这次是判断k是否大于左子树的大小，小于则搜索左子树，否则就让`k -= sz[ch[p][0]] + cnt[p]`，即减去左子树的大小和当前节点的计数，然后判断k减完之后是否小于等于0，如果是则说明当前节点就是排名为k的数，否则就搜索右子树，最后记得`splay`，代码如下：

```c++
T kth(int k) // 查询第k个数
{
    int p = rt;
    while (p) {
        if (k > sz[ch[p][0]]) {
            k -= sz[ch[p][0]] + cnt[p];
            if (k <= 0) {
                splay(p);
                return v[p];
            }
            p = ch[p][1];
        } else
            p = ch[p][0];
    }
    return T{}; // 我也不知道能返回什么，就随便写了一个(～￣▽￣)～
}
```

### 合并两棵树

严格来说，这并不算Splay树的功能。因为对合并的两棵树有严格的要求（一棵树的最大值小于另一棵树的最小值），同时由于我们是用的数组模拟而不是指针，所以基本没有可操作性。这个操作仅仅是为了在删除一个节点之后合并左右子树的而已，而非真的合并两颗任意的Splay，但为了方便理解我还是把它单独写了一个函数。
至于合并的具体步骤，首先判断是否有子树为空，有则直接让根节点等于非空的那颗树，否则就先找到左子树的最大值，然后将其旋转至根节点，然后将右子树的根节点的父亲设为我们刚旋转上来的节点就行了。

```c++
void merge(int x, int y) // 合并两棵树，要求x的最大值小于y的最小值，用于删除操作中合并左右两颗子树
{
    if (!x || !y) {
        rt = (x ? x : y);
        return;
    }
    int p = x;
    while (ch[p][1])
        p = ch[p][1];
    splay(p); // 将x的最大值旋转至根节点
    ch[p][1] = y;
    fa[y] = p;
    update(p);
}
```

### 删除指定节点

要删除一个节点，首先需要将其旋转至根节点。然后看`cnt`是否大于1，如果大于则说明有多个相同值，只要把`cnt`减1即可，否则就用上面的函数合并左右子树，然后清除对应节点，代码如下：

```c++
void del(T x) // 删除值为x的节点
{
    srank(x); // 找到x并旋转至根节点
    if (cnt[rt] > 1) {
        cnt[rt]--;
        update(rt);
        return;
    }
    if (!ch[rt][0] || !ch[rt][1]) {
        int tmp = rt;
        rt = ch[rt][!ch[rt][0]];
        fa[rt] = 0;
        clear(tmp);
        return;
    }
    int tmp = rt;
    merge(ch[rt][0], ch[rt][1]);
    clear(tmp);
}
```

### 查找前驱、后继

x的前驱的定义是小于x的最大的数，后继则是反过来，指大于x的最小的数。直接求似乎有点麻烦，我们可以换一个思路。
后继与前驱类似，我们以前驱来作为例子。首先插入x，插入之后x被自动移动至根节点，那么这时x的左子树全部小于x，x的前驱就是左子树中最大的数，找到这个数返回，然后删除x节点即可。代码如下：

```c++
T pre_nxt(int x, bool op) // 前驱、后继
{
    insert(x);
    int p = ch[rt][op];
    if (!p)
        return T{};
    while (ch[p][!op])
        p = ch[p][!op];
    del(x);
    return v[p];
}
```

## 总结

至此，Splay树的基本功能就介绍完毕了ヾ(≧▽≦*)o。以下是完整代码：

```c++
template <typename T>
class Splay
{
private:
    int tot, rt;
    std::vector<T> v;
    std::vector<int> fa, cnt, sz;
    std::vector<std::array<int, 2>> ch;
    inline bool get(int x) { return x == ch[fa[x]][1]; }
    void update(int x) { sz[x] = sz[ch[x][0]] + sz[ch[x][1]] + cnt[x]; }
    void clear(int x) { v[x] = cnt[x] = fa[x] = ch[x][0] = ch[x][1] = sz[x] = 0; }
    void rotate(int x) // 旋转
    {
        int cip = get(x), y = fa[x], z = fa[y];
        fa[x] = z, fa[y] = x;
        if (ch[x][cip ^ 1])
            fa[ch[x][cip ^ 1]] = y; // 改变父亲
        ch[y][cip] = ch[x][cip ^ 1];
        ch[x][cip ^ 1] = y;
        if (z)
            ch[z][y == ch[z][1]] = x; // 改变孩子
        update(y), update(x);
    }
    void splay(int x)
    {
        for (int f = fa[x]; f = fa[x], f; rotate(x))
            if (fa[f]) // 父节点为根节点只用旋转一次
                rotate(get(x) == get(f) ? f : x);
        rt = x;
    }    
    void merge(int x, int y) // 合并两棵树，要求x的最大值小于y的最小值，用于删除操作中合并左右两颗子树
    {
        if (!x || !y) {
            rt = (x ? x : y);
            return;
        }
        int p = x;
        while (ch[p][1])
            p = ch[p][1];
        splay(p); // 将x的最大值旋转至根节点
        ch[p][1] = y;
        fa[y] = p;
        update(p);
    }

public:
    Splay(int x = 0)
        : rt{0}, tot{0}, ch(x + 10), fa(x + 10), v(x + 10), cnt(x + 10), sz(x + 10) {}
    ~Splay() noexcept {}
    void insert(int x)
    {
        if (!rt) { // 根节点为空
            v[++tot] = x, cnt[tot]++;
            rt = tot, update(tot);
            return;
        }
        int p = rt, f = 0;
        while (true) {
            if (v[p] == x) { // 存在对应节点
                cnt[p]++;
                update(p), update(f);
                splay(p);
                break;
            }
            f = p, p = ch[p][x > v[p]];
            if (!p) { // 不存在对应节点
                v[++tot] = x, cnt[tot]++;
                ch[f][x > v[f]] = tot;
                fa[tot] = f;
                update(tot), update(f);
                splay(tot);
                break;
            }
        }
    }
    int srank(T x) // 查询x的排名
    {
        int ans = 0, p = rt;
        while (p) {
            if (x < v[p])
                p = ch[p][0];
            else if (x >= v[p]) {
                ans += sz[ch[p][0]];
                if (x == v[p]) {
                    splay(p);
                    return ans + 1;
                }
                ans += cnt[p];
                p = ch[p][1];
            }
        }
        return ans + 1;
    }
    T kth(int k) // 查询第k个数
    {
        int p = rt;
        while (p) {
            if (k > sz[ch[p][0]]) {
                k -= sz[ch[p][0]] + cnt[p];
                if (k <= 0) {
                    splay(p);
                    return v[p];
                }
                p = ch[p][1];
            } else
                p = ch[p][0];
        }
        return T{};
    }
    void del(T x) // 删除值为x的节点
    {
        srank(x); // 找到x并旋转至根节点
        if (cnt[rt] > 1) {
            cnt[rt]--;
            update(rt);
            return;
        }
        if (!ch[rt][0] || !ch[rt][1]) {
            int tmp = rt;
            rt = ch[rt][!ch[rt][0]];
            fa[rt] = 0;
            clear(tmp);
            return;
        }
        int tmp = rt;
        merge(ch[rt][0], ch[rt][1]);
        clear(tmp);
    }
    T pre_nxt(T x, bool op) // 前驱、后继
    {
        insert(x);
        int p = ch[rt][op];
        if (!p)
            return T{};
        while (ch[p][!op])
            p = ch[p][!op];
        del(x);
        return v[p];
    }
};
```

（这段代码能过洛谷的模板题，应该是没什么大问题吧……）

总的来说，Splay树作为Tarjan大神提出来的数据结构，还是非常有学习的价值的。维持平衡的操作只有一个`splay`而已，而且也不需要分那么多种情况，其他的函数都可以视情况进行修改和删除，扩展性非常强。

---

闲聊：

想给芙宁娜一个完整的家（
![辛苦你了](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/fufu.jpg)
