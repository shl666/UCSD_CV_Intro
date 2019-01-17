# 什么是特征点检测（Corner Detector）?
以下定义来自[维基百科](https://zh.wikipedia.org/wiki/%E8%A7%92%E6%A3%80%E6%B5%8B)：

特征点检测，是计算机视觉系统中用来提取特征以及推测图像内容的一种方法.角检测的应用很广，经常用在运动检测，跟踪，图像镶嵌（image mosaicing），全景图缝合（panorama stiching），三维建模以及物体识别中.

# 什么是特征点（Corner）？
以下定义来自[百度百科](https://baike.baidu.com/item/%E7%89%B9%E5%BE%81%E7%82%B9)：

图像处理中，特征点指的是图像灰度值发生剧烈变化的点或者在图像边缘上曲率较大的点(即两个边缘的交点)。图像特征点在基于特征点的图像匹配算法中有着十分重要的作用。图像特征点能够反映图像本质特征，能够标识图像中目标物体。通过特征点的匹配能够完成图像的匹配。

简单来说，特征点就是图像中具有代表性和鲁棒性的点，可以是角，也可以是局部亮点，抑或是线段终点。根据检测算法的不同，我们得到的特征点也有着区别。

# 示例
先看一下下面这两张图片（UCSD的PC）：
![original](images/original.png)

两张图片区别不大，（几乎）同一时间使用同一相机在不同角度拍下。 我们再来看看他们使用Forstner corner detection算法获得的特征点：

![corner](images/corner_detection.png)

是不是几乎都是一致的？除了个别被覆盖的部分。接下来就是对这些特征点的匹配（matching）操作。

![matching](images/corner_matching.png)

这样的matching结果，我们基本上可以看出两张图之间的变换关系了。之后利用这些关系，我们可以进行更多的应用操作。比如之前定义中提到的全景图缝合等。

# 算法流程

好了，我们开始理一理整个算法是怎么样一个东西。

## 特征检测（corner detection）
1. 我们会有一张图片$I$，首先我们要对图片进行黑白处理。一般来说图片根据不同的存储格式会有3到4个图层，我们这一步是将他们变成一个（不变也没关系，就是计算量翻倍）。

1. 然后我们需要对图片进行梯度计算，分别是横向(x轴)与纵向(y轴)。这里需要引入Five Point Central Opertor（没找到中文翻译...）。其实说到底就是一个卷积核，是根据某些数学运算推导出来的。有兴趣看深入了解的同学请点[这里](https://en.wikipedia.org/wiki/Five-point_stencil)。
$$
  k_x=1/12
  \begin{matrix}
   -1 & 8 & 0 & -8 & 1
  \end{matrix}
$$
$$
  k_y=1/12
  \begin{matrix}
   -1 & 8 & 0 & -8 & 1
  \end{matrix}^{T}
$$
我们可以得到两幅图像$I_x$和$I_y$。利用简单的矩阵运算，我们在这里最好算出$I_xx$和$I_yy$，他们分别是$I_x$和$I_y$的平方值（$I_xx(i,j) = I_{x}^{2}(i,j)$），下一步会有用。

1. 这里我们要引入下一个参数$w$，窗口（window）大小。因为我们不能从一个点的数值来判断他是不是corner，而是需要根据他和他附近区域（local area）的关系来判断，也就是这里的窗口内部区域。根据第二步的结果和窗口大小，我们就可以去计算每个点的梯度矩阵
$$
N(i,j) = 
\left[
 \begin{matrix}
   \sum_{w}{I_x^2(i,j)} & \sum_{w}{I_x(i,j)I_y(i,j)} \\
   \sum_{w}{I_x(i,j)I_y(i,j)}& \sum_{w}{I_y^2(i,j)}
  \end{matrix}
 \right] 
$$
再计算每个梯度矩阵的最小特征值(min eigenvalue)
$$ \lambda_{min}(i,j) = \frac{Tr(N(i,j)) - \sqrt{Tr^2(N(i,j) - det(N(i,j))}}{2}$$
这是一个小型矩阵的最小特征值的近似简化算法，$Tr$代表矩阵的[迹](https://baike.baidu.com/item/%E7%9F%A9%E9%98%B5%E7%9A%84%E8%BF%B9)，$det$代表矩阵的[行列式](https://baike.baidu.com/item/%E8%A1%8C%E5%88%97%E5%BC%8F)。
这里对于最小特征值的理解，我个人看来，可以简单地理解为一个分数（score），分数越高，它越可能是corner；分数越低，越不可能。

1. 再者，我们需要将图片过一个[非极大抑制（NMS）](https://baike.baidu.com/item/%E9%9D%9E%E6%9E%81%E5%A4%A7%E5%80%BC%E6%8A%91%E5%88%B6/22768283)滤波器。所谓非极大抑制呢，就是在给定区域（$w_{nms}$大小的窗口，这个窗口和之前提到的并不是同一个）内，如果中心点是最大值，那么中心点的数值保留；如果不是，则置为0。函数表达为：
$$
 J(i,j)=
 \begin{cases} 
 0 & \text{if } I(i,j)<I_{max}(i',j')\\
 I(i,j) & \text{if } I(i,j)=I_{max}(i',j')\\
 \end{cases}
$$

1. 最后一步很关键，需要引入[Forstner Corner Detection算法](https://en.wikipedia.org/wiki/Corner_detection#The_F%C3%B6rstner_corner_detector)计算实际corner与我们筛选出来的理论corner的相对位置关系。看起来很复杂，其实只需要解一个矩阵方程即可
$$
\left[
 \begin{matrix}
   \sum_{w}{I_x^2(i,j)} & \sum_{w}{I_x(i,j)I_y(i,j)} \\
   \sum_{w}{I_x(i,j)I_y(i,j)}& \sum_{w}{I_y^2(i,j)}
  \end{matrix}
 \right] 
 \left[
 \begin{matrix}
   x_{corner} \\
   y_{corner}
  \end{matrix}
 \right] 
 =
 \left[
 \begin{matrix}
   \sum_{w}{xI_x^2(i,j)+yI_x(i,j)I_y(i,j)} \\
   \sum_{w}{xI_x(i,j)I_y(i,j)+yI_y^2(i,j)}
  \end{matrix}
 \right] 
$$











