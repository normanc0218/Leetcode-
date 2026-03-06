## 1 Two Sum

## 49 Group Anagrams

**核心思路：** 找一个 key 让所有 anagram 映射到同一组。

### 方法一：排序法 O(n·k log k)

```python
def groupAnagrams(self, strs):
    group = defaultdict(list)
    for s in strs:
        key = "".join(sorted(s))
        group[key].append(s)
    return list(group.values())
```

### 方法二：字符计数法 O(n·k)

```python
def groupAnagrams(self, strs):
    group = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for c in s:
            count[ord(c) - ord('a')] += 1
        group[tuple(count)].append(s)
    return list(group.values())
```

用 26 位计数 tuple 作 key，"eat" 和 "tea" 都得到 `(1,0,0,0,1,...,1,...,0)`。

### 方法三：质数映射法 O(n·k)

每个字母映射一个质数，乘积作 key（质因数分解唯一）。缺点：字符串长时大整数变慢。

| 方法 | 时间 | 优点 | 缺点 |
|------|------|------|------|
| 排序 | O(n·k log k) | 简洁易读 | 排序开销 |
| 字符计数 | O(n·k) | 理论最优 | key 较长 |
| 质数映射 | O(n·k) | key 是单整数 | 大整数溢出 |


## 217 Contains Duplicate
## 347 Top K Frequent Elements
## 238 Product of Array Except Self

---

## 🔹 Pattern
**前缀积 × 后缀积**

---

## 💡 Core Idea

**关键洞察：答案 = 左边所有数的乘积 × 右边所有数的乘积**

- **不能用除法**（题目要求）
- **核心思想**：
  - `result[i] = (左边所有数的乘积) × (右边所有数的乘积)`
  - 前缀积（prefix）：`i` 左边所有数的乘积
  - 后缀积（suffix）：`i` 右边所有数的乘积

**优化**：用结果数组存储中间结果，空间 O(1)（不算输出数组）

**两次遍历**：
1. 从右往左：计算后缀积，存入 `res`
2. 从左往右：计算前缀积，乘到 `res` 上

---

## 🧩 Pseudo Code

```text
function productExceptSelf(nums):
    n = len(nums)
    res = [1] × n
    
    // 第一次遍历：从右往左，计算后缀积
    suffix = 1
    for i from n-1 to 0:
        res[i] = suffix          // 先存入当前后缀积
        suffix *= nums[i]        // 更新后缀积
    
    // 第二次遍历：从左往右，计算前缀积并相乘
    prefix = 1
    for i from 0 to n-1:
        res[i] *= prefix         // 乘上前缀积
        prefix *= nums[i]        // 更新前缀积
    
    return res
```

---

## ✅ Complete Code

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        res = [1] * n
        
        # 第一次遍历：从右往左，计算后缀积
        suff = 1
        for i in range(n - 1, -1, -1):
            res[i] *= suff      # res[i] = 右边所有数的乘积
            suff *= nums[i]     # 更新后缀积
        
        # 第二次遍历：从左往右，计算前缀积
        pref = 1
        for i in range(n):
            res[i] *= pref      # res[i] *= 左边所有数的乘积
            pref *= nums[i]     # 更新前缀积
        
        return res
```
---

## 🔑 核心原理

### **为什么要两次遍历？**

```
对于 nums[i]，答案是:
result[i] = (左边的乘积) × (右边的乘积)

例如: nums = [1, 2, 3, 4]
i=2 (数字3):
  左边: 1 × 2 = 2
  右边: 4 = 4
  结果: 2 × 4 = 8

第一次遍历: 存储右边的乘积 → res = [24, 12, 4, 1]
第二次遍历: 乘上左边的乘积 → res = [24, 12, 8, 6]
```

---

### **为什么从右往左计算后缀积？**

```
从右往左:
i=3: 右边没有数 → 1
i=2: 右边是 4 → 4
i=1: 右边是 3×4 → 12
i=0: 右边是 2×3×4 → 24

从左往右也可以，但需要额外变量或逻辑
从右往左更自然，直接用 res 存储
```

---

## 📊 复杂度分析

**时间复杂度**: O(n)
- 两次遍历，每次 O(n)

**空间复杂度**: O(1)
- 不算输出数组，只用了 `pref` 和 `suff` 两个变量
- （如果算输出数组则是 O(n)）

---

## 💡 另一种理解方式

```
数组:     [1,    2,    3,    4]
         
左边积:   [1,    1,   1×2, 1×2×3]
右边积:  [2×3×4, 3×4,  4,    1]
         
结果:    [1×24, 1×12, 2×4, 6×1]
        = [24,   12,   8,    6]

关键: 左边积 × 右边积 = 除了自己以外所有数的积
```

---

## 🧠 快速记忆

```
右往左存后缀积，
左往右乘前缀积，
两次遍历巧结合，
O(1) 空间真神奇！
```

---

## 🎯 Key Points

```
✅ 答案 = 左边积 × 右边积
✅ 第一次遍历：从右往左存后缀积
✅ 第二次遍历：从左往右乘前缀积
✅ 用 res 数组存储中间结果，空间 O(1)
✅ 时间 O(n)，两次遍历
```

---

**提示**: 这题的精髓在于用结果数组存储中间结果，实现 O(1) 额外空间！💪
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
# 📘 125. Valid Palindrome

---

## 🔹 Pattern
**双指针 (Two Pointers)**

---

## 💡 Core Idea

**关键洞察：双指针从两端向中间靠拢，跳过非字母数字字符**

- **双指针**：`l` 从左，`r` 从右，向中间移动
- **过滤规则**：只考虑字母和数字，忽略空格、标点
- **大小写不敏感**：统一转换为小写比较
- **核心逻辑**：
  1. 跳过左边的非字母数字
  2. 跳过右边的非字母数字
  3. 比较两边字符（转小写）
  4. 不匹配返回 `False`，匹配则继续

**为什么用双指针？** 回文串的特点是对称，从两端比较最直观高效。

---

## 🧩 Pseudo Code

```text
function isPalindrome(s):
    l = 0
    r = len(s) - 1
    
    while l < r:
        // 跳过左边的非字母数字
        if s[l] not alphanumeric:
            l += 1
        
        // 跳过右边的非字母数字
        else if s[r] not alphanumeric:
            r -= 1
        
        // 比较字符（转小写）
        else if s[l].lower() == s[r].lower():
            l += 1
            r -= 1
        
        // 不匹配
        else:
            return False
    
    return True
```

---

## ✅ Complete Code

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        l = 0
        r = len(s) - 1
        
        while l < r:
            # 跳过左边的非字母数字
            if not s[l].isalnum():
                l += 1
            
            # 跳过右边的非字母数字
            elif not s[r].isalnum():
                r -= 1
            
            # 比较字符（转小写）
            elif s[l].lower() == s[r].lower():
                l += 1
                r -= 1
            
            # 不匹配
            else:
                return False
        
        return True
```
## 📊 三种解法对比

### **方法1：双指针 + elif（推荐）**
```python
def isPalindrome(self, s: str) -> bool:
    l, r = 0, len(s) - 1
    while l < r:
        if not s[l].isalnum():
            l += 1
        elif not s[r].isalnum():
            r -= 1
        elif s[l].lower() == s[r].lower():
            l += 1
            r -= 1
        else:
            return False
    return True
```
- **时间**: O(n)
- **空间**: O(1)
- **优点**: 清晰、高效

---

### **方法2：双指针 + while 跳过**
```python
def isPalindrome(self, s: str) -> bool:
    l, r = 0, len(s) - 1
    while l < r:
        # 持续跳过左边
        while l < r and not s[l].isalnum():
            l += 1
        # 持续跳过右边
        while l < r and not s[r].isalnum():
            r -= 1
        
        if s[l].lower() != s[r].lower():
            return False
        l += 1
        r -= 1
    return True
```
- **时间**: O(n)
- **空间**: O(1)
- **优点**: 跳过逻辑更明确

---

### **方法3：预处理（最简洁）**
```python
def isPalindrome(self, s: str) -> bool:
    # 过滤并转小写
    filtered = ''.join(c.lower() for c in s if c.isalnum())
    # 比较反转
    return filtered == filtered[::-1]
```
- **时间**: O(n)
- **空间**: O(n)
- **优点**: 代码最短


## 🎯 复杂度分析

**时间复杂度**: O(n)
- 每个字符最多访问一次
- `isalnum()` 和 `lower()` 都是 O(1)

**空间复杂度**: O(1)
- 只用了两个指针
- 预处理法是 O(n)



---

## 🎯 Key Points

```
✅ 双指针从两端向中间移动
✅ 用 elif 确保逻辑互斥
✅ isalnum() 判断字母数字
✅ lower() 转小写比较
✅ 跳过非字母数字后继续循环
```

---

## 🧠 快速记忆

```
双指针两端走，
非字母数字跳，
转小写再比较，
不等就False报！
```

---

## 📝 相关题目

### **680. Valid Palindrome II**
```python
# 允许删除一个字符后判断回文
# 需要递归或双指针 + 辅助函数
```

### **5. Longest Palindromic Substring**
```python
# 找最长回文子串
# 中心扩展法或动态规划
```

### **9. Palindrome Number**
```python
# 判断数字是否回文
# 可以转字符串用双指针
```


**提示**: 双指针是最基础的模式之一，这题是入门必刷题！💪
## 167 Two Sum II
## 15 3Sum
## 11 Container With Most Water

---

## 🔹 Pattern
**双指针 (Two Pointers) + 贪心**

---

## 💡 Core Idea

**关键洞察：移动较短的边，才有可能获得更大面积**

- **面积公式**：`area = min(height[l], height[r]) × (r - l)`
- **核心策略**：
  - 从两端开始，宽度最大
  - 每次移动较短的那一边
  - 为什么？因为面积由短板决定，移动长板只会让宽度变小，面积不可能更大

**贪心思想**：
- 宽度在递减（`r - l` 越来越小）
- 要获得更大面积，必须寻找更高的边
- 移动短边才有机会找到更高的边
- 移动长边只会让情况更糟（宽度减小，高度不会超过当前短边）

---

## 🧩 Pseudo Code

```text
function maxArea(heights):
    l = 0
    r = len(heights) - 1
    maxArea = 0
    
    while l < r:
        // 计算当前面积（短板 × 宽度）
        height = min(heights[l], heights[r])
        area = height × (r - l)
        maxArea = max(maxArea, area)
        
        // 移动较短的边
        if heights[l] < heights[r]:
            l += 1  // 左边更短，移动左指针
        else:
            r -= 1  // 右边更短或相等，移动右指针
    
    return maxArea
```

---

## ✅ Complete Code

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        l = 0
        r = len(heights) - 1
        max_res = 0
        
        while l < r:
            # 计算当前面积
            h = min(heights[l], heights[r])
            max_res = max(max_res, h * (r - l))
            
            # 移动较短的边
            if heights[l] < heights[r]:
                l += 1
            else:
                r -= 1
        
        return max_res
```

## 🔑 核心原理

### **为什么移动短边？**

```
当前状态: l=0(高度1), r=8(高度7)
面积 = min(1, 7) × 8 = 8

选项1: 移动长边 r (高度7)
  新状态: l=0(1), r=7(3)
  面积 = min(1, 3) × 7 = 7 ← 更小了！
  原因: 宽度减小，高度还是1（短板）

选项2: 移动短边 l (高度1)
  新状态: l=1(8), r=8(7)
  面积 = min(8, 7) × 7 = 49 ← 可能更大！
  原因: 宽度减小，但找到了更高的边

结论: 移动短边才有可能获得更大面积
```

---

## 📊 时间复杂度分析

**时间复杂度**: O(n)
- 两个指针各遍历一次
- 每次循环 O(1) 操作

**空间复杂度**: O(1)
- 只用了常数个变量

---

## 🧠 快速记忆

```
双指针两端放，
短板决定面积量，
移动短边找更高，
贪心策略最优解！
```

---

## 🎯 Key Points

```
✅ 面积 = min(height[l], height[r]) × (r - l)
✅ 移动较短的边
✅ 贪心：只有找到更高的边才可能增大面积
✅ 时间 O(n)，空间 O(1)
```

---

**提示**: 这题的精髓在于理解"为什么移动短边"，这是贪心思想的经典应用！💪
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
### 1. dp 数组含义

`dp[i]` = 前 i 个字符的解码方法总数

### 2. 递推公式（两个 if，不是 if-elif）

```python
if s[i-1] != '0':            # 单独解码第 i 个字符
    dp[i] += dp[i-1]
if 10 <= int(s[i-2:i]) <= 26:  # 和前一个字符组合解码
    dp[i] += dp[i-2]
```

> **关键：两个 if 互不排斥，可能同时成立，都要检查。**
> 用 elif 会漏掉两者同时成立的情况。

### 3. 初始化

```python
if not s or s[0] == '0':  # 提前排除，保证 dp[1]=1 合法
    return 0
dp[0] = dp[1] = 1
```

- `dp[0] = 1`：空串作为递推基础
- `dp[1] = 1`：首字符已由提前判断保证非 '0'

### 4. 遍历顺序

从左到右，`i` 从 2 到 n（依赖 `dp[i-1]` 和 `dp[i-2]`）

### 5. 举例推导 s = "226"

```
dp = [1, 1, 0, 0]

i=2: s[1]='2'≠'0' → dp[2]+=dp[1]=1
     10≤22≤26     → dp[2]+=dp[0]=2  → dp=[1,1,2,0]

i=3: s[2]='6'≠'0' → dp[3]+=dp[2]=2
     10≤26≤26     → dp[3]+=dp[1]=3  → dp=[1,1,2,3]

结果：3 种（"BBF", "VF", "BZ"）
```

### 完整代码

```python
def numDecodings(self, s):
    if not s or s[0] == '0':
        return 0
    n = len(s)
    dp = [0] * (n + 1)
    dp[0] = dp[1] = 1
    for i in range(2, n + 1):
        if s[i-1] != '0':
            dp[i] += dp[i-1]
        if 10 <= int(s[i-2:i]) <= 26:
            dp[i] += dp[i-2]
    return dp[n]
```
## 322 Coin Change
## 152 Maximum Product Subarray

## 方法一：前后缀积（贪心）

**核心观察：** 最大乘积子数组一定从左端或右端延伸而来，遇到 0 就重置。

```python
def maxProduct(self, nums):
    n, res = len(nums), nums[0]
    prefix = suffix = 0
    for i in range(n):  # 注意：range(n) 不能少遍历
        prefix = nums[i] * (prefix or 1)
        suffix = nums[n - 1 - i] * (suffix or 1)
        res = max(res, prefix, suffix)
    return res
```

### 关键细节

- `prefix or 1`：当 prefix 为 0 时重置为 1，等于从下一个元素重新开始
- **为什么双向？** 负数个数为奇数时，一个方向会包含多余的负数，另一个方向能避开它
- 例：`[-2, 3, 4]`，前缀积 -2, -6, -24 全负；后缀积 4, 12, -24 能捕捉到 12

### 易错点

循环必须是 `range(n)`，写成 `range(n-1)` 会导致前缀漏最后一个元素、后缀漏第一个元素。

---

## 方法二：DP（max/min 追踪）

**核心思路：** 每个位置维护当前最大积和最小积，因为负负得正，最小值乘负数可能变最大。

```python
def maxProduct(self, nums):
    res = cur_max = cur_min = nums[0]
    for i in range(1, len(nums)):
        candidates = (nums[i], nums[i] * cur_max, nums[i] * cur_min)
        cur_max = max(candidates)
        cur_min = min(candidates)
        res = max(res, cur_max)
    return res
```

### 为什么要同时追踪 min？

`nums = [2, -1, -2]`：到 -1 时 cur_min=-2，再乘 -2 得 4，这就是最大值。只追踪 max 会丢失这个翻转。

---

## 对比

| | 前后缀积（贪心） | DP（max/min） |
|---|---|---|
| 时间 | O(n) | O(n) |
| 空间 | O(1) | O(1) |
| 思路 | 最优解一定从某端延伸 | 每步维护最大/最小状态 |
| 优点 | 代码简洁，不易出错 | 更通用，易推广 |
## 300 Longest Increasing Subsequence

## 方法一：DP O(n²) — 动规五部曲

### 1. dp 数组含义

`dp[i]` = 以 `nums[i]` 结尾的最长递增子序列长度

### 2. 递推公式

```python
if nums[i] > nums[j]:
    dp[i] = max(dp[i], dp[j] + 1)
```

遍历 i 之前所有的 j，只要 `nums[i] > nums[j]`，就能接在 j 的子序列后面。

### 3. 初始化

`dp = [1] * n`，每个元素自身就是长度 1 的子序列。

### 4. 遍历顺序

外层 i 从左到右，内层 j 从 0 到 i-1。`dp[i]` 依赖前面所有 `dp[j]`。

### 5. 举例推导 nums = [10, 9, 2, 5, 3, 7]

```
i=0: 无 j 可比              dp=[1,1,1,1,1,1]
i=1: 9<10                   dp=[1,1,1,1,1,1]
i=2: 2<10, 2<9              dp=[1,1,1,1,1,1]
i=3: 5>2                    dp=[1,1,1,2,1,1]
i=4: 3>2                    dp=[1,1,1,2,2,1]
i=5: 7>2, 7>5, 7>3          dp=[1,1,1,2,2,3]

结果：3（2,5,7 或 2,3,7）
```

### 完整代码

```python
def lengthOfLIS(self, nums):
    n = len(nums)
    dp = [1] * n
    res = 1
    for i in range(n):
        for j in range(i):
            if nums[i] > nums[j]:
                dp[i] = max(dp[i], dp[j] + 1)
        res = max(res, dp[i])  # 放外层循环，不要放内层
    return res
```

---

## 方法二：贪心 + 二分 O(n log n)

### 核心思路

维护 `tails` 数组，`tails[i]` = 长度为 i+1 的递增子序列的**最小末尾元素**。让末尾尽可能小，给后续留更多空间。

对每个新元素：
- 比 tails 所有元素都大 → 接在后面，长度 +1
- 否则 → 二分找到第一个 ≥ 它的位置，替换掉

### 举例 nums = [10, 9, 2, 5, 3, 7]

```
10 → tails = [10]
 9 → 替换10  tails = [9]
 2 → 替换9   tails = [2]
 5 → 接后面  tails = [2, 5]
 3 → 替换5   tails = [2, 3]
 7 → 接后面  tails = [2, 3, 7]

长度 = 3
```

> **注意：** tails 的内容不一定是真正的 LIS，但长度一定正确。

### 完整代码

```python
def lengthOfLIS(self, nums):
    tails = []
    for num in nums:
        lo, hi = 0, len(tails)
        while lo < hi:
            mid = (lo + hi) // 2
            if tails[mid] < num:
                lo = mid + 1
            else:
                hi = mid
        if lo == len(tails):
            tails.append(num)
        else:
            tails[lo] = num
    return len(tails)
```

---

## 对比

| 方法 | 时间 | 空间 | 特点 |
|---|---|---|---|
| DP | O(n²) | O(n) | 直观，动规五部曲标准题 |
| 贪心+二分 | O(n log n) | O(n) | 最优，但 tails 不是真正的 LIS |
## 62 Unique Paths
## 动规五部曲

### 1. dp 数组含义

`dp[i][j]` = 从左上角 (0,0) 到位置 (i,j) 的路径总数

### 2. 递推公式

```python
dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

只能从上方或左方到达，两个方向的路径数相加。

### 3. 初始化

第一行和第一列都是 1，因为只有一条路可以到达（一直向右或一直向下）。

```python
for i in range(m):
    dp[i][0] = 1
for j in range(n):
    dp[0][j] = 1
```

### 4. 遍历顺序

从左到右、从上到下，`i` 从 1 到 m-1，`j` 从 1 到 n-1。因为 `dp[i][j]` 依赖上方和左方的值。

### 5. 举例推导 m=3, n=3

```
1  1  1
1  2  3
1  3  6

结果：6
```

---

## 完整代码

```python
def uniquePaths(self, m, n):
    dp = [[0] * n for _ in range(m)]
    for i in range(m):
        dp[i][0] = 1
    for j in range(n):
        dp[0][j] = 1
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]
```

---

## 空间优化：O(n)

每一行只依赖当前行左边和上一行同位置，可以压缩成一维数组：

```python
def uniquePaths(self, m, n):
    dp = [1] * n
    for i in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j-1]  # dp[j] 本身是上方，dp[j-1] 是左方
    return dp[n-1]
```

---

## 数学解法：O(m+n)

本质是从 (0,0) 到 (m-1,n-1) 走 m-1 步下 + n-1 步右，选哪些步向下：

```python
from math import comb
def uniquePaths(self, m, n):
    return comb(m + n - 2, m - 1)
```

---

## 对比

| 方法 | 时间 | 空间 |
|---|---|---|
| 二维 DP | O(mn) | O(mn) |
| 一维 DP | O(mn) | O(n) |
| 组合数学 | O(m+n) | O(1) |
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


## 56 Merge Intervals
## 57 Insert Interval
## 435 Non-overlapping Intervals
## 920 Meeting Rooms
## 2402 Meeting Rooms III


## 48 Rotate Image
## 54 Spiral Matrix
## 73 Set Matrix Zeroes

## 思路演进

本质是模拟题，没有算法技巧，核心难点在于**怎么省空间**。不能边遍历边置零，否则新产生的 0 会影响后续判断。

---

## 方法一：暴力 O(mn) 空间

存所有 0 的坐标，再统一置零。不需要 copy 矩阵。

```python
def setZeroes(self, matrix):
    m, n = len(matrix), len(matrix[0])
    zeros = []
    for i in range(m):
        for j in range(n):
            if matrix[i][j] == 0:
                zeros.append((i, j))
    for i, j in zeros:
        for k in range(n):
            matrix[i][k] = 0
        for k in range(m):
            matrix[k][j] = 0
```

---

## 方法二：O(m+n) 空间 — 用 set 记录行列

坐标有冗余，其实只需要知道**哪些行**和**哪些列**要置零。

```python
def setZeroes(self, matrix):
    m, n = len(matrix), len(matrix[0])
    row = set()
    col = set()
    for i in range(m):
        for j in range(n):
            if matrix[i][j] == 0:
                row.add(i)
                col.add(j)
    for i in range(m):
        for j in range(n):
            if i in row or j in col:
                matrix[i][j] = 0
```

---

## 方法三：O(1) 空间 — 用第一行/列当标记

把 set 的功能"寄存"到矩阵的第一行和第一列上：

| set 做法 | O(1) 做法 |
|---|---|
| `row.add(i)` | `matrix[i][0] = 0` |
| `col.add(j)` | `matrix[0][j] = 0` |
| `i in row` | `matrix[i][0] == 0` |
| `j in col` | `matrix[0][j] == 0` |

### 四步，顺序不能乱

1. **先救信息**：记录第一行/列自身是否有 0（马上要被覆盖当标记了）
2. **扫描内部，做标记**：往第一行/列写标记
3. **扫描内部，置零**：读第一行/列的标记
4. **最后处理第一行/列自己**

```python
def setZeroes(self, matrix):
    m, n = len(matrix), len(matrix[0])

    # 1. 先救第一行/列的信息
    first_row_zero = any(matrix[0][j] == 0 for j in range(n))  # 别漏 == 0
    first_col_zero = any(matrix[i][0] == 0 for i in range(m))

    # 2. 扫描内部，做标记
    for i in range(1, m):
        for j in range(1, n):
            if matrix[i][j] == 0:
                matrix[i][0] = 0
                matrix[0][j] = 0

    # 3. 扫描内部，置零
    for i in range(1, m):
        for j in range(1, n):
            if matrix[i][0] == 0 or matrix[0][j] == 0:
                matrix[i][j] = 0

    # 4. 最后处理第一行/列
    if first_row_zero:
        for j in range(n):
            matrix[0][j] = 0
    if first_col_zero:
        for i in range(m):
            matrix[i][0] = 0
```

### 易错点

- `any(matrix[0][j] for ...)` 检查的是值是否 truthy（非零即 True），必须写 `== 0`
- 第一行/列必须**最后**处理，否则标记信息会被提前覆盖
- 内部循环从 1 开始，不包含第一行/列

---

## 对比

| 方法 | 时间 | 空间 | 思路 |
|---|---|---|---|
| 存坐标 | O(mn) | O(mn) | 直接记录所有 0 的位置 |
| set 记录行列 | O(mn) | O(m+n) | 只记录哪些行列要置零 |
| 第一行/列当标记 | O(mn) | O(1) | 借用矩阵自身空间存标记 |
## 202 Happy Number
## 66 Plus One
## 50 Pow(x, n)

## 136 Single Number
## 191 Number of 1 Bits
## 338 Counting Bits
## 268 Missing Number
## 371 Sum of Two Integers
## 190 Reverse Bits