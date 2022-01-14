# [每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

### 题目

请根据每日 气温 列表 temperatures ，请计算在每一天需要等几天才会有更高的温度。如果气温在这之后都不会升高，请在该位置用 0 来代替。

示例 1:

```
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```

示例 2:

```
输入: temperatures = [30,40,50,60]
输出: [1,1,1,0]
```

示例 3:

```
输入: temperatures = [30,60,90]
输出: [1,1,0]
```

**提示：**

- `1 <= temperatures.length <= 105`
- `30 <= temperatures[i] <= 100`

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

```
输入：s = "abc"
输出：3
解释：三个回文子串: "a", "b", "c"
```

示例 2：

```
输入：s = "aaa"
输出：6
解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
```

**提示：**

- `1 <= s.length <= 1000`
- `s` 由小写英文字母组成

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

#### Manacher算法

核心思想：利用回文串的对称性，记录到达最右端回文串的右端点R和中心点C，右中心点 f ( i' ) 利用左中心点 f ( i ) 赋初值，并不断更新R和C。 

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

***

# [任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

### 题目

给你一个用字符数组 tasks 表示的 CPU 需要执行的任务列表。其中每个字母表示一种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。在任何一个单位时间，CPU 可以完成一个任务，或者处于待命状态。

然而，两个 相同种类 的任务之间必须有长度为整数 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的 最短时间 。

示例 1：

```
输入：tasks = ["A","A","A","B","B","B"], n = 2
输出：8
解释：A -> B -> (待命) -> A -> B -> (待命) -> A -> B
     在本示例中，两个相同类型任务之间必须间隔长度为 n = 2 的冷却时间，而执行一个任务只需要一个单位时间，所以中间出现了（待命）状态。 
```

示例 2：

```
输入：tasks = ["A","A","A","B","B","B"], n = 0
输出：6
解释：在这种情况下，任何大小为 6 的排列都可以满足要求，因为 n = 0
["A","A","A","B","B","B"]
["A","B","A","B","A","B"]
["B","B","B","A","A","A"]
...
诸如此类
```

示例 3：

```
输入：tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2
输出：16
解释：一种可能的解决方案是：
     A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> (待命) -> (待命) -> A -> (待命) -> (待命) -> A
```

**提示：**

- `1 <= task.length <= 104`
- `tasks[i]` 是大写英文字母
- `n` 的取值范围为 `[0, 100]`

### 题解

#### 模拟 + 贪心

核心思路：先用桶模型统计字母，然后模拟时间流逝，随着time++，每次选择不在冷却期、且剩余数量最多的任务。

```c
int leastInterval(char* tasks, int tasksSize, int n) {
     int freq[26];
     memset(freq, 0, sizeof(freq));
     for (int i = 0; i < tasksSize; i++) {
         freq[tasks[i] - 'A']++;
     }

     int nextTime[26], num[26], m = 0;
     for(int i = 0; i < 26; i++) {
         if (freq[i] > 0) {
             nextTime[m] = 1;
             num[m] = freq[i];
             m++;
         }
     }

     int time = 0;
     for (int i = 0; i < tasksSize; i++) {
         time++;
         int minNext = INT_MAX;
         for (int j = 0; j < m; j++) {
             if (num[j]) {
                 minNext = fmin(minNext, nextTime[j]);
             }
         }
         time = fmax(time, minNext);

         int best = -1;
         for (int j = 0; j < m; j++) {
             if (nextTime[j] <= time && num[j] > 0) {
                 if (best == -1 || num[j] > num[best]) {
                     best = j;
                 }
             }
         }
         nextTime[best] = nextTime[best] + n + 1;
         num[best]--;
     }
     return time;
}
```

#### 构造

[题解中方法二](https://leetcode-cn.com/problems/task-scheduler/solution/ren-wu-diao-du-qi-by-leetcode-solution-ur9w/)

统计每种任务出现数量，记最多数量为max，以max为行，n + 1为列构造矩阵。其余任务有两种情况：

1. 其余任务不能满铺该矩阵。记出现max次数任务有num个，则结果 `result1 = (max - 1) * (n + 1) + num`
2. 其余任务可以满铺该矩阵。则CPU不存在休息，结果等于所有任务数量，`result2 = tasksSize`

最终结果：result = max ( result1, result2 )

```c
int leastInterval(char* tasks, int tasksSize, int n) {
    int freq[26];
    memset(freq, 0, sizeof(freq));
    for (int i = 0; i < tasksSize; i++) {
        freq[tasks[i] - 'A']++;
    }

    int max = 0; 
    for (int i = 0; i < 26; i++) {
        if (freq[i] > max) {
            max = freq[i];
        }
    }

    int num = 0;
    for (int i = 0; i < 26; i++) {
        if(freq[i] == max) {
            num++;
        }
    }

    return fmax((max - 1) * (n + 1) + num, tasksSize);
}
```

***

# [合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

### 题目

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

示例 1:

```
输入: 

​	Tree 1                     Tree 2                  
​          1                         2                             
​         / \                       / \                            
​        3   2                     1   3                        
​       /                           \   \                      
​      5                             4   7                  

输出: 

合并后的树:

​	     3  
​	    / \  
​	   4   5  
​	  / \   \  
​	 5   4   7
```

注意: 合并必须从两个树的根节点开始。

### 题解

#### DFS

```c
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */


struct TreeNode* mergeTrees(struct TreeNode* root1, struct TreeNode* root2){
    if (root1 == NULL) {
        return root2;
    }
    if (root2 == NULL) {
        return root1;
    }
    
    struct TreeNode *res = (struct TreeNode *)malloc(sizeof(struct TreeNode));
    res->val = root1->val + root2->val;
    res->left = mergeTrees(root1->left, root2->left);
    res->right = mergeTrees(root1->right, root2->right);

    return res;
}
```

***

# [最短无序连续子数组](https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray/)

### 题目

给你一个整数数组 nums ，你需要找出一个 连续子数组 ，如果对这个子数组进行升序排序，那么整个数组都会变为升序排序。

请你找出符合题意的 最短 子数组，并输出它的长度。

 

示例 1：

```
输入：nums = [2,6,4,8,10,9,15]
输出：5
解释：你只需要对 [6, 4, 8, 10, 9] 进行升序排序，那么整个表都会变为升序排序。
```

示例 2：

```
输入：nums = [1,2,3,4]
输出：0
```

示例 3：

```
输入：nums = [1]
输出：0
```

**提示：**

- `1 <= nums.length <= 104`
- `-105 <= nums[i] <= 105`

### 题解

#### 双指针

将原数组排序，找到排序数组和原数组第一个不一样的左端和右端，这就是我们要找的子数组。

```c
int cmpfunc (const void * a, const void * b)
{
   return ( *(int*)a - *(int*)b );
}

int findUnsortedSubarray(int* nums, int numsSize) {
    int tmp[numsSize];
    for (int i = 0; i < numsSize; i++) {
        tmp[i] = nums[i];
    }
    qsort(tmp, numsSize, sizeof(int), cmpfunc);

    int l = 0, r = numsSize - 1;
    while (tmp[l] == nums[l] && l < r) {
        l++;
    }
    while (tmp[r] == nums[r] && l < r) {
        r--;
    }

    if (l == r) {
        return 0;
    }

    return r - l + 1;
}
```

#### 一次遍历

一次遍历从两头进行。设max为一次遍历左端已遍历部分最大值，min为右端最小值。通过一次遍历，维护max和min：当左端当前元素小于max，则更新right；右端当前元素大于min，则更新left。最后`r - l + 1`就是答案。当r == -1（l == -1）时表示数组已有序，返回0。

```c
int findUnsortedSubarray(int* nums, int numsSize) {
    int max = INT_MIN, r = -1;
    int min = INT_MAX, l = -1;
    for (int i = 0; i < numsSize; i++) {
        if (nums[i] < max) {
            r = i;
        } else {
            max = nums[i];
        }
        if (nums[numsSize - i - 1] > min) {
            l = numsSize - i - 1;
        } else {
            min = nums[numsSize - i - 1];
        }
    }
    return r == -1 ? 0 : r - l + 1;
}
```

***

# [和为 K 的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

### 题目

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回该数组中和为 `k` 的连续子数组的个数。

**示例 1：**

```c
输入：nums = [1,1,1], k = 2
输出：2
```

**示例 2：**

```c
输入：nums = [1,2,3], k = 3
输出：2
```

**提示：**

- `1 <= nums.length <= 2 * 104`
- `-1000 <= nums[i] <= 1000`
- `-107 <= k <= 107`

### 题解

#### 暴力（超时）

```c
int subarraySum(int* nums, int numsSize, int k) {
    int sum = 0, cnt = 0;
    for (int i = 0; i < numsSize; i++) {
        for (int j = i; j < numsSize; j++) {
            sum += nums[j];
            if (sum == k) {
                cnt++;
            }
        }
        sum = 0;
    }
    return cnt;
}
```

#### 前缀和+哈希表

建立哈希表map，map [ i ] 记录前缀和 sum 为 i 的个数。一次遍历 nums ，更新当前前缀和 sum ，如果 map [sum - k] 不为0，则说明有 map [sum - k] 个前缀和和当前前缀和之差为 k ，在 cnt 中更新结果，然后更新哈希表 map 。最终结果就是一次遍历结束后的 cnt 。

```c
int subarraySum(int* nums, int numsSize, int k) {
    int *maps = (int *)calloc(1001 * 20001 * 2, sizeof(int));
    int *map = maps + 1001 * 20001;
    int sum = 0, cnt = 0;
    map[sum]++;
    for(int i = 1; i <= numsSize; i++) {
        sum += nums[i - 1];
        if (map[sum - k] > 0) {
            cnt += map[sum - k];
        }
        map[sum]++;
    }
    free(maps);
    return cnt;
}
```

