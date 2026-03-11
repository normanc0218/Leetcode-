# 🚀 LeetCode Memo (NeetCode 150)

Personal algorithm knowledge base.

---

## 📂 Categories

- [01 Array & Hashing](#01-array--hashing)
- [02 Two Pointers](#02-two-pointers)
- [03 Sliding Window](#03-sliding-window)
- [04 Stack](#04-stack)
- [05 Binary Search](#05-binary-search)
- [06 Linked List](#06-linked-list)
- [07 Trees](#07-trees)
- [08 Tries](#08-tries)
- [09 Heap / Priority Queue](#09-heap--priority-queue)
- [10 Backtracking](#10-backtracking)
- [11 Graphs](#11-graphs)
- [12 Advanced Graphs](#12-advanced-graphs)
- [13 1-D Dynamic Programming](#13-1-d-dynamic-programming)
- [14 2-D Dynamic Programming](#14-2-d-dynamic-programming)
- [15 Greedy](#15-greedy)
- [16 Intervals](#16-intervals)
- [17 Math & Geometry](#17-math--geometry)
- [18 Bit Manipulation](#18-bit-manipulation)
---
## 01. Array & Hashing
- [1 Two Sum](mainMaterial.md#1-two-sum)
- [287 Find the Duplicate Number](mainMaterial.md#287-find-the-duplicate-number)
- [49 Group Anagrams](mainMaterial.md#49-group-anagrams)
- [217 Contains Duplicate](mainMaterial.md#217-contains-duplicate)
- [347 Top K Frequent Elements](mainMaterial.md#347-top-k-frequent-elements)
- [238 Product of Array Except Self](mainMaterial.md#238-product-of-array-except-self)
- [36 Valid Sudoku](mainMaterial.md#36-valid-sudoku)
- [128 Longest Consecutive Sequence](mainMaterial.md#128-longest-consecutive-sequence)
- [242 Valid Anagram](mainMaterial.md#242-valid-anagram)
- [271 Encode and Decode Strings](mainMaterial.md#271-encode-and-decode-strings)

## 02. Two Pointers
- [125 Valid Palindrome](mainMaterial.md#125-valid-palindrome)
- [167 Two Sum II](mainMaterial.md#167-two-sum-ii)
- [15 3Sum](mainMaterial.md#15-3sum)
- [11 Container With Most Water](mainMaterial.md#11-container-with-most-water)
- [42 Trapping Rain Water](mainMaterial.md#42-trapping-rain-water)
---

## 🧠 双指针模板

```python
def two_pointers(s):
    l, r = 0, len(s) - 1
    
    while l < r:
        # 左指针处理
        if 左边不满足条件:
            l += 1
            continue  # 或用 elif
        
        # 右指针处理
        if 右边不满足条件:
            r -= 1
            continue  # 或用 elif
        
        # 双指针都满足条件，进行处理
        if 满足某个条件:
            # 处理逻辑
            l += 1
            r -= 1
        else:
            return False
    
    return True
```
## 03. Sliding Window
- [121 Best Time to Buy and Sell Stock](mainMaterial.md#121-best-time-to-buy-and-sell-stock)
- [3 Longest Substring Without Repeating Characters](mainMaterial.md#3-longest-substring-without-repeating-characters)
- [424 Longest Repeating Character Replacement](mainMaterial.md#424-longest-repeating-character-replacement)
- [567 Permutation in String](mainMaterial.md#567-permutation-in-string)
- [76 Minimum Window Substring](mainMaterial.md#76-minimum-window-substring)

## 04. Stack
- [20 Valid Parentheses](mainMaterial.md#20-valid-parentheses)
- [155 Min Stack](mainMaterial.md#155-min-stack)
- [150 Evaluate Reverse Polish Notation](mainMaterial.md#150-evaluate-reverse-polish-notation)
- [22 Generate Parentheses](mainMaterial.md#22-generate-parentheses)
- [739 Daily Temperatures](mainMaterial.md#739-daily-temperatures)
- [853 Car Fleet](mainMaterial.md#853-car-fleet)
- [84 Largest Rectangle in Histogram](mainMaterial.md#84-largest-rectangle-in-histogram)

## 05. Binary Search
- [704 Binary Search](mainMaterial.md#704-binary-search)
- [74 Search a 2D Matrix](mainMaterial.md#74-search-a-2d-matrix)
- [875 Koko Eating Bananas](mainMaterial.md#875-koko-eating-bananas)
- [153 Find Minimum in Rotated Sorted Array](mainMaterial.md#153-find-minimum-in-rotated-sorted-array)
- [33 Search in Rotated Sorted Array](mainMaterial.md#33-search-in-rotated-sorted-array)
- [981 Time Based Key Value Store](mainMaterial.md#981-time-based-key-value-store)
- [4 Median of Two Sorted Arrays](mainMaterial.md#4-median-of-two-sorted-arrays)
# 📘 二分查找 (Binary Search) - 三种区间写法

---

## 🎯 核心原则

**关键：保持区间定义的一致性**

- **区间定义决定一切**：初始化、循环条件、更新方式
- **不变式**：每次循环后，答案必在搜索区间内
- **三种写法本质相同**，只是区间定义不同

---

## 1️⃣ 左闭右闭 [left, right] - 最常用 ⭐

### 💡 区间定义
- `[left, right]` - 两端都包含
- 搜索区间内的每个元素都可能是答案

### ✅ 模板代码

```python
def binary_search(nums, target):
    left, right = 0, len(nums) - 1  # ✅ right = len(nums) - 1
    
    while left <= right:  # ✅ left == right 时区间 [left, left] 有效
        mid = left + (right - left) // 2
        
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1  # ✅ mid 已检查，排除
        else:
            right = mid - 1  # ✅ mid 已检查，排除
    
    return -1  # 未找到
```

### 🔑 关键点
```
✅ right = len(nums) - 1  (包含最后一个元素)
✅ while left <= right     (相等时还有一个元素)
✅ left = mid + 1          (mid 已检查，排除)
✅ right = mid - 1         (mid 已检查，排除)
```

---

## 2️⃣ 左闭右开 [left, right)

### 💡 区间定义
- `[left, right)` - 左包含，右不包含
- `right` 指向的元素不在搜索区间内

### ✅ 模板代码

```python
def binary_search(nums, target):
    left, right = 0, len(nums)  # ✅ right = len(nums) (不包含)
    
    while left < right:  # ✅ left == right 时区间 [left, left) 无效
        mid = left + (right - left) // 2
        
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1  # ✅ mid 已检查，排除
        else:
            right = mid  # ✅ mid 已检查，但 right 本就不包含，所以 = mid
    
    return -1  # 未找到
```

### 🔑 关键点
```
✅ right = len(nums)       (不包含最后位置)
✅ while left < right      (相等时区间为空)
✅ left = mid + 1          (mid 已检查，排除)
✅ right = mid             (right 不包含，所以 = mid)
```

---

## 3️⃣ 左开右开 (left, right)

### 💡 区间定义
- `(left, right)` - 两端都不包含
- 两个边界都不在搜索区间内

### ✅ 模板代码

```python
def binary_search(nums, target):
    left, right = -1, len(nums)  # ✅ 两端都不包含
    
    while left + 1 < right:  # ✅ 区间内至少有一个元素
        mid = left + (right - left) // 2
        
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid  # ✅ left 不包含，所以 = mid
        else:
            right = mid  # ✅ right 不包含，所以 = mid
    
    return -1  # 未找到
```

### 🔑 关键点
```
✅ left = -1, right = len(nums)  (两端都在外面)
✅ while left + 1 < right         (至少间隔2，中间有元素)
✅ left = mid                     (left 不包含，所以 = mid)
✅ right = mid                    (right 不包含，所以 = mid)
```

---

## 📊 三种写法对比

| 特性 | 左闭右闭 [l, r] | 左闭右开 [l, r) | 左开右开 (l, r) |
|------|----------------|----------------|----------------|
| **初始 right** | `len(nums) - 1` | `len(nums)` | `len(nums)` |
| **初始 left** | `0` | `0` | `-1` |
| **循环条件** | `left <= right` | `left < right` | `left + 1 < right` |
| **更新 left** | `mid + 1` | `mid + 1` | `mid` |
| **更新 right** | `mid - 1` | `mid` | `mid` |
| **推荐度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |


## 🔍 寻找左边界（第一个 >= target）

### 左闭右闭写法
```python
def lower_bound(nums, target):
    left, right = 0, len(nums) - 1
    result = len(nums)  # 默认返回 len (表示都小于 target)
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if nums[mid] >= target:
            result = mid
            right = mid - 1  # 继续向左找
        else:
            left = mid + 1
    
    return result
```

### 左闭右开写法
```python
def lower_bound(nums, target):
    left, right = 0, len(nums)
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] >= target:
            right = mid  # 继续向左找
        else:
            left = mid + 1
    
    return left  # 退出时 left == right
```

---

## 🔍 寻找右边界（最后一个 <= target）

### 左闭右闭写法
```python
def upper_bound(nums, target):
    left, right = 0, len(nums) - 1
    result = -1  # 默认返回 -1 (表示都大于 target)
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if nums[mid] <= target:
            result = mid
            left = mid + 1  # 继续向右找
        else:
            right = mid - 1
    
    return result
```

### 左闭右开写法
```python
def upper_bound(nums, target):
    left, right = 0, len(nums)
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] <= target:
            left = mid + 1  # 继续向右找
        else:
            right = mid
    
    return left - 1  # 返回 left - 1
```

---

## 🧠 如何选择？

### ⭐ **推荐：左闭右闭 [left, right]**
```
优点:
✅ 最直观：[0, len-1] 包含所有元素
✅ 对称美：left = mid + 1, right = mid - 1
✅ 最常用：大多数教材和题目使用

适用: 99% 的情况
```

### **左闭右开 [left, right)**
```
优点:
✅ Python 风格：range(0, len) 的习惯
✅ 不需要 -1：right = len(nums)
✅ 找边界时更简洁

适用: 寻找边界的问题
```

### **左开右开 (left, right)**
```
优点:
✅ 避免溢出：不直接访问边界
✅ 理论优美：开区间一致性

缺点:
❌ 不直观：left = -1 很奇怪
❌ 少用：不是主流写法

适用: 特殊情况（如需要保护边界）
```


## 📌 防溢出技巧

```python
# ❌ 可能溢出
mid = (left + right) // 2

# ✅ 防止溢出
mid = left + (right - left) // 2

# ✅ 位运算（最快，但可读性差）
mid = (left + right) >> 1
```

---

## 🎯 总结

| 场景 | 推荐写法 |
|------|---------|
| **基础二分查找** | 左闭右闭 ⭐⭐⭐⭐⭐ |
| **寻找边界** | 左闭右开 ⭐⭐⭐⭐ |
| **Python 习惯** | 左闭右开 ⭐⭐⭐⭐ |
| **特殊保护边界** | 左开右开 ⭐⭐⭐ |

**建议**：掌握**左闭右闭**作为主力，理解**左闭右开**用于边界问题，了解**左开右开**扩展知识。

---

**提示**: 选一种写法深入理解，面试时保持一致，不要混用！💪
## 06. Linked List
- [206 Reverse Linked List](mainMaterial.md#206-reverse-linked-list)
- [21 Merge Two Sorted Lists](mainMaterial.md#21-merge-two-sorted-lists)
- [143 Reorder List](mainMaterial.md#143-reorder-list)
- [19 Remove Nth Node From End](mainMaterial.md#19-remove-nth-node-from-end)
- [2 Add Two Numbers](mainMaterial.md#2-add-two-numbers)
- [141 Linked List Cycle](mainMaterial.md#141-linked-list-cycle)
- [138 Copy List With Random Pointer](mainMaterial.md#138-copy-list-with-random-pointer)
- [23 Merge k Sorted Lists](mainMaterial.md#23-merge-k-sorted-lists)

## 07. Trees
- [226 Invert Binary Tree](mainMaterial.md#226-invert-binary-tree)
- [104 Maximum Depth of Binary Tree](mainMaterial.md#104-maximum-depth-of-binary-tree)
- [543 Diameter of Binary Tree](mainMaterial.md#543-diameter-of-binary-tree)
- [110 Balanced Binary Tree](mainMaterial.md#110-balanced-binary-tree)
- [100 Same Tree](mainMaterial.md#100-same-tree)
- [572 Subtree of Another Tree](mainMaterial.md#572-subtree-of-another-tree)
- [235 LCA of BST](mainMaterial.md#235-lca-of-bst)
- [102 Binary Tree Level Order Traversal](mainMaterial.md#102-binary-tree-level-order-traversal)
- [199 Binary Tree Right Side View](mainMaterial.md#199-binary-tree-right-side-view)
- [98 Validate BST](mainMaterial.md#98-validate-bst)
- [230 Kth Smallest in BST](mainMaterial.md#230-kth-smallest-in-bst)
- [105 Construct Binary Tree from Preorder and Inorder](mainMaterial.md#105-construct-binary-tree-from-preorder-and-inorder)
- [124 Binary Tree Maximum Path Sum](mainMaterial.md#124-binary-tree-maximum-path-sum)
- [297 Serialize and Deserialize Binary Tree](mainMaterial.md#297-serialize-and-deserialize-binary-tree)

## 08. Tries
- [208 Implement Trie](mainMaterial.md#208-implement-trie)
- [211 Design Add and Search Words](mainMaterial.md#211-design-add-and-search-words)
- [212 Word Search II](mainMaterial.md#212-word-search-ii)

## 09. Heap / Priority Queue
- [215 Kth Largest Element in Array](mainMaterial.md#215-kth-largest-element-in-array)
- [703 Kth Largest Element in Stream](mainMaterial.md#703-kth-largest-element-in-stream)
- [973 K Closest Points to Origin](mainMaterial.md#973-k-closest-points-to-origin)
- [621 Task Scheduler](mainMaterial.md#621-task-scheduler)
- [295 Find Median from Data Stream](mainMaterial.md#295-find-median-from-data-stream)

## 10. Backtracking
- [78 Subsets](mainMaterial.md#78-subsets)
- [39 Combination Sum](mainMaterial.md#39-combination-sum)
- [46 Permutations](mainMaterial.md#46-permutations)
- [90 Subsets II](mainMaterial.md#90-subsets-ii)
- [40 Combination Sum II](mainMaterial.md#40-combination-sum-ii)
- [79 Word Search](mainMaterial.md#79-word-search)
- [131 Palindrome Partitioning](mainMaterial.md#131-palindrome-partitioning)
- [17 Letter Combinations of Phone Number](mainMaterial.md#17-letter-combinations-of-phone-number)
- [51 N Queens](mainMaterial.md#51-n-queens)
# 🧠 回溯模板总结

```python
def backtrack(路径, 选择列表, ...):
    # 终止条件
    if 满足结束条件:
        result.append(路径.copy())  # ⚠️ 必须copy
        return
    
    # 遍历选择列表
    for 选择 in 选择列表:
        # 剪枝
        if 不合法:
            continue 或 break
        
        # 做选择
        路径.append(选择)
        
        # 递归
        backtrack(路径, 新的选择列表, ...)
        
        # 撤销选择
        路径.pop()
```

## 🎯 Key Points

```
✅ 回溯三步：选择 → 递归 → 撤销
✅ 必须复制path：result.append(path.copy())
✅ 允许重复：递归传 i
✅ 防止重复组合：从 start 开始遍历
✅ 剪枝优化：排序后提前break
```

---
## 11. Graphs
- [200 Number of Islands](mainMaterial.md#200-number-of-islands)
- [695 Max Area of Island](mainMaterial.md#695-max-area-of-island)
- [133 Clone Graph](mainMaterial.md#133-clone-graph)
- [417 Pacific Atlantic Water Flow](mainMaterial.md#417-pacific-atlantic-water-flow)
- [130 Surrounded Regions](mainMaterial.md#130-surrounded-regions)
- [994 Rotting Oranges](mainMaterial.md#994-rotting-oranges)
- [207 Course Schedule](mainMaterial.md#207-course-schedule)
- [210 Course Schedule II](mainMaterial.md#210-course-schedule-ii)
- [684 Redundant Connection](mainMaterial.md#684-redundant-connection)
- [269 Alien Dictionary](mainMaterial.md#269-alien-dictionary)

## 12. Advanced Graphs
- [332 Reconstruct Itinerary](mainMaterial.md#332-reconstruct-itinerary)
- [1584 Min Cost to Connect All Points](mainMaterial.md#1584-min-cost-to-connect-all-points)
- [743 Network Delay Time](mainMaterial.md#743-network-delay-time)
- [778 Swim in Rising Water](mainMaterial.md#778-swim-in-rising-water)
- [787 Cheapest Flights Within K Stops](mainMaterial.md#787-cheapest-flights-within-k-stops)

## 13. 1-D Dynamic Programming
- [70 Climbing Stairs](mainMaterial.md#70-climbing-stairs)
- [746 Min Cost Climbing Stairs](mainMaterial.md#746-min-cost-climbing-stairs)
- [198 House Robber](mainMaterial.md#198-house-robber)
- [213 House Robber II](mainMaterial.md#213-house-robber-ii)
- [5 Longest Palindromic Substring](mainMaterial.md#5-longest-palindromic-substring)
- [647 Palindromic Substrings](mainMaterial.md#647-palindromic-substrings)
- [91 Decode Ways](mainMaterial.md#91-decode-ways)
- [322 Coin Change](mainMaterial.md#322-coin-change)
- [152 Maximum Product Subarray](mainMaterial.md#152-maximum-product-subarray)
- [300 Longest Increasing Subsequence](mainMaterial.md#300-longest-increasing-subsequence)

## 14. 2-D Dynamic Programming
- [62 Unique Paths](mainMaterial.md#62-unique-paths)
- [1143 Longest Common Subsequence](mainMaterial.md#1143-longest-common-subsequence)
- [309 Best Time to Buy and Sell Stock with Cooldown](mainMaterial.md#309-best-time-to-buy-and-sell-stock-with-cooldown)
- [518 Coin Change II](mainMaterial.md#518-coin-change-ii)
- [494 Target Sum](mainMaterial.md#494-target-sum)
- [97 Interleaving String](mainMaterial.md#97-interleaving-string)
- [72 Edit Distance](mainMaterial.md#72-edit-distance)
- [115 Distinct Subsequences](mainMaterial.md#115-distinct-subsequences)

## 15. Greedy
- [53 Maximum Subarray](mainMaterial.md#53-maximum-subarray)
- [55 Jump Game](mainMaterial.md#55-jump-game)
- [45 Jump Game II](mainMaterial.md#45-jump-game-ii)
- [134 Gas Station](mainMaterial.md#134-gas-station)
- [846 Hand of Straights](mainMaterial.md#846-hand-of-straights)
- [1899 Merge Triplets to Form Target Triplet](mainMaterial.md#1899-merge-triplets-to-form-target-triplet)
- [763 Partition Labels](mainMaterial.md#763-partition-labels)
- [678 Valid Parenthesis String](mainMaterial.md#678-valid-parenthesis-string)

## 16. Intervals
- [56 Merge Intervals](mainMaterial.md#56-merge-intervals)
- [57 Insert Interval](mainMaterial.md#57-insert-interval)
- [435 Non-overlapping Intervals](mainMaterial.md#435-non-overlapping-intervals)
- [920 Meeting Rooms](mainMaterial.md#920-meeting-rooms)
- [2402 Meeting Rooms III](mainMaterial.md#2402-meeting-rooms-iii)

## 17. Math & Geometry
- [48 Rotate Image](mainMaterial.md#48-rotate-image)
- [54 Spiral Matrix](mainMaterial.md#54-spiral-matrix)
- [73 Set Matrix Zeroes](mainMaterial.md#73-set-matrix-zeroes)
- [202 Happy Number](mainMaterial.md#202-happy-number)
- [66 Plus One](mainMaterial.md#66-plus-one)
- [50 Pow(x, n)](mainMaterial.md#50-powx-n)

## 18. Bit Manipulation
- [136 Single Number](mainMaterial.md#136-single-number)
- [191 Number of 1 Bits](mainMaterial.md#191-number-of-1-bits)
- [338 Counting Bits](mainMaterial.md#338-counting-bits)
- [268 Missing Number](mainMaterial.md#268-missing-number)
- [371 Sum of Two Integers](mainMaterial.md#371-sum-of-two-integers)
- [190 Reverse Bits](mainMaterial.md#190-reverse-bits)



# NeetCode 250 独有题目（不在 NeetCode 150 中）

## 1. Arrays & Hashing
- [1929 Concatenation of Array](extraMaterial.md#1929-concatenation-of-array)
- [14 Longest Common Prefix](extraMaterial.md#14-longest-common-prefix)
- [27 Remove Element](extraMaterial.md#27-remove-element)
- [169 Majority Element](extraMaterial.md#169-majority-element)
- [705 Design HashSet](extraMaterial.md#705-design-hashset)
- [706 Design HashMap](extraMaterial.md#706-design-hashmap)
- [912 Sort an Array](extraMaterial.md#912-sort-an-array)
- [75 Sort Colors](extraMaterial.md#75-sort-colors)
- [304 Range Sum Query 2D Immutable](extraMaterial.md#304-range-sum-query-2d-immutable)
- [122 Best Time to Buy And Sell Stock II](extraMaterial.md#122-best-time-to-buy-and-sell-stock-ii)
- [229 Majority Element II](extraMaterial.md#229-majority-element-ii)
- [560 Subarray Sum Equals K](extraMaterial.md#560-subarray-sum-equals-k)
- [41 First Missing Positive](extraMaterial.md#41-first-missing-positive)

## 2. Two Pointers
- [344 Reverse String](extraMaterial.md#344-reverse-string)
- [680 Valid Palindrome II](extraMaterial.md#680-valid-palindrome-ii)
- [1768 Merge Strings Alternately](extraMaterial.md#1768-merge-strings-alternately)
- [88 Merge Sorted Array](extraMaterial.md#88-merge-sorted-array)
- [26 Remove Duplicates From Sorted Array](extraMaterial.md#26-remove-duplicates-from-sorted-array)
- [18 4Sum](extraMaterial.md#18-4sum)
- [189 Rotate Array](extraMaterial.md#189-rotate-array)
- [881 Boats to Save People](extraMaterial.md#881-boats-to-save-people)

## 3. Sliding Window
- [219 Contains Duplicate II](extraMaterial.md#219-contains-duplicate-ii)
- [209 Minimum Size Subarray Sum](extraMaterial.md#209-minimum-size-subarray-sum)
- [658 Find K Closest Elements](extraMaterial.md#658-find-k-closest-elements)

## 4. Stack
- [682 Baseball Game](extraMaterial.md#682-baseball-game)
- [225 Implement Stack Using Queues](extraMaterial.md#225-implement-stack-using-queues)
- [232 Implement Queue using Stacks](extraMaterial.md#232-implement-queue-using-stacks)
- [735 Asteroid Collision](extraMaterial.md#735-asteroid-collision)
- [901 Online Stock Span](extraMaterial.md#901-online-stock-span)
- [71 Simplify Path](extraMaterial.md#71-simplify-path)
- [394 Decode String](extraMaterial.md#394-decode-string)
- [895 Maximum Frequency Stack](extraMaterial.md#895-maximum-frequency-stack)

## 5. Binary Search
- [35 Search Insert Position](extraMaterial.md#35-search-insert-position)
- [374 Guess Number Higher Or Lower](extraMaterial.md#374-guess-number-higher-or-lower)
- [69 Sqrt(x)](extraMaterial.md#69-sqrtx)
- [1011 Capacity to Ship Packages Within D Days](extraMaterial.md#1011-capacity-to-ship-packages-within-d-days)
- [81 Search In Rotated Sorted Array II](extraMaterial.md#81-search-in-rotated-sorted-array-ii)
- [410 Split Array Largest Sum](extraMaterial.md#410-split-array-largest-sum)
- [1095 Find in Mountain Array](extraMaterial.md#1095-find-in-mountain-array)

## 6. Linked List
- [92 Reverse Linked List II](extraMaterial.md#92-reverse-linked-list-ii)
- [622 Design Circular Queue](extraMaterial.md#622-design-circular-queue)
- [460 LFU Cache](extraMaterial.md#460-lfu-cache)

## 7. Trees
- [94 Binary Tree Inorder Traversal](extraMaterial.md#94-binary-tree-inorder-traversal)
- [144 Binary Tree Preorder Traversal](extraMaterial.md#144-binary-tree-preorder-traversal)
- [145 Binary Tree Postorder Traversal](extraMaterial.md#145-binary-tree-postorder-traversal)
- [701 Insert into a Binary Search Tree](extraMaterial.md#701-insert-into-a-binary-search-tree)
- [450 Delete Node in a BST](extraMaterial.md#450-delete-node-in-a-bst)
- [427 Construct Quad Tree](extraMaterial.md#427-construct-quad-tree)
- [337 House Robber III](extraMaterial.md#337-house-robber-iii)
- [1325 Delete Leaves With a Given Value](extraMaterial.md#1325-delete-leaves-with-a-given-value)

## 8. Heap / Priority Queue
- [1834 Single Threaded CPU](extraMaterial.md#1834-single-threaded-cpu)
- [767 Reorganize String](extraMaterial.md#767-reorganize-string)
- [1405 Longest Happy String](extraMaterial.md#1405-longest-happy-string)
- [1094 Car Pooling](extraMaterial.md#1094-car-pooling)
- [502 IPO](extraMaterial.md#502-ipo)

## 9. Backtracking
- [1863 Sum of All Subsets XOR Total](extraMaterial.md#1863-sum-of-all-subsets-xor-total)
- [77 Combinations](extraMaterial.md#77-combinations)
- [47 Permutations II](extraMaterial.md#47-permutations-ii)
- [473 Matchsticks to Square](extraMaterial.md#473-matchsticks-to-square)
- [698 Partition to K Equal Sum Subsets](extraMaterial.md#698-partition-to-k-equal-sum-subsets)
- [52 N Queens II](extraMaterial.md#52-n-queens-ii)
- [140 Word Break II](extraMaterial.md#140-word-break-ii)

## 10. Tries
- [2707 Extra Characters in a String](extraMaterial.md#2707-extra-characters-in-a-string)

## 11. Graphs
- [463 Island Perimeter](extraMaterial.md#463-island-perimeter)
- [953 Verifying An Alien Dictionary](extraMaterial.md#953-verifying-an-alien-dictionary)
- [997 Find the Town Judge](extraMaterial.md#997-find-the-town-judge)
- [752 Open The Lock](extraMaterial.md#752-open-the-lock)
- [1462 Course Schedule IV](extraMaterial.md#1462-course-schedule-iv)
- [721 Accounts Merge](extraMaterial.md#721-accounts-merge)
- [399 Evaluate Division](extraMaterial.md#399-evaluate-division)
- [310 Minimum Height Trees](extraMaterial.md#310-minimum-height-trees)

## 12. Advanced Graphs
- [1631 Path with Minimum Effort](extraMaterial.md#1631-path-with-minimum-effort)
- [1489 Find Critical and Pseudo Critical Edges in Minimum Spanning Tree](extraMaterial.md#1489-find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree)
- [2392 Build a Matrix With Conditions](extraMaterial.md#2392-build-a-matrix-with-conditions)
- [2709 Greatest Common Divisor Traversal](extraMaterial.md#2709-greatest-common-divisor-traversal)

## 13. 1-D Dynamic Programming
- [1137 N-th Tribonacci Number](extraMaterial.md#1137-n-th-tribonacci-number)
- [377 Combination Sum IV](extraMaterial.md#377-combination-sum-iv)
- [279 Perfect Squares](extraMaterial.md#279-perfect-squares)
- [343 Integer Break](extraMaterial.md#343-integer-break)
- [1406 Stone Game III](extraMaterial.md#1406-stone-game-iii)

## 14. 2-D Dynamic Programming
- [63 Unique Paths II](extraMaterial.md#63-unique-paths-ii)
- [64 Minimum Path Sum](extraMaterial.md#64-minimum-path-sum)
- [1049 Last Stone Weight II](extraMaterial.md#1049-last-stone-weight-ii)
- [877 Stone Game](extraMaterial.md#877-stone-game)
- [1140 Stone Game II](extraMaterial.md#1140-stone-game-ii)

## 15. Greedy
- [860 Lemonade Change](extraMaterial.md#860-lemonade-change)
- [918 Maximum Sum Circular Subarray](extraMaterial.md#918-maximum-sum-circular-subarray)
- [978 Longest Turbulent Subarray](extraMaterial.md#978-longest-turbulent-subarray)
- [1871 Jump Game VII](extraMaterial.md#1871-jump-game-vii)
- [649 Dota2 Senate](extraMaterial.md#649-dota2-senate)
- [135 Candy](extraMaterial.md#135-candy)

## 16. Intervals
- [2402 Meeting Rooms III](extraMaterial.md#2402-meeting-rooms-iii)

## 17. Math & Geometry
- [168 Excel Sheet Column Title](extraMaterial.md#168-excel-sheet-column-title)
- [1071 Greatest Common Divisor of Strings](extraMaterial.md#1071-greatest-common-divisor-of-strings)
- [2807 Insert Greatest Common Divisors in Linked List](extraMaterial.md#2807-insert-greatest-common-divisors-in-linked-list)
- [867 Transpose Matrix](extraMaterial.md#867-transpose-matrix)
- [13 Roman to Integer](extraMaterial.md#13-roman-to-integer)

## 18. Bit Manipulation
- [67 Add Binary](extraMaterial.md#67-add-binary)
- [201 Bitwise AND of Numbers Range](extraMaterial.md#201-bitwise-and-of-numbers-range)
- [3133 Minimum Array End](extraMaterial.md#3133-minimum-array-end)

- Total Problems: 150

