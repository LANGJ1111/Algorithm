#### [每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

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
int* dailyTemperatures(int* temperatures, int temperaturesSize, int* returnSize){
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
int* dailyTemperatures(int* temperatures, int temperaturesSize, int* returnSize){
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

