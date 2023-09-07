# Abstract

我们提出了一个高分辨率transformer(HRFormer)，它可以学习dense预测任务的高分辨率表示，与原始vision transformer形成对比，前者产生低分辨率表示，具有较高的内存和计算成本。我们利用高分辨率卷积网络(HRNet[46])中引入的多分辨率并行设计，以及**对小的非重叠图像窗口**[arxiv 2019：Interlaced sparse self-attention for semantic segmentation]**进行自注意的局部窗口自注意**，提高了内存和计算效率。此外，我们在FFN中引入一个卷积来横穿断开连接的图像窗口交换信息。我们证明了高分辨率transformer在人体姿态估计和语义分割任务上的有效性，例如，HRFormer在COCO姿态估计上比Swin transformer[27]高出1.3个AP，参数减少50%，flop减少30%。代码可在以下网站获得:https://github.com/HRNet/HRFormer。

# 1 Introduction

Vision Transformer(ViT)[13]在ImageNet分类任务中表现出良好的性能。后续许多工作通过知识蒸馏[42]、采用更深层次的架构[43]、直接引入卷积运算[16,48]、重新设计输入图像tokens[54]等方式提高分类精度。此外，一些研究试图扩展 transformer以解决更广泛的视觉任务，如目标检测[4]、语义分割[63,37]、姿态估计[51,23]、视频理解[61,2,30]等。本文主要研究了用于密集预测任务的 transformer，包括姿态估计和语义分割。

Vision transformer将一幅图像分割成大小为16 × 16的图像patch序列，提取每个图像块的特征表示。因此，Vision Transformer的输出特征丢失了精确密集预测所必需的细粒度空间细节。Vision Transformer只输出单一尺度的特征表示，因此缺乏处理多尺度变化的能力。为了减少特征粒度的损失并对多尺度变化进行建模，我们提出了包含更丰富空间信息的高分辨率变压器(HRFormer)，并为密集预测构建多分辨率表示。

高分辨率变压器采用HRNet[46]中采用的多分辨率并行设计。首先，**HRFormer在主干和第一阶段都采用了卷积**，因为一些同时进行的研究[11,50]也表明卷积在早期阶段表现更好。其次，HRFormer在整个过程中保持高分辨率流，并行的中分辨率和低分辨率流有助于提高高分辨率表示。因此，HRFormer具有不同分辨率的特征图，能够对多尺度变化进行建模。第三，HRFormer通过与多尺度融合模块交换多分辨率特征信息，混合了近距离和远程注意。

在每个分辨率下，采用局部窗口自注意机制来降低内存和计算复杂度。我们**将表示图划分为一组不重叠的小图像窗口，并在每个图像窗口中分别执行自注意**。这将内存和计算复杂度从二次的降低到相对于空间大小的线性。我们进一步在局部窗口自注意后的前馈网络(FFN)中引入一个3 × 3深度卷积，用于在局部窗口自注意过程中断开的图像窗口之间交换信息。这有助于扩展接受域，对于密集的预测任务是必不可少的。**图1显示了HRFormer块**的详细信息。

![image-20221026095519162](E:\MarkDown\picture\image-20221026095519162.png)

我们对图像分类、姿态估计和语义分割任务进行了实验，并在各种基准测试中取得了具有竞争力的性能。例如，与DeiT-B[42]相比，HRFormer-B在ImageNet分类上获得+1.0%的top-1精度，且参数减少40%，flop减少20%。在COCO val设置下，HRFormer-B比HRNet-W48[41]获得0.9%的AP，参数减少32%，flop减少19%。在PASCAL-Context测试和COCO-Stuff测试中，HRFormer-B + OCR比HRNet-W48 + OCR[55]分别少了25%的参数和略多的FLOPs，获得了+1.2%和+2.0%的mIoU。

# 2 Related work

在HRFormer中插入卷积的目的是不同的，除了增强局域性之外，它还**确保了在非重叠窗口之间的信息交换。**

**高密度预测的高分辨率CNN**。高分辨率卷积算法在姿态估计和语义分割方面都取得了巨大的成功。在高分辨率卷积神经网络的发展中，研究人员开发了三种主要路径，包括:(i)应用空洞卷积来去除一些下采样层[6,52]，(ii)使用解码器从低分辨率表示中恢复高分辨率表示[38,1,31,32]，以及(iii)在整个网络中保持高分辨率表示[46,15,39,64,45,59,20]。我们的HRFormer属于第三种路径，同时保留了视觉转换器和HRNet[46]的优点。

> *dense prediction tasks?*
>
> 指的是如语义分割、姿态评估这类需要对图像中每个像素所属类别进行细致分类的任务。

# 3 High-Resolution Transformer

**多分辨率并联transformer**。我们遵循HRNet[46]的设计，从一个高分辨率的卷积stem开始作为第一阶段，逐步增加一个高到低分辨率的流作为新阶段。多分辨率流是并行连接的。主体由一系列的阶段组成。在每一阶段中，每个分辨率流的特征表示由多个transformer块独立更新，跨分辨率的信息与卷积多尺度融合模块进行重复交换。

**图2说明了整个HRFormer体系结构**。卷积多尺度融合模块的设计完全遵循了HRNet的思路。我们在接下来的讨论中说明transformer块的细节，更多的细节在图1中显示。

<img src="E:\MarkDown\picture\image-20221026100037683.png" alt="image-20221026100037683" style="zoom:200%;" />

图解高分辨率Transformer体系结构。多分辨率并联transformer模块用浅蓝色区域标记。每个模块由多个连续的多分辨率并联变压器块组成。第一级采用卷积块结构，其余三级采用HRFormer块结构

**Local-window self-attention**

我们将特征映射X∈R^N×D划分为一组不重叠的小窗口:X → {X1, X2，···，XP}，其中每个窗口的大小为K × K。我们在每个窗口内独立地执行多头自注意(MHSA)。第p个窗口上的多头自注意的表达式为:
<img src="E:\MarkDown\picture\image-20221026101923292.png" alt="image-20221026101923292" style="zoom:150%;" />

我们还应用了T5模型[35]中引入的**相对位置嵌入方案**，将相对位置信息嵌入到局部窗口自注意中。

> **通过Embedding层，完成从高维稀疏特征向量到低维稠密特征向量的转换**
>
> 
>
> T5：**Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer**
>
> 绝对位置方法将输入Token的绝对位置从1编码到最大列长度。也就是说，每个位置都有一个单独的编码向量。然后将编码向量与输入Token组合,以向模型公开位置信息。
> 相对位置方法对输入元素之间的相对距离进行编码，学习符号之间的成对关系。相对位置编码(RPE)通常是通过带有可学习参数的query表与Self-attention模块中的query和key进行交互来计算的。这种模式允许模块捕获Token之间非常长的依赖关系。相对位置编码在自然语言处理中的有效性得到了验证。
>
> 相对位置嵌入不是对每个位置使用固定的嵌入，而是根据自我注意机制中比较的“键”和“查询”之间的偏移量产生不同的学习嵌入。我们使用一种简化的位置嵌入形式，其中每个“embedding”只是一个标量，添加到用于计算注意力权重的相应的logit中。为了提高效率，我们还在模型中的所有层之间共享位置嵌入参数，尽管在给定层内，每个注意力头使用不同的学习位置嵌入。通常，学习固定数量的嵌入，每个嵌入对应于一系列可能的键查询偏移量。在这项工作中，我们为所有模型使用了32个嵌入，其范围以对数方式增大到偏量 128，超过该偏移量我们将所有相对位置分配给相同的嵌入。请注意，给定层对超过128 个标记的相对位置不敏感，但后续层可以通过组合来自前一层的局部信息来构建对更大偏移量的敏感性。![image-20221027183754614](E:\MarkDown\picture\image-20221027183754614.png)
>
> 有点区别
>
> https://blog.csdn.net/chenf1995/article/details/122971023
>
> ![image-20221027185528650](E:\MarkDown\picture\image-20221027185528650.png)

通过MHSA在每个窗口内聚合信息，我们将它们合并以计算输出X^MHSA:<img src="E:\MarkDown\picture\image-20221026102227998.png" alt="image-20221026102227998" style="zoom:70%;" />

图1的左侧说明了局部窗口自注意如何更新2D输入表示，其中多头自注意在每个窗口内独立操作。



**具有depth-wise convolution的FFN**

本地窗口自注意分别对非重叠窗口执行自注意。没有跨窗口的信息交换。为了处理这个问题，我们在形成视觉转换器中的FFN的两个point-wise MLP之间添加了一个3 × 3深度卷积（深度可分离卷积的前半部分为深度卷积）:MLP(DW-Conv.(MLP()))。图1的右边部分显示了一个3 × 3深度卷积FFN如何更新2D输入表示的例子。

![image-20221027191033720](E:\MarkDown\picture\image-20221027191033720.png)

![image-20221027191208274](E:\MarkDown\picture\image-20221027191208274.png)

**Representation head designs**

如图2所示，HRFormer的输出由4个不同分辨率的特征图组成。我们举例说明了不同任务的表示头设计的细节如下:(i) ImageNet分类，我们将四分辨率的特征映射发送到一个bottleneck，输出通道分别更改为128,256,512和1024。然后，我们应用跨步卷积进行融合，输出最低分辨率的2048通道特征图。最后，我们应用全局平均池化操作，然后是最后的分类器。(ii)**姿态估计，我们只在最高分辨率的特征图上应用回归头**。(iii)语义分割，我们将语义分割头应用于连接的表示上，首先将所有低分辨率表示上采样到最高分辨率，然后将它们连接在一起。

**实例化**

![image-20221026103018431](E:\MarkDown\picture\image-20221026103018431.png)

我们在表1中说明了HRFormer的整体体系结构配置。我们分别用(M1, M2, M3, M4)和(B1, B2, B3, B4)来表示{state1, stage2, stage3, stage4}的模块数和 block数。我们用(C1, C2, C3, C4)， (H1, H2, H3, H4)和(R1, R2, R3, R4)来表示与不同分辨率相关联的变压器块中的通道数、头数和MLP膨胀比。我们在最初的HRNet基础上保持第一个阶段不变，并使用bottleneck作为基本构建块。我们将变压器块应用于其他阶段，每个变压器块由一个局部窗口自注意和一个 (3 × 3)depth-wise convolution的FFN组成。为了简单起见，我们没有在表1中包含卷积多尺度融合模块。在我们的实现中，我们将四个分辨率流上的窗口的大小默认设置为(7,7,7)。表2展示了三个不同HRFormer实例的配置细节，其中MLP扩展比(R1, R2, R3, R4)对所有模型都设置为(4,4,4,4)，没有显示出来。

![image-20221026103455204](E:\MarkDown\picture\image-20221026103455204.png)



**Analysis**

3 × 3深度卷积的好处有两个:一个是增强局部性，另一个是实现跨窗口的交互。我们演示了具有深度卷积的FFN如何能够扩展非重叠局部窗口之外的相互作用，并在图3中为它们之间的关系建模。因此，基于局部窗口自注意和3 × 3深度卷积FFN的结合，我们可以构建显著提高内存和计算效率的HRFormer块



![image-20221026102720943](E:\MarkDown\picture\image-20221026102720943.png)

# 4 Experiments

**Human Pose Estimation**

训练设置。我们研究了HRFormer在COCO[26]人体姿态估计基准上的性能，该基准包含超过200K图像和250K人实例，标记有17个关键点。我们在COCO train 2017数据集上训练我们的模型，包括57K图像和150K人实例。我们在val 2017集和test-dev 2017上评估我们的方法，分别包含5K和20K图像。

我们遵循mmpose[8]的大多数默认训练和评估设置，并将优化器从Adam更改为AdamW。在训练批大小上，由于GPU内存有限，HRFormer-T和HRFormer-S选择256,HRFormer-B选择128。每次COCO姿态估计任务HRFormer实验需要8× 32G-V100 gpu。

**Results**

表3报告了COCO验证集的比较。我们将HRFormer与具有代表性的卷积方法进行比较，如HRNet[41]和一些最近的变压器方法，包括PRTR [23]， TransPose-H-A6[51]和TokenPose-L/D24[24]。与输入尺寸为384 × 288的HRNet-W48相比，HRFormer-B增益0.9%，参数减少32%，flop减少19%。因此，我们的HRFormer-B没有使用任何先进的技术，如UDP[20]和DARK[59]，已经实现了77.2%。我们相信HRFormer-B可以通过使用UDP或DARK方案获得更好的结果。我们还报告了表4中COCO测试开发集的比较。我们的HRFormer-B在参数和flop更少的情况下比HRNet-W48性能好0.7%左右。图4显示了COCO验证集上人体姿态估计的一些示例结果。

![image-20221026105742737](E:\MarkDown\picture\image-20221026105742737.png)

 **Ablation Experiments**

**Influence of 3 × 3 depth-wise convolution within FFN**

我们在表7中研究了基于HRFormer-T的FFN中3 × 3深度卷积的影响。我们观察到，在FFN中应用3 × 3深度卷积显著提高了多个任务的性能，包括ImageNet分类、pascal -上下文分割和COCO姿态估计。例如，在ImageNet、PASCAL-Context和COCO上，HRFormer-T + FFN w/ 3× 3深度卷积比HRFormer-T + FFN w/o 3× 3深度卷积的性能分别提高了0.65%、2.9%和4.04%。

![image-20221026105931968](E:\MarkDown\picture\image-20221026105931968.png)

**Influence of shifted window scheme & 3×3 depth-wise convolution within FFN based on Swin-T**

我们将我们的方法与表8中Swin变压器[27]的移位窗口方案进行比较。为了进行公平的比较，我们按照相同的方法构造了一个与swing - t相同的Intra-Window  transformer结构,除了我们不应用移位窗口方案。我们看到，在FFN中应用3×3深度卷积改进了swing - t和Intra winT。令人惊讶的是，当在FFN中配置3× 3深度卷积时，Intrawin-T甚至优于swing - t。

![image-20221026111638473](E:\MarkDown\picture\image-20221026111638473.png)

**Shifted window scheme v.s. 3×3 depth-wise convolution within FFN based on HRFormer-T.** 

在表9中，我们比较了FFN方案中的3 × 3深度卷积与基于HRFormer-T的移位窗口方案。根据结果，我们看到在FFN中应用3×3深度卷积在所有不同的任务中显著优于应用移位窗口方案。

![image-20221026111655513](E:\MarkDown\picture\image-20221026111655513.png)



**Comparison to ViT, DeiT & Swin on pose estimation**

我们报告了基于两个知名变压器模型的COCO位姿估计结果，包括表10中的vit-large [13]， DeiT-B⚗[42]和swin - b[27]。值得注意的是，vit - large和swin - b都是在ImageNet21K上进行预先训练，然后在ImageNet1K上进行微调，分别达到85.1%和86.4%的top-1精度。

DeiT-B⚗在ImageNet1K上训练了1000个epoch，达到了85.2%的top-1精度。我们应用反卷积模块对三种方法的SimpleBaseline[49]之后的编码器输出表示进行上采样。参数和flop的数量列在表10的第四和第五列中。根据表10中的结果，我们可以看到，我们的HRFormer-B比所有三种具有较少参数和flop的方法获得了更好的性能。

![image-20221026111901529](E:\MarkDown\picture\image-20221026111901529.png)



**Comparison to HRNet**

通过将所有的transformer块替换为由两个3 × 3卷积组成的传统基本块，我们将HRFormer与具有几乎相同架构配置的卷积HRNet进行了比较。表11显示了在ImageNet、PASCAL-Context和COCO上的比较结果。我们观察到，在各种配置下，HRFormer的模型和计算复杂度都大大降低，性能明显优于HRNet。例如，HRFormer-T在三个任务上的性能分别比HRNet-T高出2.0%、1.5%和1.6%，而分别只需要大约50%的参数和flop。总而言之，HRFormer通过利用转换器(如依赖于内容的动态交互)的好处实现了更好的性能。

![image-20221026112057664](E:\MarkDown\picture\image-20221026112057664.png)

# 5 Conclusion

在这项工作中，我们提出了高分辨率变压器(HRFormer)，一个简单而有效的变压器架构，用于密集的预测任务，包括姿势估计和语义分割。

其关键思想是将HRFormer块与卷积HRNet的多分辨率并行设计相结合，HRFormer块将局部窗口自注意和FFN与深度卷积相结合，以提高记忆和计算效率。此外，HRFormer在早期采用卷积，在多尺度融合方案中混合近程和远程注意，也有一定的好处。我们通过经验验证了HRFormer在姿态估计和语义分割任务上的有效性。



![image-20221026112237184](E:\MarkDown\picture\image-20221026112237184.png)









































