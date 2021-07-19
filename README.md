今天我们来学习两种构造支序分类树（phylogenetic tree）的算法。

第一种叫做非加权组平均法（unweighted pair group method with arithmetic mean, UPGMA)。为了使用这种方法，我们必须确定数据是超度量（ultrametric）的。在实际运用中，这意味着我们观察到每一个分支的演化速率都等同（每一个分叉都是一个等腰三角形）。显然，核酸/蛋白质序列分子钟式的中性演化，最适合用这种算法分析。同理，如果选用这种方法，必须选择合适的外群（outgroup），使其与每一个内群样本之间的距离都近似。

如果要给 `N` 个样本做聚类，那么必先准备半正定、对称的度量函数 `D`，表示样本的两两距离。另外，我们另准备一个长为 `N`，每个位置初始化为 1 的数组 `L`.

```
UPGMA(D, L)：	
   
   若 D 大小为 1：
      停止运行
   
   记 D 值域中最小的非零元素为 D (x, y)
   
   定义新距离函数 D':  对 D 定义域中的任意两个样本 a, b:
      若 a, b 不是 x 或 y: D'(a,b) = D(a,b)；D'
      若 a 是 x 或 y: D'({x,y}, d) = (L(x)D(x,b) + L(y)D(y,b)) / (L(x) + L(y))

   定义新数组 L'，令 L'({x,y}) = L(x) + L(y)

   对 D 中的任意非 x 或 y 的样本 a:  L'(a) = L(a)

   报告：“结合节点 x, y 为 {x,y}：分岔高度为 D(x, y)/2”

   运行 UPGMA(D', L')
```

用人话解释，UPGMA 是一种从下往上的聚类方法：给定 `N` 个样本，找到两个距离最近的样本 `x, y`，将它们结合为支序树中的新节点 `{x,y}`，其分叉高度为 `D(x, y)/2`。接着，它计算节点 `{x,y}` 与其他样本/节点之间的距离，按 `x`, `y` 原来的大小进行加权（所有的样本的大小皆为 1，形如 `{x,y}` 的节点的大小是 `x` 大小与 `y` 大小之和），并计算所有节点的大小。这个过程持续下去，直到所有的节点都被结合、支序树完成为止。

这是一种贪心算法。如果不做任何优化、用矩阵来表示 `D`，显然复杂度是 `O(N^3)` 。如果用最小堆（min-heap）实现 `D`，可以提高为 `O(N^2 lg(N))`：`D'` 可以通过在 `D` 原位增减键值、删除项目形成，每周期的时间复杂度最多 `O(N lg(N))`，而这个算法最多 `N-1` 周期就可以结束。

用现代的计算机程序语言写这个算法显然是非常简单的，但我们有办法在蹩脚的 Ti-83 Plus 手持绘图计算器上实现这个算法吗？答案是肯定的。

=== 以下内容纯属恶搞，切勿在生产环境中使用 Ti-83 Plus ===

我们需要利用 Ti-BASIC 语言中对矩阵和列表两种数据类型的支持，实现半自动的 `O(N^3)` 算法。实现有两个成分：

* 辅助函数 `MININD`，寻找矩阵中最小的非零元素，并将它存入变量
* 真正的 `UPGMA` 算法

在运行这个算法前，需要执行以下两个步骤

1. 将矩阵 `[A]` 初始化为表示样本距离的 `N x N` 半正定对称矩阵；Ti-BASIC 支持 `1 <= N <= 99`
2. 确保变量 `X`, `Y`, `A`，以及列表 `L2` 和 `L4` 中没有需要的数据
3. 将 `L4` 初始化为长为 `N`, 值全部为 1 的列表

我们来做一个例题。假设我们想要给以下的数据聚类：

|      | Q    | Z    | W    | Y    | Out  |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Q    | 18   | 15   | 18   | 16   | 44.9 |
| Z    | 15   | 11   | 14   | 12   | 43.1 |
| W    | 18   | 14   | 17   | 15   | 42.5 |
| Y    | 16   | 12   | 15   | 14   | 43.5 |
| Out  | 44.9 | 43.1 | 42.5 | 43.5 | 0    |

（这是来自[Kim et al. (2010)](https://www.tandfonline.com/doi/full/10.1080/19768351003764973) 的真实资料。这里的四种物种实际上是采集于东海、黄海地区的小黄鱼 *Larimichthys polyactis* 以及作为外群的大黄鱼 *Larimichthys crocea*，数值是 Tamura–Nei 遗传距离乘100。小黄鱼能有什么坏心眼呢？）

注意，如果是数学上的度量，我们应当期望以上矩阵的对角元素为0，但是这是生物课不是数学课——同物种的遗传距离可以看作是遗传方差。在 UPGMA 的运行中，我们不会用对角元素的值，所以全设为 0 也是可以的。

我们首先把矩阵录入计算器

<img src="https://github.com/LykosEremos/phylo_with_ti83/blob/main/1.jpg" width="250" />

然后把 `L4` 初始化为五个 1

<img src="https://github.com/LykosEremos/phylo_with_ti83/blob/main/2.jpg" width="250" />

运行我们的程序

<img src="https://github.com/LykosEremos/phylo_with_ti83/blob/main/3.jpg" width="250" />

<img src="https://github.com/LykosEremos/phylo_with_ti83/blob/main/4.jpg" width="250" />

这里输出的意思是：

1. `ND SIZE IN L4`：提醒你要初始化 L4 为节点大小；这个每次运行 `UPGMA` 都会显示，如果你有去做就不用理会。
2. `F2M`，`G2M`：被结合的两个样本在矩阵中的索引，这里分别是 2 和 4。Ti-BASIC 是从 1 开始索引的，所以这意味着这里结合的是样本 `Z` 和 `Y`
3. `MH`：节点 `{Z,Y}` 的高度，计算方式是 `D(Z,Y)/2=6`
4. `D`：节点 `{Z,Y}` 与其他各样本之间的距离，根据这里的数据，我们可以构建下一个周期所要用的距离矩阵

|       | Q    | W    | {Y,Z} | Out  |
| ----- | ---- | ---- | ----- | ---- |
| Q     | 18   | 18   | 15.5  | 44.9 |
| W     | 18   | 17   | 14.5  | 42.5 |
| {Y,Z} | 15.5 | 14.5 | 0     | 43.3 |
| Out   | 44.9 | 43.1 | 43.3  | 0    |

我们的程序是半自动的，也就是说每一周期都必须手动运行（再次提醒大家，这是仅供娱乐的程序）。运行下一周期时，把矩阵 `[A]` 改成这个，然后 `L4` 初始化为 `{1,1,2,1}` 反应节点 `{Z,Y}` 的大小变化，就可以运行下一周期的算法了。如果你有闲心，可以把试着把整个树做出了，和文献里的 Fig 3B 进行比对。

哦对了，你要是没有耐性，也可以用 R 来构建系统树，最简便的就是`phangorn`包中的相关函数：

```R
library(phangorn)

D <- matrix(c(18,   15,   18,   16,   44.9, 
              15,   11,   14,   12,   43.1,
              18,   14,   17,   15,   42.5,
              16,   12,   15,   14,   43.5,
              44.9, 43.1, 42.5, 43.5, 0    ), byrow=TRUE, nrow = 5)
colnames(D) <- c('Q','Z','W','Y','Out')
rownames(D) <- c('Q','Z','W','Y','Out')

tree <- upgma(D)
plot(tree)
edgelabels(text= round(tree$edge.length,4))
```

做出来结果应该是这样的

<img src="https://github.com/LykosEremos/phylo_with_ti83/blob/main/5.png" width="500" />

