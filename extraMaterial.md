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
## 核心思路

从后往前填。nums1 后面有 n 个空位，从后往前不会覆盖还没处理的元素。

## 三个指针

```
nums1 = [1, 2, 3, 0, 0, 0]
              ↑i          ↑k
nums2 = [2, 5, 6]
              ↑j

i = m-1       nums1 有效部分的最后一个
j = n-1       nums2 的最后一个
k = m+n-1     nums1 要填的位置
```

每次比较 nums1[i] 和 nums2[j]，大的放到 nums1[k]。

## 完整代码

```python
class Solution:
    def merge(self, nums1, m, nums2, n):
        i = m - 1
        j = n - 1
        k = m + n - 1
        while j >= 0:
            if i >= 0 and nums1[i] > nums2[j]:
                nums1[k] = nums1[i]
                i -= 1
            else:
                nums1[k] = nums2[j]
                j -= 1
            k -= 1
```

## 走一遍示例

```
nums1 = [1, 2, 3, 0, 0, 0]   nums2 = [2, 5, 6]

i=2 j=2 k=5: 3 vs 6 → 6大 → nums1[5]=6   j=1
i=2 j=1 k=4: 3 vs 5 → 5大 → nums1[4]=5   j=0
i=2 j=0 k=3: 3 vs 2 → 3大 → nums1[3]=3   i=1
i=1 j=0 k=2: 2 vs 2 → 相等 → nums1[2]=2  j=-1

j<0，退出

结果: [1, 2, 2, 3, 5, 6]
```

## 为什么 while j >= 0 就够了

nums2 全部放完就结束。nums1 剩下的元素本来就在 nums1 的正确位置，不用移动。

反过来如果 nums1 先用完（i<0），循环继续把 nums2 剩下的填进去，由 else 分支处理。

## 为什么从后往前

从前往前会覆盖 nums1 还没处理的元素：

```
nums1 = [1, 2, 3, 0, 0, 0]   nums2 = [2, 5, 6]

从前往前: nums1[0] = 1 ✓
          nums1[1] = 2 ✓（但 nums1 原来的 2 被覆盖了吗？）
```

从后往前利用了 nums1 末尾的空位，填入不会影响还没比较的元素。

## 复杂度

- 时间：O(m + n)
- 空间：O(1)，原地操作

## 26 Remove Duplicates From Sorted Array
## 18 4Sum
## 189 Rotate Array
## 881 Boats to Save People
## 219 Contains Duplicate II
## 209 Minimum Size Subarray Sum
## 658 Find K Closest Elements
## 682 Baseball Game
## 225 Implement Stack Using Queues
# 225. Implement Stack using Queues — 笔记

## 题意

用队列实现栈的 push、pop、top、empty 操作。

---

## 核心技巧

用单个 deque，每次 push 后把前面的元素轮转到后面，保证最新元素永远在队头。

```
push(4)，队列原本是 [3, 2, 1]：
    append(4)  → [3, 2, 1, 4]
    轮转 3 次  → [4, 3, 2, 1]
```

这样 `popleft()` 就等于栈的 pop。

---

## 为什么轮转 len - 1 次而不是 len 次？

append 之后新元素已经在队列里，长度包含了它。轮转 `len - 1` 次刚好把它前面的元素全部移到后面，跳过它自己。用 `len` 次的话新元素也会被转一圈回原位，顺序就错了。

---

## 代码

```python
class MyStack:
    def __init__(self):
        self.q = deque()

    def push(self, x: int) -> None:
        self.q.append(x)
        for _ in range(len(self.q) - 1):
            self.q.append(self.q.popleft())

    def pop(self) -> int:
        return self.q.popleft()

    def top(self) -> int:
        return self.q[0]

    def empty(self) -> bool:
        return len(self.q) == 0
```

---

## 复杂度

| 操作 | 时间 |
|------|------|
| push | O(n)（轮转） |
| pop / top / empty | O(1) |
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

## 核心思路

```
1. 找节点 → BST 二分查找（左小右大）
2. 删节点 → 三种情况
3. 两子时 → 找替代值，删掉原来的
```

## 三种情况

**情况1：叶子节点** → 直接删（返回 None）

```
删除7:    5        5
         / \  →   /
        3   7    3
```

**情况2：只有一个子节点** → 用子节点替代

```
删除7:    5        5
         / \  →   / \
        3   7    3   9
             \
              9
```

**情况3：有两个子节点** → 找右子树最小值替换，再删掉那个最小值

```
删除5:    5        6
         / \  →   / \
        3   8    3   8
           /        /
          6        7
           \
            7
```

## 完整代码

```python
class Solution:
    def deleteNode(self, root, key):
        if not root:
            return None

        if key < root.val:
            root.left = self.deleteNode(root.left, key)
        elif key > root.val:
            root.right = self.deleteNode(root.right, key)
        else:
            # 找到了要删的节点
            if not root.left:        # 情况1&2: 无左子树（含叶子）
                return root.right
            if not root.right:       # 情况2: 无右子树
                return root.left

            # 情况3: 有两个子节点
            cur = root.right
            while cur.left:
                cur = cur.left       # 找右子树最小值
            root.val = cur.val       # 复制值
            root.right = self.deleteNode(root.right, cur.val)  # 删掉副本

        return root
```

## 关键细节

### `return root.right` 为什么能同时处理情况1和2

```
叶子节点:   root.right = None → return None → 节点被删掉
有右子节点: root.right = 某节点 → return 那个节点 → 替代了当前节点
```

父节点接住返回值，两种情况都正确。

### 情况3 为什么要 deleteNode 第二次

复制值后树里有两份相同的值，必须删掉原来那个：

```
      5                4               4
     / \    复制值     / \    删副本    / \
    3   8   →        3   8   →       3   8
     \                \
      4                4 ← 重复！
```

### 右子树最小 vs 左子树最大

两种都可以，题目允许多种结果：

```python
# 右子树最小值（常见写法）
cur = root.right
while cur.left:
    cur = cur.left
root.val = cur.val
root.right = self.deleteNode(root.right, cur.val)

# 左子树最大值（同样正确）
cur = root.left
while cur.right:
    cur = cur.right
root.val = cur.val
root.left = self.deleteNode(root.left, cur.val)
```

## 复杂度

- 时间：O(h)，h 是树高，已是最优
- 空间：O(h)，递归调用栈

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

## 解法一：回溯 + 剪枝

### 核心思路

把每根火柴分配到 4 条边中的一条，DFS 尝试所有分配方式，看能否让 4 条边都等于 `sum/4`。

### 完整代码

```python
class Solution:
    def makesquare(self, matchsticks: List[int]) -> bool:
        if sum(matchsticks) % 4 != 0:
            return False
        target = sum(matchsticks) // 4
        matchsticks.sort(reverse=True)  # 剪枝1: 从大到小排序

        sides = [0] * 4

        def dfs(i):
            if i == len(matchsticks):
                return sides[0] == sides[1] == sides[2] == sides[3]

            for side in range(4):
                if sides[side] + matchsticks[i] > target:  # 剪枝2: 超过目标就跳过
                    continue

                sides[side] += matchsticks[i]
                if dfs(i + 1):
                    return True
                sides[side] -= matchsticks[i]

                if sides[side] == 0:  # 剪枝3: 空边去重
                    break

            return False

        return dfs(0)
```

### 三个剪枝

1. **降序排列** — 大的先放，更早触发超限，减少搜索分支
2. **超过 target 就跳过** — 某条边已经超过 `sum/4`，没必要继续
3. **空边去重** — 当前边为 0，放进去失败了，其他空边结果一样，直接 `break`

### 剪枝3 详解

```
sides = [3, 0, 0, 0]，尝试放火柴到：
  side 1 (值为0): 放进去 → 递归失败 → 回溯到 0
  side 2 (值为0): 和 side 1 完全一样，不用试 → break
  side 3 (值为0): 同理
```

`break` 跳出的是 `for side in range(4)` 循环，然后 `return False` 回溯到上一层。不会死循环。

### 复杂度

- 时间：最坏 O(4ⁿ)，剪枝后通常很快
- 空间：O(n)

---

## 解法二：位掩码 DP

---

# 位掩码 DP 简明笔记

## 什么是位掩码？

用一个整数的二进制位来表示"哪些元素被选过"。

以 4 根火柴为例，`mask = 0b1010 = 10`：

```
火柴编号:  3  2  1  0
mask:      1  0  1  0
→ 第1根和第3根已用，第0根和第2根没用
```

## 三个核心操作

| 操作 | 代码 | 含义 |
|------|------|------|
| 检查第 i 位 | `mask & (1 << i)` | 第 i 根火柴用过没？ |
| 设置第 i 位 | `mask \| (1 << i)` | 标记第 i 根为已使用 |
| 取模归零 | `% target` | 填满一条边，开始下一条 |

## 套到火柴拼正方形（LeetCode 473）

**状态定义：** `dp[mask]` = 用了 mask 对应的火柴后，当前正在填的边累积了多少长度。`-1` 表示不可达。

**转移：** 对每个可达的 mask，尝试加入每根未使用的火柴。

```python
dp[0] = 0  # 初始：没用任何火柴，累积长度为 0

for mask in range(1 << n):
    if dp[mask] == -1:
        continue
    for i in range(n):
        if mask & (1 << i):          # 已用过，跳过
            continue
        if dp[mask] + matchsticks[i] <= target:
            dp[mask | (1 << i)] = (dp[mask] + matchsticks[i]) % target
```

**答案：** `dp[(1 << n) - 1] == 0`（所有火柴都用完，且刚好归零）。

## 走一遍示例

`matchsticks = [3, 3, 3, 3]`，`target = 3`

```
dp[0000] = 0          → 用第0根 → dp[0001] = 3 % 3 = 0
dp[0001] = 0          → 用第1根 → dp[0011] = 3 % 3 = 0
dp[0011] = 0          → 用第2根 → dp[0111] = 3 % 3 = 0
dp[0111] = 0          → 用第3根 → dp[1111] = 3 % 3 = 0 ✓
```

## 复杂度

- 时间：O(2ⁿ × n)
- 空间：O(2ⁿ)
- n ≤ 15 → 2¹⁵ = 32768，完全可接受

## 和回溯的对比

| | 回溯 + 剪枝 | 位掩码 DP |
|---|---|---|
| 时间 | 最坏 O(4ⁿ)，剪枝后通常很快 | O(2ⁿ × n)，稳定可预测 |
| 空间 | O(n) | O(2ⁿ) |
| 特点 | 依赖剪枝效果 | 不依赖，枚举所有子集 |

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

## 题意

给定课程的先修关系和一组查询 `[u, v]`，判断 u 是否是 v 的先修课（直接或间接）。

---

## 解法一：BFS 拓扑排序 + 传递可达关系（推荐）

在 Course Schedule 的拓扑排序基础上，只多加两行核心逻辑。

### 核心思路

按拓扑顺序处理时，node 的所有先修课已经算好了。处理 node → nei 这条边时：

- node 是 nei 的先修 → `reachable[nei].add(node)`
- node 的所有先修也是 nei 的先修 → `reachable[nei] |= reachable[node]`

### `|=` 是什么

集合的并集更新。`a |= b` 等价于 `a = a | b`，把 b 的所有元素合并到 a 里。

### 代码

```python
class Solution:
    def checkIfPrerequisite(self, numCourses, prerequisites, queries):
        graph = defaultdict(list)
        indegree = [0] * numCourses
        reachable = [set() for _ in range(numCourses)]

        for pre, crs in prerequisites:
            graph[pre].append(crs)
            indegree[crs] += 1

        q = deque()
        for c in range(numCourses):
            if indegree[c] == 0:
                q.append(c)

        while q:
            node = q.popleft()
            for nei in graph[node]:
                reachable[nei].add(node)
                reachable[nei] |= reachable[node]
                indegree[nei] -= 1
                if indegree[nei] == 0:
                    q.append(nei)

        return [u in reachable[v] for u, v in queries]
```

时间复杂度：O(V² + E)

---

## 解法二：DFS + 手动缓存

对每个查询跑 DFS，用字典缓存避免重复计算。

### 代码

```python
class Solution:
    def checkIfPrerequisite(self, numCourses, prerequisites, queries):
        adj = [[] for _ in range(numCourses)]
        for pre, crs in prerequisites:
            adj[pre].append(crs)

        cache = {}

        def dfs(node, target):
            if node == target:
                return True
            if (node, target) in cache:
                return cache[(node, target)]

            for nei in adj[node]:
                if dfs(nei, target):
                    cache[(node, target)] = True
                    return True

            cache[(node, target)] = False
            return False

        return [dfs(u, v) for u, v in queries]
```

时间复杂度：O(V × (V + E))，比 BFS 慢

---

## 两种解法对比

| | BFS 拓扑排序 | DFS + 缓存 |
|---|---|---|
| 核心操作 | 一次遍历传递可达关系 | 每个查询跑 DFS |
| 时间 | O(V² + E) | O(V × (V + E)) |
| 优点 | 更高效，代码简洁 | 思路直观 |
| 和 Course Schedule 的关系 | 原代码多加两行 | 需要额外写 DFS |

V = numCourses（顶点数），E = len(prerequisites)（边数）

---

## 易错点

1. 不加缓存的 DFS 会超时（TLE）
2. 建图方向和 indegree 要配套：`graph[pre].append(crs)` 对应 `indegree[crs] += 1`
3. `reachable` 要用 `[set() for ...]` 初始化，不能用 `[set()] * n`（共享同一个 set）
## 721 Accounts Merge
## 399 Evaluate Division
## 310 Minimum Height Trees
## 1631 Path with Minimum Effort
## 1489 Find Critical and Pseudo Critical Edges in Minimum Spanning Tree
## 2392 Build a Matrix With Conditions
## 2709 Greatest Common Divisor Traversal
## 1137 N-th Tribonacci Number
## 题目描述

泰波那契序列 $T_n$ 定义如下：

$$T_0 = 0,\quad T_1 = 1,\quad T_2 = 1$$

$$T_{n+3} = T_n + T_{n+1} + T_{n+2} \quad (n \geq 0)$$

给定整数 `n`，返回第 `n` 个泰波那契数 $T_n$。

---

## 解题思路

使用**动态规划（DP）**，自底向上递推计算每一项，避免递归重复计算。

### 状态定义

`dp[i]` 表示第 `i` 个泰波那契数。

### 状态转移方程

$$dp[i] = dp[i-1] + dp[i-2] + dp[i-3], \quad i \geq 3$$

### 边界条件

| n | 结果 |
|---|------|
| 0 | 0    |
| 1 | 1    |
| 2 | 1    |

---

## 代码实现

```python
class Solution:
    def tribonacci(self, n: int) -> int:
        if n == 0:
            return 0
        if 1 <= n <= 2:
            return 1

        dp = [0] * (n + 1)
        dp[0] = 0
        dp[1] = 1
        dp[2] = 1

        for i in range(3, n + 1):
            dp[i] = dp[i-1] + dp[i-2] + dp[i-3]

        return dp[n]
```

---

## 复杂度分析

| 类型 | 复杂度 |
|------|--------|
| 时间复杂度 | $O(n)$ |
| 空间复杂度 | $O(n)$ |

> **优化提示**：由于每次只需要前三项，可将 `dp` 数组替换为三个变量，将空间复杂度优化至 $O(1)$。

---

## 示例推导

以 `n = 7` 为例：

| i | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7  |
|---|---|---|---|---|---|---|---|----|
| dp[i] | 0 | 1 | 1 | 2 | 4 | 7 | 13 | 24 |

**输出：** `24`

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

## 题目描述

给定一个二进制字符串 `s` 和两个整数 `minJump`、`maxJump`。

- 起点在下标 `0`（一定是 `'0'`）
- 从 `i` 跳到 `j` 需满足：`i + minJump <= j <= i + maxJump` 且 `s[j] == '0'`

问：能否到达下标 `n-1`？

---

## 核心思路

### 为什么不能贪心？

贪心只考虑局部最优，但这题有障碍物且跳跃范围固定，贪心可能走进死路且无法回头。需要 **DP** 记录所有可达位置。

### 为什么用 DP + 前缀和？

- **DP**：记录每个位置是否可达
- **前缀和**：快速查询某个区间内有没有可达位置，避免内层循环超时

---

## 定义

### `f` 数组（即 dp）

```
f[i] = 1  表示下标 i 可达
f[i] = 0  表示下标 i 不可达
```

初始：`f[0] = 1`（起点一定可达）

### `pre` 数组（前缀和）

```
pre[i] = f[0] + f[1] + ... + f[i]   （包含 i 自身）
```

查询区间 `[l, r]` 内有没有可达位置：

$$pre[r] - pre[l-1] > 0$$

---

## 状态转移

对每个位置 `i`，能跳到它的位置 `j` 满足：

$$i - maxJump \leq j \leq i - minJump$$

所以转移方程为：

$$f[i] = \begin{cases} 1 & \text{若 } s[i] = \text{'0'} \text{ 且 } pre[r] - pre[l-1] > 0 \\ 0 & \text{其他} \end{cases}$$

其中 $l = i - maxJump$，$r = i - minJump$

---

## 初始化细节

由于从 `i = minJump` 开始遍历，需要提前预处理 `[0, minJump)` 这段的前缀和：

```python
for i in range(minJump):
    pre[i] = 1
```

因为这段范围内只有 `f[0] = 1`，其余为 `0`，前缀和全为 `1`。

---

## 完整代码

```python
class Solution:
    def canReach(self, s: str, minJump: int, maxJump: int) -> bool:
        n = len(s)
        f, pre = [0] * n, [0] * n
        f[0] = 1

        # 预处理 [0, minJump) 的前缀和
        for i in range(minJump):
            pre[i] = 1

        for i in range(minJump, n):
            left  = i - maxJump
            right = i - minJump
            if s[i] == '0':
                total = pre[right] - (0 if left <= 0 else pre[left - 1])
                f[i] = int(total != 0)
            pre[i] = pre[i - 1] + f[i]

        return bool(f[n - 1])
```

---

## 举例推导

```
s = "011010"，minJump=2，maxJump=3
```

**初始化：**
```
f   = [1, 0, 0, 0, 0, 0]
pre = [1, 1, 1, 0, 0, 0]   ← 预处理前3位
```

**i=3，s[3]='0'，查 [0, 1]：**
```
left=0, right=1
pre[1] - 0 = 1 > 0  →  f[3]=1
pre[3] = pre[2] + f[3] = 1 + 1 = 2
```

**i=4，s[4]='1'，跳过：**
```
pre[4] = pre[3] + f[4] = 2 + 0 = 2
```

**i=5，s[5]='0'，查 [2, 3]：**
```
left=2, right=3
pre[3] - pre[1] = 2 - 1 = 1 > 0  →  f[5]=1
```

**结果：**
```
f = [1, 0, 0, 1, 0, 1]
return f[5] = 1 → True ✓
```

---

## 我的写法

### 定义

```
dp[i] = True   表示下标 i 可达
dp[i] = False  表示下标 i 不可达
```

`pre` 长度为 `n+1`，**不包含自身**：

```
pre[i] = dp[0] + dp[1] + ... + dp[i-1]
```

查询区间 `[l, r]` 内有没有可达位置：

$$pre[r+1] - pre[l] > 0$$

### 初始化

```python
dp[0] = True
pre[0] = 0   # 固定开头
pre[1] = 1   # 因为 dp[0] = True
# 其余全是 0
```

用数组表示：
```
dp  = [T, F, F, F, F, F]
pre = [0, 1, 0, 0, 0, 0, 0]   ← 长度 n+1=7
```

### 完整代码

```python
class Solution:
    def canReach(self, s: str, minJump: int, maxJump: int) -> bool:
        n = len(s)
        dp = [False] * n
        dp[0] = True
        pre = [0] * (n + 1)
        pre[1] = 1

        for i in range(1, n):
            if s[i] == '0':
                l = max(0, i - maxJump)
                r = i - minJump
                if r >= 0 and pre[r + 1] - pre[l] > 0:
                    dp[i] = True
            pre[i + 1] = pre[i] + dp[i]

        return dp[n - 1]
```

### 举例推导

```
s = "011010"，minJump=2，maxJump=3
```

**初始化：**
```
dp  = [T, F, F, F, F, F]
pre = [0, 1, 0, 0, 0, 0, 0]
```

**i=3，s[3]='0'，查 [0, 1]：**
```
l=0, r=1
pre[2] - pre[0] = 1 - 0 = 1 > 0  →  dp[3]=True
pre[4] = pre[3] + dp[3] = 1 + 1 = 2
```

**i=5，s[5]='0'，查 [2, 3]：**
```
l=2, r=3
pre[4] - pre[2] = 2 - 1 = 1 > 0  →  dp[5]=True
```

**结果：**
```
dp = [T, F, F, T, F, T]
return dp[5] = True ✓
```

---

## 两种写法对比

| | 我的写法 | 官方写法 |
|--|---------|---------|
| `pre` 长度 | `n+1` | `n` |
| `pre[i]` 含义 | `f[0]..f[i-1]` 的和 | `f[0]..f[i]` 的和 |
| 区间查询 | `pre[r+1] - pre[l]` | `pre[r] - pre[l-1]` |
| 边界特判 | 不需要 | `left <= 0` 时用 `0` |

两种写法**本质相同**，只是下标偏移不同。

---

## 复杂度分析

| 类型 | 复杂度 |
|------|--------|
| 时间复杂度 | $O(n)$ |
| 空间复杂度 | $O(n)$ |

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
