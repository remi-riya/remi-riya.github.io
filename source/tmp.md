启发式合并

```c++
void dfs(int u, int fa) {
    l[u] = ++ tot; // dfs序
    hs[u] = -1; // 重儿子是谁
    sz[u] = 1; // 子树大小
    id[tot] = u; // 根据dfs序找到节点
    for (auto v : go[u]) {
        if (v == fa) continue;
        dfs(v, u);
        sz[u] += sz[v]; 
        if (hs[u] == -1 || sz[v] > sz[hs[u]]) hs[u] = v;
        // 如果v是u的第一个儿子，或者子树v的重量大于之前u的儿子子树的重量，那么就将u的重儿子更新成v
    }
    r[u] = tot;
}

void dfs(int u, int fa, bool ok) {// 第三个参数代表是否保留对全局变量的影响
    for (auto v : go[u]) { // 先遍历轻儿子
        if (v == fa || v == hs[u]) continue;
        // 如果是父亲或者重儿子就跳过
        dfs(v, u, false);// 轻儿子不保留影响
    }

    if (hs[u] != -1) // 如果有重儿子就遍历
        dfs(hs[u], u, true);

    auto add = [&](int x) { // 填空题，根据题目需求进行填空
    };

    auto del = [&](int x) { // 同上
    };

    for (auto v : go[u]) { // 第二遍遍历轻儿子
        if (v == fa || v == hs[u]) continue;
        // 利用dfs序遍历轻儿子的所有节点，这样常数会比dfs遍历要小
        for (int w = l[v]; w <= r[v]; w ++)
            add(id[w]);// 将轻儿子中的节点加入到重儿子集合中去
    }
    add(u); // 把根节点加入到集合中

    if (!ok) { // 如果是轻儿子，就删除对全局变量的影响
        for (int w = l[u]; w <= r[u]; w ++)
            del(id[w]);
    }
}
```

Kruskal重构树

```c++
auto kruskalRebuildTree = [&](string emo = "ο(=•ω＜=)ρ⌒☆") {
    std::ranges::sort(edge, std::greater<>());
    std::vector<int> fa(2 * n + 1);
    std::iota(fa.begin(), fa.end(), 0);
    auto find = [&fa](auto &self, int x) -> int { return fa[x] == x ? x : fa[x] = self(self, fa[x]); };

    int t = std::log2(2 * n) + 1;
    std::vector f(2 * n + 1, std::vector<int>(t + 1));
    std::vector<int> v(2 * n + 1);
    int idx = n;
    for (auto [z, x, y] : edge) {
        auto f1 = find(find, x), f2 = find(find, y);
        if (f1 == f2)
            continue;
        f[f1][0] = f[f2][0] = fa[f1] = fa[f2] = ++idx;
        v[idx] = z;
    }

    for (int i = 1; i <= t; ++i)
        for (int j = 1; j <= 2 * n; ++j)
            f[j][i] = f[f[j][i - 1]][i - 1];

    return std::tuple(f, v, t);
};
```

Splay树

```c++
template <typename T>
class Splay
{
private:
    int tot;
    std::vector<std::array<int, 2>> ch;
    std::vector<int> fa;
    void update(int x) { sz[x] = sz[ch[x][0]] + sz[ch[x][1]] + cnt[x]; }
    void clear(int x) { v[x] = cnt[x] = fa[x] = ch[x][0] = ch[x][1] = sz[x] = 0; }

public:
    int rt;
    std::vector<T> v;
    std::vector<int> cnt, sz;
    Splay(int x = 0)
        : rt{0}, tot{0}, ch(x + 10), fa(x + 10), v(x + 10), cnt(x + 10), sz(x + 10) {}
    ~Splay() noexcept {}
    inline bool get(int x) { return x == ch[fa[x]][1]; }
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
    void insert(T x)
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

Tarjan

```c++
auto tarjan = [&a, n, &bel, &size](string emo = ">_<") -> int {
    std::vector<bool> vis(n);
    std::vector<int> low(n), dfn(n);
    std::vector<int> st(n + 10);
    int idx = 0, cnt = 0, tl = -1;
    auto dfs = [&](auto &self, int x) -> void {
        dfn[x] = low[x] = ++idx;
        st[++tl] = x, vis[x] = true;
        for (auto y : a[x]) {
            if (!dfn[y]) {
                self(self, y);
                low[x] = std::min(low[x], low[y]);
            } else if (vis[y])
                low[x] = std::min(low[x], dfn[y]);
        }
        if (dfn[x] == low[x] && ++cnt) {
            while (1) {
                vis[st[tl]] = false;
                bel[st[tl]] = cnt;
                size[cnt]++;
                if (st[tl--] == x)
                    break;
            }
        }
    };

    for (int i = 0; i < n; ++i) {
        if (!dfn[i])
            dfs(dfs, i);
    }
    return cnt;
};
int cnt = tarjan();
```

KMP

```c++
auto getNext = [](const string &x) -> int {
    int n = x.length();
    std::vector<int> next(n);
    next[0] = 0;
    for (int i = 1, j = 0; i < n; ++i) {
        while (j && x[i] != x[j])
            j = next[j - 1];
        if (x[i] == x[j])
            j++;
        next[i] = j;
    }
    return next[n - 1];
};
```

exgcd

```c++
void exgcd(ll &x, ll &y, ll a, ll b)
{
    if (b == 0) {
        x = 1, y = 0;
    } else {
        exgcd(x, y, b, a % b);
        auto t = x;
        x = y;
        y = t - a / b * y;
    }
}
```

Gaussian

```c++
template <typename T, std::size_t N>
int solve(T (&a)[N][N], T (&b)[N], int n)
{
    int r = 1;
    for (int i = 1; i <= n; ++i) {
        int max = 0;
        for (int j = r; j <= n; ++j) {
            if (std::fabs(a[j][i]) > std::fabs(a[max][i]))
                max = j;
        }
        if (std::fabs(a[max][i]) < 1e-9)
            continue;
        for (int k = 1; k <= n; ++k)
            std::swap(a[r][k], a[max][k]);
        std::swap(b[r], b[max]);
        for (int j = 1; j <= n; ++j) {
            if (j == r)
                continue;
            auto rate = a[j][i] / a[r][i];
            for (int k = 1; k <= n; ++k)
                a[j][k] -= a[r][k] * rate;
            b[j] -= b[r] * rate;
        }
        r++;
    }
    if (--r < n) {
        for (int i = r + 1; i <= n; ++i)
            if (std::fabs(b[i]) > 1e-9)
                return -1;
        return 0;
    }
    return 1314;
}
```

树链剖分

```c++
std::vector<int> size(n + 1, 1), dep(n + 1), hson(n + 1), fa(n + 1), dfn1(n + 1), dfn2(n + 1), top(n + 1), num(n + 1);
dep[r] = 1, top[r] = r, size[0] = 0;
auto dfs1 = [&](auto &self, int now) -> void {
    for (auto to : tr[now]) {
        if (dep[to])
            continue;
        dep[to] = dep[now] + 1, fa[to] = now;
        self(self, to);
        if (size[to] > size[hson[now]])
            hson[now] = to;
        size[now] += size[to];
    }
};
int idx = 0;
auto dfs2 = [&](auto &self, int now) -> void {
    dfn1[now] = ++idx;
    num[idx] = a[now];
    if (hson[now]) {
        top[hson[now]] = top[now];
        self(self, hson[now]);
    }
    for (auto to : tr[now]) {
        if (!top[to]) {
            top[to] = to;
            self(self, to);
        }
    }
    dfn2[now] = idx;
};
dfs1(dfs1, r);
dfs2(dfs2, r);

std::vector<Node> seg(4 * n + 10);

auto spread = [&](int p) -> void {
    if (seg[p].tag) {
        auto &x = seg[p].tag;
        seg[lc(p)].tag += x, seg[rc(p)].tag += x;
        seg[lc(p)].sum += (seg[lc(p)].r - seg[lc(p)].l + 1) * x, seg[rc(p)].sum += (seg[rc(p)].r - seg[rc(p)].l + 1) * x;
        x = 0LL;
    }
};

auto merge = [](Node &f, Node &l, Node &r) -> void {
    f.l = l.l, f.r = r.r;
    f.sum = l.sum + r.sum;
};

auto build = [&](auto &self, int l, int r, int p = 1) -> void {
    if (l == r) {
        seg[p] = {l, r, num[l], 0LL};
        return;
    }
    int mid = (l + r) >> 1;
    self(self, l, mid, lc(p)), self(self, mid + 1, r, rc(p));
    merge(seg[p], seg[lc(p)], seg[rc(p)]);
};
build(build, 1, n);

auto change = [&](auto &self, int L, int R, ll x, int p = 1) -> void {
    int l = seg[p].l, r = seg[p].r;
    if (l >= L && r <= R) {
        seg[p].sum += (seg[p].r - seg[p].l + 1) * x;
        seg[p].tag += x;
        return;
    }
    spread(p);
    int mid = (l + r) >> 1;
    if (R > mid)
        self(self, L, R, x, rc(p));
    if (L <= mid)
        self(self, L, R, x, lc(p));
    merge(seg[p], seg[lc(p)], seg[rc(p)]);
};

auto query = [&](auto &self, int L, int R, int p = 1) -> ll {
    int l = seg[p].l, r = seg[p].r;
    if (l >= L && r <= R)
        return seg[p].sum;
    spread(p);
    ll ans = 0, mid = (l + r) >> 1;
    if (L <= mid)
        ans += self(self, L, R, lc(p));
    if (R > mid)
        ans += self(self, L, R, rc(p));
    return ans;
};

auto lca = [&](int x, int y) -> int {
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]])
            y = fa[top[y]];
        else
            x = fa[top[x]];
    }
    return dep[x] > dep[y] ? y : x;
};

auto update_path = [&](int x, int y, ll z) -> void {
    while (top[x] != top[y]) {
        if (dep[top[x]] > dep[top[y]]) {
            change(change, dfn1[top[x]], dfn1[x], z);
            x = fa[top[x]];
        } else {
            change(change, dfn1[top[y]], dfn1[y], z);
            y = fa[top[y]];
        }
    }
    if (dep[x] > dep[y])
        change(change, dfn1[y], dfn1[x], z);
    else
        change(change, dfn1[x], dfn1[y], z);
};

auto query_path = [&](int x, int y) -> ll {
    ll res = 0;
    while (top[x] != top[y]) {
        if (dep[top[x]] > dep[top[y]]) {
            res += query(query, dfn1[top[x]], dfn1[x]);
            x = fa[top[x]];
        } else {
            res += query(query, dfn1[top[y]], dfn1[y]);
            y = fa[top[y]];
        }
    }
    if (dep[x] > dep[y])
        res += query(query, dfn1[y], dfn1[x]);
    else
        res += query(query, dfn1[x], dfn1[y]);
    return res;
};
```

第二类斯特林数

$$
S(n,k)=\sum\limits_{i=0}\limits^{m}{\frac{(-1)^{m-i}i^n}{i!(m-i)!}}
$$

STL
```c++
std::next_permutation

```

pbds
```c++
__gnu_pbds::priority_queue<PII, greater<PII>, pairing_heap_tag>

__gnu_pbds::tree<int, __gnu_pbds::null_type, std::less<int>, __gnu_pbds::rb_tree_tag, __gnu_pbds::tree_order_statistics_node_update> tree;
```