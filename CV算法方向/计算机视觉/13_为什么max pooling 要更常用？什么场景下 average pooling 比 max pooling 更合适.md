## Pooling层的作用

1. 首要作用，下采样（downsamping），降维、去除冗余信息。增大感受野。
2. 实现非线性，在一定程度上能防止过拟合的发生。
3. 可以实\现特征不变性（feature invariant）。

## 为什么Max pooling更常用？

通常来讲，max-pooling的效果更好，虽然**max-pooling和average-pooling都对数据做了下采样，但是max-pooling感觉更像是做了特征选择，选出了分类辨识度更好的特征，提供了非线性。**这就类似于nms（非极大抑制)，一方面能抑制噪声，另一方面能提升特征图在区域内的显著性（筛选出的极大值）。

根据相关理论，特征提取的误差主要来自两个方面：（看不懂，先记下来）

- （1）邻域大小受限造成的估计值方差增大；
- （2）卷积层参数误差造成估计均值的偏移。

​    一般来说，average-pooling能减小第一种误差，更多的保留图像的背景信息，max-pooling能减小第二种误差，更多的保留纹理信息。

- 最大池化保留了纹理特征
- 平均池化保留整体的数据特征
- 全局平均池化有定位的作用(看知乎)

   **Average-pooling**更强调对整体特征信息进行一层下采样，在减少参数维度的贡献上更大一点，**更多的体现在信息的完整传递这个维度上**，在一个很大很有代表性的模型中，比如说DenseNet中的模块之间的连接大多采用average-pooling，在减少维度的同时，更有利信息传递到下一个模块进行特征提取。

但是average-pooling在**全局平均池化（global average pooling）**操作中应用也比较广，在ResNet和Inception结构中最后一层都使用了平均池化。有的时候在模型接近分类器的末端使用全局平均池化还可以代替Flatten操作，使输入数据变成一维向量。

max-pooling和average-pooling的使用性能对于我们设计卷积网络还是很有用的，虽然池化操作对于整体精度提升效果也不大，但是在减参，控制过拟合以及提高模型性能，节约计算力上的作用还是很明显的，所以池化操作时卷积设计上不可缺少的一个操作。

## 什么场景用Max_Pooling 什么场景用Average-pooling

在分类问题中，我们需要知道的是这张图像有什么object，而不大关心这个object位置在哪，在这种情况下显然max pooling比average pooling更合适。典型的如ResNet在输入全连接层之前利用Kernel Size=7的AvgPooling来降维。反之为了减少无用信息的影响时用maxpool，比如网络浅层常常见到maxpool，因为开始几层对图像而言包含较多的无关信息。

在网络比较深的地方，特征已经稀疏了，从一块区域里选出最大的，比起这片区域的平均值来，更能把稀疏的特征传递下去。当特征map中的信息都具有一定贡献的时候使用AvgPooling，例如**图像分割中**常用global avgpool来获取全局上下文关系，再比如网络走到比较深的地方，这个时候特征图的H W都比较小，包含的语义信息较多，这个时候再使用MaxPooling就不太合适了， 是因为网络深层的高级语义信息一般来说都能帮助分类器分类。这对于**对象检测类型任务**可能不好用但使用`平均池化`的一个动机是每个空间位置具有用于`期望特征`的检测器，并且通过平均每个空间位置，其行为类似于平均输入图像的不同平移的预测(有点像数据增加)。

## 卷积替代池化

avg-pooling就是一般的平均滤波卷积操作，而max-pooling操作引入了非线性，可以用stride=2的CNN+RELU替代，性能基本能够保持一致，甚至稍好。已经有最新的一些网络结构去掉了pooling层用步长为2的卷积层代替。

## 平均池化与全局平均池化

全局平均池化是在论文Network in Network中提出的

<img src="https://s2.loli.net/2024/08/08/42PlTXk35xRmsdu.png" alt="image.png" style="zoom: 80%;" />

**思想：**对于输出的每一个通道的特征图的所有像素计算一个平均值，经过全局平均池化之后就得到一个 **维度=$C_{in}$=类别数** 的特征向量，然后直接输入到softmax层

**作用：**代替全连接层，可接受任意尺寸的图像

**优点：**

1）可以更好的将类别与最后一个卷积层的特征图对应起来（每一个通道对应一种类别，这样每一张特征图都可以看成是该类别对应的类别置信图）

2）降低参数量，全局平均池化层没有参数，可防止在该层过拟合

3）整合了全局空间信息，对于输入图片的spatial translation更加鲁棒

直观的图解：

![image.png](https://s2.loli.net/2024/08/08/LeglmbTtoJ4GuEO.png)

**注意：**图上GAP指的是4*4=16。4个类别。

## 各个池化方式在Pytorch中的实现

1. 最大池化

   ```python
   maxpool_layer = torch.nn.MaxPool2d(
       kernel_size, # 池化窗口的大小
       stride=None, # 池化操作的步长，默认等于窗口大小
       padding=0,   # 零像素的边缘填充数量
       dilation=1,  # 扩张元素的数量
       return_indices=False, # 返回池化取值的索引，并通过nn.MaxUnpool2d()进行反池化
       ceil_mode=False # 在输出尺寸中是否使用向上取整代替向下取整
       )
   ```

2. 平均池化

   ```python
   avgpool_layer = torch.nn.AvgPool2d(
       kernel_size, # 池化窗口的大小
       stride=None, # 池化操作的步长，默认等于窗口大小
       padding=0,   # 零像素的边缘填充数量
       ceil_mode=False # 在输出尺寸中是否使用向上取整代替向下取整
       count_include_pad=True, # 计算均值时是否考虑填充像素
       divisor_override=None # 若指定将用作平均操作中的除数
       )
   ```

3. 全局平均池化

   ```python
   gap_layer = torch.nn.AdaptiveAvgPool2d(output_size=1)
   ```

## 参考

1. [Max Pooling和 Average Pooling的区别，使用场景分别是什么？](https://blog.csdn.net/ytusdc/article/details/104415261)
2. [全局平均池化](https://blog.csdn.net/u012370185/article/details/95591712)
3. [卷积神经网络中的池化(Pooling)层](https://0809zheng.github.io/2021/07/02/pool.html)