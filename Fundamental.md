# UofT MEng Notes

Personal ML knowledge base.

## 📂 Categories
- [Numpy](#numpy技巧)
- [激活函数和损失函数](#激活函数--损失函数-速查表)
- [KNN](#knnk-nearest-neighbors复习笔记)
- [K-means](#k-means-算法复习笔记--代码练习)
- [PCA](#pca主成分分析复习笔记--代码练习)
- [Linear Regression](#linear-regression-复习笔记)
- [logistic Regression](#logistic-regression-复习笔记)
- [Support Vector Machine](#svm支持向量机复习笔记)
- [Lagrangian SVM derivation](#拉格朗日乘子法--svm-推导笔记)
- [Random Forest & XGBoost](#决策树--random-forest--xgboost-复习笔记)










# Numpy技巧
## NumPy 广播（Broadcasting）规则速查 - Kmeans 有用

## 核心规则

**从右往左逐维度对齐，每个维度必须满足以下之一：**

1. 两个维度相同
2. 其中一个是 1 → 自动复制扩展成另一个的大小

不满足 → 报错。

---

## 图解

```
(3, 1, 2)    ← X[:, None]
(1, 2, 2)    ← centroids[None, :]
-----------
 ↓  ↓  ↓
 3  2  2     ← 结果 shape

维度2:  2 vs 2 → 相同，OK
维度1:  1 vs 2 → 1 复制成 2
维度0:  3 vs 1 → 1 复制成 3
```

---

## 常见例子

```
(4,)   + (1,)     → (4,)       ✅  1 扩展成 4
(3, 4) + (1, 4)   → (3, 4)     ✅  1 扩展成 3
(3, 4) + (4,)     → (3, 4)     ✅  左边自动补1变成 (1,4)
(3, 4) + (3, 1)   → (3, 4)     ✅  1 扩展成 4
(3, 4) + (2, 4)   → ERROR       ❌  3 vs 2，都不是1
(2, 1, 3) + (4, 3) → (2, 4, 3) ✅  左边补1变(1,4,3)，然后1→2
```

---

## 维度不够时怎么办？

**短的那个在左边补 1**，然后再按规则广播。

```
A: (3, 4)     →  看作 (1, 3, 4)
B: (2, 1, 4)

对齐后：
(1, 3, 4)
(2, 1, 4)
----------
(2, 3, 4)    ✅
```

---

## 记忆口诀

> **右对齐，逢 1 扩，不同非 1 就报错。**
---
# 激活函数 & 损失函数 速查表

---

## 1. 输出层激活函数

这些函数加在模型最后一层，把原始输出变成有意义的值。

### Sigmoid

```
sigmoid(z) = 1 / (1 + e^(-z))

输入：任意实数
输出：(0, 1) 之间的一个概率
用途：二分类
```

### Softmax

```
softmax(zi) = e^zi / Σ(e^zj)

输入：k 个实数
输出：k 个概率，加起来 = 1
用途：多分类
```
---

## 2. 损失函数

### MSE（Mean Squared Error）

```
L = (1/n) × Σ(yi - ŷi)²

配合：无激活（回归）
任务：回归
直觉：预测值和真实值差多远
```

### MAE（Mean Absolute Error）

```
L = (1/n) × Σ|yi - ŷi|

配合：无激活（回归）
任务：回归（对异常值更鲁棒）
直觉：和 MSE 类似，但不放大大误差
```

### BCE（Binary Cross Entropy）

```
L = -(1/n) × Σ[yi × log(ŷi) + (1-yi) × log(1-ŷi)]

配合：sigmoid
任务：二分类
直觉：预测概率和真实标签(0/1)差多远
```

### CE（Categorical Cross Entropy）

```
L = -(1/n) × Σ Σ yij × log(ŷij)
  = -(1/n) × Σ log(正确类别的预测概率)

配合：softmax
任务：多分类
直觉：正确类别的概率越高，损失越小
```

### Hinge Loss

```
L = (1/n) × Σ max(0, 1 - yi × ŷi)

配合：无激活（输出原始分数）
任务：二分类（SVM 用的）
直觉：只要分对且有足够间隔就不惩罚
```

---

## 3. 经典搭配（必背）

```
任务         输出层激活     损失函数         例子
─────────   ──────────    ──────────      ──────
回归         无            MSE / MAE       预测房价
二分类       sigmoid       BCE             是否垃圾邮件
多分类       softmax       CE              识别猫/狗/鸟
SVM 分类     无            Hinge Loss      文本分类
```

这是固定搭配，不要混着用：
- sigmoid + MSE → 非凸，优化困难 ❌
    Loss = (y - sigmoid(wx+b))²

    sigmoid 是 S 形曲线，套上平方之后
    损失函数变得弯弯曲曲，出现多个坑

    梯度下降可能掉进一个小坑（局部最优）出不来
- softmax + BCE → 维度不对 ❌
- 回归 + CE → 没有意义 ❌

---

## 4. 隐藏层激活函数

这些加在中间层，给模型引入非线性能力。和输出层激活函数是不同的东西。

### ReLU（最常用）

```
ReLU(z) = max(0, z)

z > 0 → 输出 z
z ≤ 0 → 输出 0

优点：计算快，缓解梯度消失
缺点：负数区域梯度为 0（"死神经元"）
```

### Leaky ReLU

```
LeakyReLU(z) = z      if z > 0
               0.01z   if z ≤ 0

解决 ReLU 死神经元问题，负数区域给一个小斜率
```

### Tanh

```
tanh(z) = (e^z - e^(-z)) / (e^z + e^(-z))

输出：(-1, 1)
类似 sigmoid 但中心在 0，收敛更快
```

### Sigmoid（作为隐藏层）

```
现在很少用在隐藏层了
缺点：梯度消失（两端梯度接近 0）、输出不以 0 为中心
```

---

## 5. 隐藏层 vs 输出层 总结

```
隐藏层激活（给模型非线性能力）：
  首选 ReLU → 大部分情况
  Leaky ReLU → 担心死神经元时
  Tanh → RNN 中常用

输出层激活（把输出变成有意义的值）：
  无 → 回归
  sigmoid → 二分类
  softmax → 多分类
```
# KNN（K-Nearest Neighbors）复习笔记

---

## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 有监督（分类 & 回归都能做） |
| **核心思想** | 看最近的 k 个邻居，少数服从多数 |
| **训练过程** | 没有！直接存储所有数据（懒学习） |
| **预测过程** | 算距离 → 找 k 个最近的 → 投票/取均值 |
| **时间复杂度** | 训练 O(1)，预测 O(n×d) |

## 2. 算法流程

```
1. 存储所有训练数据（没有训练过程）
2. 来了新数据 x：
   a. 算 x 到所有训练点的距离
   b. 找距离最近的 k 个点
   c. 分类：k 个邻居投票，多数类胜出
      回归：k 个邻居的 y 取均值
```

## 3. 距离度量

```
欧氏距离（最常用）：sqrt(Σ(xi - yi)²)
曼哈顿距离：        Σ|xi - yi|
闵可夫斯基距离：    (Σ|xi - yi|^p)^(1/p)
  p=1 → 曼哈顿
  p=2 → 欧氏
```

## 4. K 的选择

```
K 太小（如 K=1）→ 对噪声敏感，容易过拟合
K 太大（如 K=n）→ 所有点都投票，等于预测多数类，欠拟合

选择方法：交叉验证，通常选奇数（避免平票）
经验值：sqrt(n) 附近
```

## 5. 关键注意事项

```
1. 必须标准化：量纲不同的特征会主导距离
2. 维度灾难：高维空间中所有点的距离趋于相同，KNN 失效
3. 数据量大时预测很慢：每次预测都要算 n 个距离
4. 不平衡数据：多数类邻居多，可用加权投票（距离越近权重越大）
```

## 6. 手搓代码（numpy）

### 分类

```python
import numpy as np
from collections import Counter

def knn_classify(X_train, y_train, x_new, k):
    """
    X_train: (n, d) 训练数据
    y_train: (n,) 训练标签
    x_new:   (d,) 一个新数据点
    k:       int 邻居数
    返回：预测标签
    """
    # 1. 算距离
    dists = np.linalg.norm(X_train - x_new, axis=1)  # (n,)

    # 2. 找最近的 k 个
    k_idx = np.argsort(dists)[:k]

    # 3. 投票
    k_labels = y_train[k_idx]
    most_common = np.bincount(k_labels).argmax()

    return most_common
```

### 回归

```python
def knn_regress(X_train, y_train, x_new, k):
    """返回：预测值"""
    # 1. 算距离
    dists = np.linalg.norm(X_train - x_new, axis=1)

    # 2. 找最近的 k 个
    k_idx = np.argsort(dists)[:k]

    # 3. 取均值
    return y_train[k_idx].mean()
```

### 批量预测

```python
def knn_predict(X_train, y_train, X_test, k):
    """对所有测试数据预测"""
    return np.array([knn_classify(X_train, y_train, x, k) for x in X_test])
```

### 逐行解读

```
np.linalg.norm(X_train - x_new, axis=1)
  X_train - x_new → (n, d) 广播减法，每个训练点减去新点
  norm(axis=1)     → 沿特征维度求距离，得到 (n,) 个距离值

np.argsort(dists)[:k]
  argsort → 距离从小到大的索引
  [:k]    → 取前 k 个（最近的）

Counter(k_labels).most_common(1)[0][0]
  Counter → 统计每个标签出现次数
  most_common(1) → 出现最多的 1 个
  [0][0]  → 取出标签值
```

## 7. sklearn 实现

```python
from sklearn.neighbors import KNeighborsClassifier, KNeighborsRegressor
from sklearn.preprocessing import StandardScaler

# 标准化（KNN 必须做！）
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# 分类
knn = KNeighborsClassifier(
    n_neighbors=5,          # K 值
    weights='uniform',      # 'uniform' 等权投票 / 'distance' 距离加权
    metric='minkowski', p=2 # 欧氏距离
)
knn.fit(X_train_s, y_train)
y_pred = knn.predict(X_test_s)

# 回归
knn_reg = KNeighborsRegressor(n_neighbors=5)
knn_reg.fit(X_train_s, y_train)
y_pred = knn_reg.predict(X_test_s)
```

## 8. 和其他算法对比

```
              KNN              SVM            Logistic
训练速度      O(1) 无训练       慢              快
预测速度      慢 O(n×d)        快              快
需要标准化    必须              推荐            推荐
处理高维      差（维度灾难）    好              好
可解释性      直觉（看邻居）    差              好（看权重）
```

## 9. 面试高频问答

**Q1: KNN 为什么叫懒学习？**
训练时什么都不做，只存数据。所有计算都在预测时发生。

**Q2: K=1 和 K=n 的区别？**
K=1 只看最近一个点，过拟合；K=n 看所有点，等于预测整体多数类，欠拟合。

**Q3: 怎么加速 KNN？**
KD-Tree 或 Ball Tree 索引，sklearn 里设置 algorithm='kd_tree'。

**Q4: KNN 能处理多分类吗？**
天然支持，投票就行，不需要像 SVM 那样拼凑。

**Q5: KNN 和 K-Means 的区别？**
KNN 是有监督分类（K 是邻居数），K-Means 是无监督聚类（K 是簇数），完全不同。
---
# K-Means 算法复习笔记 & 代码练习

## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 无监督聚类 |
| **目标** | 最小化 WCSS（簇内平方和）：J = Σ Σ ‖x - μi‖² |
| **输入** | 数据集 X，簇数 k |
| **输出** | k 个簇的划分 + k 个质心 |
| **时间复杂度** | O(n · k · d · t) |

## 2. 算法流程（4 步循环）

```
1. 初始化：随机选 k 个质心
2. 分配（E步）：每个点 → 最近质心的簇
3. 更新（M步）：质心 = 簇内均值
4. 重复 2-3 直到收敛
```

## 3. 关键知识点

- **K-Means++**：初始化优化，首个质心随机选，后续按距离平方概率选，避免局部最优
- **选 K**：肘部法（WCSS 拐点）、轮廓系数（Silhouette Score）
- **收敛性**：保证收敛（J 单调递减），不保证全局最优 → 多次运行取最优
- **局限**：假设球形簇、对异常值敏感、需预设 k、高维失效

## 4. 对比记忆

| | K-Means | K-Medoids | GMM |
|--|---------|-----------|-----|
| 中心 | 均值 | 实际数据点 | 均值+协方差 |
| 簇形状 | 球形 | 任意 | 椭圆形 |
| 抗噪 | 弱 | 强 | 中 |
| 速度 | 快 | 慢 | 中 |

---

## 5. 代码练习

---


**参考答案 ↓**

```python
import numpy as np

def kmeans(X, k, max_iters=100):
    n, d = X.shape
    # 1. 随机初始化质心
    idx = np.random.choice(n, k, replace=False)
    centroids = X[idx]

    for _ in range(max_iters):
        '''
        X[:, None]   → 在第1个位置插入 → (n, 1, d)
        X[None, :]   → 在第0个位置插入 → (1, n, d)
        X[:, :, None] → 在第2个位置插入 → (n, d, 1)
        '''
        # 2. 分配：计算每个点到每个质心的距离 （n,1,d)-(1,k,d)-->(n,k,d)然后再去np.linalg.norm(axis=2)
        dists = np.linalg.norm(X[:, None] - centroids[None, :], axis=2)  # (n, k)
        labels = np.argmin(dists, axis=1) #计算index用

        # 3. 更新：重新计算质心 labels==i是个bool value 像个mask， e.g. [true,false,true,false...]
        new_centroids = np.array([X[labels == i].mean(axis=0) for i in range(k)])

        # 4. 收敛判断
        if np.all(np.abs(centroids - new_centroids) < 1e-6):
            break
        centroids = new_centroids

    return labels, centroids
```

关键行解读：
- `X[:, None] - centroids[None, :]` 利用广播计算 (n, k, d) 的差矩阵
- `np.argmin(dists, axis=1)` 得到每个点最近的质心索引

---


### 6 Scikit-Learn 实现（最常用）

```python
import numpy as np
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# ========== 数据准备 ==========
from sklearn.datasets import make_blobs
X, y_true = make_blobs(n_samples=300, centers=4, random_state=42)

# 标准化（重要！量纲不同时必须做）
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# ========== 基本用法 ==========
kmeans = KMeans(
    n_clusters=4,        # 簇数
    init='k-means++',    # 初始化方法，默认就是 k-means++
    n_init=10,           # 跑10次取最优，默认10
    max_iter=300,        # 最大迭代次数
    random_state=42
)
labels = kmeans.fit_predict(X_scaled)

# 关键属性
print("质心:", kmeans.cluster_centers_)   # (k, n_features)
print("标签:", kmeans.labels_)            # (n_samples,)
print("WCSS:", kmeans.inertia_)           # 簇内平方和
print("迭代次数:", kmeans.n_iter_)

# ========== 肘部法选 K ==========
wcss = []
K_range = range(1, 11)
for k in K_range:
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    km.fit(X_scaled)
    wcss.append(km.inertia_)

plt.plot(K_range, wcss, 'bo-')
plt.xlabel('K')
plt.ylabel('WCSS (inertia)')
plt.title('Elbow Method')
plt.show()

# ========== 轮廓系数选 K ==========
for k in range(2, 11):
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    lab = km.fit_predict(X_scaled)
    score = silhouette_score(X_scaled, lab)
    print(f"K={k}, Silhouette={score:.3f}")

# ========== 可视化（2D） ==========
plt.scatter(X_scaled[:, 0], X_scaled[:, 1], c=labels, cmap='viridis', s=30, alpha=0.6)
centers = kmeans.cluster_centers_
plt.scatter(centers[:, 0], centers[:, 1], c='red', marker='X', s=200, edgecolors='black')
plt.title('K-Means Clustering')
plt.show()
```

**sklearn 核心 API 速记：**

```
KMeans(n_clusters, init, n_init, max_iter, random_state)
  .fit(X)              → 训练
  .predict(X)          → 预测新数据的簇
  .fit_predict(X)      → 训练+预测
  .cluster_centers_    → 质心坐标
  .labels_             → 训练数据的标签
  .inertia_            → WCSS
  .n_iter_             → 实际迭代次数
```

---

### 练习 1：处理空簇问题

练习 1 的实现中，如果某个簇没有被分配到任何点会怎样？请修改代码处理这种边界情况。

提示：`X[labels == i]` 可能为空，导致 `.mean()` 返回 nan。常见策略是重新随机选一个数据点作为该簇质心。

例子
数据点：[1, 1, 1, 1, 10]，k=3

初始质心：μ1=1, μ2=1, μ3=10

分配（距离相同时 argmin 取第一个）：
  1 → μ1 (距离0)
  1 → μ1 (距离0)
  1 → μ1 (距离0)
  1 → μ1 (距离0)
  10 → μ3 (距离0)

结果：
  μ1 = {1,1,1,1}
  μ2 = {}          ← 空！没人分给它
  μ3 = {10}
**参考答案 ↓**

```python
# 替换更新质心的那一行：
new_centroids = np.empty_like(centroids)
for i in range(k):
    members = X[labels == i]
    if len(members) == 0:
        # 空簇：随机重选一个数据点
        new_centroids[i] = X[np.random.randint(n)]
    else:
        new_centroids[i] = members.mean(axis=0)
```

---

### 练习 2（综合）：完整 Pipeline

用 sklearn 的 make_blobs 生成数据，用你自己实现的 K-Means 聚类，并可视化结果。

```python
from sklearn.datasets import make_blobs

# TODO:
# 1. 生成 300 个点，3 个簇
# 2. 用你的 kmeans() 聚类
# 3. 用 plt.scatter 可视化，不同簇不同颜色，标出质心
```

**参考答案 ↓**

```python
from sklearn.datasets import make_blobs
import matplotlib.pyplot as plt
import numpy as np

# 1. 生成数据
X, y_true = make_blobs(n_samples=300, centers=3, random_state=42)

# 2. 聚类
labels, centroids = kmeans(X, k=3)

# 3. 可视化
plt.scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', alpha=0.6, s=30)
plt.scatter(centroids[:, 0], centroids[:, 1],
            c='red', marker='X', s=200, edgecolors='black')
plt.title('K-Means Clustering Result')
plt.show()
```

---

## 6. 面试高频问答

**Q1: K-Means 一定会收敛吗？**
是，因为 WCSS 有下界（>=0）且每步单调递减。

**Q2: 为什么不保证全局最优？**
非凸优化问题，初始化不同结果不同。

**Q3: K-Means 与 EM-GMM 的关系？**
K-Means 是 GMM 的硬分配特例（协方差为 σ²I 且 σ→0）。

**Q4: 如何处理大规模数据？**
Mini-Batch K-Means，每次用随机子集更新。

**Q5: 特征量纲不同怎么办？**
先标准化（StandardScaler），否则大量纲特征主导距离。
---
# PCA（主成分分析）复习笔记 & 代码练习



## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 无监督降维 |
| **目标** | 找到方差最大的方向，将数据投影到低维空间 |
| **输入** | 数据集 X (n, d)，目标维度 k |
| **输出** | 降维后的数据 (n, k) |
| **前提** | 数据需要先中心化（减均值） |

## 2. 算法流程（5 步）

```
1. 中心化：X_centered = X - X.mean(axis=0)
2. 计算协方差矩阵：C = (X_centered.T @ X_centered) / (n-1)    shape (d, d), cov(x, y) = Σ(xi - x̄)(yi - ȳ) / (n-1)
(协方差举证是个2D数组，所以才能算eigenvalues 和 eigenvectors)
3. 特征值分解：eigenvalues, eigenvectors = eig(C)
4. 排序：按特征值从大到小排，取前 k 个特征向量
5. 投影：X_pca = X_centered @ top_k_eigenvectors                shape (n, k)
```

## 3. 关键知识点

- **为什么中心化？** 不减均值，第一主成分会指向均值方向，不是方差最大方向
- **协方差矩阵** (d, d)：对角线是每个特征的方差，非对角线是特征间的协方差
- **特征值**：代表该主成分方向上的方差大小，越大越重要
- **特征向量**：主成分的方向，彼此正交
- **解释方差比**：eigenvalue_i / sum(eigenvalues)，表示第 i 个主成分保留了多少信息
- **选 k**：累计解释方差比达到 85%~95% 即可
- **PCA vs SVD**：实践中常用 SVD 代替特征值分解，数值更稳定

## 4. 适用场景

- 高维数据可视化（降到 2D/3D）
- 去噪（丢掉方差小的成分）
- 特征降维（减少计算量，缓解维度灾难）
- 去多重共线性（回归前处理）

## 5. 局限性

- 只能捕捉线性关系（非线性用 Kernel PCA、t-SNE、UMAP）
- 主成分不容易解释实际含义
- 对特征量纲敏感 → 需要先标准化（StandardScaler）
- 假设方差大 = 信息多，不一定总成立

---

## 6. 手写 Numpy 实现

### 完整代码

```python
import numpy as np

def pca(X, k):
    """
    参数:
        X: np.ndarray, shape (n, d)
        k: int, 目标维度
    返回:
        X_pca: np.ndarray, shape (n, k) 降维后的数据
        components: np.ndarray, shape (k, d) 主成分方向
        explained_ratio: np.ndarray, shape (k,) 解释方差比
    """
    n, d = X.shape

    # 1. 中心化
    mean = X.mean(axis=0)             # (d,)
    X_centered = X - mean             # (n, d)

    # 2. 协方差矩阵
    cov = (X_centered.T @ X_centered) / (n - 1)   # (d, d)

    # 3. 特征值分解
    eigenvalues, eigenvectors = np.linalg.eigh(cov)  # eigh 用于对称矩阵

    # 4. 排序（eigh 返回的是升序，要反转）
    idx = np.argsort(eigenvalues)[::-1]       # 从大到小的索引
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]        # 注意：列是特征向量

    # 5. 取前 k 个，这里要转置，因为算出来的eigenvectors是按列看的
    components = eigenvectors[:, :k].T         # (k, d)
    explained_ratio = eigenvalues[:k] / eigenvalues.sum()

    # 6. 投影 dot product
    X_pca = X_centered @ eigenvectors[:, :k]   # (n, d) @ (d, k) = (n, k)

    return X_pca, components, explained_ratio
```

### 逐行解读

**中心化：**
```
X.mean(axis=0) → 每列的均值，shape (d,)
X - mean → 广播，每个点减去均值
```

**协方差矩阵：**
```
X_centered.T @ X_centered → (d, n) @ (n, d) = (d, d)
除以 (n-1) → 无偏估计
```

**为什么用 eigh 不用 eig？**
```
协方差矩阵是对称矩阵 → eigh 专门处理对称矩阵
优点：更快、数值更稳定、保证特征值是实数
```

**特征向量的取法：**
```
np.linalg.eigh 返回的 eigenvectors 中，每一列是一个特征向量
所以取前 k 个是 eigenvectors[:, :k]，不是 eigenvectors[:k]
```

## 8. Scikit-Learn 实现

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# ========== 数据准备 ==========
from sklearn.datasets import load_iris
X, y = load_iris(return_X_y=True)    # (150, 4) 四维数据

# 标准化（PCA 对量纲敏感，推荐先做）
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# ========== 基本用法 ==========
pca = PCA(n_components=2)            # 降到 2 维
X_pca = pca.fit_transform(X_scaled)  # (150, 2)

# 关键属性
pca.components_              # 主成分方向 (k, d)
pca.explained_variance_      # 每个主成分的方差（特征值）
pca.explained_variance_ratio_ # 解释方差比
pca.n_components_            # 实际使用的主成分数

print("解释方差比:", pca.explained_variance_ratio_)
print("累计:", pca.explained_variance_ratio_.sum())

# ========== 用比例选 k ==========
pca_auto = PCA(n_components=0.95)    # 自动选 k，保留 95% 方差
X_auto = pca_auto.fit_transform(X_scaled)
print("自动选了", pca_auto.n_components_, "个主成分")

# ========== 可视化（降到2D） ==========
plt.scatter(X_pca[:, 0], X_pca[:, 1], c=y, cmap='viridis', s=30)
plt.xlabel('PC1')
plt.ylabel('PC2')
plt.title('PCA Visualization')
plt.show()

# ========== 解释方差图（选 k 用） ==========
pca_full = PCA().fit(X_scaled)
plt.plot(range(1, 5), pca_full.explained_variance_ratio_.cumsum(), 'bo-')
plt.xlabel('Number of Components')
plt.ylabel('Cumulative Explained Variance')
plt.title('Explained Variance vs Components')
plt.axhline(y=0.95, color='r', linestyle='--', label='95% threshold')
plt.legend()
plt.show()
```

**sklearn PCA API 速记：**

```
PCA(n_components)
  n_components=2       → 降到2维
  n_components=0.95    → 自动选k，保留95%方差

  .fit_transform(X)    → 训练+降维
  .transform(X)        → 对新数据降维（用已有的主成分）
  .inverse_transform() → 从低维还原回高维（近似）
  .components_         → 主成分方向 (k, d)
  .explained_variance_ratio_ → 解释方差比
```

---

## 9. PyTorch 实现

PyTorch 没有内置 PCA，用 SVD 实现：

```python
import torch

def pca_torch(X, k, device='cuda'):
    """
    X: tensor, shape (n, d)
    """
    X = X.to(device)

    # 1. 中心化
    mean = X.mean(dim=0)
    X_centered = X - mean

    # 2. SVD 分解（代替特征值分解）
    # X_centered = U @ diag(S) @ V.T
    # V 的列就是主成分方向
    U, S, Vt = torch.linalg.svd(X_centered, full_matrices=False)

    # 3. 取前 k 个主成分投影
    X_pca = X_centered @ Vt.T[:, :k]    # (n, d) @ (d, k) = (n, k)

    # 解释方差比
    explained_var = (S ** 2) / (X.shape[0] - 1)
    explained_ratio = explained_var / explained_var.sum()

    return X_pca.cpu(), explained_ratio[:k].cpu()


# ========== 使用示例 ==========
from sklearn.datasets import load_iris
X_np, y = load_iris(return_X_y=True)
X_tensor = torch.tensor(X_np, dtype=torch.float32)

device = 'cuda' if torch.cuda.is_available() else 'cpu'
X_pca, ratio = pca_torch(X_tensor, k=2, device=device)

print("降维后 shape:", X_pca.shape)
print("解释方差比:", ratio)
```

**PyTorch 关键函数：**
```
torch.linalg.svd(X)     → U, S, Vt 分解
torch.mean(X, dim=0)    → 沿第0维求均值
S ** 2 / (n-1)           → 从奇异值算方差
```

**为什么用 SVD 不用 eigh？**
```
协方差矩阵 C = X.T @ X / (n-1)
对 C 做 eigh 和 对 X 做 SVD 得到的主成分方向一样
但 SVD 直接分解 X，不用先算协方差矩阵，数值更稳定
```

---

## 10. TensorFlow 实现

```python
import tensorflow as tf
import numpy as np

def pca_tf(X, k):
    """
    X: np.ndarray, shape (n, d)
    """
    X_tf = tf.constant(X, dtype=tf.float32)

    # 1. 中心化
    mean = tf.reduce_mean(X_tf, axis=0)
    X_centered = X_tf - mean

    # 2. SVD
    S, U, V = tf.linalg.svd(X_centered, full_matrices=False)
    # 注意：TF 的 svd 返回顺序是 S, U, V（跟 PyTorch/NumPy 不同！）

    # 3. 投影
    X_pca = X_centered @ V[:, :k]    # (n, d) @ (d, k) = (n, k)

    # 解释方差比
    explained_var = (S ** 2) / (X.shape[0] - 1)
    explained_ratio = explained_var / tf.reduce_sum(explained_var)

    return X_pca.numpy(), explained_ratio[:k].numpy()


# ========== 使用示例 ==========
from sklearn.datasets import load_iris
X, y = load_iris(return_X_y=True)

X_pca, ratio = pca_tf(X, k=2)
print("降维后 shape:", X_pca.shape)
print("解释方差比:", ratio)
```

**注意 TF 的 SVD 返回顺序不同：**
```
PyTorch:     U, S, Vt = torch.linalg.svd(X)
NumPy:       U, S, Vt = np.linalg.svd(X)
TensorFlow:  S, U, V  = tf.linalg.svd(X)     ← 顺序不同！且 V 不是转置
```

---

## 11. 三框架对比速查

| | sklearn | PyTorch | TensorFlow |
|--|---------|---------|------------|
| 内置 PCA | PCA 类 | 无（用 SVD） | 无（用 SVD） |
| GPU 加速 | 不支持 | 原生支持 | 原生支持 |
| 核心函数 | PCA().fit_transform() | torch.linalg.svd() | tf.linalg.svd() |
| SVD 返回顺序 | - | U, S, Vt | S, U, V |
| 自动选 k | n_components=0.95 | 手动算 | 手动算 |

---

## 12. 面试高频问答

**Q1: PCA 和 LDA 的区别？**
PCA 是无监督（最大化方差），LDA 是有监督（最大化类间距离/类内距离）。

**Q2: 什么时候用 PCA 什么时候用 t-SNE？**
PCA 保持全局结构，适合降维做后续任务；t-SNE 保持局部结构，适合可视化。

**Q3: PCA 能用于分类吗？**
PCA 本身不分类，但可以降维后再接分类器，减少过拟合和计算量。

**Q4: 为什么要标准化？**
不标准化的话，量纲大的特征方差大，主成分会被它主导。

**Q5: PCA 的时间复杂度？**
O(d²n + d³)，协方差矩阵 O(d²n)，特征值分解 O(d³)。特征维度 d 很大时用 SVD 更高效。


# Linear Regression 复习笔记

---

## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 有监督回归 |
| **目标** | 最小化预测值和真实值的均方误差 |
| **模型** | ŷ = X @ w + b |
| **损失函数** | MSE = Σ(yi - ŷi)² / n |
| **求解方式** | 正规方程（解析解）或 梯度下降（迭代） |

## 2. 模型表达

```
单特征：ŷ = w × x + b
多特征：ŷ = w1×x1 + w2×x2 + ... + wd×xd + b

矩阵形式（把 b 合并进 w）：
  X 加一列 1：X → (n, d+1)
  w 包含 b： w → (d+1,)
  ŷ = X @ w
  原来：ŷ = w1×x1 + w2×x2 + b

X = [[3, 5],        w = [w1, w2]     b = b
     [1, 2]]

ŷ = X @ w + b    ← 要单独加 b

X = [[3, 5, 1],     w = [w1, w2, b]
     [1, 2, 1]]
         ↑
       加的这列

ŷ = X @ w    ← b 自动包含在里面了
```

## 3. 损失函数

```
MSE = (1/n) × Σ(yi - ŷi)²

矩阵形式：
MSE = (1/n) × (y - Xw).T @ (y - Xw)
```
(A.T).T = A                    转置两次 = 自己

(A + B).T = A.T + B.T          加法的转置 = 各自转置

(AB).T = B.T @ A.T             乘法的转置 = 反过来乘
                                （最常用！）

(ABC).T = C.T @ B.T @ A.T     多个矩阵同理，全部反过来

(cA).T = c × A.T              标量提出来，不受影响
---

## 4. 正规方程推导

### 目标：找 w 使 MSE 最小

```
L(w) = (y - Xw).T @ (y - Xw)

展开：
L(w) = y.T@y - y.T@Xw - (Xw).T@y + (Xw).T@(Xw)
     = y.T@y - 2(Xw).T@y + w.T@X.T@X@w
```

### 对 w 求导，令导数为 0

```
∂L/∂w = -2X.T@y + 2X.T@X@w = 0
```

### 解方程

```
X.T@X@w = X.T@y
w = (X.T@X)⁻¹ @ X.T @ y
```

这就是正规方程，一步算出最优 w。

### 正规方程的限制

- X.T@X 必须可逆（特征不能线性相关）
- 时间复杂度 O(d³)，特征多时很慢（d > 10000 就不推荐）
- 数据太多也慢（需要算 X.T@X）

---

## 5. 梯度下降推导

### 当正规方程不好用时，用迭代的方法

```
损失函数：L = (1/n) × Σ(yi - ŷi)²

对 w 求梯度：
∂L/∂w = -(2/n) × X.T @ (y - Xw)

对 b 求梯度：
∂L/∂b = -(2/n) × Σ(yi - ŷi)
```

### 更新规则

```
w = w - lr × ∂L/∂w
b = b - lr × ∂L/∂b

lr = 学习率（步长），通常 0.01, 0.001
```

### 三种梯度下降

```
批量梯度下降（BGD）：每次用全部数据算梯度
  优点：稳定
  缺点：数据大时慢

随机梯度下降（SGD）：每次用 1 个样本算梯度
  优点：快
  缺点：震荡，不稳定

小批量梯度下降（Mini-batch）：每次用一小批数据
  优点：平衡速度和稳定性
  实际最常用，batch_size 通常 32, 64, 128
```

### 梯度下降直觉理解

```
想象你站在山上，闭着眼想走到最低点：

梯度 = 当前位置最陡的上坡方向
负梯度 = 最陡的下坡方向
学习率 = 每步走多大

太大 → 跳过最低点，来回震荡
太小 → 走太慢，要很久才到
```

---

## 6. 正则化

### 为什么需要？

特征多、数据少时容易过拟合（w 值很大），正则化限制 w 的大小。

### Ridge（L2 正则化）

```
Loss = MSE + α × Σ(wi²)

正规方程变成：
w = (X.T@X + αI)⁻¹ @ X.T @ y

特点：w 变小但不会变成 0，保留所有特征
```

### Lasso（L1 正则化）

```
Loss = MSE + α × Σ|wi|

特点：会把一些 w 压到 0，自动做特征选择
没有解析解，只能用梯度下降
```

### Elastic Net（L1 + L2）

```
Loss = MSE + α1 × Σ|wi| + α2 × Σ(wi²)

结合两者优点
```

### 对比

```
              Ridge       Lasso       Elastic Net
正则项        Σwi²        Σ|wi|       两者都有
特征选择      不能         能           能
解析解        有           没有         没有
适用场景      特征都有用   很多无关特征  特征多且有相关性
```

---

## 7. 评估指标

```
MSE  = Σ(yi - ŷi)² / n           均方误差
RMSE = sqrt(MSE)                  均方根误差（和 y 同单位）
MAE  = Σ|yi - ŷi| / n            平均绝对误差（对异常值更鲁棒）
R²   = 1 - Σ(yi-ŷi)² / Σ(yi-ȳ)²  决定系数（1=完美，0=和均值一样差）
```

---

## 8. 关键假设

```
1. 线性关系：y 和 X 之间是线性的
2. 独立性：样本之间独立
3. 同方差性：误差的方差是常数
4. 正态性：误差服从正态分布
5. 无多重共线性：特征之间不能高度相关
```

违反这些假设时，模型效果会变差或参数估计不可靠。

---

## 9. Scikit-Learn 实现

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# ========== 数据准备 ==========
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=200, n_features=5, noise=10, random_state=42)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 标准化
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)         # 注意：test 只 transform，不 fit

# ========== 基本线性回归 ==========
lr = LinearRegression()
lr.fit(X_train_s, y_train)

y_pred = lr.predict(X_test_s)

print("权重 w:", lr.coef_)           # (d,)
print("偏置 b:", lr.intercept_)      # 标量
print("MSE:", mean_squared_error(y_test, y_pred))
print("R²:", r2_score(y_test, y_pred))

# ========== Ridge 回归（L2） ==========
ridge = Ridge(alpha=1.0)              # alpha 越大正则化越强
ridge.fit(X_train_s, y_train)
print("Ridge R²:", r2_score(y_test, ridge.predict(X_test_s)))

# ========== Lasso 回归（L1） ==========
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_s, y_train)
print("Lasso R²:", r2_score(y_test, lasso.predict(X_test_s)))
print("Lasso 非零权重数:", np.sum(lasso.coef_ != 0))   # 特征选择效果
```

**sklearn API 速记：**

```
LinearRegression()
  .fit(X, y)         → 训练
  .predict(X)        → 预测
  .coef_             → 权重 w (d,)
  .intercept_        → 偏置 b

Ridge(alpha)         → L2 正则化
Lasso(alpha)         → L1 正则化

注意：
- test 数据只用 scaler.transform()，不能 fit_transform()
- alpha 越大，正则化越强，w 越小
```

---

## 10. PyTorch 实现

```python
import torch
import torch.nn as nn
import torch.optim as optim
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# ========== 数据准备 ==========
X, y = make_regression(n_samples=200, n_features=5, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

X_train_t = torch.tensor(X_train, dtype=torch.float32)
y_train_t = torch.tensor(y_train, dtype=torch.float32).unsqueeze(1)  # (n,) → (n,1)
X_test_t = torch.tensor(X_test, dtype=torch.float32)
y_test_t = torch.tensor(y_test, dtype=torch.float32).unsqueeze(1)

# ========== 定义模型 ==========
model = nn.Linear(5, 1)              # 5个特征 → 1个输出

# ========== 损失函数 + 优化器 ==========
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)
# Ridge 效果：加 weight_decay
# optimizer = optim.SGD(model.parameters(), lr=0.01, weight_decay=0.01)

# ========== 训练 ==========
for epoch in range(1000):
    # 前向传播
    y_pred = model(X_train_t)
    loss = criterion(y_pred, y_train_t)

    # 反向传播
    optimizer.zero_grad()       # 清空旧梯度
    loss.backward()             # 算新梯度
    optimizer.step()            # 更新 w

    if (epoch + 1) % 200 == 0:
        print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# ========== 评估 ==========
with torch.no_grad():
    y_pred = model(X_test_t)
    mse = criterion(y_pred, y_test_t)
    print(f"Test MSE: {mse.item():.4f}")

# 查看权重
print("w:", model.weight.data)
print("b:", model.bias.data)
```

**PyTorch 关键点：**

```
nn.Linear(d, 1)          → 线性层 = wx + b
nn.MSELoss()             → MSE 损失
optim.SGD(lr, weight_decay)  → SGD 优化器，weight_decay = L2 正则化
optimizer.zero_grad()    → 每次迭代前清空梯度
loss.backward()          → 自动求梯度
optimizer.step()         → 用梯度更新参数
torch.no_grad()          → 评估时不算梯度，省内存
unsqueeze(1)             → (n,) 变 (n,1)，匹配 MSELoss 的输入格式
```

---

## 11. TensorFlow 实现

```python
import tensorflow as tf
import numpy as np
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# ========== 数据准备 ==========
X, y = make_regression(n_samples=200, n_features=5, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# ========== 方式 1：用 Keras Sequential ==========
model = tf.keras.Sequential([
    tf.keras.layers.Dense(1, input_shape=(5,))       # 线性层
])

model.compile(
    optimizer=tf.keras.optimizers.SGD(learning_rate=0.01),
    loss='mse'
)

# 加 L2 正则化的写法：
# tf.keras.layers.Dense(1, kernel_regularizer=tf.keras.regularizers.l2(0.01))

model.fit(X_train, y_train, epochs=1000, batch_size=32, verbose=0)

# 评估
mse = model.evaluate(X_test, y_test, verbose=0)
print(f"Test MSE: {mse:.4f}")

# 查看权重
w, b = model.layers[0].get_weights()
print("w:", w.flatten())
print("b:", b)

# ========== 方式 2：手动梯度下降 ==========
w = tf.Variable(tf.random.normal([5, 1]))
b = tf.Variable(tf.zeros([1]))
lr = 0.01

X_train_tf = tf.constant(X_train, dtype=tf.float32)
y_train_tf = tf.constant(y_train.reshape(-1, 1), dtype=tf.float32)

for epoch in range(1000):
    with tf.GradientTape() as tape:
        y_pred = X_train_tf @ w + b
        loss = tf.reduce_mean((y_train_tf - y_pred) ** 2)

    grads = tape.gradient(loss, [w, b])
    w.assign_sub(lr * grads[0])        # w = w - lr * grad
    b.assign_sub(lr * grads[1])

    if (epoch + 1) % 200 == 0:
        print(f"Epoch {epoch+1}, Loss: {loss.numpy():.4f}")
```

**TensorFlow 关键点：**

```
Keras 方式：
  Dense(1)                   → 线性层
  compile(optimizer, loss)   → 配置优化器和损失
  fit(X, y, epochs)          → 训练
  kernel_regularizer=l2()    → L2 正则化

手动方式：
  tf.GradientTape()          → 记录计算过程，自动求梯度
  tape.gradient(loss, vars)  → 算梯度
  var.assign_sub(lr * grad)  → 更新参数
```

---

## 12. 三框架对比

| | sklearn | PyTorch | TensorFlow |
|--|---------|---------|------------|
| 求解方式 | 正规方程（默认） | 梯度下降 | 梯度下降 |
| 代码量 | 3 行 | 20+ 行 | 10+ 行（Keras） |
| GPU | 不支持 | 支持 | 支持 |
| 正则化 | Ridge / Lasso 类 | weight_decay | kernel_regularizer |
| 适用 | 快速实验 | 自定义模型 | 生产部署 |

---

## 13. 面试高频问答

**Q1: 正规方程和梯度下降怎么选？**
特征少（d < 10000）用正规方程快；特征多或数据量大用梯度下降。

**Q2: 为什么要标准化？**
梯度下降时，量纲不同导致损失函数等高线是椭圆形，收敛慢。标准化后变圆形，收敛快。正规方程不受影响，但标准化后系数更好解释。

**Q3: R² 可以是负数吗？**
可以。说明模型比直接预测均值还差。

**Q4: Ridge 和 Lasso 怎么选？**
认为所有特征都有用 → Ridge；想做特征选择 → Lasso；不确定 → Elastic Net。

**Q5: 多重共线性怎么处理？**
用 Ridge（L2 让 X.T@X+αI 一定可逆），或 PCA 降维后再回归。

# Logistic Regression 复习笔记

---

## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 有监督分类（名字有 regression 但其实是分类） |
| **模型** | z = X@w+b → ŷ = sigmoid(z) |
| **输出** | 概率值 (0~1) |
| **损失函数** | Binary Cross Entropy（交叉熵） |
| **求解** | 没有解析解，只能用梯度下降 |

## 2. 和 Linear Regression 的关系

```
Linear Regression:   X@w+b → ŷ（直接输出数值）
Logistic Regression: X@w+b → sigmoid → ŷ（输出概率）

就是多了一个 sigmoid
```

## 3. Sigmoid 函数

```
sigmoid(z) = 1 / (1 + e^(-z))

z = -∞  → 0
z = 0   → 0.5
z = +∞  → 1

作用：把任意实数压到 (0, 1) 之间，当概率用
```

**决策规则：**

```
ŷ >= 0.5 → 预测为 1（正类）
ŷ < 0.5  → 预测为 0（负类）

阈值 0.5 可以调，不是固定的
```

## 4. 为什么不用 MSE？

```
如果用 MSE + sigmoid → 损失函数是非凸的 → 很多局部最优 → 梯度下降容易卡住
如果用交叉熵 + sigmoid → 损失函数是凸的 → 只有一个最优 → 梯度下降保证收敛
```

## 5. 损失函数：Binary Cross Entropy

```
单个样本：
L = -[y × log(ŷ) + (1-y) × log(1-ŷ)]

全部样本：
L = -(1/n) × Σ[yi × log(ŷi) + (1-yi) × log(1-ŷi)]
```

**直觉理解：**

```
当 y=1 时：L = -log(ŷ)
  ŷ 接近 1 → -log(1) = 0      ← 预测对了，损失小
  ŷ 接近 0 → -log(0) = ∞      ← 预测错了，损失巨大

当 y=0 时：L = -log(1-ŷ)
  ŷ 接近 0 → -log(1) = 0      ← 预测对了，损失小
  ŷ 接近 1 → -log(0) = ∞      ← 预测错了，损失巨大
```

预测越自信且越错，惩罚越大。

## 6. 梯度

```
∂L/∂w = (1/n) × X.T @ (ŷ - y)
∂L/∂b = (1/n) × Σ(ŷi - yi)

和 Linear Regression 的梯度形式几乎一样！
区别只是 ŷ 过了 sigmoid
```

## 7. 多分类扩展：Softmax

```
二分类：sigmoid → 输出 1 个概率
多分类：softmax → 输出 k 个概率，加起来 = 1

softmax(zi) = e^zi / Σ(e^zj)

例：z = [2, 1, 0.5]
e^z = [7.39, 2.72, 1.65]
sum = 11.76
softmax = [0.63, 0.23, 0.14]   ← 三个类的概率，和为 1
```

**多分类损失函数：Categorical Cross Entropy**

```
L = -Σ yi × log(ŷi)    （y 是 one-hot 编码）
```

## 8. 评估指标

```
Accuracy  = 预测对的 / 总数           （类别不平衡时不靠谱）
Precision = TP / (TP + FP)           （预测为正的里面有多少真正）
Recall    = TP / (TP + FN)           （真正的正样本里抓到多少）
F1 Score  = 2 × Precision × Recall / (Precision + Recall)
AUC-ROC   = 不同阈值下 TPR vs FPR 的曲线下面积
```

```
              预测正    预测负
实际正        TP        FN
实际负        FP        TN
```

## 9. 正则化

跟 Linear Regression 一样：

```
L2（Ridge）：Loss + α × Σwi²     → 防过拟合
L1（Lasso）：Loss + α × Σ|wi|    → 特征选择

sklearn 中参数叫 C = 1/α，C 越小正则化越强（注意是反的）
```

## 10. sklearn 实现

```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report

# 训练
model = LogisticRegression(
    C=1.0,                # 正则化强度（1/α，越小越强）
    penalty='l2',         # 正则化类型
    max_iter=1000,        # 最大迭代次数
    multi_class='multinomial',  # 多分类用 softmax
    random_state=42
)
model.fit(X_train, y_train)

# 预测
y_pred = model.predict(X_test)          # 类别标签
y_prob = model.predict_proba(X_test)    # 概率值

# 评估
print("Accuracy:", accuracy_score(y_test, y_pred))
print(classification_report(y_test, y_pred))

# 关键属性
model.coef_         # 权重 (k, d)
model.intercept_    # 偏置 (k,)
```

## 11. PyTorch 实现

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 二分类
model = nn.Linear(d, 1)
criterion = nn.BCEWithLogitsLoss()   # 内部自带 sigmoid，不用自己加
optimizer = optim.SGD(model.parameters(), lr=0.01)

for epoch in range(1000):
    logits = model(X_train_t)                # z = X@w+b（不加sigmoid）
    loss = criterion(logits, y_train_t)      # 损失函数内部加 sigmoid

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# 预测
with torch.no_grad():
    probs = torch.sigmoid(model(X_test_t))   # 这里手动加 sigmoid 得到概率
    preds = (probs >= 0.5).float()            # 概率转标签

# 多分类
model = nn.Linear(d, k)                      # k 个类
criterion = nn.CrossEntropyLoss()             # 内部自带 softmax
```

**注意：**

```
BCEWithLogitsLoss = sigmoid + 交叉熵（二分类，不用自己加 sigmoid）
CrossEntropyLoss  = softmax + 交叉熵（多分类，不用自己加 softmax）

训练时不加 sigmoid/softmax（损失函数里已经包含了）
预测时要手动加 sigmoid/softmax 才能得到概率
```

## 12. 和 Linear Regression 对比速查

```
                   Linear Regression    Logistic Regression
任务               回归                 分类
输出               连续值               概率 (0~1)
激活函数           无                   sigmoid / softmax
损失函数           MSE                  Cross Entropy
解析解             有（正规方程）        没有
梯度形式           X.T@(ŷ-y)/n         X.T@(ŷ-y)/n（一样！）
评估指标           MSE, R²             Accuracy, F1, AUC
```

## 13. 面试高频问答

**Q1: 为什么叫 Regression 但是做分类？**
历史原因。它回归的是概率值（0~1），然后用阈值分类。

**Q2: Logistic Regression 能处理非线性问题吗？**
本身不能，决策边界是线性的。但可以手动加多项式特征或用核方法。

**Q3: sigmoid 和 softmax 的关系？**
二分类的 softmax 等价于 sigmoid。softmax 是 sigmoid 的多分类推广。

**Q4: 为什么用交叉熵不用 MSE？**
MSE + sigmoid 是非凸的，交叉熵 + sigmoid 是凸的，优化更可靠。

**Q5: 类别不平衡怎么办？**
调整 class_weight='balanced'，或调阈值，或用 F1/AUC 代替 accuracy 评估。
# SVM（支持向量机）复习笔记

---

## 1. 核心概念速记

| 项目 | 内容 |
|------|------|
| **类型** | 有监督分类 |
| **目标** | 找最大间隔的决策边界 |
| **模型** | w.T@x + b = 0（超平面） |
| **损失函数** | Hinge Loss + L2 正则化 |
| **关键特点** | 只有支持向量影响决策边界 |

## 2. 核心思想

```
找一条线把两类分开，让离线最近的点（支持向量）距离最大
间隔大 → 不确定的区域很宽 → 但是数据很少落进来 → 大部分预测都很自信
间隔小 → 不确定的区域很窄 → 但是边界贴着数据 → 新数据稍微偏一点就分错

分类规则：
  w.T@x + b > 0 → +1（正类）
  w.T@x + b < 0 → -1（负类）

间隔 = 2 / ‖w‖
最大化间隔 = 最小化 ‖w‖²/2
约束：yi × (w.T@xi + b) >= 1
```

## 3. Hinge Loss

```
L = max(0, 1 - yi × f(xi))

yi × f(xi) >= 1 → 分对且间隔够 → Loss = 0，不管
yi × f(xi) < 1  → 分错或间隔不够 → Loss > 0，惩罚

对比 BCE：所有点都有损失
Hinge：分好的点完全忽略，只关注边界附近
```

## 4. 软间隔（C 参数）

```
数据有噪声，不能完美分开 → 允许一些点犯错

Loss = ‖w‖²/2 + C × Σ max(0, 1 - yi(w·xi + b))

C 大 → 不容许犯错 → 间隔小 → 容易过拟合
C 小 → 容许犯错   → 间隔大 → 可能欠拟合
```

## 5. 核技巧（处理非线性）

```
线性不可分时，把数据映射到高维空间
 x和z相当于两个原始数据的数据点
常用核函数：
  线性核：  K(x,z) = x.T@z              直接用
  多项式核：K(x,z) = (x.T@z + 1)^d      多项式边界
  RBF 核： K(x,z) = exp(-γ‖x-z‖²)       最常用，任意形状

RBF 核的 γ：
  γ 大 → 只看附近的点 → 边界复杂 → 容易过拟合
  γ 小 → 看很远的点   → 边界平滑 → 可能欠拟合
```

## 6. 手搓线性 SVM（梯度下降）

```python
import numpy as np

def svm(X, y, lr=0.001, C=1.0, max_iters=1000):
    """y 必须是 +1 或 -1"""
    n, d = X.shape
    w = np.zeros(d)
    b = 0

    for _ in range(max_iters):
        for i in range(n):
            if y[i] * (X[i] @ w + b) >= 1:
                # 分对了，只更新正则项
                w = w - lr * w
            else:
                # 分错了，更新 w 和 b
                w = w - lr * (w - C * y[i] * X[i])
                b = b + lr * C * y[i]

    return w, b

# 预测：np.sign(X @ w + b) → +1 或 -1
```

## 7. sklearn 实现

```python
from sklearn.svm import SVC

# 线性 SVM
model = SVC(kernel='linear', C=1.0)

# RBF 核 SVM（最常用）
model = SVC(kernel='rbf', C=1.0, gamma='scale')

model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# 关键属性
model.support_vectors_     # 支持向量坐标
model.n_support_           # 每类支持向量数
```

## 8. 和 Logistic Regression 对比

```
                  Logistic Regression      SVM
关注哪些点        所有点                    只看支持向量
输出              概率                      标签（原生无概率）
损失函数          BCE                       Hinge Loss
处理非线性        手动加特征                 核技巧
数据量大          快                        慢
```

## 9. 面试高频问答

**Q1: 什么是支持向量？**
离决策边界最近的点，去掉其他点不影响边界，去掉支持向量边界就变了。

**Q2: C 和 γ 怎么调？**
用网格搜索 + 交叉验证。C 控制容错，γ 控制 RBF 核的影响范围。

**Q3: SVM 能做回归吗？**
能，叫 SVR（Support Vector Regression），sklearn 里是 `from sklearn.svm import SVR`。

**Q4: SVM 为什么不适合大数据？**
训练复杂度 O(n²~n³)，数据量大时非常慢。大数据用 Logistic Regression 或深度学习。

**Q5: 核技巧的本质？**
不需要真的把数据变到高维，只算高维空间里的内积，省计算量。

# 拉格朗日乘子法 & SVM 推导笔记

---

## 1. 核心思想

**把"有约束的优化"变成"无约束的优化"。**

```
有约束：最小化 f(w)，约束 g(w) = 0
  ↓ 引入乘子 λ
无约束：最小化 L(w, λ) = f(w) + λ × g(w)
  ↓ 对所有变量求导 = 0
解出 w 和 λ
```

## 2. 等式约束的例子

```
最小化：x² + y²           （离原点最近）
约束：  x + y - 4 = 0     （必须在这条线上）

L = x² + y² + λ(x + y - 4)

∂L/∂x = 2x + λ = 0       → x = -λ/2
∂L/∂y = 2y + λ = 0       → y = -λ/2
∂L/∂λ = x + y - 4 = 0    → 约束自动满足

解：x = 2, y = 2, λ = -4
```

## 3. 不等式约束（SVM 用的）

等式约束用 λ，不等式约束用 α >= 0，多一个 KKT 条件：

```
约束：g(w) >= 0

KKT 条件：
  α >= 0
  α × g(w) = 0    ← 互补松弛条件

意思：
  约束没取等（g(w) > 0）→ α 必须 = 0（不起作用）
  α > 0 → 约束必须取等（g(w) = 0，在边界上）
```

## 4. SVM 的拉格朗日推导

### 原始问题

```
最小化：‖w‖²/2
约束：  yi(w.T@xi + b) >= 1    对所有 i
```

### 构造拉格朗日函数

```
L(w, b, α) = ‖w‖²/2 - Σ αi[yi(w.T@xi + b) - 1]

αi >= 0
```

### 对 w 求导 = 0

```
∂L/∂w = w - Σ αi×yi×xi = 0

→  w = Σ αi×yi×xi    ← 关键结论！
```

### 对 b 求导 = 0

```
∂L/∂b = -Σ αi×yi = 0

→  Σ αi×yi = 0
```

### KKT 互补松弛条件

```
αi × [yi(w.T@xi + b) - 1] = 0

两种情况：
  αi = 0 → 这个点不影响 w（不是支持向量）
  αi > 0 → yi(w.T@xi + b) = 1（在边界上，是支持向量）
```

## 5. 从 w 到核技巧

### w 代入决策函数

```
f(x) = w.T@x + b
     = [Σ αi×yi×xi].T @ x + b
     = Σ αi×yi×(xi.T@x) + b
                ─────────
                内积
```

### 把内积换成核函数

```
线性：f(x) = Σ αi×yi×(xi.T@x) + b
核：  f(x) = Σ αi×yi×K(xi, x) + b

K 可以是：
  线性核：  xi.T@x
  多项式核：(xi.T@x + 1)^d
  RBF 核：  exp(-γ‖xi-x‖²)
```

### 核函数的本质

```
K(xi, x) = φ(xi).T @ φ(x)

不用真的算 φ（可能是无限维）
直接用 K 公式得到高维空间的内积结果
```

## 6. 完整推导链条

```
有约束优化
  ↓ 拉格朗日乘子法
L(w, b, α) = ‖w‖²/2 - Σαi[yi(w.T@xi+b) - 1]
  ↓ 对 w 求导 = 0
w = Σ αi×yi×xi
  ↓ 代入决策函数
f(x) = Σ αi×yi×(xi.T@x) + b
  ↓ KKT 条件
大部分 αi = 0，只有支持向量的 αi > 0
  ↓ 核技巧
把 xi.T@x 换成 K(xi, x)，处理非线性
```

## 7. 记忆要点

```
1. 拉格朗日 = 约束塞进目标函数
2. 求导 = 0 得到 w = Σαi×yi×xi
3. KKT 条件 → 只有支持向量的 α > 0
4. w 代入决策函数 → 出现内积 xi.T@x
5. 内积换成核函数 → 高维分类
```
# 决策树 & Random Forest & XGBoost 复习笔记

---

## 1. 决策树（Decision Tree）

### 核心概念

| 项目 | 内容 |
|------|------|
| **类型** | 有监督（分类 & 回归） |
| **核心思想** | 不断问问题，把数据分成越来越纯的子集 |
| **分类树** | 叶子节点投票（多数类） |
| **回归树** | 叶子节点取均值 |

### 怎么选分裂特征？

**信息增益（Entropy，ID3/C4.5 用）：**

```
Entropy = -Σ pi × log2(pi)

全是一类：Entropy = 0（最纯）
一半一半：Entropy = 1（最不纯）

信息增益 = 分裂前的 Entropy - 分裂后的加权 Entropy
选信息增益最大的特征
```

**基尼系数（Gini，CART / sklearn 默认）：**

```
Gini = 1 - Σ pi²

全是一类：Gini = 0（最纯）
一半一半：Gini = 0.5（最不纯）

选分裂后 Gini 最小的特征
```

**回归树用 MSE：**

```
选分裂后 MSE 下降最多的特征和阈值
```

### 过拟合控制

```
max_depth：      限制树的深度
min_samples_split：节点最少要多少样本才能继续分
min_samples_leaf： 叶子最少要多少样本
max_features：    每次分裂只看部分特征
剪枝（pruning）：  先长完再砍掉不好的分支
```

### 优缺点

```
优点：可解释性强、不用标准化、能处理非线性
缺点：容易过拟合、不稳定（数据变一点树就变很多）
```

### sklearn 实现

```python
from sklearn.tree import DecisionTreeClassifier, DecisionTreeRegressor

# 分类
clf = DecisionTreeClassifier(
    criterion='gini',       # 'gini' 或 'entropy'
    max_depth=5,
    min_samples_split=10,
    random_state=42
)
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)

# 回归
reg = DecisionTreeRegressor(max_depth=5)
reg.fit(X_train, y_train)
```

---

## 2. Bagging vs Boosting

两种集成思路，区别很重要：

```
Bagging（并行）：
  多棵树独立训练，最后投票/取均值
  每棵树看到的数据和特征都不同
  目的：减少方差（过拟合）
  代表：Random Forest

Boosting（串行）：
  一棵接一棵训练，后一棵修正前一棵的错误
  每棵树关注之前分错的样本
  目的：减少偏差（欠拟合）
  代表：AdaBoost → GBDT → XGBoost
```

```
          Bagging              Boosting
训练方式   并行，独立            串行，依赖前一棵
关注点     减少方差              减少偏差
过拟合风险  低                   较高（需要控制）
代表       Random Forest        XGBoost, LightGBM
```

---

## 3. Random Forest

### 核心思想

```
训练很多棵决策树，每棵树用不同的数据和特征
最后投票（分类）或取均值（回归）

两层随机性：
  1. 数据随机：每棵树用 bootstrap 采样（有放回抽样）
  2. 特征随机：每次分裂只看 sqrt(d) 个随机特征
```

### 为什么有效？

```
单棵树：容易过拟合，不稳定
多棵不同的树投票：个别树的错误被其他树纠正

就像：
  一个人判断 → 容易出错
  100 个人投票 → 集体智慧，更靠谱
```

### sklearn 实现

```python
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor

rf = RandomForestClassifier(
    n_estimators=100,       # 树的数量
    max_depth=10,           # 每棵树的最大深度
    max_features='sqrt',    # 每次分裂看 sqrt(d) 个特征
    min_samples_split=5,
    n_jobs=-1,              # 并行训练（用所有 CPU）
    random_state=42
)
rf.fit(X_train, y_train)
y_pred = rf.predict(X_test)

# 特征重要性
importances = rf.feature_importances_    # 每个特征的重要程度
```

### 关键参数

```
n_estimators：树的数量，越多越好（但有上限，收益递减）
max_depth：单棵树深度，太深过拟合
max_features：每次分裂看多少特征
  分类默认 sqrt(d)
  回归默认 d/3
```

---

## 4. XGBoost

### 核心思想

```
Boosting 的思路：每棵新树学习前面所有树的残差（错误）

第1棵树：预测 → 算残差
第2棵树：拟合残差 → 新残差更小了
第3棵树：拟合新残差 → 残差更更小了
...
最终预测 = 所有树的预测之和
```

### 用数字理解

```
真实值 y = 100

第1棵树预测：80      → 残差 = 100-80 = 20
第2棵树拟合残差：15  → 新残差 = 20-15 = 5
第3棵树拟合残差：3   → 新残差 = 5-3 = 2

最终预测 = 80 + 15 + 3 = 98（很接近100了）
```

### XGBoost 比 GBDT 强在哪？

```
1. 正则化：损失函数自带 L1/L2，防过拟合
2. 二阶导数：用泰勒展开到二阶，优化更快更准
3. 列采样：像 Random Forest 一样随机选特征
4. 缺失值处理：自动学习缺失值该往左还是往右
5. 并行化：虽然树是串行的，但每棵树内部的分裂是并行的
6. 剪枝：先长完再剪，比 GBDT 的提前停止更好
```

### XGBoost 目标函数

```
Obj = Σ L(yi, ŷi) + Σ Ω(fk)
      ──────────     ────────
      损失函数        正则项（控制树的复杂度）

Ω(f) = γT + (1/2)λΣwj²
       ↑        ↑
     叶子数量   叶子权重的L2

γ 大 → 倾向于少的叶子 → 简单的树
λ 大 → 叶子权重小 → 预测更保守

XGBoost 的 Loss Function 不是固定的，根据任务选：
回归：   MSE        → objective='reg:squarederror'
二分类：  BCE        → objective='binary:logistic'
多分类：  CE         → objective='multi:softmax'
排序：   Pairwise   → objective='rank:pairwise'
```

### Python 实现

```python
import xgboost as xgb
from sklearn.model_selection import train_test_split

# 方式 1：sklearn 接口（最简单）
model = xgb.XGBClassifier(
    n_estimators=100,       # 树的数量
    max_depth=6,            # 每棵树深度
    learning_rate=0.1,      # 学习率（缩小每棵树的贡献）
    subsample=0.8,          # 每棵树用 80% 数据
    colsample_bytree=0.8,   # 每棵树用 80% 特征
    reg_alpha=0,            # L1 正则
    reg_lambda=1,           # L2 正则
    random_state=42
)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# 回归
reg = xgb.XGBRegressor(n_estimators=100, max_depth=6, learning_rate=0.1)

# 特征重要性
importances = model.feature_importances_

# 方式 2：原生接口（更多功能）
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

params = {
    'max_depth': 6,
    'learning_rate': 0.1,
    'objective': 'binary:logistic',    # 二分类
    'eval_metric': 'logloss',
    'subsample': 0.8,
    'colsample_bytree': 0.8,
}

model = xgb.train(
    params, dtrain,
    num_boost_round=100,
    evals=[(dtest, 'test')],
    early_stopping_rounds=10,          # 10轮不改善就停
    verbose_eval=20
)
y_pred = model.predict(dtest)
```

### 关键参数速记

```
n_estimators / num_boost_round：树的数量
max_depth：每棵树深度（默认6，比RF浅）
learning_rate / eta：学习率，缩小每棵树的贡献
  小 → 需要更多树，但更稳定
  大 → 需要更少树，但容易过拟合
subsample：行采样比例
colsample_bytree：列采样比例
reg_alpha：L1 正则
reg_lambda：L2 正则
early_stopping_rounds：提前停止
```

### 调参顺序

```
1. 先固定 learning_rate=0.1
2. 调 n_estimators（用 early_stopping 自动选）
3. 调 max_depth（3~10）
4. 调 subsample 和 colsample_bytree（0.6~1.0）
5. 调 reg_alpha 和 reg_lambda
6. 最后降低 learning_rate，增加 n_estimators
```

---

## 5. 三者对比

```
              决策树         Random Forest      XGBoost
模型          1棵树           多棵树并行          多棵树串行
思路          单独决策         投票                逐步纠错
过拟合        严重            不严重              需要控制
速度          快              中等（可并行）       慢（但有优化）
可解释性      高              中                  低
调参难度      简单            简单                复杂
通常效果      一般            好                  最好
```

---

## 6. LightGBM 和 CatBoost（补充）

```
XGBoost 之后的改进：

LightGBM（微软）：
  按叶子生长而不是按层 → 更快
  直方图优化 → 大数据更高效
  适合：数据量大、特征多

CatBoost（Yandex）：
  原生支持类别特征（不用手动 One-Hot）
  有序 Boosting → 减少过拟合
  适合：类别特征多的数据

实际选择：
  数据小 → XGBoost
  数据大 → LightGBM
  类别特征多 → CatBoost
  竞赛中三个都试 → 选最好的
```

---

## 7. 面试高频问答

**Q1: Random Forest 和 XGBoost 怎么选？**
RF 不容易过拟合，调参简单，适合快速基线。XGBoost 效果通常更好，但需要仔细调参。

**Q2: XGBoost 的 learning_rate 是什么？**
每棵新树的贡献乘以 learning_rate 缩小。0.1 意味着每棵树只贡献 10%，需要更多树但更稳定。

**Q3: 为什么 XGBoost 在 Kaggle 上这么流行？**
结构化数据上效果最好，内置正则化防过拟合，能处理缺失值，有 early_stopping。

**Q4: Bagging 减少方差，Boosting 减少偏差，什么意思？**
方差大 = 换一批数据模型变化大（过拟合），Bagging 通过平均多个模型降低这种不稳定。偏差大 = 模型太简单抓不住规律（欠拟合），Boosting 逐步增加复杂度来改善。

**Q5: 决策树需要标准化吗？**
不需要。决策树只看特征的排序，不看绝对值大小，所以量纲不影响。这也是树模型的一大优势。