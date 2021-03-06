## K Nearest Neighbor

K 临近算法，分类学习中的经典算法，属于监督学习。注意其和 K 均值算法（K-Means）不同，K-Means 用于解决聚类问题，为无监督学习。

- KNN：监督学习。已知数据集中各元素分类，现给出一个新的元素，为它进行分类
- K-Means：无监督学习。给出一个数据集，对数据集中的各元素进行聚合，生成 K 个种类

### 分类算法中的距离问题

对于分类、聚类问题，很常见的做法是使用点点直接的距离（闵可夫斯基距离）来表示元素间的相似度。但这种算法有一个普遍的问题：当不同特征的取值范围有很大差别的时候，较大的那个数据造成对算法造成的影响远远大于较小的那个数据。

举个栗子：关于房价预测的算法（入门机器学习时很常见的案例了吧），有 “房间数” 和 “房屋面积” 两个特征。房间数的取值区间为 `[1, 5]`，而房屋面积的区间则为 `[100, 1000]`，此时，如果照以前那样代入计算，则 “房间数” 这个特征对你算法的影响几乎可以不算。虽然这不是一个分类算法，但对于分类算法而言，这种问题也是存在的。

这种时候，可以采取[特征缩放](https://zh.wikipedia.org/zh/特征缩放)的手段来预处理数据集；或者针对分类/聚类算法，使用向量余弦法。

#### 向量余弦法

对于下面两个数据，它们都包含了一些特征，并由此进行分类（是什么类型的房屋）比如：

```javascript
// [房间数, 小区人口数, 房屋面积, 有独卫] ==> 民宅
const a = [3, 3000, 100, 1];
// [房间数, 小区人口数, 房屋面积, 没有独卫] ==> 青年公寓
const b = [1, 2600, 20, 0];
```

将它们放在空间坐标系后，成为两个向量。而这两个向量之间的夹角则可以很好的体验出两者之间的相似度。这样的方法就避免了较大数据造成的影响。

#### 特征缩放

还是这样的数据

```javascript
// [房间数, 小区人口数, 房屋面积, 有独卫] ==> 民宅
const a = [3, 3000, 100, 1];
// [房间数, 小区人口数, 房屋面积, 没有独卫] ==> 青年公寓
const b = [1, 2600, 20, 0];
```

在计算两者之间的距离时，为了能够消除 “小区人口” 这样大数据的影响，可以在求差之后除以 `maxRange`，即该样本数据的 `最大值 - 最小值`：

```javascript
// a - b
[2, 400, 80, 1]
// 各类数据的 range 假设如下
[3, 500, 100, 1]
// 消除大数影响
[0.67, 0.8, 0.8, 1]
```

### 数据偏差

KNN 本质上是多数表决的算法 - 计算给定数据到样本集各个数据之间的距离，然后选择距离最近的 K 个样本，在这 K 个样本中，哪种类型的数据最多，则把 input 数据划分到该类下。

这样的算法会收到样本数据集内各种数据分布的影响：当数据集中某一类的数据（距离为 a）远大于其他类型数据时，则该类数据的对算法的影响大于其他数据 - 在 K 个数据中，a 出现的概率更大一些。因此，需要对数据的权重进行处理，距离越远则权重越小。举个例子：

在某数据集中分布着 `a`, `b` 两种数据，现给出新数据 `M`，且 `K = 5`，计算得 M 附近最近的 5 个点为 `[a, b, a, a, b]`，距离分别是 `[4, 1, 3, 3, 2]`。此时应考虑权重：

```javascript
const resultGroup = '';
// K 近邻的各数据
['a', 'b', 'a', 'a', 'b']
// 数据的距离
[4, 1, 3, 3, 2]

// 在不考虑权重下
resultGroup = 'a' // a 出现了 3 次

// 考虑权重时
const disWithWeightA = 1 / 4 + 2 / 3 = 0.916; // 距离为 4 的数据出现了 1 次，距离为 3 的数据出现了 2 次
const disWithWeightB = 1 / 1 + 1 / 2 = 1.5; // 距离为 1 的数据出现了 1 次，距离为 2 的数据出现了 1 次
// ===>
resultGroup = 'b';
```

### 算法效率

在 knn 算法中，我们需要从数据集中取出距离目标点较近的前 K 个元素。普通的方法是计算每一个点到目标点的距离，然后排序之后取前 K 个。这样的方法除了慢就没啥毛病，但是当数据量很大的时候是可以慢死人的。因此必须对其进行优化。

#### 最大堆

如果我们还是需要遍历每一个点并计算距离，那么至少可以使用最大堆来优化其排序，保证堆根节点是有最大距离的元素。在遍历完成之后逐步取出根节点的元素（每次取完以后要保证最大堆的平衡）

#### k-d 树

最大堆可以提升一些效率，但无法从根本上解决问题 - 我们还是需要遍历全部数据以便计算各个点和目标点间的距离。而通过 k-d 树的方式，可以不用遍历所有点，通过对数据集的对半划分，可以极大的提升算法效率。

### Articles

- [K NEAREST NEIGHBOR 算法](http://coolshell.cn/articles/8052.html)

### Code

```bash
# run in local
$ git clone git@github.com:ecmadao/ml-in-action.git
$ cd ml-in-action
$ npm i

# normal loop
$ node knn/code/index.js

# use k-d tree
$ node knn/code/knn-kdtree.js
```
