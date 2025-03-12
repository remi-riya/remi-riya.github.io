---
title: 牛客暑期多校第一场I-Mirror Maze
katex: false
abbrlink: 52837
date: 2024-07-17 12:20:32
tags:
categories:
---

花了我将近两天时间Debug的阴间题，写完感觉人生都空虚了（；´д｀）ゞ

---

题意还是很简单的，思路也很明显，就是记忆化搜索，或者像题解一样直接都预处理出来。
这题的图跟一般的图主要的区别在于图中的环是无法从外部进入的，因为光路是可逆的，所以光路只有两种可能，要么在一个环中死循环，要么形成一条链最后射出边界，虽然听起来很简单，但是我写了整整三个版本的代码，前两个版本都是没有考虑全面导致实现有问题，都是过60%，其中一个在我找出一个错误样例并改对后信心满满的交了一发之后从过60变成了过30……

下面是翻车经历：

最开始的时候我只开了两个数组，一个记录是否被访问，一个记录答案，然后发现一面镜子可能在一条路径中被访问多次，于是又开了一个数组记录路径编号，交一发喜提wa（以下代码都经过多次修改，我也不确定是不是当初交的那一版_(:з)∠)_）

```c++
#include <bits/stdc++.h>
#define lowbit(x) ((x) & -(x))
using std::cin, std::cout, std::string;
using ll = long long;
using ull = unsigned long long;
using pii = std::pair<int, int>;
using pll = std::pair<ll, ll>;
const int inf = 0x3f3f3f3f;
const ll INF = 0x3f3f3f3f3f3f3f3f;
const int mod = 998244353;
int main()
{
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m, q;
    cin >> n >> m;
    std::vector<std::string> a(n + 2);
    for (int i = 1; i <= n; ++i)
        cin >> a[i], a[i] = " " + a[i] + " ";

    std::vector F(n + 1, std::vector<std::array<int, 4>>(m + 1));
    std::vector vis(n + 1, std::vector<std::array<int, 4>>(m + 1));
    std::vector id(n + 1, std::vector<std::array<int, 4>>(m + 1));
    int d[4][2]{
        {-1, 0}, {0, 1}, {1, 0}, {0, -1}};
    std::map<char, std::array<int, 4>> mp{
        {'/', {1, 0, 3, 2}},
        {'\\', {3, 2, 1, 0}},
        {'|', {0, 3, 2, 1}},
        {'-', {2, 1, 0, 3}}};
    std::map<std::string, int> mp1{
        {"above", 0},
        {"below", 2},
        {"left", 3},
        {"right", 1}};

    int tag = 1, lst = 1;
    std::vector<std::array<int, 3>> is;
    auto dfs = [&](auto &self, int x, int y, int v) -> std::pair<int, bool> {
        if (F[x][y][v]) {
            tag = id[x][y][v];
            return {F[x][y][v], false};
        }

        int xx = x + d[v][0], yy = y + d[v][1];
        if (xx < 1 || xx > n || yy < 1 || yy > m) {
            tag = ++lst;
            return {0, false};
        }
        // std::cerr << std::format("x:{} y:{} v:{}\n", x, y, v);
        int vv = mp[a[xx][yy]][v];

        if (vis[xx][yy][vv] == tag) {
            tag = ++lst;
            return {0, true};
        }

        vis[xx][yy][vv] = tag;
        auto [t, flag] = self(self, xx, yy, vv);
        F[x][y][v] = (!(bool)std::count(id[xx][yy].begin(), id[xx][yy].end(), tag) && v != vv);
        id[xx][yy][vv] = tag;
        if (flag)
            is.push_back({x, y, v});
        return {F[x][y][v] += t, flag};
    };

    cin >> q;
    while (q--) {
        int u, v;
        std::string s;
        cin >> u >> v >> s;
        tag = ++lst;
        auto [ans, flag] = dfs(dfs, u, v, mp1[s]);
        if (flag) {
            for (auto [x, y, v] : is)
                F[x][y][v] = ans;
            is.clear();
        }
        std::cout << ans << '\n';
        // std::cerr << '\n';
    }
}
```

之后调了半天，发现对于有的镜子在同一条路径上可能会同时以反射镜和非反射镜出现，比如下面这个样例

```text
3 5
----|
//--\
\---/
1
1 2 below
```

其中`[3][2]`处的镜子就是同时以两种形式出现的，但是我判断是否重复时直接判断的是否出现，而不是是否作为反射镜出现。其实这一版已经很接近答案了，但是我当时一拍脑袋，直接用一个外部的`set`去重不就行了？于是就有了第二版代码

```c++
#include <bits/stdc++.h>
#define lowbit(x) ((x) & -(x))
using std::cin, std::cout, std::string;
using ll = long long;
using ull = unsigned long long;
using pii = std::pair<int, int>;
using pll = std::pair<ll, ll>;
const int inf = 0x3f3f3f3f;
const ll INF = 0x3f3f3f3f3f3f3f3f;
const int mod = 998244353;
int main()
{
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m, q;
    cin >> n >> m;
    std::vector<std::string> a(n + 2);
    for (int i = 1; i <= n; ++i)
        cin >> a[i], a[i] = " " + a[i] + " ";

    std::vector F(n + 1, std::vector<std::array<int, 4>>(m + 1));
    std::vector vis(n + 1, std::vector<std::array<int, 4>>(m + 1));
    int d[4][2]{
        {-1, 0}, {0, 1}, {1, 0}, {0, -1}};
    std::map<char, std::array<int, 4>> mp{
        {'/', {1, 0, 3, 2}},
        {'\\', {3, 2, 1, 0}},
        {'|', {0, 3, 2, 1}},
        {'-', {2, 1, 0, 3}}};
    std::map<std::string, int> mp1{
        {"above", 0},
        {"below", 2},
        {"left", 3},
        {"right", 1}};

    std::vector<std::array<int, 3>> st;
    std::set<std::pair<int, int>> ok;
    int ans = 0;
    auto dfs = [&](auto &self, int x, int y, int v, int tag) -> bool {
        st.push_back({x, y, v});
        if (F[x][y][v]) {
            ans += F[x][y][v];
            return false;
        }
        // std::cerr << std::format("x:{}, y:{}, v:{}\n", x, y, v);
        int xx = x + d[v][0], yy = y + d[v][1];
        if (xx < 1 || xx > n || yy < 1 || yy > m)
            return false;
        // std::cerr << x << ' ' << y << '\n';
        int vv = mp[a[xx][yy]][v];

        if (vv != v)
            ok.insert({xx, yy});
        if (vis[xx][yy][vv] == tag)
            return true;

        vis[xx][yy][vv] = tag;

        return self(self, xx, yy, vv, tag);
    };

    int idx = 1;
    cin >> q;
    while (q--) {
        int _u, _v;
        std::string op;
        cin >> _u >> _v >> op;
        int vv = mp1[op];
        ans = 0;
        // vis[_u][_v][vv] = ++idx;
        if (dfs(dfs, _u, _v, vv, idx)) {
            for (auto [x, y, v] : st)
                F[x][y][v] = ok.size();
            st.clear();
        } else {
            std::set<std::pair<int, int>> s;
            while (!st.empty()) {
                auto [x, y, v] = st.back();
                F[x][y][v] = s.size() + ans;
                if (ok.count({x, y}))
                    s.insert({x, y});
                st.pop_back();
            }
        }
        ok.clear();
        std::cout << F[_u][_v][vv] << '\n';
    }
}
```

这一版代码里面，我直接把`dfs`返回的答案都去掉了，只用来判断是否是环，由于光路不会出现分叉，所以可以用一个外部的栈记录链上的镜子出现的顺序，然后逆推答案，只需要把所有在本次搜索中作为反射镜的点都存到`set`里，逆推时判断。
改完后调过了几个样例，感觉很对，一交60变30，寄。

这个版本的问题主要在于一条链可能是分几次走的，也就是一次询问走了后半部分，下一次询问又从前半部分开始，由于对于走过的点只存了一个答案，而丢失了路径信息，有可能一个点在后半部分已经经过了一次，但是由于第二次从前面开始时无法知道后面走过了哪些点，就会重复计算，并且无法去重。

经过一天的调试，我确定必须要给每条路径一个编号，来判断是否重复出现，同时也需要记录是否访问，否则碰到环就会死循环，于是又回到了第一版代码，又调试了一两个小时，对拍了几次，终于过了这个抽象题。

```c++
#include <bits/stdc++.h>
#define lowbit(x) ((x) & -(x))
using std::cin, std::cout, std::string;
using ll = long long;
using ull = unsigned long long;
using pii = std::pair<int, int>;
using pll = std::pair<ll, ll>;
const int inf = 0x3f3f3f3f;
const ll INF = 0x3f3f3f3f3f3f3f3f;
const int mod = 998244353;
int main()
{
    std::ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m, q;
    cin >> n >> m;
    std::vector<std::string> a(n + 2);
    for (int i = 1; i <= n; ++i)
        cin >> a[i], a[i] = " " + a[i] + " ";

    std::vector F(n + 1, std::vector<std::array<int, 4>>(m + 1));
    std::vector id(n + 1, std::vector<std::array<int, 4>>(m + 1));
    std::vector used(n + 1, std::vector<std::array<int, 4>>(m + 1));
    int d[4][2]{
        {-1, 0}, {0, 1}, {1, 0}, {0, -1}};
    std::map<char, std::array<int, 4>> mp{
        {'/', {1, 0, 3, 2}},
        {'\\', {3, 2, 1, 0}},
        {'|', {0, 3, 2, 1}},
        {'-', {2, 1, 0, 3}}};
    std::map<std::string, int> mp1{
        {"above", 0},
        {"below", 2},
        {"left", 3},
        {"right", 1}};

    int tag = 1, lst = 1;
    std::vector<std::array<int, 3>> is;
    auto dfs = [&](auto &self, int x, int y, int v) -> std::pair<int, bool> {
        if (x < 1 || x > n || y < 1 || y > m) {
            // 越界说明是一条新的链，分配新的编号
            tag = ++lst;
            return {0, false};
        }
        if (F[x][y][v]) {
            // 说明后面的链是之前走过的，把编号设置为对应的编号
            tag = id[x][y][v];
            return {F[x][y][v], false};
        }
        if (id[x][y][v] == tag) {
            // 在本次递归中访问过，说明是环
            tag = ++lst;
            return {0, true};
        }

        int vv = mp[a[x][y]][v];
        int xx = x + d[vv][0], yy = y + d[vv][1];
        // std::cerr << std::format("x:{} y:{} v:{}\n", x, y, v);

        id[x][y][v] = tag; // 临时编号
        auto [t, flag] = self(self, xx, yy, vv);
        if (v != vv) {
            // 去重
            F[x][y][v] = !(bool)std::count(used[x][y].begin(), used[x][y].end(), tag);
            used[x][y][v] = tag;
        }
        id[x][y][v] = tag; // 真正的路径编号
        if (flag)
            is.push_back({x, y, v});
        return {F[x][y][v] += t, flag};
    };

    cin >> q;
    while (q--) {
        int u, v;
        std::string s;
        cin >> u >> v >> s;
        int vv = mp1[s];
        u += d[vv][0], v += d[vv][1];
        tag = ++lst; // 分配临时编号
        auto [ans, flag] = dfs(dfs, u, v, vv);
        if (flag) {
            // 环单独处理
            for (auto [x, y, v] : is)
                F[x][y][v] = ans;
            is.clear();
        }
        std::cout << ans << '\n';
        // std::cerr << '\n';
    }
}

/*
⣿⣿⣿⣿⣿⣿⡷⣯⢿⣿⣷⣻⢯⣿⡽⣻⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣇⠸⣿⣿⣆⠹⣿⣿⢾⣟⣯⣿⣿⣿⣿⣿⣿⣽⣻⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣻⣽⡿⣿⣎⠙⣿⣞⣷⡌⢻⣟⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣿⣿⣿⣿⣿⣿⡄⠹⣿⣿⡆⠻⣿⣟⣯⡿⣽⡿⣿⣿⣿⣿⣽⡷⣯⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣟⣷⣿⣿⣿⡀⠹⣟⣾⣟⣆⠹⣯⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⢠⡘⣿⣿⡄⠉⢿⣿⣽⡷⣿⣻⣿⣿⣿⣿⡝⣷⣯⢿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣯⢿⣾⢿⣿⡄⢄⠘⢿⣞⡿⣧⡈⢷⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⢸⣧⠘⣿⣷⠈⣦⠙⢿⣽⣷⣻⣽⣿⣿⣿⣿⣌⢿⣯⢿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣟⣯⣿⢿⣿⡆⢸⡷⡈⢻⡽⣷⡷⡄⠻⣽⣿⣿⡿⣿⣿⣿⣿⣿⣿⣷⣿⣿⣿⣿⣏⢰⣯⢷⠈⣿⡆⢹⢷⡌⠻⡾⢋⣱⣯⣿⣿⣿⣿⡆⢻⡿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⡎⣿⢾⡿⣿⡆⢸⣽⢻⣄⠹⣷⣟⣿⣄⠹⣟⣿⣿⣟⣿⣿⣿⣿⣿⣿⣽⣿⣿⣿⡇⢸⣯⣟⣧⠘⣷⠈⡯⠛⢀⡐⢾⣟⣷⣻⣿⣿⣿⡿⡌⢿⣻⣿⣿
⣿⣿⣿⣿⣿⣿⣧⢸⡿⣟⣿⡇⢸⣯⣟⣮⢧⡈⢿⣞⡿⣦⠘⠏⣹⣿⣽⢿⣿⣿⣿⣿⣯⣿⣿⣿⡇⢸⣿⣿⣾⡆⠹⢀⣠⣾⣟⣷⡈⢿⣞⣯⢿⣿⣿⣿⢷⠘⣯⣿⣿
⣿⣿⣿⣿⣿⣿⣿⡈⣿⢿⣽⡇⠘⠛⠛⠛⠓⠓⠈⠛⠛⠟⠇⢀⢿⣻⣿⣯⢿⣿⣿⣿⣷⢿⣿⣿⠁⣾⣿⣿⣿⣧⡄⠇⣹⣿⣾⣯⣿⡄⠻⣽⣯⢿⣻⣿⣿⡇⢹⣾⣿
⣿⣿⣿⣿⣿⣿⣿⡇⢹⣿⡽⡇⢸⣿⣿⣿⣿⣿⣞⣆⠰⣶⣶⡄⢀⢻⡿⣯⣿⡽⣿⣿⣿⢯⣟⡿⢀⣿⣿⣿⣿⣿⣧⠐⣸⣿⣿⣷⣿⣿⣆⠹⣯⣿⣻⣿⣿⣿⢀⣿⢿
⣿⣿⣿⣿⣿⣿⣿⣿⠘⣯⡿⡇⢸⣿⣿⣿⣿⣿⣿⣿⣧⡈⢿⣳⠘⡄⠻⣿⢾⣽⣟⡿⣿⢯⣿⡇⢸⣿⣿⣿⣿⣿⣿⡀⢾⣿⣿⣿⣿⣿⣿⣆⠹⣾⣷⣻⣿⡿⡇⢸⣿
⣿⣿⣿⣿⣿⣿⣿⣿⡇⢹⣿⠇⢸⣿⣿⣿⣿⣿⣿⣿⣿⣷⣄⠻⡇⢹⣆⠹⣟⣾⣽⣻⣟⣿⣽⠁⣾⣿⣿⣿⣿⣿⣿⣇⣿⣿⠿⠛⠛⠉⠙⠋⢀⠁⢘⣯⣿⣿⣧⠘⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⡈⣿⡃⢼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣦⡙⠌⣿⣆⠘⣿⣞⡿⣞⡿⡞⢠⣿⣿⣿⣿⣿⡿⠛⠉⠁⢀⣀⣠⣤⣤⣶⣶⣶⡆⢻⣽⣞⡿⣷⠈⣿
⣿⣿⣿⣿⣿⣿⣿⣿⡿⠃⠘⠁⠉⠉⠉⠉⠉⠉⠉⠉⠉⠙⠛⠛⢿⣄⢻⣿⣧⠘⢯⣟⡿⣽⠁⣾⣿⣿⣿⣿⣿⡃⢀⢀⠘⠛⠿⢿⣻⣟⣯⣽⣻⣵⡀⢿⣯⣟⣿⢀⣿
⣿⣿⣿⣟⣿⣿⣿⣿⣶⣶⡆⢀⣿⣾⣿⣾⣷⣿⣶⠿⠚⠉⢀⢀⣤⣿⣷⣿⣿⣷⡈⢿⣻⢃⣼⣿⣿⣿⣿⣻⣿⣿⣿⡶⣦⣤⣄⣀⡀⠉⠛⠛⠷⣯⣳⠈⣾⡽⣾⢀⣿
⣿⢿⣿⣿⣻⣿⣿⣿⣿⣿⡿⠐⣿⣿⣿⣿⠿⠋⠁⢀⢀⣤⣾⣿⣿⣿⣿⣿⣿⣿⣿⣌⣥⣾⡿⣿⣿⣷⣿⣿⢿⣷⣿⣿⣟⣾⣽⣳⢯⣟⣶⣦⣤⡾⣟⣦⠘⣿⢾⡁⢺
⣿⣻⣿⣿⡷⣿⣿⣿⣿⣿⡗⣦⠸⡿⠋⠁⢀⢀⣠⣴⢿⣿⣽⣻⢽⣾⣟⣷⣿⣟⣿⣿⣿⣳⠿⣵⣧⣼⣿⣿⣿⣿⣿⣾⣿⣿⣿⣿⣿⣽⣳⣯⣿⣿⣿⣽⢀⢷⣻⠄⠘
⣿⢷⣻⣿⣿⣷⣻⣿⣿⣿⡷⠛⣁⢀⣀⣤⣶⣿⣛⡿⣿⣮⣽⡻⣿⣮⣽⣻⢯⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⢀⢸⣿⢀⡆
⠸⣟⣯⣿⣿⣷⢿⣽⣿⣿⣷⣿⣷⣆⠹⣿⣶⣯⠿⣿⣶⣟⣻⢿⣷⣽⣻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢀⣯⣟⢀⡇
⣇⠹⣟⣾⣻⣿⣿⢾⡽⣿⣿⣿⣿⣿⣆⢹⣶⣿⣻⣷⣯⣟⣿⣿⣽⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⢀⡿⡇⢸⡇
⣿⣆⠹⣷⡻⣽⣿⣯⢿⣽⣻⣿⣿⣿⣿⣆⢻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠛⢻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠇⢸⣿⠇⣼⡇
⡙⠾⣆⠹⣿⣦⠛⣿⢯⣷⢿⡽⣿⣿⣿⣿⣆⠻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠃⠎⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠏⢀⣿⣾⣣⡿⡇
⣿⣷⡌⢦⠙⣿⣿⣌⠻⣽⢯⣿⣽⣻⣿⣿⣿⣧⠩⢻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡏⢰⢣⠘⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠃⢀⢀⢿⣞⣷⢿⡇
⣿⣽⣆⠹⣧⠘⣿⣿⡷⣌⠙⢷⣯⡷⣟⣿⣿⣿⣷⡀⡹⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣈⠃⣸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠟⢀⣴⡧⢀⠸⣿⡽⣿⢀
⢻⣽⣿⡄⢻⣷⡈⢿⣿⣿⢧⢀⠙⢿⣻⡾⣽⣻⣿⣿⣄⠌⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠛⢁⣰⣾⣟⡿⢀⡄⢿⣟⣿⢀
⡄⢿⣿⣷⢀⠹⣟⣆⠻⣿⣿⣆⢀⣀⠉⠻⣿⡽⣯⣿⣿⣷⣈⢻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠋⢀⣠⠘⣯⣷⣿⡟⢀⢆⠸⣿⡟⢸
⣷⡈⢿⣿⣇⢱⡘⢿⣷⣬⣙⠿⣧⠘⣆⢀⠈⠻⣷⣟⣾⢿⣿⣆⠹⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠋⣠⡞⢡⣿⢀⣿⣿⣿⠇⡄⢸⡄⢻⡇⣼
⣿⣷⡈⢿⣿⡆⢣⡀⠙⢾⣟⣿⣿⣷⡈⠂⠘⣦⡈⠿⣯⣿⢾⣿⣆⠙⠻⠿⠿⠿⠿⡿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠿⠛⢋⣠⣾⡟⢠⣿⣿⢀⣿⣿⡟⢠⣿⢈⣧⠘⢠⣿
⣿⣿⣿⣄⠻⣿⡄⢳⡄⢆⡙⠾⣽⣿⣿⣆⡀⢹⡷⣄⠙⢿⣿⡾⣿⣆⢀⡀⢀⢀⢀⢀⢀⢀⢀⢀⢀⢀⢀⢀⣀⣠⣴⡿⣯⠏⣠⣿⣿⡏⢸⣿⡿⢁⣿⣿⢀⣿⠆⢸⣿
⣿⣿⣿⣿⣦⡙⣿⣆⢻⡌⢿⣶⢤⣉⣙⣿⣷⡀⠙⠽⠷⠄⠹⣿⣟⣿⣆⢙⣋⣤⣤⣤⣄⣀⢀⢀⢀⢀⣾⣿⣟⡷⣯⡿⢃⣼⣿⣿⣿⠇⣼⡟⣡⣿⣿⣿⢀⡿⢠⠈⣿
⣿⣿⣿⣿⣿⣷⣮⣿⣿⣿⡌⠁⢤⣤⣤⣤⣬⣭⣴⣶⣶⣶⣆⠈⢻⣿⣿⣆⢻⣿⣿⣿⣿⣿⣿⣷⣶⣤⣌⣉⡘⠛⠻⠶⣿⣿⣿⣿⡟⣰⣫⣴⣿⣿⣿⣿⠄⣷⣿⣿⣿
*/
```

其中，`F[x][y][v]`表示从点`[x][y]`以方向`v`射入的答案，由于环不同于链，环上的点答案都是相同的，所以需要在搜索完后单独处理，为了判定是出现了环还是走到了之前走过的点，`dfs`的返回值需要有一个布尔值。`id`是这个点所在的路径的编号，`used`是反射镜所在路径的编号，由于之前说过的同时以反射镜和非反射镜出现的情况，对于反射镜必须单独开一个数组。
总结来说，递归的主要目的其实就两个：

1. 确定是否是环
2. 确定当前的链是一条之前没走过的链还是后半部分是之前走过的链

为了确定这一点，需要在递归完成后再进行编号，如果是之前走过的链，就让`tag`等于那条链对应的编号，否则就分配一个新的编号，然后在回溯时将编号赋值给访问到的所有点，同时根据反射前后的方向判断当前镜子是否是反射镜，是就标记上当前的路径编号，由于一面镜子也可能同时作为两条路径的反射镜，所以所有标记数组都需要记录方向。
除此之外，虽然编号要在回溯时才能确定，但标记要在递归前标记，否则就无法判断是否是环，为此，可以在单独开一个数组用于标记，或者像我一样分配一个临时编号，实测可以少50ms（其实也没什么用(～￣▽￣)～）具体的步骤都在代码里有注释。

---

其实最后总结一下也没有那么多东西要考虑，单纯之前考虑不够全面导致一直wa还把接近答案的代码抛弃换了一个全新的写法，考虑清楚之后做法还是很清晰的。
