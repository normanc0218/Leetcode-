# UofT MEng Notes

Personal ML knowledge base.

## 📂 Categories

- [K-means](#01-K-Means-算法复习笔记-&-代码练习)
- [PCA](#02-pca主成分分析复习笔记--代码练习)
- [Linear Regression](#linear-regression-复习笔记)














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

# 01 K-Means 算法复习笔记 & 代码练习

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
# 02 PCA（主成分分析）复习笔记 & 代码练习



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