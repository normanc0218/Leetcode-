## 1 Two Sum

## 49 Group Anagrams
## 217 Contains Duplicate
## 347 Top K Frequent Elements
## 238 Product of Array Except Self
## 36 Valid Sudoku
## 128 Longest Consecutive Sequence

---

## 🔹 Pattern
**HashSet + 智能起点**

---

## 💡 Core Idea

**关键洞察：只从序列的起点开始计数，避免重复**

- **HashSet** 用于 O(1) 查找
- **智能起点**：只从 `n-1` 不存在的数字开始
- **为什么？** 如果 `n-1` 存在，说明 `n` 不是序列起点，跳过它避免重复计算

**例子**：`[100, 4, 200, 1, 3, 2]`
- 1: `0` 不存在 ✓ 起点 → 计数 `1,2,3,4` → 长度4
- 2: `1` 存在 ✗ 跳过（不是起点）
- 3: `2` 存在 ✗ 跳过
- 4: `3` 存在 ✗ 跳过
- 100: `99` 不存在 ✓ 起点 → 计数 `100` → 长度1
- 200: `199` 不存在 ✓ 起点 → 计数 `200` → 长度1

**时间复杂度**：O(n) - 每个数字最多被访问2次（一次判断起点，一次被计数）

---

## 🧩 Pseudo Code

```text
function longestConsecutive(nums):
    if empty: return 0
    
    numSet = set(nums)  // O(1) 查找
    maxLength = 0
    
    for num in nums:
        // 只从序列起点开始
        if (num - 1) not in numSet:
            currentNum = num
            currentLength = 1
            
            // 向右延伸，计数连续数字
            while (currentNum + 1) in numSet:
                currentNum += 1
                currentLength += 1
            
            maxLength = max(maxLength, currentLength)
    
    return maxLength
```

---

## ✅ Complete Code

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        numSet = set(nums)  # O(1) 查找
        maxLength = 0
        
        for num in nums:
            # 只从序列起点开始（n-1 不存在）
            if num - 1 not in numSet:
                currentNum = num
                currentLength = 1
                
                # 向右延伸
                while currentNum + 1 in numSet:
                    currentNum += 1
                    currentLength += 1
                
                maxLength = max(maxLength, currentLength)
        
        return maxLength
```


### 2️⃣ **为什么检查 `n-1`？**
```python
# 不检查 n-1（错误）:
# 序列 [1,2,3,4]
# 会计算4次: 1→4, 2→4, 3→4, 4→1
# 时间复杂度 O(n²)

# 检查 n-1（正确）:
# 只从 1 开始计算一次
# 时间复杂度 O(n)
```

---

## 📊 方法对比

| 方法 | 时间 | 空间 | 说明 |
|------|------|------|------|
| **排序** | O(n log n) | O(1) | 排序后遍历 |
| **HashSet（智能起点）** | O(n) | O(n) | ✅ 最优 |
| **Union Find** | O(n α(n)) | O(n) | 复杂度类似但代码复杂 |


---

## 🎯 Key Points

```
✅ 用 Set 实现 O(1) 查找
✅ 只从起点开始（n-1 不存在）
✅ 每个数字最多访问2次
✅ 时间 O(n)，空间 O(n)
```

---

## 🧠 快速记忆

```
Set 快速查找，
起点判断 n-1，
向右延伸计数，
O(n) 轻松搞定！
```


**提示**: 这题的精髓在于"智能起点"，避免重复计算！💪
## 242 Valid Anagram
## 271 Encode and Decode Strings

## 125 Valid Palindrome
## 167 Two Sum II
## 15 3Sum
## 11 Container With Most Water
## 42 Trapping Rain Water

## 121 Best Time to Buy and Sell Stock
## 3 Longest Substring Without Repeating Characters
## 424 Longest Repeating Character Replacement
## 567 Permutation in String
## 76 Minimum Window Substring

## 20 Valid Parentheses
## 155 Min Stack
## 150 Evaluate Reverse Polish Notation
## 22 Generate Parentheses
## 739 Daily Temperatures
## 853 Car Fleet
## 84 Largest Rectangle in Histogram

## 704 Binary Search
## 74 Search a 2D Matrix
## 875 Koko Eating Bananas
## 153 Find Minimum in Rotated Sorted Array
## 33 Search in Rotated Sorted Array
## 981 Time Based Key Value Store
## 4 Median of Two Sorted Arrays

## 206 Reverse Linked List
## 21 Merge Two Sorted Lists
## 143 Reorder List
## 19 Remove Nth Node From End
## 2 Add Two Numbers
## 141 Linked List Cycle
## 138 Copy List With Random Pointer
## 23 Merge k Sorted Lists

## 226 Invert Binary Tree
## 104 Maximum Depth of Binary Tree
## 543 Diameter of Binary Tree
## 110 Balanced Binary Tree
## 100 Same Tree
## 572 Subtree of Another Tree
## 235 LCA of BST
## 102 Binary Tree Level Order Traversal
## 199 Binary Tree Right Side View
## 98 Validate BST
## 230 Kth Smallest in BST
## 105 Construct Binary Tree from Preorder and Inorder
## 124 Binary Tree Maximum Path Sum
## 297 Serialize and Deserialize Binary Tree

## 208 Implement Trie
## 211 Design Add and Search Words
## 212 Word Search II

## 215 Kth Largest Element in Array
## 703 Kth Largest Element in Stream
## 973 K Closest Points to Origin
## 621 Task Scheduler
## 295 Find Median from Data Stream

### 🔹 Pattern
**双堆（Two Heaps）/ 数据流**

### 💡 Core Idea
**关键洞察：用两个堆维护有序序列的"中间分割点"**

- **max_heap**（大顶堆）：存储较小的一半元素
- **min_heap**（小顶堆）：存储较大的一半元素
- **核心不变式**：
  1. `max(max_heap) <= min(min_heap)` - 左边所有元素 <= 右边所有元素
  2. `len(max_heap) == len(min_heap)` 或 `len(max_heap) == len(min_heap) + 1`

**为什么要"推来推去"？**
- 新元素不知道应该放哪个堆
- 先push到max_heap，让堆自动调整，找出最大值
- 把最大值pop出来push到min_heap
- 这样保证了不变式：max_heap的最大值 <= min_heap的所有值
- 如果min_heap太多，平衡回来

**本质**：通过堆的性质，新元素会和所有元素"隐式比较"，找到正确的分界点

### 🧩 Pseudo Code

```text
class MedianFinder:
    max_heap = []  // 存较小一半（用负数模拟大顶堆）
    min_heap = []  // 存较大一半（小顶堆）
    
    function addNum(num):
        // 步骤1: 先push到max_heap
        // 这一步相当于和max_heap的所有元素比较
        push(max_heap, -num)
        
        // 步骤2: pop出max_heap的最大值，push到min_heap
        // 保证 max(max_heap剩余) <= min_heap的所有值
        max_val = -pop(max_heap)
        push(min_heap, max_val)
        
        // 步骤3: 平衡两个堆
        // max_heap最多比min_heap多1个
        if len(min_heap) > len(max_heap):
            min_val = pop(min_heap)
            push(max_heap, -min_val)
    
    function findMedian():
        if len(max_heap) > len(min_heap):
            // 奇数个元素，max_heap多1个
            return -max_heap[0]
        else:
            // 偶数个元素，取两个堆顶的平均值
            return (-max_heap[0] + min_heap[0]) / 2
```

### ✅ Complete Code

```python
import heapq

class MedianFinder:
    def __init__(self):
        # max_heap存较小的一半（Python用负数模拟大顶堆）
        self.max_heap = []
        # min_heap存较大的一半（Python默认小顶堆）
        self.min_heap = []

    def addNum(self, num: int) -> None:
        # 步骤1: 新元素先进max_heap
        heapq.heappush(self.max_heap, -num)
        
        # 步骤2: 取出max_heap的最大值，推到min_heap
        # 这保证了 max(max_heap) <= min(min_heap)
        max_val = -heapq.heappop(self.max_heap)
        heapq.heappush(self.min_heap, max_val)
        
        # 步骤3: 平衡长度
        # 保持 len(max_heap) == len(min_heap) 或 len(max_heap) == len(min_heap) + 1
        if len(self.min_heap) > len(self.max_heap):
            min_val = heapq.heappop(self.min_heap)
            heapq.heappush(self.max_heap, -min_val)

    def findMedian(self) -> float:
        # 如果max_heap多一个，说明总数是奇数
        if len(self.max_heap) > len(self.min_heap):
            return -self.max_heap[0]
        # 否则总数是偶数，返回两个堆顶的平均值
        else:
            return (-self.max_heap[0] + self.min_heap[0]) / 2.0
```

### 🎯 Key Points

```
✅ 两个堆的分工明确：max_heap存小的，min_heap存大的
✅ Python只有小顶堆，用负数模拟大顶堆
✅ 推来推去的本质：利用堆性质自动筛选分界点
✅ 不变式1: max(max_heap) <= min(min_heap)
✅ 不变式2: 长度差 <= 1，max_heap可以多1个
✅ 中位数看max_heap：奇数取堆顶，偶数取平均
```

**时间复杂度**: 
- addNum: O(log n)
- findMedian: O(1)

**空间复杂度**: O(n)

---


## 78 Subsets
## 39 Combination Sum
# 🔹 Pattern
**回溯 (Backtracking) / DFS**

---

## 💡 Core Idea

**关键洞察：穷举所有可能的组合，用回溯剪枝优化**

- **可以重复使用**同一个数字（无限次）
- **不考虑顺序**：`[2,2,3]` 和 `[2,3,2]` 是同一个组合
- **回溯三步曲**：
  1. **做选择** - 把数字加入路径
  2. **递归** - 继续寻找剩余部分
  3. **撤销选择** - 回溯，尝试其他可能

- **关键技巧**：
  - `start` 参数防止重复组合（保证递增顺序）
  - `递归传 i` 而不是 `i+1`，允许重复使用当前数字
  - 排序后可以提前剪枝（`current_sum > target` 时 break）

**为什么用回溯？** 因为需要找出**所有**满足条件的组合，DP虽然能做但复杂度高且不直观。

---

## 🧩 Pseudo Code

```text
function combinationSum(nums, target):
    result = []
    sort(nums)  // 排序方便剪枝优化
    
    function dfs(start, path, current_sum):
        // 终止条件1: 找到一个有效组合
        if current_sum == target:
            add copy of path to result
            return
        
        // 终止条件2: 超过目标，剪枝
        if current_sum > target:
            return
        
        // 遍历选择列表（从start开始，避免重复组合）
        for i from start to len(nums):
            num = nums[i]
            
            // 可选剪枝优化：如果排序过，可以提前break
            if current_sum + num > target:
                break
            
            // 1. 做选择
            add num to path
            
            // 2. 递归（⚠️ 注意传i不是i+1，允许重复使用）
            dfs(i, path, current_sum + num)
            
            // 3. 撤销选择（回溯）
            remove last element from path
    
    dfs(0, [], 0)
    return result
```

---

## ✅ Complete Code

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        result = []
        candidates.sort()  # 排序（可选，但方便剪枝优化）
        
        def dfs(start, path, current_sum):
            # 终止条件：找到一个有效组合
            if current_sum == target:
                result.append(path.copy())  # ⚠️ 必须复制！
                return
            
            # 终止条件：超过目标，剪枝
            if current_sum > target:
                return
            
            # 从 start 开始遍历，避免产生重复组合
            for i in range(start, len(candidates)):
                # 做选择
                path.append(candidates[i])
                
                # 递归：传 i 允许重复使用当前数字
                dfs(i, path, current_sum + candidates[i])
                
                # 撤销选择（回溯）
                path.pop()
        
        dfs(0, [], 0)
        return result
```

### 4️⃣ **剪枝优化（可选但推荐）**

```python
# 方法1：基础剪枝
if current_sum > target:
    return

# 方法2：提前break（需要排序）
candidates.sort()
for i in range(start, len(candidates)):
    if current_sum + candidates[i] > target:
        break  # 后面的数字更大，直接跳出
    # ...
```

---

## 📊 复杂度分析

**时间复杂度**: O(N^(T/M))
- N = candidates 长度
- T = target
- M = candidates 中的最小值
- 最坏情况：target很大，最小值很小（如1）

**空间复杂度**: O(T/M)
- 递归栈深度 = 最长组合长度
---
## 46 Permutations
## 90 Subsets II
## 40 Combination Sum II
## 79 Word Search


---

## 🔹 Pattern
**回溯 (Backtracking) + DFS + 需要撤销状态**

---

## 💡 Core Idea

**关键洞察：在二维网格中寻找路径，需要回溯以尝试不同路径**

- **目标**：在网格中找到一条路径拼出目标单词
- **限制**：每个格子只能使用一次（在同一条路径中）
- **回溯关键**：
  - `visited.add()` - 标记当前路径占用的格子
  - `visited.remove()` - 回溯时释放格子，让其他路径可以使用
  
- **为什么需要 remove？**
  - 因为同一个格子可能出现在**不同的路径尝试**中
  - 路径1失败了，要释放格子让路径2使用

**与 Pacific Atlantic 的区别**：
- Word Search：寻找**特定路径** → 需要回溯（add + remove）
- Pacific Atlantic：标记**所有可达格子** → 不需要回溯（只add）

---

## 🧩 Pseudo Code

```text
function exist(board, word):
    m = rows, n = cols
    visited = set()
    directions = [(0,1), (1,0), (0,-1), (-1,0)]
    
    function dfs(i, j, index):
        // 终止条件：找到完整单词
        if index == len(word):
            return True
        
        // 边界检查 + 字符不匹配 + 已访问
        if i < 0 or j < 0 or i >= m or j >= n:
            return False
        if board[i][j] != word[index]:  // ⚠️ 注意是 !=
            return False
        if (i, j) in visited:
            return False
        
        // 1. 做选择：标记当前格子为已访问
        add (i, j) to visited
        
        // 2. 递归：尝试四个方向
        for each direction (dx, dy):
            nx = i + dx
            ny = j + dy
            if dfs(nx, ny, index + 1):
                return True  // 找到了立即返回
        
        // 3. 撤销选择：回溯，释放格子
        remove (i, j) from visited
        
        return False
    
    // 遍历所有格子作为起点
    for i in 0 to m-1:
        for j in 0 to n-1:
            if dfs(i, j, 0):
                return True
    
    return False
```

---

## ✅ Complete Code

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        if not board or not board[0]:
            return False
        
        m, n = len(board), len(board[0])
        visited = set()
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
        
        def dfs(i, j, index):
            # 终止条件：找到完整单词
            if index == len(word):
                return True
            
            # 边界检查、字符不匹配、已访问
            if (i < 0 or j < 0 or i >= m or j >= n or 
                board[i][j] != word[index] or  # ⚠️ 是 != 不是 ==
                (i, j) in visited):
                return False
            
            # 1. 做选择：标记访问
            visited.add((i, j))
            
            # 2. 递归：尝试四个方向
            for dx, dy in directions:
                nx, ny = i + dx, j + dy
                if dfs(nx, ny, index + 1):
                    return True
            
            # 3. 撤销选择：回溯（释放格子）
            visited.remove((i, j))
            
            return False
        
        # 遍历所有起点
        for i in range(m):
            for j in range(n):
                if dfs(i, j, 0):
                    return True
        
        return False
```

---

---
## 🎯 复杂度分析

**时间复杂度**: O(m × n × 4^L)
- m × n：遍历所有起点
- 4^L：每个位置最多4个方向，最多递归L层（L = word长度）

**空间复杂度**: O(L)
- 递归栈深度 = 单词长度
- visited 最多存 L 个坐标


---

## 🧠 回溯模板（寻路问题）

```python
def backtrack_path(位置, 路径状态, 目标):
    # 终止条件
    if 达到目标:
        return True
    
    # 剪枝条件
    if 不合法:
        return False
    
    # 1. 做选择（标记占用）
    visited.add(位置)
    
    # 2. 递归尝试
    for 下一个位置 in 所有可能:
        if backtrack_path(下一个位置, 新状态, 目标):
            return True
    
    # 3. 撤销选择（释放占用）⚠️ 关键！
    visited.remove(位置)
    
    return False
```

---

## 🎯 Key Points

```
✅ 寻路问题 → 需要回溯（add + remove）
✅ 字符不匹配才返回False（!= 不是 ==）
✅ 必须遍历所有格子作为起点
✅ 递归返回值要用 if 判断
✅ visited 代表"当前路径占用的格子"
```

---

## 📝 快速记忆

### **回溯三步曲**
```
1. 标记访问: visited.add((i, j))
2. 尝试方向: if dfs(...): return True
3. 撤销标记: visited.remove((i, j))  ← 关键！
```

### **判断条件**
```
越界？ → False
不匹配？ → False  ⚠️ 注意是 !=
访问过？ → False
```
---

**提示**: 回溯的关键在于"撤销"，寻路问题几乎都需要回溯！💪
## 131 Palindrome Partitioning
## 17 Letter Combinations of Phone Number

## 200 Number of Islands
## 695 Max Area of Island
## 133 Clone Graph
## 417 Pacific Atlantic Water Flow
## 🔹 Pattern
**反向DFS / 图遍历**

### 💡 Core Idea
**关键洞察：正向思考很难（每个点要判断能否到达两个海洋），反向思考很简单！**

- **从边界开始反向DFS**，把问题反过来想
- 不问"这个格子能流到哪"，而是问"哪些格子能流到这里"
- 从太平洋边界出发，标记所有能流向太平洋的格子
- 从大西洋边界出发，标记所有能流向大西洋的格子
- **关键规则**：反向DFS时，只有当 `heights[next] >= heights[current]` 才能继续（水往高处流）
- 最后取两个集合的**交集**，就是答案

**为什么反向？** 因为从边界开始只需要4条边，而从每个格子正向搜索需要m×n个起点！

### 🧩 Pseudo Code

```text
function pacificAtlantic(heights):
    if empty: return []
    
    m = rows, n = cols
    pacific = set()    // 能流向太平洋的格子
    atlantic = set()   // 能流向大西洋的格子
    directions = [(0,1), (1,0), (0,-1), (-1,0)]
    
    function dfs(x, y, visited):
        if (x,y) already in visited:
            return
        
        add (x,y) to visited
        
        for each direction (dx, dy):
            nx = x + dx
            ny = y + dy
            
            // 检查边界
            if nx,ny out of bounds:
                continue
            
            // ⚠️ 关键：反向流动，只有更高或相等才能流过来
            if heights[nx][ny] >= heights[x][y]:
                dfs(nx, ny, visited)
    
    // 从四条边界开始DFS
    for i in 0 to m-1:
        dfs(i, 0, pacific)        // 左边界（太平洋）
        dfs(i, n-1, atlantic)     // 右边界（大西洋）
    
    for j in 0 to n-1:
        dfs(0, j, pacific)        // 上边界（太平洋）
        dfs(m-1, j, atlantic)     // 下边界（大西洋）
    
    // 取交集
    result = []
    for each cell in pacific:
        if cell also in atlantic:
            add cell to result
    
    return result
```

### ✅ Complete Code

```python
class Solution:
    def pacificAtlantic(self, heights: List[List[int]]) -> List[List[int]]:
        if not heights or not heights[0]:
            return []
        
        m, n = len(heights), len(heights[0])
        pacific = set()
        atlantic = set()
        
        def dfs(x, y, visited):
            # 已访问过，直接返回
            if (x, y) in visited:
                return
            
            # 标记为已访问
            visited.add((x, y))
            
            # 四个方向
            directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
            for dx, dy in directions:
                nx, ny = x + dx, y + dy
                
                # 检查边界
                if 0 <= nx < m and 0 <= ny < n:
                    # 反向DFS：只有高度 >= 当前格子才能流过来
                    if heights[nx][ny] >= heights[x][y]:
                        dfs(nx, ny, visited)
        
        # 从太平洋边界（上边和左边）开始DFS
        for i in range(m):
            dfs(i, 0, pacific)      # 左边界
            dfs(i, n - 1, atlantic) # 右边界
        
        for j in range(n):
            dfs(0, j, pacific)      # 上边界
            dfs(m - 1, j, atlantic) # 下边界
        
        # 返回交集
        return list(pacific & atlantic)
```

### 🎯 Key Points

```
✅ 反向思维：从海洋往陆地流
✅ 不变式：heights[next] >= heights[current]
✅ 两次DFS：pacific和atlantic分别标记
✅ 最终答案：两个集合的交集
```

**时间复杂度**: O(m × n) - 每个格子最多访问2次  
**空间复杂度**: O(m × n) - 两个visited集合

---
## 130 Surrounded Regions
## 994 Rotting Oranges
## 207 Course Schedule
## 210 Course Schedule II
## 684 Redundant Connection

## 332 Reconstruct Itinerary
## 1584 Min Cost to Connect All Points
## 743 Network Delay Time
## 778 Swim in Rising Water
## 787 Cheapest Flights Within K Stops

## 70 Climbing Stairs
## 746 Min Cost Climbing Stairs
## 198 House Robber
## 213 House Robber II
## 5 Longest Palindromic Substring
## 647 Palindromic Substrings
## 91 Decode Ways
## 322 Coin Change
## 152 Maximum Product Subarray
## 300 Longest Increasing Subsequence

## 62 Unique Paths
## 1143 Longest Common Subsequence
## 309 Best Time to Buy and Sell Stock with Cooldown
## 518 Coin Change II
## 494 Target Sum
## 97 Interleaving String
## 72 Edit Distance
## 115 Distinct Subsequences

## 53 Maximum Subarray
## 55 Jump Game
## 45 Jump Game II
## 134 Gas Station
## 846 Hand of Straights
## 1899 Merge Triplets to Form Target Triplet
## 763 Partition Labels
## 678 Valid Parenthesis Str