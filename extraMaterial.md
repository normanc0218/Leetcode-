## 1929 Concatenation of Array
## 14 Longest Common Prefix
## 27 Remove Element
## 169 Majority Element

## 5. Majority Element（多数元素）

**思路一：HashMap 计数（最直觉）**

遍历数组统计次数，找到出现超过 `n//2` 次的元素。

```python
def majorityElement(self, nums):
    count = {}
    for n in nums:
        count[n] = count.get(n, 0) + 1
        if count[n] > len(nums) // 2:
            return n
```

时间 O(n)，空间 O(n)

---

**思路二：排序**

排序后中间位置一定是多数元素（因为它超过一半）。

```python
def majorityElement(self, nums):
    nums.sort()
    return nums[len(nums) // 2]
```

时间 O(n log n)，空间 O(1)

---

**思路三：Boyer-Moore 投票算法（最优）**

多数元素和其他所有元素互相抵消，最后剩下的一定是它。

```python
def majorityElement(self, nums):
    candidate, count = None, 0
    for n in nums:
        if count == 0:
            candidate = n       # 选当前元素为候选
        count += 1 if n == candidate else -1
    return candidate
```

**模拟一遍** `[2, 2, 1, 1, 2]`：

```
n=2: count=0 → candidate=2, count=1
n=2:                        count=2
n=1:                        count=1  # 抵消
n=1:                        count=0  # 抵消
n=2: count=0 → candidate=2, count=1
返回 2 ✅
```

**三种方法对比：**

| 方法 | 时间 | 空间 | 核心思想 |
|------|------|------|---------|
| HashMap | O(n) | O(n) | 直接计数 |
| 排序 | O(n log n) | O(1) | 中位数必是答案 |
| Boyer-Moore | O(n) | O(1) | 多数派抵消少数派后必剩余 |

---
## 705 Design HashSet
## 706 Design HashMap
## 912 Sort an Array
## 75 Sort Colors

## 题目概述

给定数组 `nums`，只包含 0（红）、1（白）、2（蓝），原地排序使得相同颜色相邻，顺序为 0 → 1 → 2。

```
输入：[2, 0, 2, 1, 1, 0]
输出：[0, 0, 1, 1, 2, 2]
```

---

## 解法一：计数排序（两遍扫描）

数一下 0、1、2 各有几个，覆盖回去。

```python
def sortColors(self, nums):
    count = [0, 0, 0]
    for n in nums:
        count[n] += 1
    i = 0
    for color in range(3):
        for _ in range(count[color]):
            nums[i] = color
            i += 1
```

- 时间：O(n)，但遍历两遍
- 空间：O(1)

---

## 解法二：三指针 — 荷兰国旗（一遍扫描）

### 思路

维护三个指针，把数组分成四个区域：

```
[0, lo)    → 全是 0
[lo, mid)  → 全是 1
[mid, hi]  → 未处理
(hi, end]  → 全是 2
```

`mid` 从左往右扫，根据当前值做不同操作：

- `nums[mid] == 0` → 和 `lo` 交换，`lo++`，`mid++`
- `nums[mid] == 1` → 已在正确区域，`mid++`
- `nums[mid] == 2` → 和 `hi` 交换，`hi--`，**mid 不动**

### 为什么遇到 2 时 mid 不动？

关键记忆：**换过来的值见过就走，没见过就留。**

- 和 `lo` 交换：`lo` 左边 `mid` 都走过，换过来的只可能是 1，见过的，`mid++`
- 和 `hi` 交换：`hi` 右边是未处理区域，换过来的可能是 0、1、2，没见过，`mid` 不动

### 代码

```python
def sortColors(self, nums):
    lo, mid, hi = 0, 0, len(nums) - 1
    while mid <= hi:
        if nums[mid] == 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi -= 1
            # mid 不动！
```

### 演练

`[2, 0, 2, 1, 1, 0]`

| 步骤 | lo | mid | hi | 操作 | 数组 |
|------|-----|------|-----|------|------|
| 1 | 0 | 0 | 5 | nums[0]=2，和 hi 换，hi-- | [0,0,2,1,1,2] |
| 2 | 0 | 0 | 4 | nums[0]=0，和 lo 换，lo++,mid++ | [0,0,2,1,1,2] |
| 3 | 1 | 1 | 4 | nums[1]=0，和 lo 换，lo++,mid++ | [0,0,2,1,1,2] |
| 4 | 2 | 2 | 4 | nums[2]=2，和 hi 换，hi-- | [0,0,1,1,2,2] |
| 5 | 2 | 2 | 3 | nums[2]=1，mid++ | [0,0,1,1,2,2] |
| 6 | 2 | 3 | 3 | nums[3]=1，mid++ | [0,0,1,1,2,2] |
| mid > hi，结束 ✓ |

### 复杂度

- 时间：O(n)，一遍扫描
- 空间：O(1)

---

## 解法三：覆盖法 — 层层涂色（一遍扫描）

### 思路

三个指针 `zero`、`one`、`two`，`two` 是主循环负责扫描，`zero` 和 `one` 是跟随指针。

每到一个位置，先假设是 2，再根据实际值逐层覆盖：

```
第一步：nums[two] = 2          → 先涂蓝
第二步：如果 tmp < 2，nums[one] = 1  → 再涂白（覆盖一个 2）
第三步：如果 tmp < 1，nums[zero] = 0 → 再涂红（覆盖一个 1）
```

### 跟随指针的速度

`tmp` 的值决定谁跟上来：

```
tmp = 2 → 谁都不跟
tmp = 1 → one 跟一步
tmp = 0 → one 跟一步，zero 也跟一步
```

速度关系永远是 `zero ≤ one ≤ two`，自然把数组分成三段。

### 代码

```python
def sortColors(self, nums):
    zero = one = 0
    for two in range(len(nums)):
        tmp = nums[two]
        nums[two] = 2
        if tmp < 2:
            nums[one] = 1
            one += 1
        if tmp < 1:
            nums[zero] = 0
            zero += 1
```

### 演练

`[2, 0, 2, 1, 1, 0]`

| two | tmp | 写 2 | 写 1 | 写 0 | zero | one | 数组 |
|-----|-----|------|------|------|------|-----|------|
| 0 | 2 | [0]=2 | — | — | 0 | 0 | [2,0,2,1,1,0] |
| 1 | 0 | [1]=2 | [0]=1 | [0]=0 | 1 | 1 | [0,2,2,1,1,0] |
| 2 | 2 | [2]=2 | — | — | 1 | 1 | [0,2,2,1,1,0] |
| 3 | 1 | [3]=2 | [1]=1 | — | 1 | 2 | [0,1,2,2,1,0] |
| 4 | 1 | [4]=2 | [2]=1 | — | 1 | 3 | [0,1,1,2,2,0] |
| 5 | 0 | [5]=2 | [3]=1 | [1]=0 | 2 | 4 | [0,0,1,1,2,2] |

结果：`[0, 0, 1, 1, 2, 2]` ✓

### 复杂度

- 时间：O(n)，一遍扫描
- 空间：O(1)

---

## 三种解法对比

| | 计数排序 | 三指针（荷兰国旗） | 覆盖法（层层涂色） |
|--|---------|----------------|--------------|
| 扫描次数 | 两遍 | 一遍 | 一遍 |
| 操作方式 | 数数再覆盖 | 交换 | 直接覆盖 |
| 易错点 | 无 | mid 遇到 2 不能 ++ | 无 |
| 面试推荐 | 保底方案 | 经典考点 | 加分写法 |

## 相关题目

- **LeetCode 283** — Move Zeroes（双指针，把 0 移到末尾）
- **LeetCode 215** — Kth Largest Element（快速选择，用到类似分区思想）
- **LeetCode 324** — Wiggle Sort II（三向切分的应用）

## 304 Range Sum Query 2D Immutable
## 122 Best Time to Buy And Sell Stock II
## 229 Majority Element II
## 560 Subarray Sum Equals K
## 41 First Missing Positive
## 43 Multiply String

**考点：手动模拟竖式乘法**（不能用 `int()` 直接转换）

---

### 如何提取数字字符的值

```python
ord('5') - ord('0')  # → 5
```

ASCII 表中 `'0'`~`'9'` 连续排列（48~57），任意数字字符减去 `'0'` 即得对应整数。

---

### 核心：位置关系

`num1[i]` × `num2[j]` 的结果一定落在 `res[i+j+1]`，进位传到 `res[i+j]`。

```
    1 2 3      index: 0 1 2
  ×   4 5      index:   0 1
  -------
结果最多 5 位   index: 0 1 2 3 4

num1[2]='3' × num2[1]='5' → res[2+1+1] = res[4]  ✅ 个位
num1[2]='3' × num2[0]='4' → res[2+0+1] = res[3]  ✅ 十位
```

`+1` 的原因：结果数组第 0 位预留给最高位进位，所以所有位置整体右移一位。

---

### 代码

```python
class Solution:
    def multiply(self, num1: str, num2: str) -> str:
        m, n = len(num1), len(num2)
        res = [0] * (m + n)  # 列表，不能用字符串（字符串不可变）

        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                d1 = ord(num1[i]) - ord('0')
                d2 = ord(num2[j]) - ord('0')

                pos = i + j + 1
                res[pos] += d1 * d2
                res[pos - 1] += res[pos] // 10  # 进位传到上一位
                res[pos] %= 10                   # 当前位只保留个位

        result = ''.join(map(str, res)).lstrip('0')
        return result if result else '0'
```

---

### 常见 Bug

**Bug 1：用字符串存结果**
```python
res = '0' * (m + n)
res[k] = ...  # ❌ 字符串不支持下标赋值，必须用列表
```

**Bug 2：用全局 k 递减代替固定位置**
```python
# ❌ 错误：k 是全局递减的，无法对应正确的乘法位置
k = m + n - 1
res[k] = (d1 * d2) % 10
k -= 1

# ✅ 正确：位置由 i+j+1 唯一确定
pos = i + j + 1
res[pos] += d1 * d2
```

**Bug 3：进位时机错误**

每对 `(i,j)` 相乘后应**立即**将进位传给 `res[pos-1]`，不能用全局 carry 在外层处理，因为多对乘法会累加到同一个位置。

**Bug 4：忘记处理全零情况**
```python
# lstrip('0') 会把 "0000" 变成 ""，需要兜底
return result if result else '0'
```

| | 时间复杂度 | 空间复杂度 |
|--|-----------|-----------|
| Multiply Strings | O(m × n) | O(m + n) |

---
## 344 Reverse String

## 题目

原地反转字符数组，不能返回新数组。

## 常见错误

```python
s = s[::-1]  # ❌ 创建新 list，局部变量指向新 list，原 list 不变
```

```
调用前：外部 s → [h, e, l, l, o]
函数内：s = s[::-1]
        局部 s → [o, l, l, e, h]（新 list）
        外部 s → [h, e, l, l, o]（没变）
```

`s =` 是改变局部变量的指向，`s[:] =` 才是修改原 list 的内容。

## 解法：双指针

左右两个指针，相向而行，逐个交换。

```python
class Solution:
    def reverseString(self, s: List[str]) -> None:
        l, r = 0, len(s) - 1
        while l < r:
            s[l], s[r] = s[r], s[l]
            l += 1
            r -= 1
```

## 模拟示例

`s = ["h", "e", "l", "l", "o"]`

```
l=0, r=4: 交换 h ↔ o → [o, e, l, l, h]
l=1, r=3: 交换 e ↔ l → [o, l, l, e, h]
l=2, r=2: l == r，停止
```

结果：`["o", "l", "l", "e", "h"]` ✅

## `s =` vs `s[:] =`

| 写法 | 效果 |
|------|------|
| `s = s[::-1]` | 局部变量指向新 list，原 list 不变 ❌ |
| `s[:] = s[::-1]` | 原 list 内容被替换 ✅ |
| 双指针交换 | 真正的原地修改 ✅ |

## 复杂度

- **时间**：O(n)
- **空间**：O(1)（原地修改）

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
# 删除有序数组中的重复项 — 完整讲解

## 题目描述

给你一个**非严格递增排列**的整数数组 `nums`，请你**原地**删除重复出现的元素，使每个元素只出现一次，返回删除后数组的新长度。

---

## 代码

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        l = 1
        for r in range(1, len(nums)):
            if nums[r] != nums[r-1]:
                nums[l] = nums[r]
                l += 1
        return l
```

---

## 核心思路：双指针

用两个指针 `l`（左/慢）和 `r`（右/快）：

- `r` 负责**遍历扫描**整个数组
- `l` 负责**记录下一个不重复元素应该放的位置**

> 因为数组已经有序，**重复的元素一定相邻**，只需比较 `nums[r]` 和 `nums[r-1]` 是否相同即可。

---

## 为什么 `l` 从 1 开始？

第一个元素 `nums[0]` 一定保留，不需要动它，所以 `l` 直接从位置 1 开始准备接收新元素。

---

## 用例子走一遍

输入：`nums = [1, 1, 2, 3, 3]`

初始状态：`l = 1`，`r` 从 1 开始遍历

| r | nums[r] | nums[r-1] | 是否不同 | 操作 | l | 数组状态 |
|---|---------|-----------|----------|------|---|----------|
| 1 | 1 | 1 | ❌ 相同 | 跳过 | 1 | [1, 1, 2, 3, 3] |
| 2 | 2 | 1 | ✅ 不同 | nums[1]=2, l++ | 2 | [1, 2, 2, 3, 3] |
| 3 | 3 | 2 | ✅ 不同 | nums[2]=3, l++ | 3 | [1, 2, 3, 3, 3] |
| 4 | 3 | 3 | ❌ 相同 | 跳过 | 3 | [1, 2, 3, 3, 3] |

返回 `l = 3`，有效部分为 `nums[0:3] = [1, 2, 3]` ✓

---

## 为什么比较 `nums[r]` 和 `nums[r-1]` 而不是 `nums[l-1]`？

两种写法都正确，效果等价：

```python
# 写法一：和前一个位置比（本题代码）
if nums[r] != nums[r-1]:

# 写法二：和慢指针前一位比
if nums[r] != nums[l-1]:
```

因为数组有序，`r` 是连续往前走的，`nums[r-1]` 就是上一个扫描过的值，和 `nums[l-1]` 在逻辑上等价。

---

## 易错点

```python
# ❌ l 从 0 开始
l = 0
for r in range(1, len(nums)):
    if nums[r] != nums[r-1]:
        nums[l] = nums[r]   # 会覆盖掉 nums[0]！
        l += 1
```

`nums[0]` 是第一个元素，必须保留，`l` 必须从 1 开始，给它留出位置。

---

## 复杂度分析

| | 复杂度 |
|---|---|
| 时间 | O(n)，r 遍历一次数组 |
| 空间 | O(1)，原地操作，无额外空间 |

---

## 扩展：如果每个元素最多保留 k 次？

把条件改为和 `l-k` 位置比较即可：

```python
# 每个元素最多保留 2 次
def removeDuplicates(self, nums):
    l = 0
    for r in range(len(nums)):
        if l < 2 or nums[r] != nums[l-2]:
            nums[l] = nums[r]
            l += 1
    return l
```

本题是 k=1 的特殊情况。

## 18 4Sum
## 189 Rotate Array
## 881 Boats to Save People
## 219 Contains Duplicate II
**考点：滑动窗口 + HashSet**

判断是否存在两个下标 `i, j` 使得 `nums[i] == nums[j]` 且 `|i - j| <= k`。

---

### 暴力解（可通过，非最优）

```python
class Solution:
    def containsNearbyDuplicate(self, nums: List[int], k: int) -> bool:
        for l in range(len(nums)):
            for r in range(l + 1, min(len(nums), l + k + 1)):
                if nums[l] == nums[r]:
                    return True
        return False
```

时间 O(n×k)，空间 O(1)

---

### 最优解：滑动窗口 + HashSet

维护一个大小为 k 的窗口，用 HashSet 存窗口内元素，新元素进来前先查重。

```python
class Solution:
    def containsNearbyDuplicate(self, nums: List[int], k: int) -> bool:
        window = set()
        l = 0
        for r in range(len(nums)):
            if r - l > k:             # 窗口超过大小 k，移除最左边
                window.remove(nums[l])
                l += 1
            if nums[r] in window:     # 新元素已在窗口内，找到重复
                return True
            window.add(nums[r])
        return False
```

**窗口大小判断：**

题目要求 `|i - j| <= k`，窗口最多容纳 k+1 个元素（包含两端），所以 `r - l > k` 时需要收缩窗口。

| | 时间复杂度 | 空间复杂度 |
|--|-----------|-----------|
| 暴力双循环 | O(n×k) | O(1) |
| 滑动窗口 + Set | O(n) | O(k) |

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

## 题目概述

给定整数数组 `nums` 和整数 `k`，将数组分成 `k` 个非空连续子数组，使得所有子数组的和的最大值**最小化**。

```
输入：nums = [7,2,5,10,8], k = 2
输出：18

分法：[7,2,5] 和 [10,8] → max(14, 18) = 18（最优）
```

---

## 解法一：二分答案 + 贪心验证（推荐）

### 思维路径

```
正面想怎么切 → 组合太多，太复杂
    ↓
换角度：猜一个上限 mid，验证能不能做到
    ↓
发现验证很简单（贪心扫一遍）
    ↓
发现 mid 有单调性（越大越容易满足）
    ↓
二分搜索找最小的可行 mid
```

### 关键洞察：二分答案 ≠ 二分查找

二分需要的不是"数组排好序"，而是**搜索空间有单调性**。

| | 二分查找 | 二分答案 |
|--|---------|---------|
| 搜索空间 | 排好序的数组 | 答案的取值范围 |
| 需要排序吗 | 需要 | 不需要，数字天然有序 |
| 单调性来自 | 数组有序 | 可行性的单调变化 |

本题答案范围 `[max(nums), sum(nums)]`，天然有序：

```
mid:    10   11   12   ...  17   18   19   ...  32
可行：   ✗    ✗    ✗   ...   ✗    ✓    ✓   ...   ✓
        ←——— 红色（不可行）——→ ←—— 蓝色（可行）——→
                              ↑
                          答案在这里（红蓝边界）
```

### 验证函数（贪心）

规定每段和不超过 `mid`，从左到右贪心地塞，塞不下就切一刀：

```python
def can_split(nums, mid, k):
    count = 1       # 至少一段
    curr_sum = 0
    for num in nums:
        if curr_sum + num > mid:
            count += 1      # 切一刀
            curr_sum = num   # 新段从当前元素开始
        else:
            curr_sum += num
    return count <= k
```

### 完整代码

```python
def splitArray(self, nums, k):
    lo, hi = max(nums), sum(nums)

    while lo < hi:
        mid = (lo + hi) // 2
        if self.can_split(nums, mid, k):
            hi = mid        # mid 可以，试试更小的
        else:
            lo = mid + 1    # mid 不行，需要更大
    return lo

def can_split(self, nums, mid, k):
    count = 1
    curr_sum = 0
    for num in nums:
        if curr_sum + num > mid:
            count += 1
            curr_sum = num
        else:
            curr_sum += num
    return count <= k
```

### 复杂度

- 时间：O(n × log(sum - max)) — 二分次数 × 每次验证
- 空间：O(1)

---

## 解法二：动态规划（保底方案）

### 思路

`dp[i][j]` = 前 `i` 个元素分成 `j` 段，最大子数组和的最小值。

对于每个可能的分割点 `m`：
- 前 `m` 个元素分 `j-1` 段 → `dp[m][j-1]`
- 第 `m+1` 到第 `i` 个元素作为最后一段 → `sum(nums[m:i])`

取两者的 max（因为要的是最大子数组和），再对所有 `m` 取 min（因为要最小化）：

```
dp[i][j] = min( max(dp[m][j-1], sum(nums[m:i])) )  对所有有效的 m
```

### 用前缀和加速区间求和

```
sum(nums[m:i]) = prefix[i] - prefix[m]
```

### 完整代码

```python
def splitArray(self, nums, k):
    n = len(nums)

    # 前缀和
    prefix = [0] * (n + 1)
    for i in range(n):
        prefix[i + 1] = prefix[i] + nums[i]

    # dp[i][j] = 前 i 个元素分 j 段的最小最大和
    dp = [[float('inf')] * (k + 1) for _ in range(n + 1)]
    dp[0][0] = 0

    for i in range(1, n + 1):          # 前 i 个元素
        for j in range(1, min(i, k) + 1):  # 分 j 段
            for m in range(j - 1, i):       # 分割点
                last_sum = prefix[i] - prefix[m]
                dp[i][j] = min(dp[i][j], max(dp[m][j - 1], last_sum))

    return dp[n][k]
```

### 演练

`nums = [7, 2, 5, 10, 8]`，k = 2，prefix = `[0, 7, 9, 14, 24, 32]`

填 `dp[i][1]`（分 1 段 = 前缀和）：

| i | dp[i][1] |
|---|----------|
| 1 | 7 |
| 2 | 9 |
| 3 | 14 |
| 4 | 24 |
| 5 | 32 |

填 `dp[i][2]`（分 2 段，枚举分割点 m）：

| i | 枚举 m | 计算 | dp[i][2] |
|---|--------|------|----------|
| 2 | m=1: max(dp[1][1], sum[1:2]) = max(7, 2) = 9 | | 9 |
| 3 | m=1: max(7, 7)=7, m=2: max(9, 5)=9 | min(7, 9) | 7 |
| 4 | m=1: max(7, 17)=17, m=2: max(9, 15)=15, m=3: max(14, 10)=14 | min(17, 15, 14) | 14 |
| 5 | m=1: max(7, 25)=25, m=2: max(9, 23)=23, m=3: max(14, 18)=18, m=4: max(24, 8)=24 | min(25, 23, 18, 24) | **18** |

答案：`dp[5][2] = 18` ✓

### 复杂度

- 时间：O(n² × k)
- 空间：O(n × k)

---

## 两种解法对比

| | 二分 + 贪心 | DP |
|--|------------|-----|
| 时间 | O(n × log(sum)) | O(n² × k) |
| 空间 | O(1) | O(n × k) |
| 思路 | 猜答案，验证可行性 | 枚举所有分割方式 |
| 难点 | 想到"二分答案" | 状态转移方程 |
| 面试策略 | 最优解，优先写 | 保底方案 |

## 同类题目（二分答案 + 贪心验证）

- **LeetCode 875** — Koko Eating Bananas（吃香蕉的速度）
- **LeetCode 1011** — Capacity To Ship Packages（船的载重量）
- **LeetCode 1482** — Minimum Number of Days to Make m Bouquets（做花束的天数）

这类题的共同特征：直接求最优很难，但**给定一个值验证可行性很简单**，且可行性有单调性。

## 1095 Find in Mountain Array
## 92 Reverse Linked List II

## 题目概述

给定链表的头节点和两个整数 `left`、`right`，反转从位置 `left` 到位置 `right` 的节点，返回反转后的链表。

```
输入：1 → 2 → 3 → 4 → 5, left=2, right=4
输出：1 → 4 → 3 → 2 → 5
```

注意：`left` 和 `right` 是**位置**，不是值。

---

## 核心思路：两件事

**第一件事：反转中间一段**

就是标准的反转链表操作，用 `prev`、`cur`、`nxt` 三个变量。

**第二件事：两根线接回去**

反转会把链表断成三截，需要两个锚点把它们接起来。

---

## 四个关键指针

```
prev_left  → 左边断口的锚点，反转后负责接新头
tail       → 右边断口的锚点，反转后负责接后半段
prev       → 反转中用的指针，反转完是新头
cur        → 反转中用的指针，反转完是后半段起点
```

dummy 不参与操作，只是给 `prev_left` 一个落脚点（处理 left=1 的边界），最后用 `dummy.next` 返回答案。

---

## 图解全过程

### 第一步：找到 prev_left 和 tail

```
dummy → 1 → 2 → 3 → 4 → 5
        ↑   ↑
  prev_left  tail
```

`prev_left`：left 前面那个节点（节点 1）
`tail`：反转段的起点（节点 2，反转后变尾巴）

### 第二步：反转中间段

```
prev=None, cur=节点2

第一轮：None ← 2    3 → 4 → 5
第二轮：None ← 2 ← 3    4 → 5
第三轮：None ← 2 ← 3 ← 4    cur=5
```

反转完：`prev` = 节点 4（新头），`cur` = 节点 5（后半段）

### 第三步：两根线接回去

链表断成三截：

```
第一截：dummy → 1
第二截：4 → 3 → 2
第三截：5 → ...
```

接线：

```
prev_left.next = prev    节点1 → 节点4（第一截接第二截）
tail.next = cur           节点2 → 节点5（第二截接第三截）
```

结果：

```
dummy → 1 → 4 → 3 → 2 → 5 ✓
```

---

## 代码

```python
def reverseBetween(self, head, left, right):
    dummy = ListNode(next=head)

    # 走到 left 前一个
    prev_left = dummy
    for _ in range(left - 1):
        prev_left = prev_left.next

    # 记住反转段起点（反转后变尾巴）
    tail = prev_left.next

    # 反转 right - left + 1 个节点
    prev = None
    cur = tail
    for _ in range(right - left + 1):
        nxt = cur.next
        cur.next = prev
        prev = cur
        cur = nxt

    # 两根线接回去
    prev_left.next = prev   # 前半段 → 新头
    tail.next = cur          # 旧头（现在是尾） → 后半段

    return dummy.next
```

---

## 易错点

| 易错点 | 说明 |
|--------|------|
| left/right 是位置不是值 | 用 `for _ in range()` 按位置走，不要按值找 |
| 反转次数 | `right - left + 1` 次，不是 `right - left` |
| 忘记接回去 | 反转完必须加两根线，否则链表断裂 |
| left=1 的边界 | 需要 dummy 节点，否则 prev_left 无处落脚 |

## 复杂度

- **时间**：O(n) — 最多遍历一次链表
- **空间**：O(1) — 只用了几个指针

## 相关题目

- **LeetCode 206** — Reverse Linked List（反转整条链表，基础版）
- **LeetCode 25** — Reverse Nodes in k-Group（每 k 个一组反转，进阶版）
- **LeetCode 24** — Swap Nodes in Pairs（两两交换，k=2 的特例）

## 622 Design Circular Queue
## 460 LFU Cache
## 94 Binary Tree Inorder Traversal

## 题目概述

给定二叉树的根节点，返回中序遍历的结果。

中序遍历顺序：**左 → 根 → 右**

```
    1
     \
      2
     /
    3

输出：[1, 3, 2]
```

## 三种遍历对比

| 遍历方式 | 顺序 | 记忆方式 |
|---------|------|---------|
| 前序 Preorder | 根 → 左 → 右 | 根在**前** |
| 中序 Inorder | 左 → 根 → 右 | 根在**中** |
| 后序 Postorder | 左 → 右 → 根 | 根在**后** |

## 解法一：递归（DFS）

### 闭包写法（推荐）

`res` 定义在外层，内层函数直接访问，不需要传参。

```python
def inorderTraversal(self, root):
    res = []
    def dfs(node):
        if node is None:
            return
        dfs(node.left)       # 左
        res.append(node.val) # 根
        dfs(node.right)      # 右
    dfs(root)
    return res
```

### 传参写法（也可以）

```python
def inorderTraversal(self, root):
    def dfs(node, res):
        if node is None:
            return
        dfs(node.left, res)
        res.append(node.val)
        dfs(node.right, res)
    res = []
    dfs(root, res)
    return res
```

两种效果一样，闭包写法更简洁。

## 解法二：迭代（用栈模拟递归）

```python
def inorderTraversal(self, root):
    res = []
    stack = []
    curr = root
    while curr or stack:
        while curr:            # 一路向左走到底
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()     # 弹出 = 访问
        res.append(curr.val)
        curr = curr.right      # 转向右子树
    return res
```

### 迭代思路

递归隐式用了调用栈，迭代就是手动维护这个栈：
1. 一路往左走，沿途节点压栈
2. 走到底了，弹出栈顶（这就是"访问根"）
3. 转向右子树，重复

## 复杂度

- **时间**：O(n) — 每个节点访问一次
- **空间**：O(h) — h 为树高，最坏 O(n)（链状树），平衡树 O(log n)

## 关键点速记

- 中序遍历 = 左根右，对 BST 来说结果是**升序排列**
- 递归写法：base case 是 `node is None`
- 闭包可以直接访问外层变量，不需要传 `res`
- 迭代写法的核心：**一路向左压栈，弹出访问，转向右子树**

## 相关题目

- **LeetCode 144** — 前序遍历（根左右）
- **LeetCode 145** — 后序遍历（左右根）
- **LeetCode 98** — 验证 BST（中序遍历应为升序）
- **LeetCode 230** — BST 第 K 小元素（中序遍历第 K 个）

## 144 Binary Tree Preorder Traversal
## 145 Binary Tree Postorder Traversal
## 701 Insert into a Binary Search Tree
# BST 插入节点 — 完整讲解

## 核心性质

BST 的核心性质：**左子树所有值 < 根节点 < 右子树所有值**

利用这个性质，插入时只需不断"缩小范围"找到合适的空位即可。

---

## 递归思路

每次比较 `val` 和当前节点值：

- `val < root.val` → 应该插入**左子树**，递归处理左边
- `val > root.val` → 应该插入**右子树**，递归处理右边
- 当前节点为 `null` → 找到插入位置，**新建节点返回**

```python
def insertIntoBST(root, val):
    if root is None:
        return TreeNode(val)   # 插入点
    if val < root.val:
        root.left = insertIntoBST(root.left, val)
    else:
        root.right = insertIntoBST(root.right, val)
    return root
```

---

## 用具体例子走一遍

假设树长这样，插入 `val = 3`：

```
    4
   / \
  2   6
```

**第1次调用** `insert(4, 3)`
- 3 < 4，所以往左走
- `root.left = insert(2, 3)` ← 先去求右边这个值

**第2次调用** `insert(2, 3)`
- 3 > 2，所以往右走
- `root.right = insert(null, 3)` ← 继续往下求

**第3次调用** `insert(null, 3)`
- root 是 null！
- 创建新节点，`return TreeNode(3)` ← **从这里开始往回传**

### 返回过程（关键！）

```
第3次返回 TreeNode(3)
        ↓
回到第2次：root.right = TreeNode(3)  ← 3 挂到 2 的右边
           return root(2)             ← 把 2 这棵子树返回
        ↓
回到第1次：root.left = 2的子树       ← 重新接回 4 的左边
           return root(4)
```

最终结果：

```
    4
   / \
  2   6
   \
    3   ✓
```

---

## 为什么要 `root.left = ...` 而不是直接递归？

```python
# ❌ 错误写法
insertIntoBST(root.left, val)  # 返回值被丢弃，树没变化

# ✅ 正确写法
root.left = insertIntoBST(root.left, val)  # 把结果接回来
```

绝大多数情况下，`root.left` 等于传进去的子树（没变化），只有在**找到插入点那一层**，才会把新节点"传递回来"并接上。

---

## 为什么要 `return root`？

### 情况一：空树

```python
insertIntoBST(None, 3)
```

这说明**整棵树是空树**，新节点就是整棵树的根，必须 return 出去，外面才能接住它。

### 情况二：正常插入

每一层都需要把子树"接回来"，所以每一层都要 return 自己：

- **非插入层**：return 原来的 root，没有变化，只是传回去让上层接着用
- **插入层**（root == null 那层）：return 新节点，上层接住后挂到树上

### 如果不 return root 会怎样？

```python
# 假设只在 null 时 return，其他层不 return
if root is None:
    return TreeNode(val)
if val < root.val:
    root.left = insertIntoBST(root.left, val)
# 没有 return root ← 函数返回 None
```

上层会执行：

```python
root.left = None  # ← 把整个左子树清空了！
```

> **一句话总结：** `return root` 不是因为 root 变了，而是因为**上层需要一个返回值来接**，不 return 就会把子树弄丢。

---

## 新节点一定插在叶子位置

> 新节点**一定插在叶子位置** —— 因为 BST 插入沿路径走到底，不需要调整已有结构（不像平衡树 AVL/红黑树需要旋转）。

---

## 迭代写法

用一个指针遍历树，找到第一个"该往下走但子节点为空"的地方，直接挂上去：

```python
def insertIntoBST(root, val):
    if not root:
        return TreeNode(val)
    cur = root
    while cur:
        if val < cur.val:
            if cur.left is None:
                cur.left = TreeNode(val)
                break
            cur = cur.left
        else:
            if cur.right is None:
                cur.right = TreeNode(val)
                break
            cur = cur.right
    return root
```

---

## 复杂度分析

|  | 平均 | 最坏（退化链表） |
|---|---|---|
| 时间 | O(log n) | O(n) |
| 空间（递归栈） | O(log n) | O(n) |

---

## 普通 BST vs 平衡树

| | 普通 BST | 平衡树 (AVL/红黑) |
|---|---|---|
| 插入方式 | 只找空位挂上 | 插入后还要旋转调整 |
| 维持性质 | BST 大小顺序 | BST + 高度平衡 |
| 复杂度 | 平均 O(log n) | 严格 O(log n) |

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

## 题目概述

给定一组单词和一个外星字母顺序，判断这些单词是否按该顺序排好了。

```
输入：words = ["hello","leetcode"], order = "hlabcdefgijkmnopqrstuvwxyz"
输出：true（因为 h 排在 l 前面）
```

## 与 Alien Dictionary（LeetCode 269）的区别

| | 953（本题） | 269（Alien Dictionary） |
|--|------------|----------------------|
| 给你什么 | 完整的字母顺序 + 一组单词 | 只给一组单词 |
| 问什么 | 单词是否按顺序排好了 | 推导字母顺序是什么 |
| 本质 | 验证 | 推导 |
| 解法 | 逐对比较 | 拓扑排序 |

---

## 核心思路

没有算法技巧，就是纯模拟字典序比较。

**字典序的定义**：比较两个单词，找到第一个不同的字符，谁的字符排前面谁就排前面。前缀相同的话，短的排前面。

### 两步走

第一步：建映射，把外星字母顺序转成数字

```python
order_map = {c: i for i, c in enumerate(order)}
```

第二步：逐对比较相邻单词

---

## 比较两个单词的三种结果

```
相同字符     → 继续看下一个
不同且顺序错 → return False
不同且顺序对 → break（答案已出，不用再看）
前缀相同 w1 更长 → return False
```

---

## 代码

```python
def isAlienSorted(self, words, order):
    order_map = {c: i for i, c in enumerate(order)}

    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i + 1]
        for j in range(len(w1)):
            if j >= len(w2):                        # w1 更长，前缀相同
                return False
            if w1[j] != w2[j]:                      # 第一个不同字符
                if order_map[w1[j]] > order_map[w2[j]]:
                    return False                    # 顺序错
                break                                # 顺序对，看下一对
    return True
```

---

## 易错点

| 易错点 | 说明 |
|--------|------|
| 越界检查顺序 | `j >= len(w2)` 必须在访问 `w2[j]` **之前**，否则 IndexError |
| 缺少 break | 第一个不同字符已经决定顺序，不 break 会误判后面的字符 |
| break 位置 | 必须在 `w1[j] != w2[j]` 的分支**内部**，不能放外面（否则第一个字符就跳出） |
| 前缀相同 w1 更长 | "apple" vs "app" 应返回 False |

## 复杂度

- **时间**：O(m × n) — m 个单词，每个最长 n 个字符
- **空间**：O(1) — order_map 固定 26 个字母

## 相关题目

- **LeetCode 269** — Alien Dictionary（推导字母顺序，拓扑排序）
- **LeetCode 242** — Valid Anagram（字符频率比较）

## 997 Find the Town Judge
# 找到小镇的法官

## 题意

小镇有 `n` 个人，编号 `1` 到 `n`。法官的条件：被所有其他人信任（入度 = n-1），且不信任任何人（出度 = 0）。

## 解法一：入度 + 出度

```python
def findJudge(n, trust):
    indegree = defaultdict(int)
    outdegree = defaultdict(int)
    for a, b in trust:
        indegree[b] += 1
        outdegree[a] += 1
    for i in range(1, n + 1):  # 编号从1开始
        if indegree[i] == n - 1 and outdegree[i] == 0:
            return i
    return -1
```

## 解法二：一个数组（更简洁）

```python
def findJudge(n, trust):
    count = [0] * (n + 1)
    for a, b in trust:
        count[a] -= 1  # 出度
        count[b] += 1  # 入度
    for i in range(1, n + 1):
        if count[i] == n - 1:
            return i
    return -1
```

入度 - 出度 = n-1，说明被所有人信任且不信任任何人。

## 示例

```
n = 3, trust = [[1,3],[2,3]]

解法一:
  indegree:  {3: 2}
  outdegree: {1: 1, 2: 1}
  i=3: indegree=2==n-1, outdegree=0 → 返回 3

解法二:
  count = [0, -1, -1, 2]
           0   1   2  3
  count[3] = 2 == n-1 → 返回 3
```

## 易错点

人的编号从 `1` 到 `n`，遍历时用 `range(1, n+1)` 而不是 `range(n)`。

## 复杂度

- 时间：O(n + E)，E 为 trust 数组长度
- 空间：O(n)

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
**考点：Dijkstra 变种**
 
effort = 路径上所有相邻格子高度差的**最大值**，求从左上到右下的最小 effort。
 
---
 
### 和普通最短路径的区别
 
| | 普通最短路径 | 这题 |
|--|------------|------|
| 堆里存 | `(累计距离, node)` | `(最大差值, row, col)` |
| 更新方式 | `curr_dist + edge_weight` | `max(curr_effort, 边的高度差)` |
| 核心操作 | 累加 | 取最大值 |
 
用 `max` 而不是累加，是因为 effort 定义就是路径上差值的最大值，每走一步要维护"到目前为止最大的差值"。
 
---
 
### 为什么不需要显式构建 adj
 
网格图的邻居关系固定（上下左右），用方向数组即时生成，边权也动态计算：
 
```python
directions = [(0,1),(0,-1),(1,0),(-1,0)]
for dr, dc in directions:
    nr, nc = r + dr, c + dc
    weight = abs(heights[nr][nc] - heights[r][c])
```
 
---
 
### Dijkstra 堆优化版（最优解）
 
```python
class Solution:
    def minimumEffortPath(self, heights: List[List[int]]) -> int:
        visited = set()
        rows, cols = len(heights), len(heights[0])
        minHeap = [[0, 0, 0]]  # [effort, row, col]
 
        while minHeap:
            diff, r, c = heapq.heappop(minHeap)
            if (r, c) in visited:
                continue
            visited.add((r, c))
 
            if (r, c) == (rows-1, cols-1):
                return diff
 
            for dr, dc in [(0,1),(1,0),(0,-1),(-1,0)]:
                nr, nc = r + dr, c + dc
                if nr < 0 or nc < 0 or nr >= rows or nc >= cols or (nr, nc) in visited:
                    continue
                newdiff = max(diff, abs(heights[r][c] - heights[nr][nc]))
                heapq.heappush(minHeap, [newdiff, nr, nc])
 
        return 0
```
 
用了 `visited` 集合后可以省略 `dist` 数组——堆保证第一次 pop 出来的就是最优解。
 
**常见 Bug：** `return 0` 写在 while 循环内部，导致第一次迭代就返回。
 
---
 
### 朴素 Dijkstra（无堆）
 
```python
class Solution:
    def minimumEffortPath(self, heights: List[List[int]]) -> int:
        rows, cols = len(heights), len(heights[0])
        dist = [[float('inf')] * cols for _ in range(rows)]
        dist[0][0] = 0
        visited = set()
 
        while True:
            curr_dist, r, c = float('inf'), -1, -1
            for i in range(rows):
                for j in range(cols):
                    if (i, j) not in visited and dist[i][j] < curr_dist:
                        curr_dist, r, c = dist[i][j], i, j
 
            if r == -1:
                break
            if (r, c) == (rows-1, cols-1):
                return dist[r][c]
            visited.add((r, c))
 
            for dr, dc in [(0,1),(1,0),(0,-1),(-1,0)]:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and (nr, nc) not in visited:
                    new_dist = max(curr_dist, abs(heights[r][c] - heights[nr][nc]))
                    if new_dist < dist[nr][nc]:
                        dist[nr][nc] = new_dist
 
        return dist[rows-1][cols-1]
```
 
朴素版必须维护 `dist` 数组，因为没有堆自动排序，靠线性扫描找最小值。
 
---
 
### 四种解法对比
 
| 方法 | 时间复杂度 | 核心思想 |
|------|-----------|---------|
| Dijkstra + 堆 | O(n log n) | 贪心，每次取最优节点扩展 |
| Binary Search + BFS | O(n log h) | 二分答案，验证 effort=mid 时能否到达 |
| SPFA | O(n) 平均 | 队列松弛，节点可重复入队 |
| Union Find | O(n log n) | 按边权从小到大加边直到起终点连通 |
 
**Binary Search + BFS 的思路：** 答案有单调性（effort 越大越容易到达），二分猜答案，BFS 只走高度差 <= mid 的边验证可达性。
 
**Union Find 的思路：** 本质是 Kruskal 最小生成树变种，把所有边按高度差排序，从小到大加边，直到起点终点连通，此时边权即为答案。
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

**思路：贪心 + 最大堆**

每次取出最重的两块石头相撞，用最大堆模拟。

```python
class Solution:
    def lastStoneWeight(self, stones: List[int]) -> int:
        new = [-s for s in stones]
        heapq.heapify(new)

        while len(new) > 1:
            s1 = -1 * heapq.heappop(new)
            s2 = -1 * heapq.heappop(new)
            if s1 > s2:
                heapq.heappush(new, -(s1 - s2))

        return -new[0] if new else 0
```

**常见 Bug：**
- `heapq.heapify()` 返回 `None`，不能赋值给变量
- 返回时要取 `new[0]`（堆顶），不是 `new[-1]`

---

## 2. Last Stone Weight II（最后一块石头的重量 II）

**思路：0/1 背包 DP**

每块石头可以分配 `+1` 或 `-1`，目标是让结果绝对值最小。

**转化：** 把石头分成两组，令总和为 `S`，让其中一组尽量接近 `S//2`，则答案为 `S - 2 * P`。

```python
class Solution:
    def lastStoneWeightII(self, stones: List[int]) -> int:
        S = sum(stones)
        target = S // 2

        dp = [0] * (target + 1)

        for stone in stones:
            for j in range(target, stone - 1, -1):
                dp[j] = max(dp[j], dp[j - stone] + stone)

        P = dp[target]
        return S - 2 * P
```

**dp 含义：** `dp[j]` = 背包容量为 `j` 时，能凑出的最大重量（重量 = 价值）

**为什么倒序遍历 j？** 防止同一块石头被重复使用（0/1 背包特性）

**为什么不用堆？**

| | Heap | 有序数组 + 二分 |
|--|------|----------------|
| get 复杂度 | O(n log n) | O(log n) |
| 数据是否破坏 | ❌ pop 会删除 | ✅ 只读 |
| 适合场景 | 反复取最值 | 有序数据查找 |

---
## 877 Stone Game
# 877. Stone Game（石子游戏）

## 题目

偶数堆石子排成一排，Alice 和 Bob 轮流从两端取，Alice 先手，两人都最优操作。总数为奇数所以没有平局。判断 Alice 是否赢。

## 核心思路

不能贪心（每次拿两端较大的），因为有时候故意拿小的能控制未来局面。

### 贪心反例

`piles = [3, 9, 1, 2]`

```
贪心：Alice 拿 3 → Bob 拿 9 → Alice 拿 2 → Bob 拿 1
Alice: 5, Bob: 10 → Alice 输

最优：Alice 拿 2 → 不管 Bob 拿 3 还是 1，Alice 下轮都能拿 9
Alice: 11, Bob: 4 → Alice 赢
```

### 为什么用区间 DP

每次只能拿两端，拿完之后剩下的还是一段连续区间。子问题天然就是"面对某个区间怎么玩最优"。

## DP 定义

`dp[i][j]` = 面对 `piles[i..j]`，**当前玩家**比对手最终多拿的最大石子数。

注意"当前玩家"不固定是 Alice，谁轮到谁就是当前玩家。

## 状态转移

面对 `piles[i..j]`，两个选择：

```
拿左端：我赚 piles[i]，剩下 [i+1, j] 给对手
        对手在 [i+1, j] 的优势是 dp[i+1][j]
        我的净优势 = piles[i] - dp[i+1][j]

拿右端：我赚 piles[j]，剩下 [i, j-1] 给对手
        对手在 [i, j-1] 的优势是 dp[i][j-1]
        我的净优势 = piles[j] - dp[i][j-1]
```

取 max：

```
dp[i][j] = max(piles[i] - dp[i+1][j], piles[j] - dp[i][j-1])
```

**为什么是减号**：对手的优势就是我的劣势。对手比我多拿 `dp[i+1][j]`，那我就要减掉这个。

## Base Case

```
dp[i][i] = piles[i]  （只剩一堆，直接拿走）
```

## 计算顺序

`dp[i][j]` 依赖 `dp[i+1][j]`（下方）和 `dp[i][j-1]`（左方），所以 i 从下往上、j 从左往右：

```
        j →
    0   1   2   3
i 0 [3]  →   →   →
  1     [9]  →   →
  2         [1]  →
  3             [2]

算 dp[i][j] 需要：
  ↑ i 从下往上（dp[i+1][j] 已算好）
  → j 从左往右（dp[i][j-1] 已算好）
```

## 代码

```python
class Solution:
    def stoneGame(self, piles):
        n = len(piles)
        dp = [[0] * n for _ in range(n)]

        # base case：只剩一堆，直接拿
        for i in range(n):
            dp[i][i] = piles[i]

        # i 从下往上，j 从左往右
        for i in range(n - 2, -1, -1):
            for j in range(i + 1, n):
                dp[i][j] = max(
                    piles[i] - dp[i + 1][j],   # 拿左端
                    piles[j] - dp[i][j - 1]    # 拿右端
                )

        return dp[0][n - 1] > 0
```

## 模拟示例

`piles = [3, 9, 1, 2]`

**Base case**：

```
dp[0][0]=3  dp[1][1]=9  dp[2][2]=1  dp[3][3]=2
```

**i=2**：

```
dp[2][3] = max(1-2, 2-1) = max(-1, 1) = 1   面对[1,2]，拿2，优势1
```

**i=1**：

```
dp[1][2] = max(9-1, 1-9) = max(8, -8) = 8   面对[9,1]，拿9，优势8
dp[1][3] = max(9-1, 2-8) = max(8, -6) = 8   面对[9,1,2]，拿9，优势8
```

**i=0**：

```
dp[0][1] = max(3-9, 9-3) = max(-6, 6) = 6   面对[3,9]，拿9，优势6
dp[0][2] = max(3-8, 1-6) = max(-5, -5) = -5  面对[3,9,1]，怎么拿都亏
dp[0][3] = max(3-8, 2-(-5)) = max(-5, 7) = 7  面对[3,9,1,2]，拿2，优势7
```

`dp[0][3] = 7 > 0` → Alice 赢 ✅

## 复杂度

- **时间**：O(n²)
- **空间**：O(n²)

## 数学 Trick

这道题其实 Alice 必赢，直接 `return True`。因为 Alice 可以预先算出奇数位总和和偶数位总和，然后选更大的那组全拿。但面试中通常要求写出 DP 解法。

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
### 题意

有 n 间会议室（编号 0 到 n-1），安排规则：
1. 优先选**编号最小的空闲房间**
2. 没有空闲就等**最早结束的房间**（结束时间相同选编号小的）

返回使用次数最多的房间编号。

---

### 解法一：暴力 O(m × n)

用数组 `rooms[i]` 记录每间房的结束时间，每次线性扫描。

**两件事**：
- 从房间 0 开始找第一个空闲房间（`rooms[i] <= s`）
- 同时记录结束时间最早的房间（`min_room`）

```python
def mostBooked(self, n, meetings):
    meetings.sort()
    rooms = [0] * n
    count = [0] * n

    for s, e in meetings:
        min_room = 0
        found = False
        for i in range(n):
            if rooms[i] <= s:
                found = True
                count[i] += 1
                rooms[i] = e
                break
            if rooms[min_room] > rooms[i]:
                min_room = i
        if found:
            continue
        count[min_room] += 1
        rooms[min_room] += e - s

    return count.index(max(count))
```

---

### 解法二：双堆 O(m log n)

**从暴力到优化的思路**：暴力中每次都线性扫描找最小值，而"反复取最小值"正是堆擅长的事。

**空闲房间和使用中房间需要不同的排序规则**，所以拆成两个堆：
- `free` 堆：按房间号排序 → 取编号最小的空闲房间
- `busy` 堆：按 (结束时间, 房间号) 排序 → 取最早结束的

**暴力中隐含的释放操作需要显式做**：分配前把 `busy` 中已结束的房间转移到 `free`。

```python
def mostBooked(self, n, meetings):
    meetings.sort()
    free = list(range(n))
    busy = []
    count = [0] * n

    for s, e in meetings:
        # 释放已结束的房间
        while busy and busy[0][0] <= s:
            _, room = heapq.heappop(busy)
            heapq.heappush(free, room)

        if free:
            room = heapq.heappop(free)
            heapq.heappush(busy, (e, room))
        else:
            earliest_end, room = heapq.heappop(busy)
            heapq.heappush(busy, (earliest_end + e - s, room))

        count[room] += 1

    return count.index(max(count))
```

**暴力 vs 双堆对应关系**：

| 操作 | 暴力 | 双堆 |
|------|------|------|
| 找空闲房间 | 线性扫描 O(n) | `free` 堆弹出 O(log n) |
| 找最早结束 | 线性扫描 O(n) | `busy` 堆弹出 O(log n) |
| 释放已结束房间 | 隐含在扫描中 | 显式循环转移 |

---

### 解法三：单堆 + 拉平技巧 O(m log n)

**核心想法**：把所有房间放在一个堆里，按 `(结束时间, 房间号)` 排序。

**问题**：多个空闲房间结束时间不同，堆顶不一定是编号最小的。

**解决**：每次分配前，把所有结束时间 < start 的房间的结束时间**统一改成 start**（拉平），这样空闲房间第一个排序键相同，堆自动按房间号排。

```python
def mostBooked(self, n, meetings):
    meetings.sort()
    available = [(0, i) for i in range(n)]
    count = [0] * n

    for start, end in meetings:
        # 拉平：所有空闲房间的结束时间统一改成 start
        # start 是单调增加的因为我们sort,所以我们可以巧妙的把available 铺平再做剩下的计算
        # 所以我们可以只用available的第二个index去做排序找数字最小的空闲房间
        while available and available[0][0] < start:
            end_time, room = heapq.heappop(available)
            heapq.heappush(available, (start, room))

        end_time, room = heapq.heappop(available)
        heapq.heappush(available, (end_time + (end - start), room))
        count[room] += 1

    return count.index(max(count))
```

**拉平技巧的两个作用**：

1. **统一排序键**：空闲房间结束时间相同 → 堆按房间号排序 → 选出编号最小的
2. **修正公式**：`end_time + (end - start)` 对空闲房间也成立（`start + (end - start) = end`）

**拉平不影响后续的原因**：会议已排序，`start` 单调递增，后续的 while 循环总会把未使用的房间重新拉平到新的 `start`。

---

### 拉平技巧演练

3 间房，会议 `[[1,3],[0,5],[2,7],[4,6],[8,10]]`，排序后 `[[0,5],[1,3],[2,7],[4,6],[8,10]]`

初始堆：`[(0,0), (0,1), (0,2)]`

| 会议 | while 拉平后 | 弹出 | 推入 | count |
|------|-------------|------|------|-------|
| [0,5] | 不动 | (0,0) | (5,0) | [1,0,0] |
| [1,3] | (0,1)→(1,1), (0,2)→(1,2) | (1,1) | (3,1) | [1,1,0] |
| [2,7] | (1,2)→(2,2) | (2,2) | (7,2) | [1,1,1] |
| [4,6] | (3,1)→(4,1) | (4,1) | (6,1) | [1,2,1] |
| [8,10] | (5,0)→(8,0), (6,1)→(8,1), (7,2)→(8,2) | (8,0) | (10,0) | [2,2,1] |

最后一步：三间都空闲，拉平到 `(8,0)(8,1)(8,2)`，堆自动按房间号排序，选出房间 0。

结果：`count = [2, 2, 1]`，返回 **0**。

---

## 三道题的递进关系

```
252（有没有冲突）
 ↓ 如果有冲突，需要多少间房？
253（最少几间房）
 ↓ 有固定房间数，哪间用得最多？
2402（哪间房最忙）
```

## 通用优化思路

```
暴力（线性扫描找最值）
 ↓ 发现瓶颈：反复取最小值
 ↓ 选对数据结构：堆
优化（堆 O(log n) 替代扫描 O(n)）
```

这个"暴力 → 找瓶颈 → 换数据结构"的套路适用于很多题目。

## 168 Excel Sheet Column Title
## 1071 Greatest Common Divisor of Strings
# LeetCode 1071 — Greatest Common Divisor of Strings

## 题目概述

给定两个字符串 `str1` 和 `str2`，找到最长的字符串 `x`，使得 `x` 重复若干次能拼成 `str1`，也能拼成 `str2`。

```
输入：str1 = "ABCABC", str2 = "ABC"
输出："ABC"

输入：str1 = "LEET", str2 = "CODE"
输出：""（不存在公因子）
```

---

## 核心洞察：这是一道数学题

和 KMP、模式匹配无关。本质就是字符串版本的最大公约数。

### 两个关键点

**第一：怎么判断有没有公因子？**

```python
str1 + str2 == str2 + str1
```

如果 `x` 能整除两者，那 `str1 = x × a 次`，`str2 = x × b 次`，两边拼接都是 `x × (a+b) 次`，必然相等。不等就说明不存在公因子。

**第二：公因子多长？**

和数字的 GCD 完全一样：

```
str1 = "ABCABC"  长度 6
str2 = "ABC"      长度 3
gcd(6, 3) = 3
答案 = str1[:3] = "ABC"
```

---

## 不需要关心谁长谁短

- 拼接比较两边都试了，顺序无关
- `gcd(6, 3) = gcd(3, 6) = 3`
- 公因子在两个字符串开头都存在，`str1[:gcd值]` 或 `str2[:gcd值]` 都一样

---

## 代码

### 用 math.gcd

```python
from math import gcd

def gcdOfStrings(self, str1, str2):
    if str1 + str2 != str2 + str1:
        return ""
    return str1[:gcd(len(str1), len(str2))]
```

### 手写 GCD（辗转相除法）

```python
def gcdOfStrings(self, str1, str2):
    if str1 + str2 != str2 + str1:
        return ""

    def gcd(a, b):
        while b:
            a, b = b, a % b
        return a

    return str1[:gcd(len(str1), len(str2))]
```

### 辗转相除法演练

```
gcd(6, 4):
  a=6, b=4 → 4, 2
  a=4, b=2 → 2, 0
  b=0，返回 2

gcd(3, 6):
  a=3, b=6 → 6, 3    ← 自动交换
  a=6, b=3 → 3, 0
  b=0，返回 3
```

原理：`gcd(a, b) = gcd(b, a % b)`，不断缩小直到余数为 0。

---

## 代码逻辑总结

```
第一步：有没有公因子？  → str1 + str2 == str2 + str1
第二步：公因子多长？    → gcd(len1, len2)
第三步：取出来         → str1[:gcd值]
```

GCD 运算全在长度（数字）上做，字符串只做了拼接比较和切片。

---

## 与类似题目的区别

| | LeetCode 1071（本题） | LeetCode 28 |
|--|---------------------|-------------|
| 问题 | 最大公因子串 | 字符串匹配 |
| 数字类比 | 12 和 8 的 GCD | 12 里包含 3 吗 |
| 核心操作 | 整除 | 匹配 |
| 算法 | GCD | KMP |

## 复杂度

- **时间**：O(n + m) — 拼接比较
- **空间**：O(n + m) — 拼接产生的新字符串

## 相关题目

- **LeetCode 28** — Find the Index of the First Occurrence（KMP 模式匹配）
- **LeetCode 459** — Repeated Substring Pattern（判断单个字符串是否由重复子串构成）

## 2807 Insert Greatest Common Divisors in Linked List
# 链表插入最大公约数节点

## 题意

给定链表，在每对相邻节点之间插入一个新节点，值为两者的最大公约数（GCD）。

## 解法：遍历 + 插入

```python
from math import gcd

def insertGreatestCommonDivisors(head):
    curr = head
    while curr and curr.next:
        g = gcd(curr.val, curr.next.val)
        new_node = ListNode(g)
        new_node.next = curr.next
        curr.next = new_node
        curr = new_node.next  # 跳过新节点，防止死循环

    return head
```

## 逐步示例

输入：`18 → 6 → 10 → 3`

```
第一步: gcd(18,6)=6   → 18 → [6] → 6 → 10 → 3
第二步: gcd(6,10)=2   → 18 → 6 → 6 → [2] → 10 → 3
第三步: gcd(10,3)=1   → 18 → 6 → 6 → 2 → 10 → [1] → 3
```

输出：`18 → 6 → 6 → 2 → 10 → 1 → 3`

## 关键点

`curr = new_node.next` 跳过刚插入的节点，否则会对新节点和下一个节点再算一次 GCD，导致死循环。

## GCD 辗转相除法

面试可以直接用 `math.gcd`，但要能手写：

```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a
```

原理：`gcd(a, b) = gcd(b, a % b)`，重复直到余数为 0。

```
gcd(18, 6):  18%6=0  → 6
gcd(10, 6):  10%6=4 → 6%4=2 → 4%2=0 → 2
gcd(10, 3):  10%3=1 → 3%1=0 → 1
```

## 复杂度

- 时间：O(n)，遍历一次链表
- 空间：O(1)，不算新插入的节点

## 867 Transpose Matrix
## 13 Roman to Integer
## 67 Add Binary
## 201 Bitwise AND of Numbers Range
## 3133 Minimum Array End
