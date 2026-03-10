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