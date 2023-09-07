官方代码仅提供用于图像分类的代码



## MLP

多层感知器（Muti-Layer Perception ，MLP）



## AS-MLP：上海科技&腾讯优图开源首个检测与分割领域MLP架构

paper: https://arxiv.org/abs/2107.08391
Code: https://github.com/svip-lab/AS-MLP

> 设计了一种轴向移位操作以便于进行空间信息交互。在架构方面，AS-MLP采用了类似PVT的分层架构，因为可以轻易的迁移到下游任务。所提方法在ImageNet数据集上取得了优于其他MLP架构的性能，在COC检测与**ADE20K分割任务上取得了与Swin相当的性能**。值得一提的是，**AS-MLP是首个迁移到下游任务的MLP架构**。注：CycleMLP与AS-MLP属于同一时期的工作，发到arxiv的时间也只差两天，说两者都是首个其实也可以。

[1](https://jishuin.proginn.com/p/763bfbd619d2)  [2](https://zhuanlan.zhihu.com/p/399200591)

patch partition 将图像分割为四块

patch merging 进行块合并，将相邻2* 2块合并得到尺寸为4C* H/8，后线性映射为2C * H/8

![image-20210916105510602](E:\MarkDown\picture\image-20210916105510602.png)



![image-20210916105555646](E:\MarkDown\picture\image-20210916105555646.png)

如上图b所示，我们以水平移动进行说明。假设输入尺寸为 *C×h×w* ，为方便起见，我们忽略了h并假设 *C=3,w=5。当移动尺寸为3时，输入特征被分为三部分，每部分分别沿水平方向移动 *{-1,0,1}* 步长。注：此时我们采用了“zero-padding”。垂直移动操作与水平移动非常类似。通过水平移动与垂直移动，特征可以进行了单一空间方向上的汇聚。在接下来的通道投影操作，两个方向的信息将进行汇聚。

在AS-MLP中使用不同的shift size和dilation rate，因此使得网络具有不同的感受野。例如，图四中的第六张图显示了当shift size为3，dilation rate为2时候的感受野大小。

![image-20210916105925324](E:\MarkDown\picture\image-20210916105925324.png)

![img](E:\MarkDown\picture\v2-30ca3f9e4a62216aa9c56bf12019350a_1440w.jpg)

**（三）在ADE20K数据集上的语义分割性能**

batch size(2*8GPUs)，输入图片分辨率512，初始化权重为在ImageNet-1K上预训练，迭代次数160K

**utilize UperNet and AS-MLP backbone**

FLOPs 理论计算量，MS是多尺度？

表三显示了我们的 AS-MLP 在 ADE20K  数据集上的性能。请注意，我们也是第一个将基于 MLP 框架应用于语义分割的工作。在更少的计算量的情况下，AS-MLP-T 取得了比 Swin-T 更好的结果（46.5 vs. 45.8 MS mIoU）。对于大型模型，UperNet + Swin-B 有着 49.7 MS  mIoU，121M 参数和 1188 GFLOPs，UperNet + AS-MLP-B 有 49.5 MS mIoU，121M 参数和  1166 GFLOPs，这也显示了我们 的AS-MLP在处理下游任务时的有效性。图六也显示了我们的方法在ADE20K数据集上的语义分割的结果。

<img src="E:\MarkDown\picture\image-20210911102720147.png" alt="image-20210911102720147" style="zoom: 33%;" />

**对比SETR:**

<img src="E:\MarkDown\picture\image-20210911110721732.png" alt="image-20210911110721732" style="zoom: 50%;" />

![image-20210912160657694](E:\MarkDown\picture\image-20210912160657694.png)

![image-20210912160441441](E:\MarkDown\picture\image-20210912160441441.png)







