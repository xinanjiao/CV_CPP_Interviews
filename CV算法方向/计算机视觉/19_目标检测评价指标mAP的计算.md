## 前置知识

### 混淆矩阵

<img src="https://s2.loli.net/2024/08/10/MgcNOK8pxLCJFIR.png" alt="image.png" style="zoom:50%;" />

**表中的四个参数说明如下：**

**True Positive（TP）：**预测为正例，实际为正例，即算法预测正确（True） **（真实值为正样本）**

**False Positive（FP）：**预测为正例，实际为负例，即算法预测错误（False） **（真实值为负样本）**

**True Negative（TN）：**预测为负例，实际为负例，即算法预测正确（True）	**（真实值为负样本）**

**False Negative（FN）：**预测为负例，实际为正例，即算法预测错误（False）	**（真实值为正样本）**

### Precision 精确率

​	精确率，指的是==正确预测的正样本数占所有预测为正样本的数量的比值==，也就是说所有预测为正样本的样本中，多少为真正的正样本。该指标只关注正样本，公式如下：(简化记忆（预测都为P）)
$$
Precision=\frac{TP}{TP+FP}=\frac{TP}{allDetection}
$$

### Recall 召回率

​	召回率，指的是==正确预测的正样本数占真实正样本总数的比值==，也就是这些预测的样本中能找到多少个正样本，公式如下：
$$
Recall=\frac{TP}{TP+FN}=\frac{TP}{allGroundTruth}
$$

### Accuray 准确率

​	准确率，表示所有样本有多少是被准确预测（包含正样本和负样本）了（使用准确率的一个问题是当数据不平衡时，没法准确评估精度，数据越不平衡，问题就越严重。）
$$
Acc=\frac{TP+TN}{TP+FP+TN+FN}
$$
​	上公式中的TP+TN即为所有的正确预测为正样本的数据与正确预测为负样本的数据的总和，TP+TN+FP+FN即为总样本的个数

### IOU（Intersection over Union）交并比

​	交并比，指的是ground truth bbox与predict bbox的交集面积占两者并集面积的一个比率，IoU值越大说明预测检测框的模型算法性能越好，通常在目标检测任务里将IoU>=0.7的区域设定为正例（目标），而将IoU<=0.3的区域设定为负例（背景），其余的会丢弃掉，形象化来说可以用如下图来解释IoU：

![image.png](https://s2.loli.net/2024/08/10/qNcUQSOWue7lMhb.png)

### **平均精度 AP（Average Precision）**

​	PR曲线（红线）以下与横轴、纵轴之间的面积。PR曲线是由Precision（精准率或者查准率）与Recall（召回率或者查全率）构成的曲线，横轴为Recall，纵轴为Precision。

AP即PR曲线的面积，计算方式有:

- 积分求解
- 插值求解

![image.png](https://s2.loli.net/2024/08/13/s23mwVEpYkUJQ8O.png)

### mAP计算

​	对于mAP（mean of Average Precision）的计算，其实就是在目标检测中我们可能是在检测多个目标，因此我们对每种类型的目标都可以计算出这个种类的AP，最后再对各个种类进行求平均就可以了。