这不是一个非常精确的匹配算法，只能筛选出大致正确的匹配特征点。如果以后有时间，我会继续介绍进一步的精确匹配算法（Outer Lier Rejection和M-estimator Sample Consensus(MSAC)的组合应用）。听起来好像又给自己挖了个坑。。。。。

好了，扯远了，下面开始介绍这个粗糙的匹配算法吧。

# 一对一特征点匹配算法
我们已经检测出了图像1（$I_1$）和图像2（$I_2$）的特征点（corner）。

![corner](https://raw.githubusercontent.com/shl666/UCSD_CV_Intro/master/chapter_2/images/corner_detection.png)

接下来要对这些特征点进行匹配。

这个算法中有几个重要的参数（最后的字母为在公式中的变量名）：
1. 匹配窗口大小（matching window size）- w
1. 相关性系数阈值（correlation coefficient threshold）- t
1. 距离比率阈值（distance ratio threshold）- d
1. 特征点距离阈值 - p

中间涉及到的一些定义会在用到的时候提及。

在最开始，我们需要计算一个**相关性系数矩阵**。

**相关性系数矩阵（Correlation Matrix）**：用来描述两幅图中所有特征点相关性的矩阵（仅在不同图之间比较，即同一幅图中的特征点相关性不需计算）。假设第一幅图中有m个特征点，第二幅图中有n个特征点，那么这个相关性系数矩阵的大小就是$m \times n$.

**相关系数（Correlation）**：用来描述两个特征点之间的相关性，区间范围在[-1,1]；数值越大，相关性越好。（具体计算方式会在后面提到）

算法主体：

1. 寻找到**相关性系数矩阵**中最大值的位置$（i，j）$（第i行，第j列）。
2. 存下这个最大值（$best$）。
3. 将**相关性系数矩阵**中的$（i，j）$值设为-1。
4. 寻找到当前**相关性系数矩阵**第i行中的最大值（$max_i$）和第j列的最大值($max_j$)。
5. 如果满足$1-best<(1-max_i)\times d$ ， $1-best<(1-max_j)\times d$ 和 这两个特征点之间的距离小于p这三个条件， 那么这个特征点组合将被记录下来，并将**相关性系数矩阵**中第i行与第j列的所有元素都设为-1；反之，这个特征点组合不符合条件（不够独特）。值得注意的是，在特征点组合不符合条件的情况下，如果违背的是条件1，那么第i行的所有元素都设为-1；如果违背的是条件2，那么第j列的所有元素都设为-1。
6. 重复整个过程，直到**相关性系数矩阵**中的最大值都不超过**相关性系数阈值（t）**。

整个算法描述完了，我们好像遗漏了一个很关键的步骤——如何计算**相关性系数**？

## 相关性系数
相关性系数的计算方式有很多种，下面这张表展示了其中的一部分。这个相关性系数算法和之前的特征点检测算法一样，都需要一个窗口大小（window size），毕竟匹配的不可能是一个像素点，而是一小块区域，具体参数大小可以根据实际情况来调整。
![match matrix](https://raw.githubusercontent.com/shl666/UCSD_CV_Intro/master/chapter_2/images/matchmatrix.png)

我用的是NCC算法。最后的结果长这样：
![corner matching](https://raw.githubusercontent.com/shl666/UCSD_CV_Intro/master/chapter_2/images/corner_matching.png)
emmmm结果依旧不是特别好，但是勉强可以用。我先把Outer Lier Rejection和M-estimator Sample Consensus(MSAC)的算法结果po上来吧。那个就看起来很舒服，有空再来填坑。![corner matching](https://raw.githubusercontent.com/shl666/UCSD_CV_Intro/master/chapter_2/images/outerlierrejection.png)