---
title: BSGS算法
tags:
  - 算法
  - BSGS
categories:
  - ACM
abbrlink: 60907
katex: true
date: 2024-03-29 13:46:49
---

BSGS算法（baby step giant step，大步小步算法），又叫北上广深算法（不是），是一种基于中间相遇思想求解离散对数的算法，即求解类似于$a^x\equiv b\pmod m$的方程。

## BSGS算法

首先介绍最普通的BSGS算法。普通的BSGS算法要求$m$和$a$要互质，既然互质，那么根据欧拉定理
$$
a^{\varphi(m)} \equiv 1\pmod m
$$
也就是说，$a^x \pmod m$以$\varphi(m)$为周期循环，那么如果方程有解，解一定在$0$到$\varphi(m)$之间，如果暴力枚举，当$m$为质数时$\varphi(m)$最大为$m-1$，最坏复杂度为$O(m)$。

普通的BSGS算法就是对上述暴力算法的一个优化，我们可以把$x$改成$At - B$，那么原式变成$a^{At - B} \equiv b \pmod m$，由于$a$和$m$互质，$a^{B}$在模$m$意义下有逆元，所以可以把它移到右边，变成$a^{At} \equiv ba^{B}$。然后我们可以预处理出右边的所有可能的$B$对应的值，然后枚举左边的$t$，如果某个$t$的值出现过，那么就可以得出解。

不难看出，当$A$取$\lceil\sqrt{\varphi(m)}\rceil$时，需要枚举的次数是最少的，但由于算欧拉函数本身就比较麻烦，一般直接取$\lceil\sqrt{m}\rceil$，代码如下：

```c++
ll BSGS(ll a, ll b, const ll p) // 离散对数
{
    if (1LL % p == b % p)
        return 0; // 特判0
    ll t = std::sqrt(p) + 1;
    std::map<ll, ll> mp;
    ll d = 1; // a^t
    for (int i = 0; i < t; ++i) {
        mp[b * d % p] = i; // 可以直接覆盖，如果有重复的会取更大的，这样At - B就是最小的
        d = d * a % p;
    }
    a = 1;
    for (int i = 1; i <= t; ++i) {
        a = a * d % p;
        if (mp.count(a)) // 存在
            return i * t - mp[a];
    }
    return -INF; // 无解
}
```

有时候$a$和$b$可能会比较大，在传参之前记得取模（血的教训＞﹏＜）。

## 扩展BSGS算法

普通的BSGS算法要求$a$和$m$一定要互质，那如果不互质呢，一种很简单的想法是转换成互质的情况再用普通的BSGS算法求解，具体来说，我们可以从原式$a^x \equiv b \pmod m$的左边提出一个$a$，变成$a \cdot a^{x-1} \equiv b \pmod m$，然后算出$a$和$m$的$\gcd$，然后等式两边和模数$m$同时除以$\gcd$，等式仍然成立，而$a$和$m$的公共质因子却变少了，一直重复这个过程，直到$a$与$m$互质，然后就可以用普通的BSGS算法解决了，假如这个过程进行了$k$次，第$i$次的最大公因数为$g_i$，等式就变成了
$$
\prod\limits_i^k \frac{a}{g_i} \cdot a^{x-k} \equiv \frac{b}{\prod\limits_i^k g_i} \pmod {\frac{m}{\prod\limits_i^k g_i}}
$$
这个时候因为$a$与$m$互质，我们可以用普通的BSGS算法解决，虽然相比与普通的BSGS左边多了一个系数$\prod\limits_i^k \frac{a}{g_i}$，但对算法正确性没有影响，只要把原来的代码稍微改一下，最后的结果就是BSGS的返回值加上$k$。
如果在迭代的过程中出现了$b$不能被$gcd$整除的情况，根据斐蜀定理，把原式转换成$a^{x-1}a - ym = b$，可得$b$一定是$\gcd(a,m)$的倍数，所以如果不是就说明原方程无解。

```c++
ll BSGS(ll a, ll b, const ll p, ll k = 1LL) // k是多加的系数
{
    if (k % p == b % p) // 特判0
        return 0;
    ll t = std::sqrt(p) + 1;
    std::map<ll, ll> mp;
    ll d = 1;
    for (int i = 0; i < t; ++i) {
        mp[b * d % p] = i;
        d = d * a % p;
    }
    a = k; // 正常这里是1，加了系数所以是k
    for (int i = 1; i <= t; ++i) {
        a = a * d % p;
        if (mp.count(a))
            return i * t - mp[a];
    }
    return -INF; // 无解
}
ll exBSGS(ll a, ll b, ll p)
{
    ll k = 1; // k是系数
    for (int i = 0;; ++i) {
        if (k % p == b % p) // 迭代中特判0
            return i;
        ll gcd = std::gcd(a, p);
        if (b % gcd)
            return -INF; // 无解
        if (gcd == 1)
            return BSGS(a, b, p, k) + i;
        k = k * a / gcd % p, b /= gcd, p /= gcd;
    }
}
```
