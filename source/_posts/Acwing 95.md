---
title: Acwing 95
tags:
  - 算法
categories:
  - 题解
abbrlink: 24657
---

原题链接——[Acwing95](https://www.acwing.com/problem/content/description/97/)  
解法来源《算法竞赛进阶指南》

---

想清楚之后并不是很难的题，但个人认为这是对思维提升非常大的一道题，刷新了我对递推和枚举的认识，故记录一下。

首先注意这一题的数据量，$n \leq 500$，数据量较小，同时矩阵也只有$5 \times 5$，那么我们可以考虑遍历状态空间，然后取最小值。
问题在于如何遍历，如果直接枚举所有选择，那么就有$\sum_{i=1}^{6}{C_{25}^{i}} \times n$种情况，大约是1e8种，大概率会T~~也许放牛客上能过~~。

那么接下来让我们回到题目本身，题目的目的是把所有灯都打开。假如我们已经枚举了第一排的可能的操作，但第一排仍有未打开的灯，那么显然只能通过开关相应的第二排的开关来把第一排还未打开的灯打开，同时第二排其他的开关又不能动，因为会破坏第一排本来已经打开的灯的状态，也就是说，第二排的开关操作是固定的，同理，如果第二排操作完后仍有灯关着，那么只能通过操作第三排的相应开关来打开……以此类推，如果在操作完最后一排后最后一排仍有灯关着或者操作步数大于6，那么就说明无法在6步以内将灯全部打开。所以，我们其实只需要枚举第一排的操作，就可以遍历所有可能的操作（这里的可能指的是可能将灯全部打开的操作，而非所有可以选择的操作）。

```c++
// Acwing 95
#include <bits/stdc++.h>
using namespace std;
string a[5];
int dx[4] = {1, 0, -1, 0}, dy[4] = {0, 1, 0, 0}, ans;
void rev(int x, int y, string a[])  // 反转函数（上面的灯不反转不影响结果，可以偷下懒）
{
    for (int i = 0; i < 4; ++i) {
        int xx = x + dx[i], yy = y + dy[i];
        if (xx < 0 || xx > 4 || yy < 0 || yy > 4)
            continue;
        a[yy][xx] ^= 1; //'0' = 48
    }
}
void check(int cnt) // 判断函数
{
    string tmp[5] = {a[0], a[1], a[2], a[3], a[4]}; // 备份
    for (int i = 0; i < 4; i++) {
        for (int j = 0; j < 5; j++) {
            if (tmp[i][j] == '0') {
                if (++cnt > 6)
                    return;
                rev(j, i + 1, tmp);
            }
        }
    }
    for (int i = 0; i < 5; i++)
        if (tmp[4][i] == '0')
            return;
    if (cnt <= 6) {
        ans = min(ans, cnt);
    }
}
void dfs(int now, int cnt)  // 枚举第一行的操作
{
    if (now < 4) {
        dfs(now + 1, cnt);  // now不反转
        rev(now, 0, a);
        dfs(now + 1, cnt + 1);  // now反转
        rev(now, 0, a);
    } else if (now == 4) {
        check(cnt);
        rev(now, 0, a);
        check(cnt + 1);
        rev(now, 0, a);
    }
}
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    while (n--) {
        ans = 7;
        for (int i = 0; i < 5; ++i) {
            cin >> a[i];
        }
        dfs(0, 0);
        if (ans > 6)
            cout << -1 << '\n';
        else
            cout << ans << '\n';
    }
}

```

这种解法最让人迷惑的点大概就是枚举第一排操作为什么有时候要把本来已经打开的灯关掉（我想了好久才懂），要注意的是，我们是在遍历可能的状态空间，而非直接去找最小的操作步数，相比与去分析输入的矩阵寻找答案，这种枚举的思想也许更适合计算机。

---

![img](https://remi-riya-img.oss-cn-beijing.aliyuncs.com/1694003151402.jpg)
