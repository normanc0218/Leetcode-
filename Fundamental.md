# UofT MEng Notes

Personal ML knowledge base.
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

## 📂 Categories

- [K-means](#01-K-Means-算法复习笔记-&-代码练习)
- [PCA](#02-pca主成分分析复习笔记--代码练习)















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

    # 5. 取前 k 个
    components = eigenvectors[:, :k].T         # (k, d)
    explained_ratio = eigenvalues[:k] / eigenvalues.sum()

    # 6. 投影
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

---

## 7. 伪代码练习

```python
import numpy as np

def pca(X, k):
    n, d = X.shape

    # Step 1: 中心化
    # 提示：每个特征减去该特征的均值
    mean = ______
    X_centered = ______

    # Step 2: 协方差矩阵
    # 提示：X_centered 转置乘自己，除以 (n-1)
    cov = ______

    # Step 3: 特征值分解
    # 提示：用 np.linalg.eigh 处理对称矩阵
    eigenvalues, eigenvectors = ______

    # Step 4: 按特征值从大到小排序
    # 提示：argsort 返回升序索引，[::-1] 反转成降序
    idx = ______
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]

    # Step 5: 投影到前 k 个主成分
    # 提示：X_centered 乘以前 k 个特征向量
    X_pca = ______

    return X_pca
```

**参考答案 ↓**

```python
def pca(X, k):
    n, d = X.shape

    mean = X.mean(axis=0)
    X_centered = X - mean

    cov = (X_centered.T @ X_centered) / (n - 1)

    eigenvalues, eigenvectors = np.linalg.eigh(cov)

    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]

    X_pca = X_centered @ eigenvectors[:, :k]

    return X_pca
```

---

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