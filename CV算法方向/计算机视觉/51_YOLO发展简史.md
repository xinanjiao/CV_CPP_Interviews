## YOLO各个系列对比

| 模型   | backbone                            | 是否解耦 | 正负样本匹配策略         | 损失函数                                                     | 是否anchor free |      |      |      |      |
| ------ | ----------------------------------- | -------- | ------------------------ | ------------------------------------------------------------ | --------------- | ---- | ---- | ---- | ---- |
| yolov3 | Darknet-53                          | 否       | Max-IOU匹配策略          | IOU_Loss                                                     | 否              |      |      |      |      |
| yolov4 | CSPDarknet-53                       | 否       | Multi anchor匹配策略     | CIOU_Loss<br />DIOU_Loss                                     | 否              |      |      |      |      |
| yolov5 | CSPDarknet-53+Focus                 | 否       | 跨分支、网格、anchor预测 | GIOU_Loss<br />DIOU_Loss                                     | 否              |      |      |      |      |
| yolox  | Darknet-53                          | 是       | simOTA动态标签分配策略   | CIOU_Loss<br />DIOU_Loss                                     | 是              |      |      |      |      |
| yolov6 | EfficientRep 参数重构化             | 是       | simOTA动态标签分配策略   | SIOU_Loss<br />DIOU_Loss                                     | 是              |      |      |      |      |
| yolov7 | Darknet-53(CSP模块替换了ELAN模块)   | 否       | simOTA动态标签分配策略   | CIOU_Loss<br />DIOU_Loss                                     | 否              |      |      |      |      |
| yolov8 | Darknet-53（c3模块替换为了c2f模块） | 是       | TaskAlignedAssigner      | CIOU_Loss<br />DFL_Loss<br />(Distribution Focal Loss)<br />DIOU_Loss | 是              |      |      |      |      |

![image.png](https://s2.loli.net/2024/07/15/zXOqaxKAQ2kTwhc.png)

### 一些问题

**使用==anchor free==方法有如下好处：**

(1) 降低了计算量，不涉及IoU计算，另外产生的预测框数量也更少。

- 假设 feature map的尺度为 $80\times80$，anchor based 方法在Feature Map上，每个单元格一般设置三个不同尺寸大小的锚框，因此产生 3 × 80 × 80 = 19200 个预测框。而使用anchor free的方法，则仅产生 80 × 80 = 6400 个预测框，降低了计算量。

(2) 缓解了正负样本不平衡问题

- anchor free方法的预测框只有anchor based方法的1/3，而预测框中大部分是负样本，因此anchor free方法可以减少负样本数，进一步缓解了正负样本不平衡问题。

(3) 避免了anchor的调参

- anchor based方法的anchor box的尺度是一个超参数，不同的超参设置会影响模型性能，anchor free方法避免了这一点。
  

之前yolo 系列都是使用coupled head， yolox使用decoupled head。

**==coupled head== 和 ==decoupled head== 有什么差异？**

- 当使用coupled head时，网络直接输出shape (1,85,80,80);
- 如果使用 ==decoupled head==，网络会分成回归分支和分类分支，最后再汇总在一起，得到shape同样为 (1,85,80,80)。

![image.png](https://s2.loli.net/2024/07/15/Ui5TONlsmMftrHC.png)

**为什么用decoupled head？**

- 如果使用coupled head，输出channel将分类任务和回归任务放在一起，这2个任务存在==冲突性==。

- 通过实验发现替换为Decoupled Head后，不仅是模型精度上会提高，同时网络的==收敛速度也加快==了，使用Decoupled Head的表达能力更好。

## 目标检测——map概念、Miou计算、IoU汇总IoU、GIoU、DIoU、CIoU、SIoU、EIoU、Wiou、Focal、alpha

**Reference**:[目标检测——map概念、Miou计算、IoU汇总IoU、GIoU、DIoU、CIoU、SIoU、EIoU、Wiou、Focal、alpha_miou ciou-CSDN博客](https://blog.csdn.net/weixin_45464524/article/details/128649991)

