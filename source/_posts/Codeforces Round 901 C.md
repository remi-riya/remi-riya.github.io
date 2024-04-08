---
title: Codeforces Round 901 C
katex: true
categories:
  - 题解
tags:
  - 算法
abbrlink: 13293
date: 2023-10-01 08:28:01
---

[原题链接](https://codeforces.com/contest/1875/problem/C)

---

这题的思路其实很简单，就是直接贪心加暴力即可。有$n$个苹果和$m$个人，要想平均分配同时使切苹果的次数最少，那么就只需要切多出来的那部分就行了，也就是$n \bmod m$，其他的既然已经能够平均分配，切了之后也依然能平均分配，没必要画蛇添足地去切。所以只需要一个while循环就能解决：

```c++
int a = n % m;
while (a) {
    ans += a;
    a = a * 2 % m;
}
```

当然，如果直接这样写的话，就会发现样例都过不了。因为有些情况是无论怎么切都无法平均分配的，上述代码遇到这种情况显然会直接死循环，所以这题的关键就在于如何判断能否平均分配。

首先，假如某种情况下可以均分，比如每个人分到了1.5个苹果，那么我们就可以把那些重量为1的苹果都切成0.5，也就是切成最小的，同时苹果依然可以均分。这么做的意义在于，当我们把所有苹果都切成最小的那种时，其实就相当于在每一次切苹果时都是直接切所有苹果，也就是直接让苹果的数量翻倍。也就是说，可以均分和这个式子等价（具体的充分必要性我就不证了，~~绝不是不会~~）：
\\[n \times 2^k \bmod m = 0\\]（或者反过来考虑，即全部切成碎块之后再把每个人分到的碎块合起来）在数据范围内，即使考虑最差情况，$m$也不会超过$2^{31}$，如果能均分的话，最多乘上31次2，也就是循环31次，时间肯定够。所以我们只需要先判断一下能否均分，如果可以就直接暴力模拟，就能过这题。

至于如何判断能否整除，可以从质因数分解的角度考虑。每次乘2就是往$n$上加一个质因子2，也就是说如果$m / \gcd(n,m)$不是2的次幂，也就是含有2以外的质因子，那么我们无论如何往$n$上加2都无法1整除$m$。完整代码如下：

```c++
#include <bits/stdc++.h>
#define lowbit(x) ((x) & -x)
using namespace std;
using ll = long long;
using pii = pair<int, int>;
int a[100], b[100];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    cin >> t;
    while (t--) {
        int n, m;
        cin >> n >> m;
        ll ans = 0;
        int a = n % m;
        int gcd = __gcd(a, m);
        if (m / gcd - lowbit(m / gcd) && a != 0)
            ans = -1;
        else
            while (a) {
                ans += a;
                a = a * 2 % m;
            }
        cout << ans << '\n';
    }
}
```

---

![fufu](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/112113626_p1.png)
