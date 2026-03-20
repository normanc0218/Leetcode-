## 1 Two Sum
## 题目描述

给定一个整数数组 `nums` 和一个目标值 `target`，在数组中找出**和为目标值的两个整数**，返回它们的下标。

- 每种输入只对应一个答案
- 不能重复使用同一个元素

---

## 解题思路

使用**哈希表（HashMap）**，一次遍历完成查找，避免暴力双层循环。

### 核心思想

遍历数组时，对于当前元素 `n`，计算其补数：

$$diff = target - n$$

- 若 `diff` **已在哈希表中**，直接返回 `[mp[diff], i]`
- 否则，将当前元素及其下标存入哈希表：`mp[n] = i`

### 图示

```
nums = [2, 7, 11, 15],  target = 9

i=0: n=2, diff=7  → 7 不在 mp 中 → mp={2:0}
i=1: n=7, diff=2  → 2 在 mp 中   → 返回 [0, 1] ✓
```

---

## 代码实现

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        mp = {}
        for i, n in enumerate(nums):
            diff = target - n
            if diff in mp:
                return [mp[diff], i]
            mp[n] = i
```

---

## 复杂度分析

| 类型 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | $O(n)$ | 只需遍历数组一次 |
| 空间复杂度 | $O(n)$ | 哈希表最多存储 n 个元素 |

> **对比暴力法**：暴力双层循环时间复杂度为 $O(n^2)$，哈希表将查找从 $O(n)$ 降至 $O(1)$，整体提升显著。

---

## 示例推导

以 `nums = [3, 2, 4]`，`target = 6` 为例：

| i | n | diff | mp 中存在？ | mp 状态 |
|---|---|------|------------|---------|
| 0 | 3 | 3    | ✗          | {3: 0}  |
| 1 | 2 | 4    | ✗          | {3: 0, 2: 1} |
| 2 | 4 | 2    | ✓          | —       |

**输出：** `[1, 2]`

## 287 Find the Duplicate Number

把 `index → nums[index]` 当链表，重复值 = 环入口，用 Floyd 判圈（同 142 题）。

```
nums = [1,3,4,2,2]

0 → 1 → 3 → 2 → 4 → 2 → 4 ...
                 ↑_________↓
环入口 = 2 = 重复的数
```

1. 快慢指针从 0 出发，相遇在环内
2. 一个回到 0，都走一步，再次相遇就是入口

```python
class Solution:
    def findDuplicate(self, nums):
        slow = fast = 0
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]
            if slow == fast:
                break
        slow = 0
        while slow != fast:
            slow = nums[slow]
            fast = nums[fast]
        return slow
```
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

## 🧩 题目概述

验证一个 9x9 的数独是否合法：
- 每一**行**没有重复数字
- 每一**列**没有重复数字
- 每个 **3x3 格子**没有重复数字
- `.` 表示空格，忽略不计

---

## 💡 核心思路

用三组 `set` 分别记录每行、每列、每个 3x3 格子出现过的数字，遍历一次，发现重复就返回 `False`。

```
row[i]   → 第 i 行出现过的数字
col[j]   → 第 j 列出现过的数字
cell[k]  → 第 k 个 3x3 格子出现过的数字
```

---

## 🔑 最难的部分：3x3 格子索引

把 9 个 3x3 格子编号 0-8：

```
┌─────┬─────┬─────┐
│  0  │  1  │  2  │
├─────┼─────┼─────┤
│  3  │  4  │  5  │
├─────┼─────┼─────┤
│  6  │  7  │  8  │
└─────┴─────┴─────┘
```

```python
idx = (r // 3) * 3 + (c // 3)
```

| 含义 | 计算 |
|------|------|
| `r // 3` | 在第几行的格子（0, 1, 2） |
| `c // 3` | 在第几列的格子（0, 1, 2） |
| `* 3 + ` | 二维坐标转一维索引 |

举例：`r=4, c=7`
```
r//3 = 1   → 第1行的格子
c//3 = 2   → 第2列的格子
idx = 1*3 + 2 = 5  → 格子5 ✅
```

---

## ✅ 完整代码

```python
class Solution:
    def isValidSudoku(self, board: List[List[str]]) -> bool:
        row  = [set() for _ in range(9)]
        col  = [set() for _ in range(9)]
        cell = [set() for _ in range(9)]

        for r in range(9):
            for c in range(9):
                val = board[r][c]
                if val == '.':
                    continue

                idx = (r // 3) * 3 + (c // 3)

                if val in row[r] or val in col[c] or val in cell[idx]:
                    return False

                row[r].add(val)
                col[c].add(val)
                cell[idx].add(val)

        return True
```

---

## 🧠 关键点总结

| 问题 | 答案 |
|------|------|
| 为什么用 set？ | 快速判断是否重复，O(1) 查找 |
| 为什么先 continue？ | `.` 不需要验证，跳过 |
| 为什么先检查再 add？ | 检查到重复直接返回，add 是确认合法后记录 |
| idx 公式的本质 | 二维坐标 `(r//3, c//3)` 转一维索引 |

---

## 📊 复杂度

| | 复杂度 |
|--|--------|
| 时间 | O(1)，棋盘固定 9x9 = 81 格 |
| 空间 | O(1)，set 最多存 9 个数字 × 27 个 set |
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
# 有效的字母异位词

## 题意

给定两个字符串 `s` 和 `t`，判断 `t` 是否是 `s` 的字母异位词（即字母种类和数量完全相同）。

## 解法一：HashMap 计数

```python
def isAnagram(s, t):
    mp = defaultdict(int)
    for ss in s:
        mp[ss] += 1
    for tt in t:
        mp[tt] -= 1
        if mp[tt] < 0:
            return False
    return sum(mp.values()) == 0
```

遍历 `s` 加计数，遍历 `t` 减计数。中途出现负数说明 `t` 有多余字符，提前返回。最后检查是否全部归零。

## 解法二：Counter 一行

```python
from collections import Counter

def isAnagram(s, t):
    return Counter(s) == Counter(t)
```

## 示例

```
s = "anagram", t = "nagaram"

计数 s: {a:3, n:1, g:1, r:1, m:1}
减去 t: {a:0, n:0, g:0, r:0, m:0}
全部归零 → True
```

```
s = "rat", t = "car"

计数 s: {r:1, a:1, t:1}
减去 t: r→0, a→0, c→-1 < 0 → 提前返回 False
```

## 复杂度

- 时间：O(n)
- 空间：O(1)，最多 26 个字母

## 271 Encode and Decode Strings

## 125 Valid Palindrome

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
## 题目概述

给定一个**已排序**的数组 `numbers` 和目标值 `target`，找到两个数使它们的和等于 `target`，返回它们的下标（1-indexed）。

```
输入：numbers = [2,7,11,15], target = 9
输出：[1,2]（因为 numbers[0] + numbers[1] = 2 + 7 = 9）
```

---

## 为什么双指针是最优解

题目的关键条件是**已排序**，三种解法对比：

| 解法 | 时间 | 空间 | 用到排序了吗 |
|------|------|------|------------|
| HashMap | O(n) | O(n) | ✗ 没用到 |
| Binary Search | O(n log n) | O(1) | ✓ 但时间更慢 |
| 双指针 | O(n) | O(1) | ✓ 且空间最优 |

- HashMap 是 Two Sum I（LeetCode 1）的解法，没利用排序条件，空间 O(n)
- Binary Search 利用了排序但时间更差
- 双指针**既利用了排序又做到了 O(1) 空间**，是本题的最优解

题目 Follow-up 也明确说：要求空间 O(1)，所以双指针是期望解法。

---

## 双指针思路

左指针指最小，右指针指最大，根据当前和调整：

```
和太小 → 需要更大的数 → 左指针右移
和太大 → 需要更小的数 → 右指针左移
刚好等于 → 找到答案
```

---

## 代码

```python
def twoSum(self, numbers, target):
    l, r = 0, len(numbers) - 1
    while l < r:
        cur = numbers[l] + numbers[r]
        if cur == target:
            return [l + 1, r + 1]  # 1-indexed
        elif cur < target:
            l += 1
        else:
            r -= 1
```

---

## 为什么双指针不会漏解

假设答案是 `numbers[i]` 和 `numbers[j]`（i < j）。

- 左指针从 0 往右走，一定会经过 i
- 右指针从末尾往左走，一定会经过 j
- 在到达 i 和 j 之前，不可能跳过它们，因为：
  - 左指针只在和太小时右移（说明当前左值不够大，不是答案）
  - 右指针只在和太大时左移（说明当前右值不够小，不是答案）

所以每次移动都是安全地排除一个不可能的值。

## 复杂度

- **时间**：O(n) — 每个元素最多被访问一次
- **空间**：O(1) — 只用两个指针

## 关键点速记

- 已排序 + 两数之和 → 双指针
- 1-indexed 返回值，别忘了 +1
- 题目保证唯一解，while 循环内一定能 return

## 相关题目

- **LeetCode 1** — Two Sum（无序数组，用 HashMap）
- **LeetCode 15** — 3Sum（排序 + 双指针，外层多一层循环）
- **LeetCode 11** — Container With Most Water（双指针，移动较短的一边）

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

## 题意

给定一组柱子高度，计算能接多少雨水。

---

## 核心思路：单调栈横着算水

维护一个**递减的单调栈**。当遇到比栈顶高的柱子时，说明形成了凹槽，可以接水。每次弹出一个凹槽底部，算那一层的水量。

---

## 每一步在做什么

```
遇到 height[i] >= height[栈顶]：
    弹出栈顶 → 这是凹槽底部 (mid)
    如果栈还有元素：
        左边界 = height[新栈顶]
        右边界 = height[i]
        h = min(左, 右) - mid    ← 水的高度
        w = i - 新栈顶 - 1       ← 水的宽度
        res += h * w
```

三个角色：左边界（新栈顶）、底部（弹出的）、右边界（当前 i）。

---

## 手算示例

```
height: [0, 2, 0, 3, 1, 0, 1, 3, 2, 1]
         0  1  2  3  4  5  6  7  8  9

i=0: push 0, 栈 [0]
i=1: 弹出 0(底部=0), 栈空无左边界, push 1, 栈 [1]
i=2: push 2, 栈 [1,2]
i=3: 弹出 2(底部=0), 左=2 右=3, h=2 w=1, +2, res=2
     弹出 1(底部=2), 栈空无左边界
     push 3, 栈 [3]
i=4: push 4, 栈 [3,4]
i=5: push 5, 栈 [3,4,5]
i=6: 弹出 5(底部=0), 左=1 右=1, h=1 w=1, +1, res=3
     弹出 4(底部=1), 左=3 右=1, h=0 w=2, +0, res=3
     push 6, 栈 [3,6]
i=7: 弹出 6(底部=1), 左=3 右=3, h=2 w=3, +6, res=9
     弹出 3(底部=3), 栈空无左边界
     push 7, 栈 [7]
i=8: push 8, 栈 [7,8]
i=9: push 9, 栈 [7,8,9]

最终 res = 9
```

---

## 代码

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        stack = []
        res = 0

        for i in range(len(height)):
            while stack and height[i] >= height[stack[-1]]:
                index = stack.pop()
                mid = height[index]
                if stack:
                    right = height[i]
                    left = height[stack[-1]]
                    h = min(right, left) - mid
                    w = i - stack[-1] - 1
                    res += h * w
            stack.append(i)

        return res    # 注意缩进在 for 循环外面
```

---

## 为什么用单调栈

- 栈维护递减序列，保证弹出时能找到左边界
- 每个元素最多入栈一次、出栈一次，时间 O(n)
- 横着一层一层算水，和双指针竖着算是两种不同的视角

---

## 易错点

1. `return res` 要在 for 循环外面，缩进错了会第一轮就返回
2. 弹出底部后要检查 `if stack`，栈空说明没有左边界，不能接水
3. 栈里存的是**下标**不是高度，因为需要算宽度 `i - stack[-1] - 1`
## 121 Best Time to Buy and Sell Stock
## 3 Longest Substring Without Repeating Characters
## 424 Longest Repeating Character Replacement
## 核心公式

```
窗口长度 - 窗口内最多字符次数 ≤ k → 合法
```

含义：保留出现最多的字符，剩下的全部替换。`max_v` 就是"你决定保留哪个字符"。

```
窗口: A A B A B
max_v = 3 (A出现3次)

保留: A A _ A _   ← 保留所有A
替换: _ _ B _ B   ← 需要替换2个，2 ≤ k 就合法
```

选出现最多的保留，需要替换的最少，窗口能撑到最大。

## 最优解法（if + max_v 只增不减）

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        hashmap = defaultdict(int)
        res = 0
        l = 0
        max_v = 0
        for r in range(len(s)):
            hashmap[s[r]] += 1
            max_v = max(max_v, hashmap[s[r]])
            if r - l + 1 - max_v > k:
                hashmap[s[l]] -= 1
                l += 1
            res = max(res, r - l + 1)
        return res
```

## 为什么用 if 而不是 while

`max_v` 只增不减，所以窗口永远不会缩小，每次最多收缩一步。

```
while 版本窗口大小变化: 3 → 4 → 3 → 2 → 4 → 5  （会缩小再变大）
if 版本窗口大小变化:    3 → 4 → 4 → 4 → 4 → 5  （只扩大或平移）
```

窗口不缩小，只是往右滑，等到遇到更好的情况才扩大。最终 `res` 一样。

## 为什么 max_v 不需要精确维护

收缩时 `max_v` 可能偏大（没更新），但这只会让窗口保持原大小不收缩，不会让它变得更大。`res` 只在 `max_v` 真正变大时才更新，那时 `max_v` 是准确的。

```
max_v 准确 → 窗口可能扩大 → 更新 res ✓
max_v 偏大 → 窗口不收缩，保持原大小 → res 不变，无害
max_v 偏小 → 不可能，因为只取 max 不减少
```

## while 版本（也正确，但稍慢）

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        hashmap = defaultdict(int)
        res = 0
        l = 0
        for r in range(len(s)):
            hashmap[s[r]] += 1
            while r - l + 1 - max(hashmap.values()) > k:
                hashmap[s[l]] -= 1
                l += 1
            res = max(res, r - l + 1)
        return res
```

区别：每次收缩要调用 `max(hashmap.values())`（O(26)），在 while 里可能调用多次。

## 两种写法对比

| | if 版本 | while 版本 |
|---|---|---|
| 时间 | O(n)，每步 O(1) | O(n)，每步 O(26) |
| 窗口行为 | 只扩大或平移 | 会缩小再扩大 |
| max_v | 只增不减，O(1) 维护 | 每次重新算，O(26) |
| 面试推荐 | ✓ 更简洁高效 | 也可以，但多解释 |

## 567 Permutation in String
## 76 Minimum Window Substring
# 76. Minimum Window Substring — 笔记

## 题意

给定字符串 s 和 t，找 s 中包含 t 所有字符（含重复）的最短子串。

---

## 核心思路：滑动窗口

两个指针，right 扩窗口收字符，left 缩窗口找最短。

```
right 每走一步：
    加入 s[right]，更新窗口计数
    
    while 窗口满足条件（have == need）：
        更新最短答案
        移出 s[left]，更新计数
        left 右移
```

---

## 需要的数据结构

| 变量 | 作用 |
|------|------|
| `hashmap` | t 中每个字符需要出现的次数 |
| `window` | 当前窗口中每个字符的计数 |
| `need` | 需要满足的字符种类数（`len(hashmap)`） |
| `have` | 已经满足的字符种类数 |

为什么需要 `have` 和 `need`？没有它们就要每次遍历整个 map 判断是否满足，有了就是 O(1) 判断。

---

## have 什么时候变化

- **加 1**：加入字符后 `window[c] == hashmap[c]`（刚好达标）
- **减 1**：移出字符后 `window[c] < hashmap[c]`（刚好不达标）

关键是用 `==` 和 `<` 而不是 `>=` 和 `<=`，保证 have 只在临界点变化，不会重复加减。

---

## 代码

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        hashmap = {}
        for c in t:
            hashmap[c] = hashmap.get(c, 0) + 1

        window = {}
        have, need = 0, len(hashmap)
        start, res_len = -1, float('inf')
        l = 0

        for r in range(len(s)):
            char = s[r]
            window[char] = window.get(char, 0) + 1
            if char in hashmap and window[char] == hashmap[char]:
                have += 1

            while have == need:
                if r - l + 1 < res_len:
                    start = l
                    res_len = r - l + 1
                window[s[l]] -= 1
                if s[l] in hashmap and window[s[l]] < hashmap[s[l]]:
                    have -= 1
                l += 1

        return s[start:start + res_len] if res_len != float('inf') else ''
```

---

## 易错点

1. `hashmap` 是字符计数，不是 `{char: index}`
2. `res_len` 初始值是 `float('inf')`（找最小），不是 `float('-inf')`
3. 缩窗口时 `l += 1`（往右移），不是 `l -= 1`
4. 判断用 `==` 和 `<`，不是 `>=` 和 `<=`，避免 have 重复变化
## 239 Sliding Window Maximum
**考点：单调递减队列（Deque）**

---

### 为什么不用堆？

最大堆无法高效删除过期元素（不支持任意位置删除），懒删除可以绕过这个问题，但时间复杂度更差：

| | 最大堆 + 懒删除 | 单调递减 Deque |
|--|----------------|----------------|
| 时间复杂度 | O(n log n) | O(n) |
| 空间复杂度 | O(n) | O(k) |
| 原因 | 每次 push O(log n) | 每个元素最多入队出队各一次 |

---

### 堆解法（可通过，非最优）

```python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        heap = []
        output = []
        for i in range(len(nums)):
            heapq.heappush(heap, (-nums[i], i))
            if i >= k - 1:
                while heap[0][1] <= i - k:  # 队头过期就懒删除
                    heapq.heappop(heap)
                output.append(-heap[0][0])
        return output
```

存 `(-nums[i], i)` 是因为 Python 只有最小堆，取负值模拟最大堆。

---

### Deque 最优解

维护一个存 index 的双端队列，保证对应值从左到右单调递减，队头始终是窗口最大值。

每次滑动三步：
1. 新元素入队前，pop 掉队尾所有比它小的（它们永远不可能是最大值）
2. 队头过期（index <= i - k）则从队头移除
3. 窗口形成后（i >= k-1），队头即为答案

```python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        dq = collections.deque()  # 存 index，对应值单调递减
        output = []

        for i in range(len(nums)):
            # 1. pop 掉队尾所有比新元素小的
            while dq and nums[dq[-1]] < nums[i]:
                dq.pop()
            dq.append(i)

            # 2. 队头过期则移除
            if dq[0] <= i - k:
                dq.popleft()

            # 3. 窗口形成后记录答案
            if i >= k - 1:
                output.append(nums[dq[0]])

        return output
```

**为什么队头永远是最大值？** 队列单调递减，队头值最大；只要它没过期，它就一直在窗口内。

---
## 20 Valid Parentheses
# 有效的括号 — 完整讲解

## 题目描述

给定一个只包含 `(`、`)`、`{`、`}`、`[`、`]` 的字符串 `s`，判断字符串是否有效。

有效条件：
- 开括号必须由**同类型**闭括号闭合
- 开括号必须以**正确的顺序**闭合
- 每个闭括号都有对应的开括号

---

## 代码

```python
class Solution:
    def isValid(self, s: str) -> bool:
        mp = {'}': '{',
              ']': '[',
              ')': '('}
        stack = []  # 存放开括号

        for i in range(len(s)):
            if s[i] in mp:                        # 是闭括号
                if stack and mp[s[i]] == stack[-1]:
                    stack.pop()
                else:
                    return False
            else:                                 # 是开括号
                stack.append(s[i])

        return len(stack) == 0
```

---

## 核心思路：栈

括号匹配天然符合**后进先出**的特性：

> 最后遇到的开括号，应该最先被闭合。

所以用栈来存放还没被匹配的开括号，每次遇到闭括号就检查栈顶是不是对应的开括号。

---

## 哈希表的作用

```python
mp = {'}': '{', ']': '[', ')': '('}
```

用闭括号作为 key，对应的开括号作为 value，这样遇到闭括号时可以 O(1) 查到它期望匹配的开括号是什么，不需要写大量 if-else。

---

## 逐步走一遍例子

输入：`s = "{[]}"`

| 步骤 | 字符 | 类型 | 操作 | 栈状态 |
|------|------|------|------|--------|
| 1 | `{` | 开括号 | 入栈 | `[{]` |
| 2 | `[` | 开括号 | 入栈 | `[{, []` |
| 3 | `]` | 闭括号 | mp[`]`]=`[`，栈顶是`[`，匹配 → pop | `[{]` |
| 4 | `}` | 闭括号 | mp[`}`]=`{`，栈顶是`{`，匹配 → pop | `[]` |

栈为空，返回 `True` ✓

---

## 再走一个失败的例子

输入：`s = "([)]"`

| 步骤 | 字符 | 类型 | 操作 | 栈状态 |
|------|------|------|------|--------|
| 1 | `(` | 开括号 | 入栈 | `[(]` |
| 2 | `[` | 开括号 | 入栈 | `[(, []` |
| 3 | `)` | 闭括号 | mp[`)`]=`(`，栈顶是`[`，**不匹配** → return False | — |

返回 `False` ✓

---

## 两个边界条件

### 1. 遇到闭括号但栈为空

```python
if stack and mp[s[i]] == stack[-1]:
```

`stack` 为空时，`stack and ...` 短路求值直接为 `False`，进入 `else` 返回 `False`。

如果不判断 `stack`，直接访问 `stack[-1]` 会报 `IndexError`。

### 2. 全是开括号

```python
return len(stack) == 0
```

比如输入 `"((("`, 循环结束后栈里还有 3 个开括号，返回 `False`。  
必须检查栈是否为空，不能直接 `return True`。

---

## 易错点总结

| 错误 | 原因 | 正确做法 |
|------|------|----------|
| 不判断 `stack` 是否为空就访问 `stack[-1]` | 空栈访问会报错 | 加 `stack and` 做短路判断 |
| 循环结束直接 `return True` | 忽略了未闭合的开括号 | `return len(stack) == 0` |
| 用开括号做 key | 遇到闭括号时无法直接查询 | 用闭括号做 key，开括号做 value |

---

## 复杂度分析

| | 复杂度 |
|---|---|
| 时间 | O(n)，遍历一次字符串 |
| 空间 | O(n)，最坏情况全是开括号全部入栈 |

## 155 Min Stack

## 题目在考什么

不只是设计一个 stack，关键是 `getMin()` 要 O(1)。普通 stack 找最小值需要遍历 O(n)，所以需要额外的结构来"记住"每个状态下的最小值。

## 核心思路

用两个栈：主栈存数据，最小栈的栈顶始终是当前的最小值。

```
操作       主栈        最小栈
push(5)   [5]         [5]        最小是5
push(3)   [5,3]       [5,3]      最小变成3
push(7)   [5,3,7]     [5,3,3]    最小还是3
pop()     [5,3]       [5,3]      最小还是3
pop()     [5]         [5]        最小变回5
```

两个栈同步 push 和 pop，`getMin()` 直接看最小栈栈顶，O(1)。

## 完整代码

```python
class MinStack:

    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)
        if self.min_stack:
            val = min(self.min_stack[-1], val)
        self.min_stack.append(val)

    def pop(self) -> None:
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

## push 的关键逻辑

```python
if self.min_stack:
    val = min(self.min_stack[-1], val)
self.min_stack.append(val)
```

新的最小值 = min(之前的最小值, 新元素)。如果最小栈为空（第一个元素），直接 push。

## 为什么不能用 hashset

```
push(3) → {3}
push(3) → {3}    set 不存重复
pop()   → {3}?   3 还在不在？不知道
```

set 不能存重复元素，也不知道顺序，无法跟踪栈的状态变化。

## 复杂度

| 操作 | 时间 | 空间 |
|------|------|------|
| push | O(1) | O(1) |
| pop | O(1) | O(1) |
| top | O(1) | O(1) |
| getMin | O(1) | O(1) |
| 总空间 | | O(n)（两个栈） |

## 150 Evaluate Reverse Polish Notation
## 22 Generate Parentheses
## 739 Daily Temperatures
## 6. Daily Temperatures（每日温度）

**思路：单调递减栈**

找"右边第一个比我大的元素"，用单调递减栈。

栈里存的是**还没找到答案的那些天的 index**，从栈底到栈顶温度单调递减。

遍历到第 `i` 天时：
- 新温度 **>** 栈顶温度 → 栈顶找到答案，pop，记录天数差 `i - index`
- 重复上面直到栈空或栈顶 >= 新温度
- 无条件将 `i` 入栈

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        stack = []
        res = [0] * len(temperatures)
        for i in range(len(temperatures)):
            while stack and temperatures[stack[-1]] < temperatures[i]:
                index = stack.pop()
                res[index] = i - index
            stack.append(i)  # 每一天都入栈，无条件执行
        return res
```

**常见 Bug：`else` 接在 `while` 上**

```python
# ❌ 错误写法
while stack and temperatures[stack[-1]] < temperatures[i]:
    index = stack.pop()
    res[index] = i - index
else:
    stack.append(i)  # while-else：循环结束后总会执行，包括 pop 之后
                     # 导致刚 pop 掉的 index 又被重新 push 回去
```

`while-else` 中的 `else` 是"循环正常结束后执行"，不是"循环没有执行才执行"，所以每次 pop 完之后还会额外 append，逻辑错误。

| | 时间复杂度 | 空间复杂度 |
|--|-----------|-----------|
| 暴力双循环 | O(n²) | O(1) |
| 单调栈（最优） | O(n) | O(n) |

---
## 853 Car Fleet
## 84 Largest Rectangle in Histogram

## 704 Binary Search
## 74 Search a 2D Matrix
## 题意

在一个每行递增且下一行首元素大于上一行末元素的矩阵中，搜索目标值。

---

## 核心思路：两次二分

1. **第一次二分**：找到 target 所在的行
2. **第二次二分**：在该行中找 target

---

## 第一次二分怎么定位行

检查 target 是否落在 `matrix[mid][0] <= target <= matrix[mid][-1]` 范围内：

- 是 → 找到目标行，break
- `matrix[mid][-1] < target` → target 在更下面的行，`l = mid + 1`
- `matrix[mid][0] > target` → target 在更上面的行，`r = mid - 1`
- 循环正常结束（没 break）→ 不存在目标行，返回 False

---

## while...else 语法

```python
while l <= r:
    ...
    if 找到了:
        break
    ...
else:
    return False  # 循环没有被 break 打断时执行
```

`else` 块只在循环**正常结束**时执行，被 break 跳出时不执行。用它可以优雅地处理"没找到"的情况。

---

## 代码

```python
class Solution:
    def searchMatrix(self, matrix, target):
        m = len(matrix)
        n = len(matrix[0])

        l, r = 0, m - 1
        while l <= r:
            mid = (r + l) // 2
            if matrix[mid][0] <= target <= matrix[mid][-1]:
                row = mid
                break
            if matrix[mid][-1] > target:
                r = mid - 1
            else:
                l = mid + 1
        else:
            return False

        l, r = 0, n - 1
        while l <= r:
            mid = (r + l) // 2
            if matrix[row][mid] == target:
                return True
            if matrix[row][mid] > target:
                r = mid - 1
            else:
                l = mid + 1

        return False
```

---

## 易错点

1. 第一次二分找不到目标行时要返回 False，否则 `matrix[row]` 会报错（row 未定义）
2. 用 `while...else` 或在第二次二分前加越界检查都可以处理这个边界
3. 时间复杂度 O(log m + log n) = O(log(m×n))
## 875 Koko Eating Bananas
## 153 Find Minimum in Rotated Sorted Array
## 33 Search in Rotated Sorted Array
## 核心思路

旋转数组用 `nums[-1]` 分成两段，二分时判断 target 和 mid 分别在哪一段来决定搜索方向。

```
[4, 5, 6, 7, 0, 1, 2]
 ↑---------↑  ↑-----↑
 左段(>nums[-1])  右段(≤nums[-1])
```

## is_blue 函数

`is_blue(mid)` 返回 True = **答案 ≤ mid**（target 在 mid 左边或就是 mid）。

```python
def is_blue(mid):
    if nums[mid] > nums[-1]:           # mid 在左段
        return target > nums[-1] and nums[mid] >= target
    else:                               # mid 在右段
        return target > nums[-1] or nums[mid] >= target
```

注意：必须用 `>=` 不能用 `>`，否则 `nums[mid] == target` 时返回 False，会跳过答案。

### 两个分支的含义

**mid 在左段 (`nums[mid] > nums[-1]`)：**

```
[4, 5, 6, 7, 0, 1, 2]
       ↑mid

target 在 mid 左边（含 mid）的条件：
  target 也在左段 (target > nums[-1])
  并且 target ≤ nums[mid]
```

**mid 在右段 (`nums[mid] ≤ nums[-1]`)：**

```
[4, 5, 6, 7, 0, 1, 2]
                ↑mid

target 在 mid 左边（含 mid）的条件：
  target 在左段 (target > nums[-1])  ← 左段整体在 mid 左边
  或者 target ≤ nums[mid]            ← 在右段但不超过 mid
```

## 完整代码

```python
class Solution:
    def search(self, nums, target):
        def is_blue(mid):
            if nums[mid] > nums[-1]:
                return target > nums[-1] and nums[mid] >= target
            else:
                return target > nums[-1] or nums[mid] >= target

        l = 0
        r = len(nums)
        while l < r:                    # 左闭右开 [l, r)
            mid = l + (r - l) // 2
            if is_blue(mid):
                r = mid
            else:
                l = mid + 1

        return l if l < len(nums) and nums[l] == target else -1
```

## 易错点

**1. `>=` vs `>`**

```python
nums[mid] >= target   # 正确：mid == target 时返回 True，r = mid 保留答案
nums[mid] > target    # 错误：mid == target 时返回 False，l = mid+1 跳过答案
```

**2. 返回值检查边界**

```python
# 错误：r 可能等于 len(nums)，越界
return nums[r] if nums[r] == target else -1

# 正确：检查边界
return l if l < len(nums) and nums[l] == target else -1
```

## 验证

`nums = [4, 5, 6, 7, 0, 1, 2]`，`target = 5`，`mid = 3`（值为 7）：

```
nums[mid]=7 > nums[-1]=2 → 左段
return 5 > 2 and 7 >= 5 → True
r = mid → 往左搜 ✓
```

`nums = [1]`，`target = 1`，`mid = 0`（值为 1）：

```
nums[mid]=1 > nums[-1]=1? 否 → 右段
return 1 > 1 or 1 >= 1 → True
r = mid = 0 → l == r = 0
nums[0] == 1 → 返回 0 ✓
```

## 981 Time Based Key Value Store
## 4 Median of Two Sorted Arrays
## 题意

给定两个有序数组，找它们合并后的中位数，要求时间复杂度 O(log(m+n))。

---

## 核心思路：二分切割

不需要真的合并数组。把两个数组各切一刀，使得左半边总共有 `(m+n+1)//2` 个元素，且左半边所有元素 ≤ 右半边所有元素。

### 为什么在短数组上二分？

对短数组（长度 m）二分，切割位置 `m_i` 确定后，长数组的切割位置 `n_i = med_len - m_i` 也就确定了。在短数组上二分保证 `n_i` 不会越界。

---

## 切割后的四个值

```
nums1:  [..., left_1 | right_1, ...]
nums2:  [..., left_2 | right_2, ...]
```

| 变量 | 含义 | 越界处理 |
|------|------|----------|
| `left_1 = nums1[m_i - 1]` | nums1 左半边最大值 | `m_i == 0` 时取 `-inf` |
| `left_2 = nums2[n_i - 1]` | nums2 左半边最大值 | `n_i == 0` 时取 `-inf` |
| `right_1 = nums1[m_i]` | nums1 右半边最小值 | `m_i == m` 时取 `inf` |
| `right_2 = nums2[n_i]` | nums2 右半边最小值 | `n_i == n` 时取 `inf` |

---

## 二分方向怎么记

只需要交叉比较：

- `left_1 > right_2`：nums1 给左半边太多了 → **r = m_i - 1**（往左缩）
- `left_2 > right_1`：nums1 给左半边太少了 → **l = m_i + 1**（往右扩）
- 否则：找到正确切割位置

记忆：**谁大了就缩谁**。left_1 大了，缩 nums1 的右边界。

---

## 奇偶判断

- **奇数**：中位数就一个，返回 `max(left_1, left_2)`
- **偶数**：中位数是两个的平均，返回 `(max(left_1, left_2) + min(right_1, right_2)) / 2`

注意 `(m+n) % 2 == 1` 是奇数，`== 0` 是偶数，别搞反。

---

## 代码

```python
class Solution:
    def findMedianSortedArrays(self, nums1, nums2):
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1
        m, n = len(nums1), len(nums2)
        med_len = (m + n + 1) // 2

        l, r = 0, m
        while l <= r:
            m_i = (l + r) // 2
            n_i = med_len - m_i

            left_1 = nums1[m_i - 1] if m_i >= 1 else float('-inf')
            left_2 = nums2[n_i - 1] if n_i >= 1 else float('-inf')
            right_1 = nums1[m_i] if m_i < m else float('inf')
            right_2 = nums2[n_i] if n_i < n else float('inf')

            if left_1 > right_2:
                r = m_i - 1
            elif left_2 > right_1:
                l = m_i + 1
            else:
                if (m + n) % 2:
                    return max(left_1, left_2)
                else:
                    return (max(left_1, left_2) + min(right_1, right_2)) / 2
```

---

## 易错点

1. **始终在短数组上二分**，先 swap 保证 `len(nums1) <= len(nums2)`
2. **二分方向**：`left_1 > right_2` 时缩右边界（r），不是扩左边界
3. **奇偶搞反**：奇数返回一个值，偶数才求平均
4. **越界处理**：四个边界值都要判断，用 `inf` / `-inf` 兜底

---

## 复杂度

- 时间：O(log(min(m, n)))，只在短数组上二分
- 空间：O(1)
## 206 Reverse Linked List
## 21 Merge Two Sorted Lists
## 143 Reorder List
## 题目核心

给定链表 `L0 → L1 → ... → Ln-1 → Ln`，重排为 `L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → ...`

就是把链表的头和尾交替拼接。

示例：`1 → 2 → 3 → 4 → 5` 变成 `1 → 5 → 2 → 4 → 3`

---

## 整体思路：四步走

不能用数组（要求原地操作），所以拆成四个经典链表操作：

```
原始：1 → 2 → 3 → 4 → 5

第一步 找中点：  [1 → 2 → 3] [4 → 5]
第二步 断开：    1 → 2 → 3    4 → 5
第三步 翻转后半：1 → 2 → 3    5 → 4
第四步 交替连接：1 → 5 → 2 → 4 → 3
```

---

## 第一步：快慢指针找中点

### 技巧：快慢指针

slow 走一步，fast 走两步。fast 到底时，slow 正好在中间。

```python
slow = head
fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

### 为什么 `while fast and fast.next`

两个条件缺一不可：

- `fast`：防止偶数长度时 fast 已经是 None
- `fast.next`：防止奇数长度时 fast 在最后一个节点，再走就越界

### slow 停在哪

| 链表长度 | 示例 | slow 停在 |
|----------|------|-----------|
| 奇数 (5) | 1→2→3→4→5 | 3（正中间） |
| 偶数 (4) | 1→2→3→4 | 2（中间偏左） |

两种情况下，`slow.next` 都是后半段的起点。

---

## 第二步：断开

```python
second_part = slow.next
slow.next = None
```

把链表一刀切成两半：

```
前半：1 → 2 → 3 → None
后半：4 → 5 → None
```

`slow.next = None` 很关键，不断开的话前半段还连着后半段，最后会成环。

---

## 第三步：翻转后半段

### 技巧：三指针翻转

```python
prev = None
while second_part:
    nxt = second_part.next      # 先存下一个
    second_part.next = prev     # 指针反转
    prev = second_part          # prev 前进
    second_part = nxt           # curr 前进
```

### 逐步模拟

翻转 `4 → 5 → None`：

```
初始：prev=None, curr=4

第一轮：
  nxt = 5
  4.next = None    (4 → None)
  prev = 4
  curr = 5

第二轮：
  nxt = None
  5.next = 4       (5 → 4)
  prev = 5
  curr = None

结束：prev = 5，即翻转后的头 → 5 → 4 → None
```

### 为什么需要 nxt 变量

因为 `second_part.next = prev` 会把原来的 next 覆盖掉。如果不提前存，就找不到下一个节点了，链表就断了。

---

## 第四步：交替连接

### 技巧：双指针交替穿插

```python
first = head        # 前半段：1 → 2 → 3
second = prev       # 后半段：5 → 4

while second:
    tmp1 = first.next       # 存好下一步
    tmp2 = second.next
    first.next = second     # first 指向 second
    second.next = tmp1      # second 指向 first 的下一个
    first = tmp1            # 两个指针各前进一步
    second = tmp2
```

### 逐步模拟

```
初始：
  first: 1 → 2 → 3
  second: 5 → 4

第一轮：
  tmp1 = 2, tmp2 = 4
  1.next = 5      → 1 → 5
  5.next = 2      → 1 → 5 → 2 → 3
  first = 2, second = 4

第二轮：
  tmp1 = 3, tmp2 = None
  2.next = 4      → 1 → 5 → 2 → 4
  4.next = 3      → 1 → 5 → 2 → 4 → 3
  first = 3, second = None

second = None → 结束

结果：1 → 5 → 2 → 4 → 3
```

### 为什么 `while second` 而不是 `while first`

后半段长度 ≤ 前半段。奇数长度时前半段多一个节点（中间那个），后半段先走完，所以用 second 控制循环。

### 为什么需要 tmp1 和 tmp2

和翻转一样的道理：修改 next 指针会破坏原有链接，必须先把后续节点存起来。

---

## 完整代码

```python
class Solution:
    def reorderList(self, head: Optional[ListNode]) -> None:
        # 第一步：快慢指针找中点
        slow = head
        fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # 第二步：断开
        second_part = slow.next
        slow.next = None

        # 第三步：翻转后半段
        prev = None
        while second_part:
            nxt = second_part.next
            second_part.next = prev
            prev = second_part
            second_part = nxt

        # 第四步：交替连接
        first = head
        second = prev
        while second:
            tmp1 = first.next
            tmp2 = second.next
            first.next = second
            second.next = tmp1
            first = tmp1
            second = tmp2
```

---

## 链表技巧总结

这道题集合了四个经典链表技巧，很多题都会复用：

| 技巧 | 本题用途 | 其他常见题目 |
|------|---------|-------------|
| 快慢指针 | 找中点 | 环检测、找环入口、判断回文链表 |
| 断开链表 | 分成两半 | 合并排序链表、分割链表 |
| 三指针翻转 | 翻转后半段 | 反转链表、K 个一组翻转 |
| 双指针穿插 | 交替连接 | 合并两个有序链表 |

### 通用注意点

- **改 next 之前先存**：任何时候修改 `.next`，都要先把原来的下一个节点存到临时变量里
- **断开防成环**：拆分链表时一定要把前半段的尾部指向 None
- **循环条件想清楚**：用长度短的那一段控制循环，防止空指针

---

## 复杂度

- **时间**：O(n)，每一步都是 O(n)，总共四步
- **空间**：O(1)，只用了几个指针变量，原地操作
## 19 Remove Nth Node From End
## 2 Add Two Numbers
## 141 Linked List Cycle
## 138 Copy List With Random Pointer
## 23 Merge k Sorted Lists

## 核心思路

堆里始终维护 k 个节点（每个链表一个），每次 pop 最小的接到结果上，再把它的 next push 进去。

```
lists: 1→4→5, 1→3→4, 2→6

堆里只有 3 个节点（k=3）:
[1, 1, 2]
 ↓  ↓  ↓
 4  3  6   ← 在链表里等着，不在堆里
 ↓  ↓
 5  4
```

## 完整代码

```python
class Solution:
    def mergeKLists(self, lists):
        heap = []
        for i, l in enumerate(lists):
            if l:                          # 链表可能为空
                heapq.heappush(heap, (l.val, i, l))

        dummy = cur = ListNode(0)
        while heap:
            val, i, node = heapq.heappop(heap)
            cur.next = node
            cur = cur.next
            if node.next:
                heapq.heappush(heap, (node.next.val, i, node.next))

        return dummy.next
```

## 走一遍示例

```
lists: 1→4→5, 1→3→4, 2→6

初始堆: [1, 1, 2]（三个链表的头）

pop 1 → 结果: 1→       push 4    堆: [1, 2, 4]
pop 1 → 结果: 1→1→     push 3    堆: [2, 3, 4]
pop 2 → 结果: 1→1→2→   push 6    堆: [3, 4, 6]
pop 3 → 结果: 1→1→2→3→ push 4    堆: [4, 4, 6]
pop 4 → 结果: ...→4→   push 5    堆: [4, 5, 6]
pop 4 → 结果: ...→4→4→ 无next    堆: [5, 6]
pop 5 → 结果: ...→5→   无next    堆: [6]
pop 6 → 结果: ...→6    堆空，结束

最终: 1→1→2→3→4→4→5→6
```

## 为什么堆里要放 i

当两个节点 val 相同时，Python 会比较第二个元素来打破平局。ListNode 不能比较，放个 `i` 作为 tiebreaker 就不会报错。

```python
heapq.heappush(heap, (l.val, i, l))
                       ↑     ↑  ↑
                      排序  平局 实际节点
```

## 易错点

push 之前要检查链表是否为空：

```python
# 错误：lists 里可能有 None
for i, l in enumerate(lists):
    heapq.heappush(heap, (l.val, i, l))    # None.val 报错

# 正确
for i, l in enumerate(lists):
    if l:
        heapq.heappush(heap, (l.val, i, l))
```

## 三种方法对比

| 方法 | 时间 | 空间 | 特点 |
|------|------|------|------|
| 逐一合并 | O(nk) | O(1) | 最慢，重复遍历多 |
| 两两合并/分治 | O(n log k) | O(log k) | 每轮处理 n 个，共 log k 轮 |
| 最小堆 | O(n log k) | O(k) | 最常用，代码最简洁 |

## 为什么是 O(n log k) 不是 O(n log n)

堆大小始终 ≤ k（每个链表只有一个代表在堆里），所以每次 push/pop 是 O(log k)。总共 n 个节点，每个进出堆一次，所以 O(n log k)。

## 226 Invert Binary Tree
## 104 Maximum Depth of Binary Tree
## 543 Diameter of Binary Tree
## 110 Balanced Binary Tree
## 100 Same Tree
## 572 Subtree of Another Tree
## 235 LCA of BST

## 🧩 题目理解

给一棵树和两个节点 `p`、`q`，找它们的**最近公共祖先**。

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

| p | q | LCA | 原因 |
|---|---|-----|------|
| 6 | 4 | 5   | 分别在 5 的左右两侧 |
| 5 | 4 | 5   | 5 本身就是 4 的祖先 |
| 6 | 7 | 5   | 分别在 5 的左右两侧 |

---

## 💡 两种情况

```
情况1：p 和 q 分别在某节点的左右两侧  → 该节点是 LCA
情况2：p 本身就是 q 的祖先            → p 是 LCA
```

---

## 🌲 版本一：普通二叉树（后序遍历 + 回溯）

### 思路
不知道 p 和 q 在哪 → 左右都要搜 → 靠返回值往上传结果

### 返回值逻辑
```
左右都不为空  → 当前节点是 LCA，返回自己
只有左不为空  → 返回左
只有右不为空  → 返回右
自己是 p 或 q → 返回自己，不再往下找
```

### 代码
```python
def lowestCommonAncestor(self, root, p, q):
    if not root:
        return None
    if root.val == p.val or root.val == q.val:
        return root                              # 找到目标，直接返回
    
    left = self.lowestCommonAncestor(root.left, p, q)
    right = self.lowestCommonAncestor(root.right, p, q)
    
    if left and right:                           # 左右都找到 → 当前节点是 LCA
        return root
    return left or right                         # 哪边不为空返回哪边
```

### 为什么有回溯？
```python
left = dfs(root.left)    # 递归下去
right = dfs(root.right)  # 递归下去
if left and right:        # ← 回来之后再判断，这就是回溯
    return root
```
**回溯 = 递归返回后还需要做额外处理**，这题两边都搜完才能判断 LCA 在哪。

---

## 🌳 版本二：BST（直接比大小）

### 思路
BST 特性：左子树 < 根 < 右子树  
→ 直接比大小就知道往哪走，不需要两边都搜

### 三种情况（else 全包）
```
p 和 q 都小于 root  → 往左走
p 和 q 都大于 root  → 往右走
其他所有情况         → else，返回 root
```

`else` 包含的情况：
```
root == p           → p 本身就是 LCA
root == q           → q 本身就是 LCA
p 在左，q 在右      → 分叉点，root 是 LCA
```
三种情况答案都是返回 `root`，所以一个 `else` 全包。

### 递归版
```python
def lowestCommonAncestor(self, root, p, q):
    if root.val > p.val and root.val > q.val:
        return self.lowestCommonAncestor(root.left, p, q)
    elif root.val < p.val and root.val < q.val:
        return self.lowestCommonAncestor(root.right, p, q)
    else:
        return root
```

### 迭代版（最简洁）
```python
def lowestCommonAncestor(self, root, p, q):
    cur = root
    while cur:
        if p.val > cur.val and q.val > cur.val:
            cur = cur.right
        elif p.val < cur.val and q.val < cur.val:
            cur = cur.left
        else:
            return cur
```

---

## 🔑 核心对比

| | 普通二叉树 | BST |
|--|-----------|-----|
| 方法 | 后序遍历 | 直接比大小 |
| 需要回溯吗 | ✅ 需要 | ❌ 不需要 |
| 搜索方向 | 左右都搜 | 只走一个方向 |
| 时间复杂度 | O(N) | O(H)，H为树高 |

---

## 🧠 回溯 vs 递归

```
递归 = 函数调用自己
回溯 = 递归返回后，还需要做额外处理
```

```python
# 有回溯（普通二叉树）
left = dfs(root.left)    # 递归
right = dfs(root.right)  # 递归
if left and right:        # ← 回来之后判断，这是回溯
    return root

# 无回溯（BST 递归）
return dfs(root.left)    # 直接 return，不需要回来判断

# 无回溯（BST 迭代）
while cur:               # 完全没有递归
    cur = cur.left/right
```

---

## ⚠️ 易错点

`return left or right` 等价于：
```python
if left and not right:
    return left
elif right and not left:
    return right
else:
    return None   # 两边都为空
```
一行搞定三种情况。

---

## 🔗 相关题目

| 题号 | 题目 | 关联点 |
|------|------|--------|
| LC 236 | LCA of Binary Tree | 普通二叉树版本 |
| LC 235 | LCA of BST | BST 版本 |
| LC 543 | Diameter of Binary Tree | 同样用后序遍历回溯 |
| LC 124 | Binary Tree Maximum Path Sum | 同样用后序遍历回溯 |
## 102 Binary Tree Level Order Traversal
## 199 Binary Tree Right Side View
## 98 Validate BST
## 230 Kth Smallest in BST
## 105 Construct Binary Tree from Preorder and Inorder
## 题意

给定一棵二叉树的前序遍历和中序遍历数组（值唯一），重建这棵二叉树。

---

## 核心洞察

两个数组各司其职：

- **preorder**：告诉你"谁是根"——每段的第一个元素就是当前子树的根
- **inorder**：告诉你"怎么分左右"——根的位置把元素分成左子树和右子树

左子树的节点数（`l_size`）从 inorder 算出来后，又反过来帮你切分 preorder。

---

## 递推过程图示

```
preorder: [3, 9, 20, 15, 7]
inorder:  [9, 3, 15, 20, 7]

第一步：preorder[0] = 3 是根
        inorder 中 3 的位置把数组分成：
        左子树 inorder: [9]      → l_size = 1
        右子树 inorder: [15,20,7]

        对应切分 preorder：
        左子树 preorder: [9]         → preStart+1 开始，长度 l_size
        右子树 preorder: [20,15,7]   → preStart+l_size+1 开始

第二步：对左右子树递归，重复同样的过程
```

---

## 思考框架

| 问题 | 回答 |
|------|------|
| 每一步在做什么？ | 从 preorder 取根，用 inorder 分左右 |
| 什么时候结束？ | `start > end`，区间为空 |
| 怎么划分子问题？ | 用 `l_size = index - start` 切分两个数组 |
| 怎么加速找根的位置？ | 预建 hashmap：`{val: i for i, val in enumerate(inorder)}` |

---

## 代码

```python
class Solution:
    def buildTree(self, preorder, inorder):
        idx_map = {val: i for i, val in enumerate(inorder)}

        def dfs(preStart, start, end):
            if start > end:
                return None

            nVal = preorder[preStart]
            node = TreeNode(val=nVal)

            index = idx_map[nVal]
            l_size = index - start

            node.left = dfs(preStart + 1, start, index - 1)
            node.right = dfs(preStart + l_size + 1, index + 1, end)
            return node

        return dfs(0, 0, len(inorder) - 1)
```

---

## 参数含义

| 参数 | 含义 |
|------|------|
| `preStart` | 当前子树的根在 preorder 中的位置 |
| `start` | 当前子树在 inorder 中的左边界 |
| `end` | 当前子树在 inorder 中的右边界 |

---

## 易错点

1. 终止条件是 `start > end`，不是判断 node 是否为 None（node 还没创建）
2. 右子树的 preStart 是 `preStart + l_size + 1`，要跳过左子树的所有节点
3. 别忘了预建 hashmap，否则每次线性查找会超时
## 124 Binary Tree Maximum Path Sum
# Binary Tree Maximum Path Sum — 解题笔记

## 题目核心

给定一棵非空二叉树，找出任意路径的最大路径和。路径可以从任意节点开始、任意节点结束，不一定经过根节点，节点不能重复使用。

## 关键思路

对于每个节点，考虑**以它为拐点（最高点）**的路径。用后序遍历，对每个节点做两件事：

1. **更新全局答案**：左链 + 当前节点 + 右链（完整路径，不往上传）
2. **向上返回**：当前节点 + max(左链, 右链)（只选一边，给父节点接上）

如果某一边是负数，取 0 不要它。

---

## 树形 DP 五部曲

### 1. dp 数组含义

`dp[node]` = 从该节点出发**往下走**的最大单链和（只走一边）

### 2. 递推公式

```
dp[node] = node.val + max(dp[left], dp[right], 0)
```

全局答案在每个节点处额外计算：

```
max_sum = max(max_sum, node.val + max(dp[left], 0) + max(dp[right], 0))
```

### 3. 初始化

- 叶子节点：`dp[leaf] = leaf.val`
- 空节点：`dp[None] = 0`

### 4. 遍历顺序

后序遍历（左 → 右 → 根），先算子树才能算当前节点

### 5. 举例推导

```
      -10
      /  \
     9    20
         /  \
        15    7
```

| 节点 | dp[left] | dp[right] | dp[node] | 拐点路径和 | max_sum |
|------|----------|-----------|----------|-----------|---------|
| 9    | 0        | 0         | 9        | 9         | 9       |
| 15   | 0        | 0         | 15       | 15        | 15      |
| 7    | 0        | 0         | 7        | 7         | 15      |
| 20   | 15       | 7         | 35       | 42        | 42      |
| -10  | 9        | 35        | 25       | 34        | **42**  |

以节点 20 为拐点：15 + 20 + 7 = 42，即最终答案。

---

## 两行 max 的区别

```python
max_sum[0] = max(max_sum[0], node.val + left + right)  # 第一行
return node.val + max(left, right)                       # 第二行
```

**第一行 — 完整路径（更新全局答案）：**

```
    left
       \
      node  ← 拐点，左+右都取
       /
    right
```

以当前节点为拐点，左右都要，这个值只用来更新答案，不往上传。

**第二行 — 单链（返回给父节点）：**

```
  parent
    |
   node  ← 只能带一边
   / \
 left  right（二选一）
```

路径不能分叉，只能选一边给父节点接上。

---

## 代码：递归 DFS（推荐）

```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        max_sum = [float('-inf')]

        def dfs(node):
            if not node:
                return 0
            left = max(dfs(node.left), 0)
            right = max(dfs(node.right), 0)
            max_sum[0] = max(max_sum[0], node.val + left + right)
            return node.val + max(left, right)

        dfs(root)
        return max_sum[0]
```

## 代码：显式 DP（迭代后序遍历）

```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        # 后序遍历拿到节点顺序
        order = []
        stack = [(root, False)]
        while stack:
            node, visited = stack.pop()
            if not node:
                continue
            if visited:
                order.append(node)
            else:
                stack.append((node, True))
                stack.append((node.right, False))
                stack.append((node.left, False))

        # 从底往上填 dp 表
        dp = {}
        max_sum = float('-inf')

        for node in order:
            left = max(dp.get(node.left, 0), 0)
            right = max(dp.get(node.right, 0), 0)
            max_sum = max(max_sum, node.val + left + right)
            dp[node] = node.val + max(left, right)

        return max_sum
```

---

## 递归 vs 显式 DP 对比

| | 递归 DFS | 显式 DP |
|---|---|---|
| dp 值存在哪 | 递归返回值 | `dp` 字典 |
| 遍历顺序 | 递归天然后序 | 手动后序遍历 |
| 填表方向 | 回溯时自动从底到顶 | 显式从底到顶循环 |
| 核心逻辑 | 完全一样 | 完全一样 |

递归版本更简洁，面试中推荐使用。

---

## 这题为什么是 Hard

**dp 返回值和最终答案不是同一个东西。** dp 存的是单链和（给父节点用），但答案是拐点路径和（左+根+右），需要一个额外的 `max_sum` 在遍历过程中记录。这个"返回值"和"答案"的分离是这道题最容易搞混的地方。

---

## 复杂度

- **时间**：O(n)，每个节点访问一次
- **空间**：O(h)，递归栈深度，h 为树高；显式 DP 为 O(n)
## 297 Serialize and Deserialize Binary Tree
## 核心概念

**序列化（Serialize）**：把内存里的树 → 变成字符串  
**反序列化（Deserialize）**：把字符串 → 还原回原来的树

字符串格式自己定，只要两个函数互相配合能还原就行。

---

## 为什么需要记录 null？

只存节点值，信息不够用。下面两棵树序列化后都是 `"1,2,3"`，无法区分：

```
    1          1
   / \          \
  2   3          2
                  \
                   3
```

**必须用 `N` 记录 null**，才能唯一确定树的结构：
- 左边那棵：`"1,2,3,N,N,N,N"`
- 右边那棵：`"1,N,2,N,3,N,N"`

---

## BFS 解法

### Serialize

用队列一层层读节点，null 也要 append 进队列（不然位置信息会丢失）：

```python
def serialize(self, root: Optional[TreeNode]) -> str:
    store = []
    q = deque([root])
    while q:
        node = q.popleft()
        if node:
            store.append(str(node.val))
            q.append(node.left)   # 不管是不是 null 都 append
            q.append(node.right)
        else:
            store.append("N")     # null 不加孩子
    return ','.join(store)
```

**示例walkthrough：**
```
    1
   / \
  2   3
     / \
    4   5

queue: [1]
弹出 1 → "1"，加入 2, 3
弹出 2 → "2"，加入 N, N
弹出 3 → "3"，加入 4, 5
弹出 N → "N"
弹出 N → "N"
弹出 4 → "4"，加入 N, N
弹出 5 → "5"，加入 N, N
...

结果: "1,2,3,N,N,4,5,N,N,N,N"
```

---

### Deserialize

同样用 BFS，每弹出一个节点，就从 vals 取两个值分配给它的左右孩子：

```python
def deserialize(self, data: str) -> Optional[TreeNode]:
    if not data or data == "N":
        return None
    vals = data.split(',')
    root = TreeNode(int(vals[0]))
    q = deque([root])
    i = 1
    while q:
        node = q.popleft()
        if vals[i] != 'N':
            node.left = TreeNode(int(vals[i]))
            q.append(node.left)
        i += 1
        if vals[i] != 'N':
            node.right = TreeNode(int(vals[i]))
            q.append(node.right)
        i += 1
    return root
```

**关键点**：每弹出一个节点，消费 vals 里的两个值（左孩子 + 右孩子），用 `i` 追踪位置。

---

## 常见坑

| 问题 | 原因 | 修复 |
|------|------|------|
| `ValueError: invalid literal for int() with base 10: 'N'` | 空树 serialize 成 `"N"`，deserialize 时直接 `int("N")` 炸了 | 加 `if not data or data == "N": return None` |
| 两棵不同的树 serialize 结果一样 | 没有记录 null 的位置 | null 也要 append 进队列并记录 `"N"` |
| `join` 后分不清节点边界 | 用了 `''.join()` | 改成 `','.join()` 用逗号分隔 |

---

## 复杂度

| | 时间 | 空间 |
|---|---|---|
| Serialize | O(n) | O(n) |
| Deserialize | O(n) | O(n) |

两个操作都是每个节点恰好访问一次。

---

## 完整代码

```python
from collections import deque
from typing import Optional

class Codec:
    def serialize(self, root: Optional[TreeNode]) -> str:
        store = []
        q = deque([root])
        while q:
            node = q.popleft()
            if node:
                store.append(str(node.val))
                q.append(node.left)
                q.append(node.right)
            else:
                store.append("N")
        return ','.join(store)

    def deserialize(self, data: str) -> Optional[TreeNode]:
        if not data or data == "N":
            return None
        vals = data.split(',')
        root = TreeNode(int(vals[0]))
        q = deque([root])
        i = 1
        while q:
            node = q.popleft()
            if vals[i] != 'N':
                node.left = TreeNode(int(vals[i]))
                q.append(node.left)
            i += 1
            if vals[i] != 'N':
                node.right = TreeNode(int(vals[i]))
                q.append(node.right)
            i += 1
        return root
```
## 208 Implement Trie

## 题目核心

实现一个前缀树，支持三个操作：插入单词、搜索完整单词、判断是否存在某前缀。

---

## Trie 是什么

Trie 是一棵多叉树，每条边代表一个字符，从根到某个节点的路径拼起来就是一个前缀。

插入 "apple" 和 "app" 后的结构：

```
        root
         |
         a
         |
         p
         |
         p  ← is_end=True ("app")
         |
         l
         |
         e  ← is_end=True ("apple")
```

---

## 节点设计

每个节点两样东西：

- `children`：字典，key 是字符，value 是子节点
- `is_end`：标记该节点是否是某个单词的结尾

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False
```

---

## 三个操作的逻辑

### insert — 插入单词

逐字符往下走，没有路就创建新节点，最后标记 `is_end = True`。

```python
def insert(self, word):
    node = self.root
    for w in word:
        if w not in node.children:
            node.children[w] = TrieNode()
        node = node.children[w]
    node.is_end = True
```

### search — 搜索完整单词

逐字符往下走，走不通返回 False，走完了检查 `is_end`。

```python
def search(self, word):
    node = self.root
    for w in word:
        if w not in node.children:
            return False
        node = node.children[w]
    return node.is_end  # 不是 return True！
```

### startsWith — 判断前缀是否存在

逐字符往下走，走不通返回 False，走完了直接返回 True（不需要检查 is_end）。

```python
def startsWith(self, prefix):
    node = self.root
    for w in prefix:
        if w not in node.children:
            return False
        node = node.children[w]
    return True
```

---

## search vs startsWith 的区别

这是这道题最容易犯的错：

| | search("app") | startsWith("app") |
|---|---|---|
| 只插入了 "apple" | **False** | **True** |
| 区别 | 要求 is_end=True | 只要路径存在就行 |

`search` 最后返回 `node.is_end`，`startsWith` 最后返回 `True`。如果 search 也返回 True，那它就和 startsWith 一模一样了，无法区分"完整单词"和"前缀"。

---

## 完整代码

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for w in word:
            if w not in node.children:
                node.children[w] = TrieNode()
            node = node.children[w]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self.root
        for w in word:
            if w not in node.children:
                return False
            node = node.children[w]
        return node.is_end

    def startsWith(self, prefix: str) -> bool:
        node = self.root
        for w in prefix:
            if w not in node.children:
                return False
            node = node.children[w]
        return True
```

---

## 复杂度

| 操作 | 时间 | 空间 |
|------|------|------|
| insert | O(m) | O(m)，m 为单词长度 |
| search | O(m) | O(1) |
| startsWith | O(m) | O(1) |

整棵树的空间最坏 O(总字符数)，但公共前缀会共享节点，实际远小于此。
## 211 Design Add and Search Words
## 212 Word Search II

## 🧩 题目概述

给定一个 `m × n` 的字母矩阵 `board` 和一个单词列表 `words`，找出所有能在矩阵中搜索到的单词。

- 每个单词必须由**相邻格子**（上下左右）的字母构成
- 同一个格子在一个单词中**不能重复使用**

---

## 💡 核心思路

### 为什么用 Trie？

| 方法 | 时间复杂度 | 问题 |
|------|-----------|------|
| 对每个单词单独 DFS | O(W × M × N × 4^L) | 重复遍历公共前缀 |
| **Trie + DFS（最优）** | O(M × N × 4 × 3^(L-1)) | 共享前缀，一次遍历多词 |

**Trie 的关键优势**：多个单词共享前缀时，DFS 只需走一次这段路径。

---

## 🏗️ 数据结构

```python
class Trie:
    def __init__(self):
        self.children = {}   # char -> Trie node
        self.word = None     # 到达此节点时完整的单词（None 表示非词尾）
```

---

## 🔄 算法流程

```
1. 建 Trie
   └── 遍历 words，将每个单词插入 Trie
       └── 在末节点存储 node.word = word

2. 遍历 board 每个格子作为起点
   └── 对每个格子调用 dfs(i, j, root)

3. DFS(i, j, node)
   ├── 越界 / 已访问('#') / 字符不在 Trie → return
   ├── 找到 node.word → 加入结果集，置 None（剪枝）
   ├── 标记当前格子为 '#'（防止重复访问）
   ├── 向四个方向递归
   ├── 恢复格子原字符（回溯）
   └── 若 node.children 为空 → 删除该分支（Trie 剪枝）
```

---

## ✅ 最终代码（含优化）

```python
class Trie:
    def __init__(self):
        self.children = {}
        self.word = None

class Solution:
    def findWords(self, board: List[List[str]], words: List[str]) -> List[str]:
        # 1. 建 Trie
        root = Trie()
        for word in words:
            node = root
            for char in word:
                if char not in node.children:
                    node.children[char] = Trie()
                node = node.children[char]
            node.word = word

        result = set()
        m, n = len(board), len(board[0])

        def dfs(i, j, node):
            if i < 0 or i >= m or j < 0 or j >= n:
                return
            char = board[i][j]
            if char == '#' or char not in node.children:
                return

            next_node = node.children[char]

            # 找到单词
            if next_node.word:
                result.add(next_node.word)
                next_node.word = None          # ⭐ 优化1：避免重复记录

            # 回溯
            board[i][j] = '#'
            for di, dj in [(1,0), (-1,0), (0,1), (0,-1)]:
                dfs(i+di, j+dj, next_node)
            board[i][j] = char

            # ⭐ 优化2：删除已耗尽的 Trie 分支
            if not next_node.children:
                del node.children[char]

        for i in range(m):
            for j in range(n):
                dfs(i, j, root)

        return list(result)
```

---

## ⭐ 两大关键优化

### 优化 1：找到单词后置 `None`

```python
if next_node.word:
    result.add(next_node.word)
    next_node.word = None   # ← 关键！
```

**作用**：防止同一个单词被多次找到（从不同起点或路径）时重复加入结果。  
**影响**：最显著的性能优化，尤其当 words 中有重复或 board 中该单词出现多次时。

---

### 优化 2：删除空 Trie 分支

```python
if not next_node.children:
    del node.children[char]
```

**作用**：当一个节点没有子节点了（叶子且已找到词），删除它。  
**影响**：随着搜索进行，Trie 越来越小，后续 DFS 的无效检查减少。

---

## 🎯 易错点总结

| 易错点 | 错误写法 | 正确写法 |
|--------|---------|---------|
| 回溯恢复 | 忘记 `board[i][j] = char` | 四方向 DFS 后必须恢复 |
| 重复记录 | 直接用 `list`，同词加多次 | 用 `set` 或 `node.word = None` |
| 越界检查 | 检查顺序混乱 | 先越界，再检查 `#`，再查 Trie |
| 剪枝时机 | 在递归前剪枝 | **回溯之后**检查 `children` 是否为空 |

---

## 📊 复杂度分析

| | 复杂度 | 说明 |
|--|--------|------|
| **时间** | O(M × N × 4 × 3^(L-1)) | 每格出发，每步最多3个方向（来路已标#） |
| **空间** | O(W × L) | Trie 存储所有单词 |

> L = 最长单词长度，W = 单词数量

---

## 🧠 记忆口诀

```
建 Trie → 每格出发 DFS
找到词 → 记录 + 置 None
标 # → 四向递归 → 恢复
无子节点 → 删 Trie 分支
```

---

## 🔗 相关题目

| 题号 | 题目 | 关联点 |
|------|------|--------|
| LC 79 | Word Search | 本题单词版，无 Trie |
| LC 208 | Implement Trie | Trie 基础实现 |
| LC 211 | Design Add and Search Words | Trie + 通配符 |
## 215 Kth Largest Element in Array
## 703 Kth Largest Element in Stream
# 数据流中的第 K 大元素

## 题意

设计一个类，初始化时给定 `k` 和初始数组，之后每次调用 `add(val)` 返回当前数据流中第 `k` 大的元素。

## 核心思路：最小堆，只维护 k 个元素

最小堆保留最大的 k 个元素，堆顶（最小的那个）就是第 k 大。

## 解法

```python
import heapq

class KthLargest:

    def __init__(self, k, nums):
        self.k = k
        self.minheap = []
        for n in nums:
            heapq.heappush(self.minheap, n)
            if len(self.minheap) > k:
                heapq.heappop(self.minheap)

    def add(self, val):
        heapq.heappush(self.minheap, val)
        if len(self.minheap) > self.k:
            heapq.heappop(self.minheap)
        return self.minheap[0]
```

## 为什么用最小堆而不是最大堆

- 最小堆保留最大的 k 个数，堆顶 = 这 k 个里最小的 = 第 k 大
- 每次 push 后如果超过 k 个，pop 掉最小的，剩下的永远是最大的 k 个

## 逐步示例

`k=3, nums=[4,5,8,2]`

```
初始化:
  push 4 → [4]
  push 5 → [4, 5]
  push 8 → [4, 5, 8]
  push 2 → [2, 4, 5, 8] → 超过k, pop 2 → [4, 5, 8]

add(3): push → [3,4,5,8] → pop 3 → [4,5,8] → 返回 4
add(9): push → [4,5,8,9] → pop 4 → [5,8,9] → 返回 5
```

## 常见错误

用最大堆然后每次 pop k 次再 push 回去，虽然能过但效率低：

```python
# ❌ 每次 add 是 O(k log n)
for i in range(self.k):
    node = heapq.heappop(self.maxheap)

# ✅ 最小堆每次 add 只需 O(log k)
heapq.heappush(self.minheap, val)
if len(self.minheap) > self.k:
    heapq.heappop(self.minheap)
```

## 复杂度

- 初始化：O(n log k)
- 每次 add：O(log k)
- 空间：O(k)

## 973 K Closest Points to Origin
# Quick Select 简明笔记

## 一句话总结

Quick Select = 快排砍一半。每次 partition 后只递归需要的那一半。

## 核心思路

找前 k 小的元素，不要求有序：

1. 选 pivot，把数组分成 "≤ pivot" 和 "> pivot" 两部分
2. 看 pivot 落在位置 `p`：
   - `p == k` → 结束，`arr[:k]` 就是答案
   - `p < k` → 前面不够，去右半边继续找
   - `p > k` → 前面太多，去左半边缩小
3. 重复直到 `p == k`

## Partition 详解

```python
def partition(l, r):
    pivot = dist(points[r])   # 选最右边作为基准
    p = l                      # "小于区域"的边界
    for i in range(l, r):
        if dist(points[i]) <= pivot:
            points[i], points[p] = points[p], points[i]
            p += 1
    points[p], points[r] = points[r], points[p]
    return p
```

`p` 就是一个隔板：左边全部 ≤ pivot，右边全部 > pivot。

### 走一遍

```
距离: [20, 1, 18, 5]，pivot = 5
       i→              p=0

i=0: 20 <= 5? 否                    p=0
i=1:  1 <= 5? 是 → swap(p=0, i=1)  p=1
      [1, 20, 18, 5]
i=2: 18 <= 5? 否                    p=1

最后 swap(p=1, r=3):
      [1, 5, 18, 20]
          ↑ p=1  pivot 归位
```

## 完整代码（LeetCode 973）

```python
class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        def dist(p):
            return p[0] * p[0] + p[1] * p[1]

        def partition(l, r):
            pivot = dist(points[r])
            p = l
            for i in range(l, r):
                if dist(points[i]) <= pivot:
                    points[i], points[p] = points[p], points[i]
                    p += 1
            points[p], points[r] = points[r], points[p]
            return p

        l, r = 0, len(points) - 1
        while l <= r:
            pivot_idx = partition(l, r)
            if pivot_idx == k:
                break
            elif pivot_idx < k:
                l = pivot_idx + 1
            else:
                r = pivot_idx - 1

        return points[:k]
```

## 三种方法对比

| 方法 | 平均时间 | 最坏时间 | 空间 | 结果有序？ |
|------|---------|---------|------|-----------|
| 最小堆（全部 push） | O(n log n) | O(n log n) | O(n) | 可以 |
| 最大堆（大小 k） | O(n log k) | O(n log k) | O(k) | 可以 |
| Quick Select | **O(n)** | O(n²) | O(1) | 否 |

## 什么时候用 Quick Select

- 要前 k 个且**不要求有序** → Quick Select 最优
- 要前 k 个且**要求有序** → 堆更方便
- **动态数据流**维护 top k → 只能用堆
- Quick Select 会**修改原数组**，不是所有场景都适合

## 为什么是 O(n)

每次处理范围大概减半：n + n/2 + n/4 + ... ≈ 2n = O(n)

最坏 O(n²) 是 pivot 每次选到最大/最小值，可以随机选 pivot 避免：

```python
import random

def partition(l, r):
    rand = random.randint(l, r)
    points[rand], points[r] = points[r], points[rand]
    # 后面一样...
```

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
# 355 Design Twitter

## 题意

设计一个简化版 Twitter，支持发推、关注、取关、获取最新 10 条动态。

---

## 数据结构选择

| 需求 | 数据结构 | 原因 |
|------|----------|------|
| 关注关系 | `defaultdict(set)` | unfollow 需要 O(1) 删除，set 天然去重 |
| 推文列表 | `defaultdict(list)` | append 按时间有序，末尾就是最新的 |
| 时间戳 | `self.time` 计数器 | 每发一条 +1，用来比较新旧 |
| 全局 Heap | ❌ 不需要 | 只在 getNewsFeed 时临时创建 |

---

## getNewsFeed 核心：合并 K 个有序链表

### 朴素做法

所有关注的人的推文全部入堆，堆大小 = 总推文数。能过但不高效。

### 优化做法

每个人只放最新一条进堆，pop 出来后再从那个人补一条。堆始终最多 K 个元素（K = 关注人数）。

```
堆里：
[-100, tweetId, 用户A, index=5]   ← A 剩余最新的
[-95,  tweetId, 用户B, index=2]   ← B 剩余最新的
[-80,  tweetId, 用户C, index=9]   ← C 剩余最新的

pop 出 -100（全局最新），然后从 A 的 index=4 补一条进来
```

列表管单人顺序，堆管全局排序，两者配合。

### 为什么 index 从末尾开始

`postTweet` 用 `append`，列表天然按时间排列，末尾最新。`index = len - 1` 开始，每次 `-1` 就是从新到旧。

---

## 代码

```python
class Twitter:
    def __init__(self):
        self.time = 0
        self.followMap = defaultdict(set)
        self.tweetMap = defaultdict(list)

    def postTweet(self, userId, tweetId):
        self.time += 1
        self.tweetMap[userId].append((self.time, tweetId))

    def getNewsFeed(self, userId):
        heap = []
        self.followMap[userId].add(userId)       # 自己也算

        # 每个人只放最新一条进堆
        for fid in self.followMap[userId]:
            if fid in self.tweetMap and self.tweetMap[fid]:
                tweets = self.tweetMap[fid]
                index = len(tweets) - 1
                time, tweetId = tweets[index]
                heapq.heappush(heap, (-time, tweetId, fid, index))
                # 存: [-时间戳, 推文ID, 来自谁, 列表下标]

        res = []
        while heap and len(res) < 10:
            time, tweetId, fid, index = heapq.heappop(heap)  # 全局最新
            res.append(tweetId)
            if index > 0:                                      # 那个人还有更早的
                index -= 1
                time, tweetId = self.tweetMap[fid][index]
                heapq.heappush(heap, (-time, tweetId, fid, index))
        return res

    def follow(self, followerId, followeeId):
        self.followMap[followerId].add(followeeId)

    def unfollow(self, followerId, followeeId):
        if followeeId in self.followMap[followerId]:
            self.followMap[followerId].remove(followeeId)
```

---

## Python 最大堆技巧

`heapq` 只有最小堆，时间戳取负就变成最大堆：`-time` 最小的就是 `time` 最大的（最新的）。

---

## 易错点

1. getNewsFeed 要包含自己的推文：`followMap[userId].add(userId)`
2. `self.tweetMap` 不要忘了 `self`
3. unfollow 时要先检查是否存在，否则 `remove` 会抛异常（或用 `discard`）
4. 堆里存 index 和 fid，才能在 pop 后知道从哪里补下一条
## 46 Permutations


## 题意

给定一组不重复的数字，返回所有可能的全排列。

---

## 核心思路：回溯

每一步从未使用的数字中选一个加入 path，选完所有数字就得到一个排列。

---

## 回溯三要素

| 要素 | 本题 |
|------|------|
| 选择 | 从未使用的数字中选一个 |
| 约束 | `used[j]` 标记已使用，不能重复选 |
| 终止 | `len(path) == len(nums)` |

---

## 代码

```python
class Solution:
    def permute(self, nums):
        res = []
        used = [False] * len(nums)

        def dfs(path):
            if len(path) == len(nums):
                res.append(path.copy())
                return

            for j in range(len(nums)):
                if used[j]:
                    continue
                path.append(nums[j])
                used[j] = True
                dfs(path)
                path.pop()
                used[j] = False

        dfs([])
        return res
```

---

## 三个重点

1. **used 数组**：标记哪些元素已经用了，避免重复使用同一个元素
2. **path.copy()**：收集结果时必须拷贝，否则回溯会改掉 path，最终全是空列表
3. **做选择和撤销完全对称**：`append/pop`、`True/False` 成对出现

---

## 和 N 皇后的对比

| | 全排列 | N 皇后 |
|---|--------|--------|
| 选择 | 选一个未使用的数字 | 选一个合法的列 |
| 约束 | `used[j]` | 列 + 两条对角线 |
| 撤销 | `pop` + `used = False` | `pop` + `remove` 三个 set |
| 模板 | 完全一样 | 完全一样 |
## 90 Subsets II

## 核心思路

"选或不选"回溯，去重关键：**不选当前元素时，跳过所有相同的元素**。

## 完整代码

```python
class Solution:
    def subsetsWithDup(self, nums: List[int]) -> List[List[int]]:
        res = []
        nums.sort()  # 排序让相同元素相邻

        def dfs(i, path):
            if i == len(nums):
                res.append(path.copy())
                return

            # 选 nums[i]
            path.append(nums[i])
            dfs(i + 1, path)
            path.pop()

            # 不选 nums[i]，跳过所有相同的
            while i + 1 < len(nums) and nums[i + 1] == nums[i]:
                i += 1
            dfs(i + 1, path)

        dfs(0, [])
        return res
```

## 为什么需要 while 跳过

以 `[1, 2, 2]` 为例，如果不跳过：

```
不选 i=1 的 2 → 选 i=2 的 2 → [2]
选 i=1 的 2 → 不选 i=2 的 2 → [2]   ← 重复！
```

两条不同路径产生了同一个 `[2]`。

加了 while 后，"不选"就意味着后面所有相同的元素全部跳过，不会偷偷从后面选一个回来。

## while 循环走一遍

`nums = [1, 2, 2, 2]`，当前 `i=1`，决定不选：

```
开始: i=1, nums[1]=2

while: nums[2]==nums[1] → 2==2 ✓ → i=2
while: nums[3]==nums[2] → 2==2 ✓ → i=3
while: i+1=4 越界 → 退出

dfs(4, path) → 直接到末尾，所有的 2 都不选
```

while 的效果：**停在最后一个重复元素上**，`i+1` 刚好跳到下一个不同的元素。

## 决策树（`[1, 2, 2]`）

```
dfs(0, [])
├── 选1 → dfs(1, [1])
│   ├── 选2 → dfs(2, [1,2])
│   │   ├── 选2 → [1,2,2] ✓
│   │   └── 不选2 → [1,2] ✓
│   └── 不选2 → 跳过重复 → [1] ✓
└── 不选1 → dfs(1, [])
    ├── 选2 → dfs(2, [2])
    │   ├── 选2 → [2,2] ✓
    │   └── 不选2 → [2] ✓
    └── 不选2 → 跳过重复 → [] ✓
```

结果：`[1,2,2], [1,2], [1], [2,2], [2], []` — 6 个，无重复。

## 直觉理解

对于连续 k 个相同元素，合法的选法只有 k+1 种（选 0 个、1 个、...、k 个）：

```
三个 2 的情况：
选 0 个: 不选 → while 跳过所有 → 结束
选 1 个: 选 → 不选 → while 跳过剩余
选 2 个: 选 → 选 → 不选
选 3 个: 选 → 选 → 选
```

每种数量只出现一次，while 保证了这一点。

## 40 Combination Sum II

## 核心思路

从排序后的数组中选元素，使总和等于 target。每个元素只能用一次，结果不能重复。

去重关键：**排序 + 跳过同一层的重复元素**。

## 解法一：for 循环 + continue

```python
class Solution:
    def combinationSum2(self, candidates, target):
        path = []
        res = []
        candidates.sort()

        def dfs(i, cur):
            if cur == target:
                res.append(path[:])
                return
            for j in range(i, len(candidates)):
                if j > i and candidates[j] == candidates[j-1]:
                    continue
                if cur + candidates[j] > target:
                    break
                path.append(candidates[j])
                dfs(j + 1, cur + candidates[j])
                path.pop()

        dfs(0, 0)
        return res
```

### j > i 的含义

同一层里，第一个重复元素可以用，后面的跳过：

```
candidates = [1, 2, 2, 3]，i=1

j=1: candidates[1]=2 → 第一个2，可以用
j=2: candidates[2]=2, j>i 且等于前一个 → 跳过
j=3: candidates[3]=3 → 不重复，可以用
```

### 走一遍 [2,2,2,2,2,2] target=8

```
dfs(0, 0):
  j=0: 选2 → dfs(1, 2)
    j=1: 选2 → dfs(2, 4)
      j=2: 选2 → dfs(3, 6)
        j=3: 选2 → dfs(4, 8) == target ✓ → [2,2,2,2]
        j=4: j>3 且重复 → skip
        j=5: skip
      j=3: j>2 且重复 → skip
    j=2: j>1 且重复 → skip
  j=1: j>0 且重复 → skip
```

每一层第一个重复元素可以用，所以能连续选四个 2，但不会产生重复组合。

## 解法二：选或不选 + while 跳过

```python
class Solution:
    def combinationSum2(self, candidates, target):
        ans = []
        candidates.sort()

        def dfs(index, path, total):
            if total == target:
                ans.append(path.copy())
                return
            if total > target or index == len(candidates):
                return

            # 选当前元素
            path.append(candidates[index])
            dfs(index + 1, path, total + candidates[index])
            path.pop()

            # 不选当前元素，跳过所有相同的
            while index + 1 < len(candidates) and candidates[index + 1] == candidates[index]:
                index += 1
            dfs(index + 1, path, total)

        dfs(0, [], 0)
        return ans
```

### while 跳过的含义

不选当前元素时，后面所有相同的也必须不选，否则会重复：

```
candidates = [2, 2, 3]

不选第1个2 → 选第2个2 → [2]
选第1个2 → 不选第2个2 → [2]  ← 重复！

加了 while：不选第1个2 → 跳过第2个2 → 所有2都不选
```

## 两种模板对比

| | for 循环 + continue | 选或不选 + while |
|---|---|---|
| 分支 | 每层选一个往下递归 | 每个元素两个分支 |
| 去重 | 同一层 continue 跳过重复 | 不选时 while 跳过所有相同 |
| 适用 | 组合类问题更常见 | 子集类问题更直觉 |

两种都能 AC，选更熟悉的就行。

## 必须排序

去重依赖相同元素相邻，不排序就无法跳过：

```
不排序: [1, 7, 1]
选第1个1，不选第3个1 → [1]
不选第1个1，选第3个1 → [1]  ← 不相邻，跳不过

排序后: [1, 1, 7]
while 或 continue 都能正确跳过
```

## 复杂度

- 时间：O(2ⁿ)，最坏情况尝试所有子集
- 空间：O(n)，递归深度 + path
## 79 Word Search

## 核心思路

从每个格子出发，DFS 尝试匹配 word 的每个字符。用 visited 防止同一条路径重复使用格子。

## 推荐模板：检查放在 dfs 开头

```python
class Solution:
    def exist(self, board, word):
        direction = [(0,1),(1,0),(0,-1),(-1,0)]
        visited = set()

        def dfs(r, c, index):
            if index == len(word):
                return True
            if r < 0 or r >= len(board) or c < 0 or c >= len(board[0]):
                return False
            if (r, c) in visited:
                return False
            if board[r][c] != word[index]:
                return False

            visited.add((r, c))
            for i, j in direction:
                if dfs(r+i, c+j, index+1):
                    return True
            visited.remove((r, c))
            return False

        for i in range(len(board)):
            for j in range(len(board[0])):
                if dfs(i, j, 0):
                    return True
        return False
```

## 为什么把检查放在 dfs 开头

所有检查集中在一个地方，调用方只管调用：

```python
# 外层干干净净
if dfs(i, j, 0):
    return True

# dfs 内部统一处理
def dfs(r, c, index):
    if ...: return False      # 不合法直接返回
    visited.add((r, c))       # 自己 add
    ...
    visited.remove((r, c))    # 自己 remove
    return False
```

如果把检查放在调用前（递归邻居之前），visited 的 add/remove 会分散在外层和 dfs 内部两个地方，容易漏。

## dfs 结构详解

```python
def dfs(r, c, index):
    # 1. 成功条件
    if index == len(word): return True

    # 2. 所有失败条件
    if 越界: return False
    if 访问过: return False
    if 字符不匹配: return False

    # 3. 标记当前格子
    visited.add((r, c))

    # 4. 递归四个方向
    for i, j in direction:
        if dfs(r+i, c+j, index+1):
            return True

    # 5. 回溯
    visited.remove((r, c))
    return False
```

## 走一遍示例

```
board:        word = "ABC"
A  B  E
C  F  G

dfs(0,0,0): board[0][0]='A'==word[0] ✓
  visited: {(0,0)}
  dfs(0,1,1): board[0][1]='B'==word[1] ✓
    visited: {(0,0),(0,1)}
    dfs(0,2,2): board[0][2]='E'!=word[2] → False
    dfs(1,1,2): board[1][1]='F'!=word[2] → False
    dfs(0,0,2): (0,0) in visited → False
    dfs(-1,1,2): 越界 → False
    回溯 remove (0,1)
  dfs(1,0,1): board[1][0]='C'!=word[1] → False
  dfs(-1,0,1): 越界 → False
  dfs(0,-1,1): 越界 → False
  回溯 remove (0,0)
  return False

dfs(0,1,0): board[0][1]='B'!=word[0] → False
...继续尝试其他起点
```

## 常见 bug

**1. visited 的 add/remove 不对称**

```python
# 错误：add 邻居而不是自己
visited.add((r+i, c+j))

# 正确：add 自己
visited.add((r, c))
```

**2. for 循环结束没有 return False**

```python
# 错误：邻居都失败后函数没有返回值（隐式返回 None）
for i, j in direction:
    if dfs(r+i, c+j, index+1):
        return True
# 缺了 return False

# 正确
visited.remove((r, c))
return False    # ← 必须有
```

**3. 外层 add 了 visited 但没 remove**

```python
# 错误
visited.add((i, j))
if dfs(i, j, 0):
    return True
# 漏了 visited.remove((i, j))

# 解决方案：用推荐模板，外层不管 visited
if dfs(i, j, 0):
    return True
```

## 复杂度

- 时间：O(m × n × 4^L)，L 是 word 长度，每个格子出发最多 4 个方向递归 L 层
- 空间：O(L)，递归深度 + visited 大小

**提示**: 回溯的关键在于"撤销"，寻路问题几乎都需要回溯！💪
## 131 Palindrome Partitioning
## 17 Letter Combinations of Phone Number
## 51 N Queens
## 核心洞察

每一行必须且只能放一个皇后，所以问题简化为：**逐行选择皇后放在哪一列**。

---

## 回溯五问模板

| # | 问题 | N皇后的回答 |
|---|------|-------------|
| 1 | 每一步在做什么选择？ | 在第 `row` 行选一个 `col` 放皇后 |
| 2 | 什么时候结束？ | `row == n`，所有行都放完了 |
| 3 | 怎么判断合不合法？ | 三个冲突：同列、主对角线 `row-col`、副对角线 `row+col` |
| 4 | 需要维护哪些状态？ | `cols` set、`diag1` set、`diag2` set、`path` 列表 |
| 5 | 怎么撤销选择？ | 加了什么就去掉什么（remove / pop） |

---

## 对角线判断怎么记

在棋盘上：

- `\` 方向：同一条线上所有格子 **行 - 列** 相同
- `/` 方向：同一条线上所有格子 **行 + 列** 相同

画个 4×4 格子验证一下就不会忘：

```
       col 0   col 1   col 2   col 3
row 0:  0-0=0   0-1=-1  0-2=-2  0-3=-3    (row - col)
row 1:  1-0=1   1-1=0   1-2=-1  1-3=-2
row 2:  2-0=2   2-1=1   2-2=0   2-3=-1
row 3:  3-0=3   3-1=2   3-2=1   3-3=0
```

相同值连起来就是 `\` 对角线。`row + col` 同理对应 `/` 对角线。

---

## 代码骨架（填空版）

```python
def backtrack(row, path):
    # 终止条件（第2问）
    if ___:
        收集结果
        return

    for col in range(n):        # 每一步的选择（第1问）
        if 不合法:              # 判断合法性（第3问）
            continue

        做选择，更新状态        # 第4问
        backtrack(row + 1, path)
        撤销选择                # 第5问
```

---

## 完整参考代码

```python
class Solution:
    def solveNQueens(self, n):
        cols, diag1, diag2 = set(), set(), set()
        res = []

        def backtrack(row, path):
            if row == n:
                board = []
                for c in path:
                    board.append('.' * c + 'Q' + '.' * (n - c - 1))
                res.append(board)
                return

            for col in range(n):
                if col in cols or (row - col) in diag1 or (row + col) in diag2:
                    continue

                path.append(col)
                cols.add(col)
                diag1.add(row - col)
                diag2.add(row + col)

                backtrack(row + 1, path)

                path.pop()
                cols.remove(col)
                diag1.remove(row - col)
                diag2.remove(row + col)

        backtrack(0, [])
        return res
```

---

## 易错点提醒

1. 构建 board 后别忘了 `res.append(board)`
2. board 不需要在回溯过程中维护，终止时按需生成即可
3. 撤销选择要和做选择完全对称，少一个就会出 bug
## 200 Number of Islands
## 695 Max Area of Island
## 133 Clone Graph

## 题目核心

给定一个无向连通图中的一个节点，返回该图的**深拷贝**。图中有环，必须防止无限循环。

## 关键思路

用一个 **`lookup` 哈希表**（原节点 → 克隆节点）同时解决两个问题：

1. **防止重复访问**：已经克隆过的节点不再重复创建
2. **映射关系**：随时能通过原节点找到对应的克隆节点来建立邻居关系

---

## BFS 解法

```python
def cloneGraph(self, node):
    if not node:
        return None
    lookup = {}
    clone = Node(node.val, [])
    lookup[node] = clone
    q = deque([node])
    while q:
        tmp = q.popleft()
        for n in tmp.neighbors:
            if n not in lookup:          # 没克隆过 → 创建 + 入队
                lookup[n] = Node(n.val, [])
                q.append(n)
            lookup[tmp].neighbors.append(lookup[n])  # 在 if 外面，总是执行
    return clone
```

**流程（以 1-2-3-4 环为例）：**

| 步骤 | 取出 | 处理邻居 | 队列 | lookup |
|------|------|----------|------|--------|
| 初始 | — | 创建 1' | [1] | {1} |
| 1 | 1 | 2 不在→创建 2'，4 不在→创建 4' | [2, 4] | {1,2,4} |
| 2 | 2 | 1 已在→直接连，3 不在→创建 3' | [4, 3] | {1,2,3,4} |
| 3 | 4 | 1 已在→连，3 已在→连 | [3] | {1,2,3,4} |
| 4 | 3 | 2 已在→连，4 已在→连 | [] | {1,2,3,4} |

---

## DFS 解法

```python
def cloneGraph(self, node):
    if not node:
        return None
    lookup = {}

    def dfs(node):
        if node in lookup:
            return lookup[node]
        copy = Node(node.val, [])
        lookup[node] = copy
        for nei in node.neighbors:
            copy.neighbors.append(dfs(nei))
        return copy

    return dfs(node)
```

**递归过程：**

```
dfs(1) → 创建 1'
  dfs(2) → 创建 2'
    dfs(1) → 已在 lookup，返回 1'  ← 防止无限递归
    dfs(3) → 创建 3'
      dfs(2) → 已在，返回 2'
      dfs(4) → 创建 4'
        dfs(1) → 已在，返回 1'
        dfs(3) → 已在，返回 3'
        → 4' 邻居 = [1', 3']
      → 3' 邻居 = [2', 4']
    → 2' 邻居 = [1', 3']
  dfs(4) → 已在，返回 4'
  → 1' 邻居 = [2', 4']
```

---

## BFS vs DFS 的 if 判断区别

两者都是**每个邻居都要加到邻居列表**，但判断重复的位置不同：

**BFS**：`if` 只控制"要不要创建新克隆 + 入队"，`append` 在 if 外面，总是执行。

```python
if n not in lookup:         # 只控制创建和入队
    lookup[n] = Node(...)
    q.append(n)
lookup[tmp].neighbors.append(lookup[n])  # 不管有没有，都连
```

**DFS**：判断藏在递归入口，`dfs(nei)` 总会返回正确的克隆节点（新建的或已有的），所以外层直接 append，不需要 if。

```python
for nei in node.neighbors:
    copy.neighbors.append(dfs(nei))  # dfs 内部处理了重复
```

> 如果 DFS 中加了 `if nei not in lookup`，会导致已访问的邻居被跳过，克隆图的边会丢失。

---

## 常见 Bug

| Bug | 后果 |
|-----|------|
| `deque(node)` 而非 `deque([node])` | 尝试迭代 Node 对象，报错 |
| 变量名不一致（`q` vs `queue`） | NameError |
| `lookup[tmp].neighbors.append(lookup[tmp])` | 把自己加成自己的邻居 |
| 忘记 `return clone` | 函数返回 None |
| DFS 中多加 `if nei not in lookup` | 丢失边，图不完整 |

---

## 复杂度

- **时间**：O(N + E)，每个节点和边各处理一次
- **空间**：O(N)，lookup 存储所有节点的映射
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
## 题目核心

给定 n 门课和先修关系，判断能否修完所有课。本质就是**有向图判断是否有环**。

---

## 建图

`prerequisites = [[course, pre]]` 意思是"先上 pre 才能上 course"。

建边方向：**pre → course**（从先修课指向后续课）。

```python
graph = defaultdict(list)
for course, pre in prerequisites:
    graph[pre].append(course)
```

示例 `[[1,0], [2,0], [3,1]]`：

```
课程0 → 课程1 → 课程3
  \
   → 课程2
```

graph = {0: [1, 2], 1: [3]}

---

## 解法一：DFS 三色标记法

### 核心思想

从每个节点出发做 DFS，如果走着走着回到了"正在访问"的节点，说明有环。

### 三个状态

| state | 含义 | 遇到时的行为 |
|-------|------|-------------|
| 0 | 未访问 | 开始检查 |
| 1 | 访问中（在当前路径上） | **有环！** 返回 True |
| 2 | 已完成（确认安全） | 跳过，返回 False（剪枝） |

### 为什么需要 state=2

避免重复检查。比如：

```
0 → 1 → 3
0 → 2 → 3
```

检查完 0→1→3 后，3 标记为 2。走 0→2→3 时看到 3 是 2，直接跳过，不用再查一遍。没有这个剪枝，大图会超时。

### 代码

```python
class Solution:
    def canFinish(self, numCourses, prerequisites):
        graph = defaultdict(list)
        for course, pre in prerequisites:
            graph[pre].append(course)

        # 0=未访问, 1=访问中, 2=已完成
        state = [0] * numCourses

        def has_cycle(node):
            if state[node] == 1:    # 回到了正在走的路径 → 有环
                return True
            if state[node] == 2:    # 之前确认过安全 → 跳过
                return False
            state[node] = 1         # 标记为"正在访问"
            for nei in graph[node]:
                if has_cycle(nei):
                    return True
            state[node] = 2         # 所有后续都安全，标记完成
            return False

        for i in range(numCourses):
            if has_cycle(i):
                return False
        return True
```

### DFS 模拟过程

图：`0 → 1 → 3, 0 → 2 → 3`

```
has_cycle(0): state[0]=1
  has_cycle(1): state[1]=1
    has_cycle(3): state[3]=1
      无邻居 → state[3]=2, 返回 False
    → state[1]=2, 返回 False
  has_cycle(2): state[2]=1
    has_cycle(3): state[3]==2, 直接返回 False  ← 剪枝！
    → state[2]=2, 返回 False
  → state[0]=2, 返回 False

has_cycle(1): state[1]==2, 跳过  ← 剪枝！
has_cycle(2): state[2]==2, 跳过  ← 剪枝！
has_cycle(3): state[3]==2, 跳过  ← 剪枝！

全部无环 → 返回 True
```

---

## 解法二：BFS 拓扑排序（Kahn 算法）

### 核心思想

不断把"没有先修课"（入度为 0）的课拿掉，看最后能不能全部拿完。拿不完说明剩下的课互相依赖（有环）。

### indegree（入度）是什么

入度 = 指向该节点的边数 = 这门课有几门先修课还没上。

```
0 → 1 → 3
0 → 2 → 3
```

| 节点 | 入度 | 含义 |
|------|------|------|
| 0 | 0 | 没有先修课，可以直接上 |
| 1 | 1 | 需要先上 0 |
| 2 | 1 | 需要先上 0 |
| 3 | 2 | 需要先上 1 和 2 |

### 代码

```python
class Solution:
    def canFinish(self, numCourses, prerequisites):
        graph = defaultdict(list)
        indegree = [0] * numCourses

        for course, pre in prerequisites:
            graph[pre].append(course)
            indegree[course] += 1

        # 入度为 0 的节点入队（没有先修课，可以直接上）
        q = deque([i for i in range(numCourses) if indegree[i] == 0])
        count = 0

        while q:
            node = q.popleft()
            count += 1
            for nei in graph[node]:
                indegree[nei] -= 1      # 上完 node，后续课的先修要求少了一门
                if indegree[nei] == 0:   # 所有先修课都上完了 → 入队
                    q.append(nei)

        return count == numCourses       # 全部上完 → 无环
```

### BFS 模拟过程

图：`0 → 1 → 3, 0 → 2 → 3`

初始入度：`[0, 1, 1, 2]`，入度为 0 的：`q = [0]`

| 步骤 | 取出 | 处理 | 入度变化 | 队列 | count |
|------|------|------|---------|------|-------|
| 1 | 0 | 1 的入度 -1, 2 的入度 -1 | [0,0,0,2] | [1,2] | 1 |
| 2 | 1 | 3 的入度 -1 | [0,0,0,1] | [2] | 2 |
| 3 | 2 | 3 的入度 -1 | [0,0,0,0] | [3] | 3 |
| 4 | 3 | 无后续 | [0,0,0,0] | [] | 4 |

count=4 == numCourses=4 → 返回 True

### 有环的情况

```
0 → 1 → 2 → 0（环）
```

入度：`[1, 1, 1]`，没有入度为 0 的节点，队列一开始就空，count=0 ≠ 3 → 返回 False。

---

## 两种解法对比

| | DFS 三色标记 | BFS Kahn 算法 |
|---|---|---|
| 核心问题 | 能不能找到环？ | 能不能全部拿完？ |
| 判断方式 | 遇到 state=1 → 有环 | count ≠ n → 有环 |
| 额外数据 | state 数组（3 种状态） | indegree 数组 |
| 时间 | O(V + E) | O(V + E) |
| 空间 | O(V + E) | O(V + E) |
| 优势 | 代码简短 | 直觉清晰，能输出上课顺序 |
| 延伸 | 适合判环 | 适合拓扑排序（Course Schedule II） |

---

## 记忆口诀

**DFS**：走到标记 1 就是环，走完标记 2 就安全。

**BFS**：入度为 0 就能上，上完减邻居，最后数一数。
## 210 Course Schedule II
## 684 Redundant Connection
## 269 Alien Dictionary 
## 🧩 题目理解

外星语言用 a-z 的字母，但顺序未知。给你一个**按该语言排好序的字典**，反推字母顺序。

**关键洞察**：字典已排序 → 相邻两个词之间隐藏了字母大小关系 → 建图 → 拓扑排序

---

## 🧠 思维路线

### 第一步：发现这是一个图的问题

相邻词对比，找第一个不同的字母：
```
"wrt" vs "wrf"  →  t 在 f 前面  →  t → f
"wrf" vs "er"   →  w 在 e 前面  →  w → e
"er"  vs "ett"  →  r 在 t 前面  →  r → t
```

每一对字母大小关系 = 图中一条**有向边**

### 第二步：认出这是拓扑排序

有向边代表顺序依赖 → 要输出一个满足所有依赖的排列 → **拓扑排序**

等价题目：Course Schedule II（输出修课顺序）

### 第三步：想清楚非法情况

| 情况 | 例子 | 检测方式 |
|------|------|---------|
| 有环 | `a→b→a` | 拓扑排序后 `len(res) != len(indegree)` |
| 前缀非法 | `["abc", "ab"]` | `len(w1) > len(w2)` 且 w1 前缀 == w2 |

---

## 🏗️ 建图细节

### 初始化
```python
graph = defaultdict(set)
indegree = {c: 0 for word in words for c in word}
#           ↑ 所有出现过的字母都要在 indegree 里
#             否则最后统计节点数会出错
```

### 为什么用 set 不用 list？
避免重复边。比如两对词都推出 `w → e`，`set` 自动去重，`indegree` 也不会多加。

```python
if w2[j] not in graph[w1[j]]:   # 没有这条边才加
    graph[w1[j]].add(w2[j])
    indegree[w2[j]] += 1
```

### 为什么找到第一个不同字母就 break？
```
"wrt" vs "wrf"
      t    f   ← 只有这里能推出信息
         ↑
      后面的字母无法比较，因为前提是前面都相同
```

### 数据结构可视化
以 `["wrt", "wrf", "er"]` 为例：
```
graph = {
  't': {'f'},   ← t → f
  'w': {'e'},   ← w → e
}

indegree = {
  'w': 0,
  'r': 0,
  't': 0,
  'f': 1,   ← 被 t 指向
  'e': 1,   ← 被 w 指向
}
```

---

## ✅ 完整代码

```python
from collections import defaultdict, deque

class Solution:
    def foreignDictionary(self, words: List[str]) -> str:
        # 1. 初始化图
        graph = defaultdict(set)
        indegree = {c: 0 for word in words for c in word}

        # 2. 建图
        for i in range(len(words) - 1):
            w1, w2 = words[i], words[i+1]
            min_l = min(len(w1), len(w2))

            # 非法：前缀情况
            if len(w1) > len(w2) and w1[:min_l] == w2[:min_l]:
                return ""

            for j in range(min_l):
                if w1[j] != w2[j]:
                    if w2[j] not in graph[w1[j]]:   # 去重
                        graph[w1[j]].add(w2[j])
                        indegree[w2[j]] += 1
                    break                            # 只看第一个不同字母

        # 3. 拓扑排序（BFS）
        queue = deque([c for c in indegree if indegree[c] == 0])
        res = []

        while queue:
            node = queue.popleft()
            res.append(node)
            for nei in graph[node]:
                indegree[nei] -= 1
                if indegree[nei] == 0:
                    queue.append(nei)

        # 4. 判环
        if len(res) != len(indegree):
            return ""

        return "".join(res)
```

---

## ⭐ 关键点总结

| 问题 | 答案 |
|------|------|
| 为什么能建图？ | 字典已排序，相邻词第一个不同字母揭示大小关系 |
| 为什么拓扑排序？ | 字母顺序 = 有依赖关系的节点排列 |
| indegree 为什么要初始化所有字母？ | 最后用 `len(indegree)` 统计总节点数判断有无环 |
| 为什么用 set？ | 避免重复边导致 indegree 计算错误 |
| 怎么判环？ | `len(res) != len(indegree)` |
| 怎么判前缀非法？ | 长词是短词前缀却排在前面 |

---

## 🧩 与 Course Schedule 的对比

| | Course Schedule II | Alien Dictionary |
|--|-------------------|-----------------|
| 图的来源 | 直接给你 `[a,b]` | 从 words 推导 |
| 节点 | 课程编号 | 字母 |
| 边的含义 | a 是 b 的先修课 | 字母 a 排在 b 前面 |
| 输出 | 修课顺序 | 字母顺序 |
| 判环 | 返回 `[]` | 返回 `""` |

**本质完全相同，唯一难点是建图。**

---

## 🔗 相关题目

| 题号 | 题目 | 关联点 |
|------|------|--------|
| LC 207 | Course Schedule | 拓扑排序判环 |
| LC 210 | Course Schedule II | 拓扑排序输出结果 |
| LC 269 | Alien Dictionary | 本题 |
## 329 Longest Increasing Path in a Matrix
## 题目概述

给定一个 m × n 的整数矩阵，找到最长的**严格递增路径**长度。只能上下左右移动，不能斜着走。

```
输入：
9 9 4
6 6 8
2 1 1

输出：4（路径：1 → 2 → 6 → 9）
```

---

## 为什么纯 DFS 不够

从不同起点出发会重复经过同一个格子。比如从 (0,0) 和 (1,0) 出发都可能经过 (0,1)，以 (0,1) 为起点的路径会被重复计算。

**解决**：加记忆化，每个格子只算一次。

## 为什么不需要 visited 数组

严格递增条件本身就阻止了环——不可能从 5 走到 3 再走回 5，所以不会重复访问。

---

## 解法一：DFS + @cache（推荐）

```python
from functools import cache

def longestIncreasingPath(self, matrix):
    rows, cols = len(matrix), len(matrix[0])

    @cache
    def dfs(r, c):
        best = 1  # 至少包含自己
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                best = max(best, 1 + dfs(nr, nc))
        return best

    return max(dfs(r, c) for r in range(rows) for c in range(cols))
```

### @cache 缓存了什么

输入 `(r, c)` → 输出（从该位置出发的最长路径长度）：

```
dfs(0, 0) → 3    缓存 {(0,0): 3}
dfs(0, 1) → 2    缓存 {(0,0): 3, (0,1): 2}
...
下次再调用 dfs(0, 0) 时直接返回 3，不再递归
```

等价于手动写 memo 字典：

```python
memo = {}
def dfs(r, c):
    if (r, c) in memo:
        return memo[(r, c)]
    best = ...
    memo[(r, c)] = best
    return best
```

### @cache 使用须知

- `from functools import cache`（Python 3.9+）
- 更低版本用 `@lru_cache(maxsize=None)`，效果一样
- LeetCode 上默认已导入，面试写白板时要提一下

---

## 解法二：DFS + 手动 memo

```python
def longestIncreasingPath(self, matrix):
    rows, cols = len(matrix), len(matrix[0])
    memo = {}

    def dfs(r, c):
        if (r, c) in memo:
            return memo[(r, c)]

        best = 1
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                best = max(best, 1 + dfs(nr, nc))

        memo[(r, c)] = best
        return best

    ans = 0
    for r in range(rows):
        for c in range(cols):
            ans = max(ans, dfs(r, c))
    return ans
```

---

## 解法三：DP + 排序

按值从小到大排序后填表，填每个格子时比它小的邻居都已算好。

```python
def longestIncreasingPath(self, matrix):
    rows, cols = len(matrix), len(matrix[0])
    dp = [[1] * cols for _ in range(rows)]

    cells = []
    for r in range(rows):
        for c in range(cols):
            cells.append((matrix[r][c], r, c))
    cells.sort()

    ans = 1
    for val, r, c in cells:
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] < val:
                dp[r][c] = max(dp[r][c], 1 + dp[nr][nc])
        ans = max(ans, dp[r][c])

    return ans
```

---

## 三种解法对比

| | DFS + @cache | DFS + 手动 memo | DP + 排序 |
|--|-------------|----------------|----------|
| 时间 | O(m × n) | O(m × n) | O(mn × log(mn)) |
| 空间 | O(m × n) | O(m × n) | O(m × n) |
| 思路 | 自顶向下 | 自顶向下 | 自底向上 |
| 代码量 | 最简洁 | 稍多 | 稍多 |
| 面试推荐 | ✓ 首选 | ✓ 万能写法 | 提一下即可 |

## 关键点速记

- DFS + 记忆化，不是纯 DFS
- 严格递增天然防环，不需要 visited
- `@cache` = 自动记忆化，key 是参数，value 是返回值
- DP 需要先排序确定填表顺序，多了 log 开销，反而不如 DFS + memo

## 相关题目

- **LeetCode 200** — Number of Islands（矩阵 DFS 基础）
- **LeetCode 695** — Max Area of Island（矩阵 DFS + 计数）
- **LeetCode 1092** — Shortest Common Supersequence（记忆化搜索）

## 332 Reconstruct Itinerary
## 1584 Min Cost to Connect All Points
## 743 Network Delay Time

## 核心思路

从起点出发，用最小堆每次取出距离最小的节点，确定它的最短距离，再扩展它的邻居。

## 完整代码

```python
class Solution:
    def networkDelayTime(self, times, n, k):
        min_dist = {}
        adj = defaultdict(list)
        for u, v, w in times:
            adj[u].append((v, w))

        heap = []
        heapq.heappush(heap, (0, k))

        while heap:
            time2cur, cur = heapq.heappop(heap)
            if cur in min_dist:        # 已确定，跳过
                continue
            min_dist[cur] = time2cur   # pop 时记录

            for v, w in adj[cur]:
                if v not in min_dist:
                    heapq.heappush(heap, (time2cur + w, v))

        if len(min_dist) != n:
            return -1
        return max(min_dist.values())
```

## 堆和 min_dist 的配合

堆负责排序，min_dist 负责去重，两个配合实现 Dijkstra。

```
堆:       负责"下一个该处理谁" → 每次给出距离最小的节点
min_dist: 负责"谁已经确定了"  → 记录已确定最短距离的节点
```

每一步：

```
1. 堆 pop 出距离最小的节点
2. min_dist 检查 → 处理过？跳过。没处理过？记录。
3. 把未确定的邻居 push 进堆
```

## 走一遍示例

```
A --1--> B --1--> C
A --3--> C
```

```
          堆                    min_dist         动作
初始:     [(0,A)]               {}

pop A(0): []                    {A:0}           push B(1), C(3)
          [(1,B),(3,C)]         {A:0}

pop B(1): [(3,C)]              {A:0, B:1}       push C(2)
          [(2,C),(3,C)]         {A:0, B:1}

pop C(2): [(3,C)]              {A:0, B:1, C:2}  C确定了！

pop C(3): C在min_dist → 跳过                    过时的，扔掉
```

堆里会有过时的副本（C 出现了两次），min_dist 帮忙挡掉。

## 为什么第一次 pop 就是最短距离

堆按距离从小到大 pop。如果 C 第一次被 pop 时距离是 2，堆里剩下的所有路径距离都 ≥ 2。所以不可能有更短的路径还没被发现。

前提：**边权必须非负**。如果有负权边，一条"当前更长"的路径可能经过负权边变得更短，Dijkstra 就不成立了。

## 容易写错的地方

**错误1：push 时记录距离而不是 pop 时**

```python
# 错误：push 时不一定是最短
for v, w in adj[cur]:
    min_dist[v] = time2cur + w
    heapq.heappush(heap, (time2cur + w, v))

# 正确：pop 时才确定
time2cur, cur = heapq.heappop(heap)
if cur in min_dist:
    continue
min_dist[cur] = time2cur
```

**错误2：忘了 pop 时去重**

```python
# 少了这两行，同一个节点被处理多次，浪费时间
if cur in min_dist:
    continue
```

**一句话记忆：pop 时记录，pop 时去重。**

## 复杂度

- 时间：O(E log V)，E 是边数，V 是节点数
- 空间：O(V + E)

## 778 Swim in Rising Water
## 787 Cheapest Flights Within K Stops

## 70 Climbing Stairs
## 746 Min Cost Climbing Stairs
## 198 House Robber
## 题意

一排房子，每间有金额，不能偷相邻的两间，求最大金额。

---

## 核心思路

对每间房子只有两个选择：偷或不偷。

- **偷第 i 间**：不能偷第 i-1 间，收益 = `dp[i-2] + nums[i]`
- **不偷第 i 间**：收益 = `dp[i-1]`

取两者最大值。

---

## dp 定义

`dp[i]` = 考虑前 i+1 间房子，能偷到的最大金额

### 转移方程

```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

### 初始化

- `dp[0] = nums[0]`（只有一间，直接偷）
- `dp[1] = max(nums[0], nums[1])`（两间选大的）

---

## 代码

```python
class Solution:
    def rob(self, nums):
        if len(nums) == 1:
            return nums[0]

        dp = [0] * len(nums)
        dp[0] = nums[0]
        dp[1] = max(nums[0], nums[1])

        for i in range(2, len(nums)):
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])

        return dp[len(nums) - 1]
```

---

## 可选优化：空间 O(1)

dp[i] 只依赖前两个值，可以用两个变量代替数组：

```python
prev2, prev1 = nums[0], max(nums[0], nums[1])
for i in range(2, len(nums)):
    prev2, prev1 = prev1, max(prev1, prev2 + nums[i])
return prev1
```

---

## 易错点

1. 别忘了处理 `len(nums) == 1` 的边界
2. `dp[1]` 是 `max(nums[0], nums[1])`，不是 `nums[1]`
3. 返回 `dp[len(nums)-1]`，不是 `dp[len(nums)]`

---

## 复杂度

- 时间：O(n)
- 空间：O(n)，优化后 O(1)
## 213 House Robber II
# 213. House Robber II（打家劫舍 II）

## 题目

房屋围成一圈，相邻房屋不能同时偷。求能偷到的最大金额。

## 核心思路

环形问题的关键：**第一间和最后一间不能同时偷**。

拆成两个子问题（普通 House Robber）：

- `nums[1:]` — 不偷第一间
- `nums[:-1]` — 不偷最后一间

取两者最大值即可。

## 代码

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        def l_rob(nums):
            n = len(nums)
            if n == 1:
                return nums[0]
            dp = [0] * len(nums)
            dp[0] = nums[0]
            dp[1] = max(nums[0], nums[1])
            for i in range(2, n):
                dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
            return dp[n - 1]

        if len(nums) == 1:
            return nums[0]
        else:
            return max(l_rob(nums[1:]), l_rob(nums[:-1]))
```

## 逐层解析

### 外层：处理环形

| 代码 | 作用 |
|------|------|
| `len(nums) == 1` | 只有一间房，直接偷 |
| `l_rob(nums[1:])` | 跳过第一间，从第 2 间到最后一间 |
| `l_rob(nums[:-1])` | 跳过最后一间，从第 1 间到倒数第 2 间 |
| `max(...)` | 两种方案取最大值 |

### 内层 `l_rob`：标准 House Robber

**状态定义**：`dp[i]` = 前 `i+1` 间房能偷到的最大金额

**转移方程**：

```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
              ↑              ↑
         不偷第i间       偷第i间
```

| 代码 | 作用 |
|------|------|
| `dp[0] = nums[0]` | 只有一间房，偷它 |
| `dp[1] = max(nums[0], nums[1])` | 两间房，偷较大的 |
| `dp[i-1]` | 不偷当前房，沿用前一间的最优解 |
| `dp[i-2] + nums[i]` | 偷当前房，加上隔一间的最优解 |

## 模拟示例

输入：`[2, 3, 2]`（环形）

```
l_rob([3, 2])：dp = [3, 3] → 3
l_rob([2, 3])：dp = [2, 3] → 3
max(3, 3) = 3 ✅
```

输入：`[1, 2, 3, 1]`（环形）

```
l_rob([2, 3, 1])：
  dp[0] = 2
  dp[1] = max(2, 3) = 3
  dp[2] = max(3, 2+1) = 3 → 3

l_rob([1, 2, 3])：
  dp[0] = 1
  dp[1] = max(1, 2) = 2
  dp[2] = max(2, 1+3) = 4 → 4

max(3, 4) = 4 ✅
```

## 复杂度

- **时间**：O(n)
- **空间**：O(n)（可优化为 O(1)，只保留前两个值）

## 空间优化版本

```python
def l_rob(nums):
    prev2, prev1 = 0, 0
    for num in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + num)
    return prev1
```

用两个变量替代整个 dp 数组，空间从 O(n) 降为 O(1)。

## 5 Longest Palindromic Substring
## 题目核心

给定字符串 s，返回其中**最长的回文子串**。

示例：`"babad"` → `"bab"` 或 `"aba"`

---

## 和 Palindromic Substrings 的关系

两道题的核心逻辑完全一样，只是统计目标不同：

| 题目 | 目标 | 核心区别 |
|------|------|---------|
| Palindromic Substrings | 数回文子串个数 | count += 1 |
| Longest Palindromic Substring | 找最长回文子串 | 记录最长的起点和长度 |

---

## 解法一：DP

### dp 含义

`dp[i][j]` = s[i..j] 是否是回文

### 递推

当 `s[i] == s[j]` 时：

- `j - i <= 1`：长度 1 或 2，直接是回文
- `j - i >= 2`：看 `dp[i+1][j-1]`

每次发现回文时，和当前最长的比较，更长就更新。

### 代码

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n = len(s)
        dp = [[False] * n for _ in range(n)]
        res_idx = 0
        res_len = 0

        for i in range(n - 1, -1, -1):       # 注意：n-1 不是 n
            for j in range(i, n):
                if s[i] == s[j] and (j - i <= 1 or dp[i + 1][j - 1]):
                    dp[i][j] = True
                    if j - i + 1 > res_len:   # 比当前最长更长就更新
                        res_idx = i
                        res_len = j - i + 1

        return s[res_idx:res_idx + res_len]
```

### 模拟过程

s = `"babad"`，n = 5

| i | j | s[i]==s[j]? | 条件 | dp[i][j] | 更新最长？ |
|---|---|-------------|------|----------|-----------|
| 4 | 4 | d==d ✓ | j-i=0 | True | "d" len=1 |
| 3 | 3 | a==a ✓ | j-i=0 | True | — |
| 3 | 4 | a==d ✗ | — | False | — |
| 2 | 2 | b==b ✓ | j-i=0 | True | — |
| 2 | 3 | b==a ✗ | — | False | — |
| 2 | 4 | b==d ✗ | — | False | — |
| 1 | 1 | a==a ✓ | j-i=0 | True | — |
| 1 | 2 | a==b ✗ | — | False | — |
| 1 | 3 | a==a ✓ | dp[2][2]=True | True | **"aba" len=3** |
| 1 | 4 | a==d ✗ | — | False | — |
| 0 | 0 | b==b ✓ | j-i=0 | True | — |
| 0 | 1 | b==a ✗ | — | False | — |
| 0 | 2 | b==b ✓ | dp[1][1]=True | True | "bab" len=3（不更新，等长） |
| 0 | 3 | b==a ✗ | — | False | — |
| 0 | 4 | b==d ✗ | — | False | — |

结果：`s[1:4]` = `"aba"`

---

## 解法二：中心扩展

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        res = ""

        def expand(l, r):
            while l >= 0 and r < len(s) and s[l] == s[r]:
                l -= 1
                r += 1
            return s[l + 1:r]    # 退出时 l 和 r 各多走了一步

        for i in range(len(s)):
            odd = expand(i, i)        # 奇数长度
            even = expand(i, i + 1)   # 偶数长度
            longer = odd if len(odd) > len(even) else even
            if len(longer) > len(res):
                res = longer

        return res
```

### 注意 `s[l+1:r]`

退出 while 时，l 和 r 已经各多走了一步（不满足条件的那一步），所以实际回文范围是 `[l+1, r-1]`，切片写成 `s[l+1:r]`。

---

## 两种解法对比

| | DP | 中心扩展 |
|---|---|---|
| 时间 | O(n²) | O(n²) |
| 空间 | O(n²) | O(1) |
| 代码量 | 较长 | 较短 |
| 优势 | dp 表可复用 | 省空间，直观 |

---

## 常见 Bug

| Bug | 后果 |
|-----|------|
| `range(n, -1, -1)` 从 n 开始 | s[n] 索引越界 |
| 中心扩展返回 `s[l:r]` 而非 `s[l+1:r]` | 多包含了不匹配的字符 |
| 只做奇数或偶数扩展 | 漏掉一半情况 |
| 更新最长时用 `>=` 而非 `>` | 不影响正确性但多做无用更新 |

---

## 复杂度

| | 时间 | 空间 |
|---|---|---|
| DP | O(n²) | O(n²) |
| 中心扩展 | O(n²) | O(1) |
## 647 Palindromic Substrings
## 题目核心

给定字符串 s，返回其中**回文子串的数量**。单个字符也算回文。

示例：`"aaa"` → 6 个回文子串：`a, a, a, aa, aa, aaa`

---

## 解法一：中心扩展法（推荐）

### 核心思想

每个回文都有一个中心。从每个可能的中心出发，向两边扩展，只要两边相等就继续。

中心有两种：

```
奇数长度：中心是一个字符    "aba" → 从 b 往两边扩
偶数长度：中心是两个字符之间  "abba" → 从 bb 往两边扩
```

### 代码

```python
class Solution:
    def countSubstrings(self, s: str) -> int:
        def expand(l, r):
            count = 0
            while l >= 0 and r < len(s) and s[l] == s[r]:
                count += 1
                l -= 1
                r += 1
            return count

        result = 0
        for i in range(len(s)):
            result += expand(i, i)      # 奇数长度，中心是 s[i]
            result += expand(i, i + 1)  # 偶数长度，中心是 s[i] 和 s[i+1] 之间
        return result
```

### 模拟过程

s = `"aab"`

| 中心 | expand(i,i) 奇数 | expand(i,i+1) 偶数 | 找到的回文 |
|------|------------------|--------------------|-----------| 
| i=0 | "a" → 1 | "aa" → 1 | a, aa |
| i=1 | "a" → 1 | "ab" 不等 → 0 | a |
| i=2 | "b" → 1 | 越界 → 0 | b |

总计：1 + 1 + 1 + 0 + 1 + 0 = **4**

### 为什么需要两次 expand

一个回文要么是奇数长度要么是偶数长度，中心不同：

```
expand(i, i)：   从一个字符往外扩 → 奇数长度
expand(i, i+1)： 从两个字符往外扩 → 偶数长度
```

只调一次会漏掉另一种情况。

---

## 解法二：动态规划

### dp 含义

`dp[i][j]` = s[i..j] 这个子串是否是回文（True/False）

### 三种情况

当 `s[i] == s[j]` 时：

| 情况 | 条件 | 示例 | 判断 |
|------|------|------|------|
| 单个字符 | j - i == 0 | "a" | 一定是回文 |
| 两个字符 | j - i == 1 | "aa" | 一定是回文 |
| 三个及以上 | j - i >= 2 | "aba" | 看 dp[i+1][j-1] |

合并情况 1 和 2：`j - i <= 1` 时直接为 True。

### 遍历顺序

`dp[i][j]` 依赖 `dp[i+1][j-1]`（左下方）：

```
    j →
i   . . . .
↓   . . . .
    . [依赖] .
    . . [当前] .
```

所以 **i 从下往上，j 从左往右**。

### 代码

```python
class Solution:
    def countSubstrings(self, s: str) -> int:
        n = len(s)
        dp = [[False] * n for _ in range(n)]
        count = 0

        for i in range(n - 1, -1, -1):       # i 从下往上
            for j in range(i, n):              # j 从 i 开始往右
                if s[i] == s[j]:
                    if j - i <= 1:             # 长度 1 或 2
                        dp[i][j] = True
                        count += 1
                    elif dp[i + 1][j - 1]:     # 长度 3+，看去掉两端是否回文
                        dp[i][j] = True
                        count += 1

        return count
```

### 模拟过程

s = `"aab"`，n = 3

填表顺序（i 从 2 到 0，j 从 i 到 2）：

| i | j | s[i]==s[j]? | 条件 | dp[i][j] | count |
|---|---|-------------|------|----------|-------|
| 2 | 2 | b==b ✓ | j-i=0 ≤ 1 | True | 1 |
| 1 | 1 | a==a ✓ | j-i=0 ≤ 1 | True | 2 |
| 1 | 2 | a==b ✗ | — | False | 2 |
| 0 | 0 | a==a ✓ | j-i=0 ≤ 1 | True | 3 |
| 0 | 1 | a==a ✓ | j-i=1 ≤ 1 | True | 4 |
| 0 | 2 | a==b ✗ | — | False | 4 |

结果：**4**

---

## 两种解法对比

| | 中心扩展 | DP |
|---|---|---|
| 思路 | 从中心往外扩 | 区间子问题 |
| 时间 | O(n²) | O(n²) |
| 空间 | O(1) | O(n²) |
| 代码量 | 更短 | 更长 |
| 优势 | 省空间，直观 | 能复用 dp 表做其他题 |
| 延伸 | 最长回文子串 | 最长回文子串、回文分割 |

面试推荐中心扩展法，简洁且空间 O(1)。DP 适合需要记录所有子串回文信息的场景（比如 Palindrome Partitioning）。

---

## 常见 Bug

| Bug | 后果 |
|-----|------|
| `dp` 初始化漏掉 `=` | 语法错误 |
| `j - 1 <= 1` 写成减数字 1 | 只有 j≤2 时才对，后面全错 |
| i 的范围从 `n` 开始而非 `n-1` | 索引越界 |
| 中心扩展只调一次 expand | 漏掉奇数或偶数长度的回文 |

---

## 复杂度

| | 时间 | 空间 |
|---|---|---|
| 中心扩展 | O(n²) | O(1) |
| DP | O(n²) | O(n²) |
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

**1. dp定义：** `dp[i]` = 凑出金额 `i` 所需的最少硬币数

**2. 递推公式：** `dp[i] = min(dp[i], dp[i - c] + 1)`，对每个硬币 `c`

**3. 初始化：** `dp[0] = 0`，其余 `inf`

**4. 遍历顺序：** 外层遍历硬币，内层正序遍历金额（完全背包）

**5. 举例验证：** coins=[1,2,5], amount=11 → dp[11]=3（5+5+1）

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0
        for c in coins:
            for i in range(c, amount + 1):
                dp[i] = min(dp[i], dp[i - c] + 1)
        return dp[amount] if dp[amount] != float('inf') else -1
```
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

## 核心思路

`dp[i][j]` = word1 前 i 个字符变成 word2 前 j 个字符的最少操作数。

三种操作：插入、删除、替换。

## 完整代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m = len(word1)
        n = len(word2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        for i in range(1, m + 1):
            dp[i][0] = i                  # 删 i 次
        for j in range(1, n + 1):
            dp[0][j] = j                  # 插 j 次

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if word1[i-1] == word2[j-1]:
                    dp[i][j] = dp[i-1][j-1]              # 相同，不操作
                else:
                    dp[i][j] = min(dp[i-1][j-1] + 1,     # 替换
                                   dp[i][j-1] + 1,        # 插入
                                   dp[i-1][j] + 1)        # 删除

        return dp[m][n]
```

## 为什么比较 word1[i-1] 和 word2[j-1] 而不是 word1[i] 和 word2[j]

dp 下标和字符串下标错开了一位：

```
dp 下标:    0    1    2    3
            ↑
            空   w    o    r
字符串下标:      0    1    2
```

`dp[i]` 表示前 i 个字符，最后一个字符是 `word1[i-1]`：

```
dp[0] → ""          没有字符
dp[1] → "w"         最后一个字符是 word1[0]
dp[2] → "wo"        最后一个字符是 word1[1]
```

多出的一行一列是为了 base case（空字符串）。几乎所有字符串 DP 都用这个错位写法。

## Base Case

```python
dp[i][0] = i   # word1 前 i 个字符 → 空串：删 i 次
dp[0][j] = j   # 空串 → word2 前 j 个字符：插 j 次
```

这是真实答案，不是占位符，所以初始化用 0 不用 inf。每个格子都会被按顺序算到，不存在"读到未计算的格子"的情况。

## 状态转移详解

```
             word2
              j-1  j
word1  i-1  [ 左上  上 ]     左上 dp[i-1][j-1] + 1 → 替换
       i    [ 左   当前]     左   dp[i][j-1] + 1   → 插入
                              上   dp[i-1][j] + 1   → 删除
```

字符相同时，左上不加 1（不需要操作）。

## 走一遍示例

word1 = "horse", word2 = "ros"

```
     ""  r  o  s
""    0  1  2  3
h     1  1  2  3
o     2  2  1  2
r     3  2  2  2
s     4  3  3  2
e     5  4  4  3  ← 答案
```

以 dp[1][1] 为例：h → r

```
h ≠ r，取 min：
  dp[0][0] + 1 = 1  替换 h→r
  dp[1][0] + 1 = 2  删除 h 再插入 r
  dp[0][1] + 1 = 2  插入 r
min = 1
```

## 易错点

**忘了区分字符相同和不同：**

```python
# 错误：不相等时 dp[i-1][j-1] 没加 1
dp[i][j] = min(dp[i-1][j-1], dp[i][j-1]+1, dp[i-1][j]+1)

# 正确：不相等时替换也是一次操作
if word1[i-1] == word2[j-1]:
    dp[i][j] = dp[i-1][j-1]
else:
    dp[i][j] = min(dp[i-1][j-1]+1, dp[i][j-1]+1, dp[i-1][j]+1)
```

## 复杂度

- 时间：O(m × n)
- 空间：O(m × n)，可优化到 O(n) 用滚动数组

## 115 Distinct Subsequences

## 题意

给定字符串 `s` 和 `t`，求 `s` 的子序列中等于 `t` 的个数。

---

## 思路：0/1 背包类比

把 `s` 的每个字符看作"物品"，`t` 看作"目标"，每个字符选或不选：

- **选**：`s[j]` 和 `t[i-1]` 匹配，方案数从 `dp[i-1]` 转移过来
- **不选**：跳过这个字符，`dp[i]` 保持不变

和 0/1 背包一样，压缩成一维后需要**倒序遍历**，防止同一轮的结果被覆盖。

---

## dp 数组含义

`dp[i]` = 用已遍历的 `s` 的字符，匹配 `t` 的前 `i` 个字符的方案数

| dp 下标 | 对应 t 的内容 | 初始值 |
|---------|--------------|--------|
| `dp[0]` | 空串 | 1（空串总能匹配） |
| `dp[1]` | `t[0]` | 0 |
| `dp[2]` | `t[1]` | 0 |
| `dp[i]` | `t[i-1]` | 0 |

### 为什么比较 `t[i-1]` 而不是 `t[i]`？

因为 `dp[0]` 代表空串，整个数组往后偏移了一位。`dp[i]` 对应的是 t 的第 i 个字符，下标是 `i-1`。

---

## 代码

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        dp = [0] * (len(t) + 1)
        dp[0] = 1

        for ss in s:
            for i in range(len(t), 0, -1):   # 倒序，0/1 背包套路
                if ss == t[i - 1]:            # dp[i] 对应 t[i-1]
                    dp[i] += dp[i - 1]

        return dp[len(t)]                     # 匹配整个 t
```

---

## 易错点

1. **索引偏移**：`dp[i]` 对应 `t[i-1]`，不是 `t[i]`
2. **返回值**：返回 `dp[len(t)]`，不是 `dp[len(t)-1]`
3. **倒序遍历**：正序会导致同一个字符被重复使用，变成完全背包
## 53 Maximum Subarray
## 55 Jump Game
# LeetCode 55 — Jump Game

## 题目概述

给定数组 `nums`，每个元素表示从该位置最多能跳几步。从位置 0 出发，判断能否到达最后一个位置。

```
输入：nums = [2,3,1,1,4]
输出：true（0→1→4 或 0→2→3→4）

输入：nums = [3,2,1,0,4]
输出：false（被位置3的0卡住）
```

---

## 核心思路：贪心

维护一个变量 `cover`，表示当前能到达的最远位置。遍历每个位置，如果能到达就更新 `cover`。

```
每一步问自己：
1. 我能到这里吗？（i <= cover）
2. 从这里出发能到更远吗？（更新 cover）
3. 够到终点了吗？（cover >= len - 1）
```

---

## 代码

```python
def canJump(self, nums):
    cover = 0
    for i in range(len(nums)):
        if i <= cover:
            cover = max(i + nums[i], cover)
        if cover >= len(nums) - 1:
            return True
    return False
```

---

## 易错点：`i <= cover` 不是 `i < cover`

当 `i == cover` 时，你站在能到达的最远位置上，还应该尝试从这里跳。

例子 `nums = [2, 0, 0]`：

```
用 i < cover：
  i=0: 0 < 0? 不满足，跳过 → 返回 False ✗

用 i <= cover：
  i=0: 0 <= 0? 满足，cover = max(0+2, 0) = 2 → 返回 True ✓
```

---

## 演练

`nums = [2, 3, 1, 1, 4]`

| i | i <= cover? | cover 更新 | cover >= 4? |
|---|------------|-----------|------------|
| 0 | 0 <= 0 ✓ | max(2, 0) = 2 | 2 >= 4? ✗ |
| 1 | 1 <= 2 ✓ | max(4, 2) = 4 | 4 >= 4? ✓ return True |

`nums = [3, 2, 1, 0, 4]`

| i | i <= cover? | cover 更新 | cover >= 4? |
|---|------------|-----------|------------|
| 0 | 0 <= 0 ✓ | max(3, 0) = 3 | ✗ |
| 1 | 1 <= 3 ✓ | max(3, 3) = 3 | ✗ |
| 2 | 2 <= 3 ✓ | max(3, 3) = 3 | ✗ |
| 3 | 3 <= 3 ✓ | max(3, 3) = 3 | ✗ |
| 4 | 4 <= 3 ✗ | 跳过 | ✗ |
| 循环结束 → return False |

---

## 复杂度

- **时间**：O(n) — 遍历一次
- **空间**：O(1) — 只用一个变量

## 相关题目

- **LeetCode 45** — Jump Game II（最少跳几次到终点，贪心进阶）
- **LeetCode 1306** — Jump Game III（能否到达值为 0 的位置，BFS/DFS）
- **LeetCode 1345** — Jump Game IV（最少跳几次，BFS）

## 45 Jump Game II
## 134 Gas Station
## 846 Hand of Straights
## 1899 Merge Triplets to Form Target Triplet
## 763 Partition Labels
## 678 Valid Parenthesis Str


## 56 Merge Intervals
## 57 Insert Interval
## 题目描述

给定一个**无重叠**且**按起点升序排列**的区间列表 `intervals`，以及一个新区间 `newInterval`。

将新区间插入列表中，确保列表仍然有序且无重叠（如有必要，合并重叠区间）。

---

## 核心思路

遍历每个区间，对于每个 `intervals[i]`，只有三种情况：

```
情况1：新区间在左边，完全不重叠
        newInterval:  [1, 2]
        intervals[i]:          [3, 5]
        → 直接插入 newInterval，后面全部追加，结束

情况2：新区间在右边，完全不重叠
        newInterval:          [4, 6]
        intervals[i]:  [1, 2]
        → 保留 intervals[i]，继续遍历

情况3：重叠，需要合并
        newInterval:  [2, 5]
        intervals[i]:    [3, 7]
        → 合并为 [min(2,3), max(5,7)] = [2, 7]
```

---

## 判断条件

| 条件 | 含义 | 操作 |
|------|------|------|
| `newInterval[1] < intervals[i][0]` | 新区间完全在左 | 插入并返回 |
| `newInterval[0] > intervals[i][1]` | 新区间完全在右 | 保留当前区间 |
| 其他 | 重叠 | 合并 |

---

## 代码实现

```python
class Solution:
    def insert(self, intervals: List[List[int]], newInterval: List[int]) -> List[List[int]]:
        res = []
        for i in range(len(intervals)):
            # 情况1：新区间完全在当前区间左边，直接插入，后面全部追加
            if newInterval[1] < intervals[i][0]:
                res.append(newInterval)
                return res + intervals[i:]
            # 情况2：新区间完全在当前区间右边，保留当前区间
            elif newInterval[0] > intervals[i][1]:
                res.append(intervals[i])
            # 情况3：重叠，合并
            else:
                newInterval = [min(newInterval[0], intervals[i][0]),
                               max(newInterval[1], intervals[i][1])]
        # 遍历结束，新区间还没插入（在最右边或一直在合并）
        res.append(newInterval)
        return res
```

---

## 举例推导

```
intervals = [[1,3],[6,9]]，newInterval = [2,5]
```

**i=0，intervals[0]=[1,3]：**
```
newInterval[1]=5 >= intervals[0][0]=1  → 不是情况1
newInterval[0]=2 <= intervals[0][1]=3  → 不是情况2
→ 情况3，合并：newInterval = [min(2,1), max(5,3)] = [1, 5]
```

**i=1，intervals[1]=[6,9]：**
```
newInterval[1]=5 < intervals[1][0]=6  → 情况1！
→ res.append([1,5])，return [[1,5],[6,9]]
```

**输出：** `[[1,5],[6,9]] ✓`

---

## 再看一个例子

```
intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]]，newInterval = [4,8]
```

| i | intervals[i] | 情况 | 操作 |
|---|-------------|------|------|
| 0 | [1,2] | 新区间在右 | res=[[1,2]] |
| 1 | [3,5] | 重叠 | newInterval=[3,8] |
| 2 | [6,7] | 重叠 | newInterval=[3,8] |
| 3 | [8,10] | 重叠 | newInterval=[3,10] |
| 4 | [12,16] | 新区间在左 | 插入并返回 |

**输出：** `[[1,2],[3,10],[12,16]] ✓`

---

## 复杂度分析

| 类型 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | $O(n)$ | 只遍历一次 |
| 空间复杂度 | $O(n)$ | 结果数组 |

---

## 关键细节

- 循环结束后必须 `res.append(newInterval)`，因为有两种情况新区间还未插入：
  1. 新区间一直在右边，遍历完还没插入
  2. 新区间一直在合并，合并结果还未加入 `res`

## 435 Non-overlapping Intervals
# 435. Non-overlapping Intervals（无重叠区间）

## 题目

给定一个区间数组 `intervals`，返回需要移除的最少区间数量，使剩余区间互不重叠。

## 解题思路

**贪心策略**：按起点排序后，遇到重叠时，保留结束更早的区间（给后续区间留更多空间）。

### 关键步骤

1. **按起点排序** `intervals.sort(key=lambda x: x[0])`
2. **遍历检测重叠**：若当前区间的起点 < 前一个区间的终点，说明重叠
3. **贪心选择**：重叠时移除结束更晚的那个（保留终点更小的），计数 +1

### 为什么保留结束更早的区间？

```
区间A: [1, 10]
区间B: [2, 3]
区间C: [4, 6]
```

保留 A（结束晚）→ 只能放 A，移除 B 和 C
保留 B（结束早）→ 可以放 B 和 C，只移除 A ✅

## 代码

```python
class Solution:
    def eraseOverlapIntervals(self, intervals: List[List[int]]) -> int:
        res = 0
        intervals.sort(key=lambda x: x[0])
        for i in range(1, len(intervals)):
            if intervals[i][0] < intervals[i - 1][1]:  # 检测重叠
                res += 1
                intervals[i][1] = min(intervals[i - 1][1], intervals[i][1])  # 保留更早结束的
        return res
```

## 逐行解析

| 代码 | 作用 |
|------|------|
| `intervals.sort(key=lambda x: x[0])` | 按起点升序排列 |
| `intervals[i][0] < intervals[i-1][1]` | 当前起点 < 前一个终点 → 重叠 |
| `res += 1` | 需要移除一个区间 |
| `intervals[i][1] = min(...)` | 把当前终点更新为两者中较小的（等价于保留结束更早的区间） |

## 模拟示例

输入：`[[1,2], [2,3], [3,4], [1,3]]`

排序后：`[[1,2], [1,3], [2,3], [3,4]]`

```
i=1: [1,3] 起点 1 < 前一个终点 2 → 重叠，res=1，更新终点为 min(2,3)=2
i=2: [2,3] 起点 2 ≥ 前一个终点 2 → 不重叠
i=3: [3,4] 起点 3 ≥ 前一个终点 3 → 不重叠
```

输出：`1` ✅

## 复杂度

- **时间**：O(n log n)（排序）
- **空间**：O(1)（原地修改）

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

## 题意

反复对 `n` 的每一位数字求平方和，如果最终能变成 1 就是快乐数，否则会陷入无限循环。

---

## 解法一：HashSet 检测循环

最直观的思路——记录见过的数，重复出现就说明进入了环。

```python
def isHappy(self, n: int) -> bool:
    visit = set()

    def sum_digit(n):
        output = 0
        while n:
            digit = n % 10
            output += digit * digit
            n //= 10
        return output

    while n not in visit:
        visit.add(n)
        n = sum_digit(n)
        if n == 1:
            return True
    return False
```

时间 O(n)，空间 O(n)。

---

## 解法二：快慢指针（优化空间）

### 关键洞察：这就是一条链

每次 `sum_digit` 的结果就是"下一个节点"，整条链一定有环：

```
快乐数:   19 → 82 → 68 → 100 → 1 → 1 → 1 → ...（1 自循环）
不快乐数:  2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 → ...（回到 4）
```

所以问题变成：**环里的值是不是 1？**

### 为什么不需要像 142 题那样回起点再走一次？

- 142 题（链表环入口）：需要找**环的入口在哪**，所以相遇后要回起点再走一次来定位
- 本题：只关心**环里是不是 1**，不关心入口位置。1 如果在环里一定是自循环（`1 → 1`），fast 一定会经过 1，相遇时看值就够了

### 代码

```python
def isHappy(self, n: int) -> bool:
    def sum_digit(n):
        output = 0
        while n:
            digit = n % 10
            output += digit * digit
            n //= 10
        return output

    slow, fast = n, sum_digit(n)
    while fast != 1 and slow != fast:
        slow = sum_digit(slow)
        fast = sum_digit(sum_digit(fast))
    return fast == 1
```

时间 O(n)，空间 O(1)。

---

## 易错点

1. fast 初始值是 `sum_digit(n)` 而不是 `n`，否则 `slow == fast` 一开始就成立直接退出
2. 循环条件要同时检查 `fast != 1` 和 `slow != fast`，少一个都可能死循环或漏判
## 66 Plus One
## 50 Pow(x, n)

## 136 Single Number
## 4. Single Number（只出现一次的数字）

**思路：位运算 XOR**

利用 XOR 的两个性质：
- `a ^ a = 0`（相同的数异或为 0）
- `a ^ 0 = a`（任何数与 0 异或为本身）

所有出现两次的数自己和自己抵消，最后只剩单独的数。

```python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        res = 0
        for n in nums:
            res = res ^ n
        return res
```

**示例：** `[4, 1, 2, 1, 2]`

```
0 ^ 4 = 4
4 ^ 1 = 5
5 ^ 2 = 7
7 ^ 1 = 6   # 两个 1 抵消
6 ^ 2 = 4   # 两个 2 抵消
返回 4 ✅
```

| | 时间复杂度 | 空间复杂度 |
|--|-----------|-----------|
| HashMap 计数 | O(n) | O(n) |
| XOR（最优） | O(n) | O(1) |

---

## 191 Number of 1 Bits

## 核心思路

数二进制里有多少个 1。

## 解法一：逐位检查

每次检查最后一位，然后右移去掉它。

```python
class Solution:
    def hammingWeight(self, n: int) -> int:
        count = 0
        while n:
            if n & 1:       # 最后一位是1吗
                count += 1
            n >>= 1         # 右移，去掉最后一位
        return count
```

### 走一遍

```
n = 13 (1101)

n & 1 = 1 ✓  count=1    n >>= 1 → 110 (6)
n & 1 = 0    count=1    n >>= 1 → 11  (3)
n & 1 = 1 ✓  count=2    n >>= 1 → 1   (1)
n & 1 = 1 ✓  count=3    n >>= 1 → 0

n=0，退出，返回 3
```

### 易错点

```python
if n &= 1:   # 错误：&= 是赋值，会把 n 改掉
if n & 1:    # 正确：只检查，不修改 n

n >= 1       # 错误：这是比较，不做任何事
n >>= 1      # 正确：右移赋值
```

## 解法二：n & (n-1) 去掉最低位的 1（更优）

每次直接去掉一个 1，循环次数 = 1 的个数。

```python
class Solution:
    def hammingWeight(self, n: int) -> int:
        count = 0
        while n:
            n = n & (n - 1)   # 去掉最低位的 1
            count += 1
        return count
```

### 走一遍

```
n = 13 (1101)

n & (n-1) = 1101 & 1100 = 1100 (12)   count=1  去掉了最右边的1
n & (n-1) = 1100 & 1011 = 1000 (8)    count=2  去掉了下一个1
n & (n-1) = 1000 & 0111 = 0000 (0)    count=3  去掉了最后一个1

n=0，退出，返回 3
```

### 为什么 n & (n-1) 能去掉最低位的 1

n-1 会把最低位的 1 变成 0，它右边的 0 全变成 1，AND 后刚好抹掉那个 1：

```
n:    1 1 0 0   (12)
n-1:  1 0 1 1   (11)  ← 最低位的1变0，右边全变1
AND:  1 0 0 0   (8)   ← 最低位的1没了
```

## 两种解法对比

| | 逐位检查 | n & (n-1) |
|---|---|---|
| 循环次数 | 总位数（32次） | 1 的个数 |
| 时间 | O(32) | O(k)，k 是 1 的个数 |
| 难度 | 直觉简单 | 需要理解 n & (n-1) |

## 338 Counting Bits
## 268 Missing Number
# 268. Missing Number（丢失的数字）

## 题目

给定包含 `0` 到 `n` 中 n 个数的数组 `nums`，找出缺失的那个数。

## 核心思路

完整的总和 - 实际的总和 = 缺失的数

```
应该有：0 + 1 + 2 + ... + n
实际有：nums[0] + nums[1] + ... + nums[n-1]
缺失的 = 应该有的 - 实际有的
```

## 代码

```python
class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        res = len(nums)
        for i in range(len(nums)):
            res += i - nums[i]
        return res
```

## 为什么这样写

展开循环：

```
res = n + (0 - nums[0]) + (1 - nums[1]) + ... + (n-1 - nums[n-1])
    = (n + 0 + 1 + ... + n-1) - (nums[0] + nums[1] + ... + nums[n-1])
    = (0 + 1 + ... + n) - sum(nums)
```

`res` 初始化为 `n` 是因为循环 `range(len(nums))` 只加到 `n-1`，需要提前把 `n` 放进去。

## 逐行解析

| 代码 | 作用 |
|------|------|
| `res = len(nums)` | 先放入 n（循环只到 n-1） |
| `res += i - nums[i]` | 每步加上"应该有的 - 实际有的" |
| `return res` | 累积的差值就是缺失的数 |

## 模拟示例

`nums = [3, 0, 1]`，n = 3，缺失的是 2

```
res = 3
i=0: res += 0 - 3 = -3  → res = 0
i=1: res += 1 - 0 = 1   → res = 1
i=2: res += 2 - 1 = 1   → res = 2 ✅
```

## 复杂度

- **时间**：O(n)
- **空间**：O(1)

## 其他解法

### 数学公式

```python
def missingNumber(self, nums):
    n = len(nums)
    return n * (n + 1) // 2 - sum(nums)
```

直接用等差数列求和公式，更直观。

### 异或（XOR）

```python
def missingNumber(self, nums):
    res = len(nums)
    for i in range(len(nums)):
        res ^= i ^ nums[i]
    return res
```

利用 `a ^ a = 0` 的性质，相同的数异或抵消，剩下的就是缺失的。

## 371 Sum of Two Integers
## 190 Reverse Bits