## 1929 Concatenation of Array
## 14 Longest Common Prefix
## 27 Remove Element
## 169 Majority Element
## 705 Design HashSet
## 706 Design HashMap
## 912 Sort an Array
## 75 Sort Colors
## 304 Range Sum Query 2D Immutable
## 122 Best Time to Buy And Sell Stock II
## 229 Majority Element II
## 560 Subarray Sum Equals K
## 41 First Missing Positive
## 344 Reverse String
## 680 Valid Palindrome II
## 1768 Merge Strings Alternately
## 88 Merge Sorted Array
## 26 Remove Duplicates From Sorted Array
## 18 4Sum
## 189 Rotate Array
## 881 Boats to Save People
## 219 Contains Duplicate II
## 209 Minimum Size Subarray Sum
## 658 Find K Closest Elements
## 682 Baseball Game
## 225 Implement Stack Using Queues
## 232 Implement Queue using Stacks
## 232 Implement Queue using Stacks

两个栈：`sin` 负责入，`sout` 负责出。`sout` 空了才把 `sin` 全部倒过去，均摊 O(1)。

```python
class MyQueue:
    def __init__(self):
        self.sin = []
        self.sout = []

    def push(self, x):
        self.sin.append(x)

    def pop(self):
        self.peek()
        return self.sout.pop()

    def peek(self):
        if not self.sout:
            while self.sin:
                self.sout.append(self.sin.pop())
        return self.sout[-1]

    def empty(self):
        return not self.sin and not self.sout
```
## 735 Asteroid Collision
## 901 Online Stock Span
## 71 Simplify Path

按 `/` 分割路径，用栈逐段处理：

- `..` → 弹栈（回上一级）
- `.` 或空串 → 跳过
- 其余 → 压栈

最后用 `/` 拼接栈内容，前面加 `/`。

```python
class Solution:
    def simplifyPath(self, path: str) -> str:
        stack = []
        for part in path.split('/'):
            if part == '..':
                if stack:
                    stack.pop()
            elif part and part != '.':
                stack.append(part)
        return '/' + '/'.join(stack)
```
## 394 Decode String
## 895 Maximum Frequency Stack
## 35 Search Insert Position
## 374 Guess Number Higher Or Lower
## 69 Sqrt(x)
## 1011 Capacity to Ship Packages Within D Days
## 81 Search In Rotated Sorted Array II
## 410 Split Array Largest Sum
## 1095 Find in Mountain Array
## 92 Reverse Linked List II
## 622 Design Circular Queue
## 460 LFU Cache
## 94 Binary Tree Inorder Traversal
## 144 Binary Tree Preorder Traversal
## 145 Binary Tree Postorder Traversal
## 701 Insert into a Binary Search Tree
## 450 Delete Node in a BST
## 427 Construct Quad Tree
## 337 House Robber III
## 1325 Delete Leaves With a Given Value
## 1834 Single Threaded CPU
## 767 Reorganize String
## 1405 Longest Happy String
## 1094 Car Pooling
## 502 IPO
## 1863 Sum of All Subsets XOR Total
## 77 Combinations
## 47 Permutations II
## 473 Matchsticks to Square
## 698 Partition to K Equal Sum Subsets
## 52 N Queens II
Almost the same as N Queens

Diag1-> r-c+n or r-c
Diag2-> r+c
## 140 Word Break II
## 2707 Extra Characters in a String
## 463 Island Perimeter
## 953 Verifying An Alien Dictionary
## 997 Find the Town Judge
## 752 Open The Lock
## 1462 Course Schedule IV
## 721 Accounts Merge
## 399 Evaluate Division
## 310 Minimum Height Trees
## 1631 Path with Minimum Effort
## 1489 Find Critical and Pseudo Critical Edges in Minimum Spanning Tree
## 2392 Build a Matrix With Conditions
## 2709 Greatest Common Divisor Traversal
## 1137 N-th Tribonacci Number
## 377 Combination Sum IV
## 279 Perfect Squares
## 343 Integer Break
## 1406 Stone Game III

## 题意

两个玩家轮流从数组前端取 1、2 或 3 个石子，都以最优策略最大化自己的得分。判断 Alice（先手）赢、Bob 赢还是平局。

---

## 核心洞察

### 博弈 dp 为什么从后往前？

"取石子"是从前往后取的，取完之后剩下的是**后半段**，后半段是一个独立的子问题。所以后面的局面先确定，从后往前推才自然。

规律：遇到"从前端取东西"的博弈题，基本都是从后往前 dp。

### dp 定义

`dp[i]` = 从第 i 个石子开始，当前玩家比对手多拿的最大分数差

注意：不区分 Alice 和 Bob，因为两人策略完全一样——轮到谁，谁都想最大化自己的优势。

---

## 转移方程

当前玩家从位置 i 取 k 个石子（k = 1, 2, 3）：

```
dp[i] = max(total_k - dp[i + k])    其中 k = 1, 2, 3
```

- `total_k`：取走的 k 个石子的分数和
- `dp[i + k]`：对手从 i+k 开始比我多拿的最优差值
- `total_k - dp[i + k]`：我拿了 total，对手之后比我多 dp[i+k]，所以我比对手多的就是这个差

### 举例：stones = [1, 2, 3]

```
dp[3] = 0                 （没石子了）
dp[2] = 3 - dp[3] = 3     （取1个，多3分）
dp[1] = max(2 - dp[2], 5 - dp[3]) = max(-1, 5) = 5
dp[0] = max(1 - dp[1], 3 - dp[2], 6 - dp[3]) = max(-4, 0, 6) = 6
```

dp[0] = 6 > 0，Alice 赢。

---

## 初始化

`dp[n] = dp[n+1] = dp[n+2] = 0`（没有石子可拿，差值为 0）

实际代码中只需要 `dp[n] = 0`，因为内层循环的 `min(i+3, n)` 保证不会越界。

---

## total 的含义

`total` 不是全局累加，而是每轮重置：

```
i = 5 时：
    j = 5: total = stones[5]              → 取1个
    j = 6: total = stones[5]+stones[6]    → 取2个
    j = 7: total = stones[5]+..+stones[7] → 取3个

i = 4 时：total 重置为 0，重新累加
```

始终表示"当前玩家这一轮取走的石子和"。

---

## 代码

```python
class Solution:
    def stoneGameIII(self, stoneValue: List[int]) -> str:
        n = len(stoneValue)
        dp = [float('-inf')] * (n + 1)
        dp[n] = 0

        for i in range(n - 1, -1, -1):
            total = 0
            for j in range(i, min(i + 3, n)):
                total += stoneValue[j]
                dp[i] = max(dp[i], total - dp[j + 1])

        if dp[0] == 0:
            return "Tie"
        return "Alice" if dp[0] > 0 else "Bob"
```

---

## 和普通 dp 的对比

| | 普通 dp | 本题 |
|---|---------|------|
| 方向 | 从前往后 | 从后往前 |
| 已知起点 | `dp[0]` | `dp[n] = 0` |
| 答案位置 | `dp[n]` | `dp[0]` |
| 原因 | 前面的状态先确定 | 后面的局面先确定 |

---

## 易错点

1. dp 初始化是 `[float('-inf')] * (n + 1)`，注意括号位置
2. dp 存的是分数差，不是绝对分数，最后看正负判断胜负
3. `total` 每次外层循环重置，不是全局累加
## 63 Unique Paths II
## 64 Minimum Path Sum
## 1049 Last Stone Weight II
## 877 Stone Game
## 1140 Stone Game II
## 860 Lemonade Change
## 918 Maximum Sum Circular Subarray
## 978 Longest Turbulent Subarray
# 最长湍流子数组 — 动规五部曲详解

> LeetCode 978. Longest Turbulent Subarray

## 题目理解

**湍流子数组**：相邻元素的大小关系必须交替变化（一升一降，一降一升），像波浪一样。

```
示例：[9, 4, 2, 10, 7, 8, 8, 1, 9]
比较：  >    >    <     >    <    =    >    <

其中 [4, 2, 10, 7, 8] 对应 > < > <，符号交替，长度为 5
```

---

## 动规五部曲

### 一、确定 dp 数组及其含义

```python
dp[i][0]  # 以第 i 个元素结尾，最后一步是【下降】的最长湍流子数组长度
dp[i][1]  # 以第 i 个元素结尾，最后一步是【上升】的最长湍流子数组长度
```

用两个维度分别追踪两种结尾状态，体现"交替"的核心思想。

### 二、确定递推公式

| 当前关系 | 含义 | 转移 |
|---------|------|------|
| `arr[i] > arr[i-1]` | 当前上升 | 上升之前必须是下降：`dp[i][1] = dp[i-1][0] + 1` |
| `arr[i] < arr[i-1]` | 当前下降 | 下降之前必须是上升：`dp[i][0] = dp[i-1][1] + 1` |
| `arr[i] == arr[i-1]` | 相等 | 湍流中断，保持初始值 1 |

核心：**上升接下降，下降接上升**，交替接力。

### 三、dp 数组初始化

```python
dp = [[1] * 2 for _ in range(n)]
```

每个位置初始化为 **1**，因为单独一个元素本身就是长度为 1 的子数组。

### 四、确定遍历顺序

从左到右，`i` 从 1 到 n-1。

因为 `dp[i]` 依赖 `dp[i-1]`，所以必须**从前往后**遍历。

### 五、举例推导验证

以 `arr = [9, 4, 2, 10, 7]` 为例：

```
i=0: dp[0] = [1, 1]                                  初始
i=1: 4 < 9  下降 → dp[1][0] = dp[0][1] + 1 = 2     → [2, 1]
i=2: 2 < 4  下降 → dp[2][0] = dp[1][1] + 1 = 2     → [2, 1]  (连续下降，无法接力)
i=3: 10 > 2 上升 → dp[3][1] = dp[2][0] + 1 = 3     → [1, 3]
i=4: 7 < 10 下降 → dp[4][0] = dp[3][1] + 1 = 4     → [4, 1]
```

最终 `max_len = 4`，对应子数组 `[4, 2, 10, 7]`，符号为 `> < >`，确实交替。

---

## 完整代码

```python
class Solution:
    def maxTurbulenceSize(self, arr: List[int]) -> int:
        n = len(arr)
        if n == 1:
            return 1

        dp = [[1] * 2 for _ in range(n)]
        max_len = 1

        for i in range(1, n):
            if arr[i] > arr[i - 1]:
                dp[i][1] = dp[i - 1][0] + 1
            elif arr[i] < arr[i - 1]:
                dp[i][0] = dp[i - 1][1] + 1
            max_len = max(max_len, dp[i][0], dp[i][1])

        return max_len
```

---

## 空间优化版本

由于 `dp[i]` 只依赖 `dp[i-1]`，可以用两个变量代替整个数组：

```python
class Solution:
    def maxTurbulenceSize(self, arr: List[int]) -> int:
        inc = dec = 1  # inc=上升结尾, dec=下降结尾
        res = 1

        for i in range(1, len(arr)):
            if arr[i] > arr[i - 1]:
                inc = dec + 1
                dec = 1
            elif arr[i] < arr[i - 1]:
                dec = inc + 1
                inc = 1
            else:
                inc = dec = 1

            res = max(res, inc, dec)

        return res
```

---

## 复杂度分析

| | 二维 DP | 空间优化 |
|--|--------|---------|
| 时间复杂度 | O(n) | O(n) |
| 空间复杂度 | O(n) | O(1) |
## 1871 Jump Game VII
## 649 Dota2 Senate
## 135 Candy
## 2402 Meeting Rooms III
## 168 Excel Sheet Column Title
## 1071 Greatest Common Divisor of Strings
## 2807 Insert Greatest Common Divisors in Linked List
## 867 Transpose Matrix
## 13 Roman to Integer
## 67 Add Binary
## 201 Bitwise AND of Numbers Range
## 3133 Minimum Array End
