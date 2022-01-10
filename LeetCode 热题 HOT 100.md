# [每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

### 题目

请根据每日 气温 列表 temperatures ，请计算在每一天需要等几天才会有更高的温度。如果气温在这之后都不会升高，请在该位置用 0 来代替。

示例 1:

输入: temperatures = [73,74,75,71,69,72,76,73]

输出: [1,1,4,2,1,1,0,0]

示例 2:

输入: temperatures = [30,40,50,60]

输出: [1,1,1,0]

示例 3:

输入: temperatures = [30,60,90]

输出: [1,1,0]

### 题解

#### 暴力（超时）

```c
int* dailyTemperatures(int* temperatures, int temperaturesSize, int* returnSize) {
    int cnt;
    int *res = (int *)malloc(sizeof(int) * temperaturesSize);
    for(int i = 0; i < temperaturesSize; i++) {
        cnt = 0;
        for(int j = i + 1; j < temperaturesSize; j++) {
            if(temperatures[j] > temperatures[i]) {
                cnt = j - i;
                break;
            }
        }
        res[i] = cnt;
    }
    *returnSize = temperaturesSize;
    return res;
}
```

#### 单调栈

用一个栈存储数组每个元素下标，遍历数组并进行操作：比较当前元素和栈顶元素大小，比栈顶元素大就将栈顶元素出栈并计算出结果，然后继续比较；比栈顶元素小就将该元素入栈。最后将所有栈中剩余元素标0。

```c
int* dailyTemperatures(int* temperatures, int temperaturesSize, int* returnSize) {
    *returnSize = temperaturesSize;
    int top = -1, cnt;
    int *res = (int *)malloc(sizeof(int) * temperaturesSize); 
    int stack[temperaturesSize + 1];
    for (int i = 0; i < temperaturesSize; i++) {
        while(top != -1 && temperatures[i] > temperatures[stack[top]]) {
            res[stack[top]] = i - stack[top];
            top--;
        }
        stack[++top] = i;
    }
    while(top != -1) {
        res[stack[top--]] = 0;
    }
    return res;
}
```

***

# [回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

### 题目

给你一个字符串 s ，请你统计并返回这个字符串中 回文子串 的数目。

回文字符串 是正着读和倒过来读一样的字符串。

子字符串 是字符串中的由连续字符组成的一个序列。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

示例 1：

输入：s = "abc"

输出：3

解释：三个回文子串: "a", "b", "c"

示例 2：

输入：s = "aaa"

输出：6

解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"

### 题解

#### 中心拓展

核心思想：找到每个回文串的中心向左右进行拓展。回文串有奇数串和偶数串，奇数串有1个中心，偶数串有2个中心，因此令 l = i / 2 ， r = i / 2 + i % 2 ，既能保证遍历奇数串和偶数串中心，也能保证偶数串中心是向右拓展的，不会有算多的情况。设总长为n，则 i 遍历范围 0 ~ 2n-1。

```c
int countSubstrings(char * s) {
    int len = strlen(s);
    int res = 0;
    for (int i = 0; i < 2 * len - 1; i++) {
        int l = i / 2, r = i / 2 + i % 2;
        while (l >= 0 && r < len && s[l] == s[r]) {
            l--;
            r++;
            res++;
        }
    }
    return res;
}
```

### Manacher算法

[一文让你彻底明白马拉车算法](https://zhuanlan.zhihu.com/p/70532099)

```c
int countSubstrings(char * s) {
    int n = strlen(s);
    char p[2 * n + 4];
    p[0] = '!';
    p[1] = '#';
    for (int i = 0; i < n; i++) {
        p[2 * i + 2] = s[i];
        p[2 * i + 3] = '#';
    }
    p[2 * n + 2] = '@';
    p[2 * n + 3] = '\0';
    n = 2 * n + 3;

    int f[n];
    memset(f, 0, sizeof(f));
    int c = 0, r = 0, res = 0;

    for(int i = 1; i < n; i++) {
        if (i > r) {
            f[i] = 1;
        } else {
            f[i] = fmin(r - i + 1, f[2 * c - i]);
        }
        while (p[i + f[i]] == p[i - f[i]]) {
            f[i]++;
        }
        if (i + f[i] - 1 > r) {
            c = i;
            r = i + f[i] - 1;
        }
        res += f[i] / 2;
    }
    return res;
}
```
