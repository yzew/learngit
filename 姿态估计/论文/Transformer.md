**此模型参数非常大，我用24G显存，batch_size为1才能勉强训练**

AS-MLP有参数对比



[最好的讲解](https://zhuanlan.zhihu.com/p/403433120)

[transformor详细讲解](https://blog.csdn.net/Tink1995/article/details/105080033)

[transformor英文详解](http://jalammar.github.io/illustrated-transformer/)



# 一、SETR

## 1.自注意力	

[论文Attention Is All You Need](https://arxiv.org/abs/1706.03762) details about key,query and value.

![image-20210908155819213](E:\MarkDown\picture\image-20210908155819213.png)

n个序列x1到xn，每个序列xi为长度为d的向量，而yi中，xi为query，(x1,x1)等分别为key和value。即三个参数都来自于xi自己，所以为Self-attention

![image-20210908160359041](E:\MarkDown\picture\image-20210908160359041.png)



## 2.Transformor

[详解transformor](https://zhuanlan.zhihu.com/p/48508221)

[图解Transformer（完整版）](https://blog.csdn.net/longxinchen_ml/article/details/86533005?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163109390016780366511903%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163109390016780366511903&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-86533005.pc_search_ecpm_flag&utm_term=transformer&spm=1018.2226.3001.4187)

基于编解码架构处理序列对
与使用注意力的seq2seq不同，这纯基于注意力，无RNN了



![image-20210908110718959](E:\MarkDown\picture\image-20210908110718959.png)

**Multi-head attention多头注意力**

就是多个自注意力，每个提取不同特征。最后在特征维度concat，然后FC得到需要的维度

<img src="E:\MarkDown\picture\image-20210908110932853.png" alt="image-20210908110932853" style="zoom:80%;" />

![image-20210908111336781](E:\MarkDown\picture\image-20210908111336781.png)

**Masked multi-head attention**

<img src="E:\MarkDown\picture\image-20210908111726270.png" alt="image-20210908111726270" style="zoom:50%;" />

**Positionwise FFN基于位置的前馈网络**

就是个全连接层，b为batch_size,n为序列长度,d为dimension尺度

<img src="E:\MarkDown\picture\image-20210908112109226.png" alt="image-20210908112109226" style="zoom:50%;" />

**Add&norm**

<img src="E:\MarkDown\picture\image-20210908112334517.png" alt="image-20210908112334517" style="zoom:67%;" />

采用layer norm而不是batch norm，因为len是变化的，BN后输入的尺寸也会随着变化。batch norm以d的维度，layer norm是在每个batch里面，单样本里面，虽然长度也在变化，但稍微稳定一点



Linear projection，做了一个线性的映射
![image-20210908182127457](E:\MarkDown\picture\image-20210908182127457.png)





## 3.Rethinking Semantic Segmentation from a Sequence-to-Sequence Perspective with Transformers（SETR）

CVPR2021 基于Transformers 从序列到序列的角度重新思考语义分割

【机构】复旦大学、牛津大学、萨里大学、腾讯优图、Facebook 【论文链接】https://arxiv.org/abs/2012.15840 【代码链接】https://github.com/fudan-zvg/SETR

[讲解一](https://zhuanlan.zhihu.com/p/348418189)       [讲解加翻译二](https://blog.csdn.net/Acmer_future_victor/article/details/115789573)



**DETR-transformor对transformor进行了改进，使得训练稍快了一点**



没看：

- 卷积操作的感受野有限是传统FCN体系结构的一个内在限制。为突破该限制，逐渐提出两类方法1.改变卷积：包括增大卷积核kernel_size、Non-local(跟本文有点像，每次抽取的都是全局特征)和特征金字塔。例如DeepLab引入空洞卷积/SPP/ASPP。2.将注意力模块集成到FCN体系结构中：一次对所有像素的全局信息抽取特征。例如PSANet提出点向空间注意模块、DANet嵌入channel attention和spatial attention。
- 有人提到，本文是把ViT模型原封不动迁移过来了，替换了encoder，虽带来了精度的提升但模型的计算量和参数量都非常大。

> ViT(Vision Transformer)首次证明了纯基于transformer的图像分类模型可以达到sota。

- CNN是通过不断地堆积卷积层来完成对图像从局部信息到全局信息的提取,不断堆积的卷积层慢慢地扩大了感受野直至覆盖整个图像;但是transformer并不假定从局部信息开始,而且一开始就可以拿到全局信息,学习难度更大一些,但transformer学习长依赖的能力更强。
- CNN结构更适合底层特征,Transformer更匹配高层语义。二者无绝对差别,就是看问题的尺度差异,本质都是消息传递。





Abst

大部分最新的语义分割方法采用了具有编解码器结构的全卷积网络(FCN)。编码器逐渐降低空间分辨率，并通过更大的感受野学习更多的抽象/语义视觉概念。由于上下文模型对分割至关重要，最近的努力集中在通过空洞卷积或引入注意模块来增加感受野。但是基于FCN的编解码器体系结构保持不变。在本文中，我们旨在提供一个替代的视角，将语义分割作为一个序列到序列的预测任务。具体来说，我们部署了一个纯粹的transformor(即没有卷积和分辨率降低)来将图像编码为一系列的patch。通过在transformor每一层中的全局上下文，这个编码器可以与一个简单的解码器组合，以提供一个强大的分割模型，称为segmentation transformer (SETR)。大量的实验表明，SETR在ADE20K上达到了新的水平(50.28% mloU)，Pascal Context(55.83% mloU)和在Cityscapes数据集上有竞争力的结果。特别是，我们在提交当天就在竞争激烈的ADE20K测试服务器排行榜上获得了第一名







我们希望为语义分割方法提供另一种思路，将语义分割转变为序列到序列的预测任务。在本文中，我们使用transformer（不使用卷积和降低分辨率）将图像编码为一系列patch序列。transformer的每一层都进行了全局的上下文建模，结合常规的Decoder模块，我们得到了一个强大的语义分割模型，称之为Segmentation transformer（SETR）。大量实验表明，SETR在ADE20K（50.28％mIoU），Pascal  Context（55.83％mIoU）上达到SOTA，并在Cityscapes上取得了较好结果。	

近年来的大多数语义分割方法都采用了全卷积网络（FCN）和编码器-解码器架构。编码器多次下采样会逐步降低空间分辨率，是通过更大的感受野学习到更加抽象的语义的视觉概念。网络Layer一旦固定,每一层的感受野是受限的,因此要获得更大范围的语义信息,理论上需要更大的感受野即更深的网络结构。由于上下文建模对于分割任务来说至关重要，最近有一些研究工作着眼于通过空洞卷积或插入注意力模块来增大感受野。然而，基于「编码器-解码器」的 FCN 架构仍然是无法改变的。
如何既能够抽取**全局的语义信息,**又能尽量**不损失分辨率,**一直是语义分割的**难点**。

在本文分钟，作者旨在通过将予以分割作为一种序列到序列的预测任务来提供一种替代方法。具体而言，作者采用了一种纯 Transformer  的架构将图像编码为一个图块序列。通过在 Transformer  的每一层中对全局上下文建模，这种编码器可以与一个简单的解码器组合，从而构建强大的分割模型 SETR。

为了验证模型的性能，作者进行了大量的实验。实验结果标记名，SETR 在 ADE20K、Pascal Context  数据集上都取得了目前最佳的分割性能，并且在 Cityscape 数据集上性能也相当可观。值得一提的是，SETR 在竞争激烈的 ADE20K  竞赛中位列榜首。

本文贡献如下： （1）将图像语义分割任务重新定义为了一个序列到序列的学习问题，对目前主流的编码器解码器 FCN 模型提出了一种替代方案。 （2）探索了将 Transformer 框架用于通过将图像序列化实现全注意力特征表征。



### 解决方法(contributions)

- 用常用于NLP领域的transformer作为Encoder来抽取全局的语义信息(整个过程不损失image分辨率),代替传统FCN的编码部分,从序列-序列学习的角度,为语义分割问题提供了一种新的视角；
- 将图像序列化处理,利用Transformer框架、完全用注意力机制来实现Encoder的功能；
- 提出三种复杂度不同的Decoder结构。

### 整体网络结构

![image-20210916113827638](E:\MarkDown\picture\image-20210916113827638.png)

如下图,模型本质上是一个ViT+Decoder结构。

<img src="E:\MarkDown\picture\v2-15af87e0bba26d2742126e56b94739ef_1440w.jpg" alt="img" style="zoom: 50%;" />

- part1 图像序列化处理

<img src="E:\MarkDown\picture\v2-403ad4559d26389f3c72fa874d15358e_1440w.jpg" alt="img" style="zoom:33%;" />

> 因为NLP中transformation的输入是一维序列,所以需要把图像(H*W*C)转换成1D。
> 法1: 按pixel-wise进行flatten。将输入展平为一个列向量，这样暴力的做法无疑会造成计算量的爆炸,考虑到计算量问题所以此方法不通。
> 法2: 按patch-wise进行flatten。本文采用此方法。

1. 将H*W*3的图像序列化为 256个`H/16*W/16*3`的patch。这样transformer的输入sequence length就是`H/16*W/16` ；
2.  为了对每个切片的空间信息进行编码，可以为每个局部位置都学习一个特定的嵌入`p_i`，并将其添加到一个线性的投影函数`e_i` 中来形成最终的输入序列E = { e1 + p1, e2 + p2, ..., e_L + p_L }(`e_i`是patch embedding,`p_i`是position embedding)

> 整个过程维度变化是：(source image) H × W × C --> (patch) H/16 × W/16 × C --> (transformer sequence) (H × W)/256 × C
> 注：NLP transformation的input维度要求是 L × C。L是序列长度，C是channel-size。

- part2 Transformer

  标准的transformor:

  <img src="E:\MarkDown\picture\v2-a9134defcf8a7b6bbc743f56b1dc60f7_1440w.jpg" alt="标准的transformor" style="zoom: 50%;" />

  这里用到的：

  > 这里linear projection layers一般是指全连接层，用于改变通道维度。具体用处见[详解transformor](https://zhuanlan.zhihu.com/p/48508221)

![img](E:\MarkDown\picture\v2-957aa6a3835fe7c0ea4484c39e3335cb_1440w.jpg)

每个Transformer层由多头注意力、LN层、MLP层构成。其输出结果即`{Z1,Z2,Z3…ZLe}`将上一步得到的`E`输入到24个串联的transformer中,即每个transformer的感受野是整张image。

说明：

1. self-attention输入是一个三元组 (q, k, v)。

2. 三元组(q, k, v)的计算方式如图,其中WQ,WK,WV是随机矩阵(learnable params)，是三个线性投影层的可学习参数。
   ![image-20210908212220159](E:\MarkDown\picture\image-20210908212220159.png)

   ![image-20210908212429683](E:\MarkDown\picture\image-20210908212429683.png)

   ![image-20210908212622329](E:\MarkDown\picture\image-20210908212622329.png)

3. 多头注意力机制是指有多组(q, k, v)矩阵,一组(q, k, v)矩阵代表一次注意力机制的运算,将这多个矩阵拼接起来后再乘以一个参数矩阵WO,即可得出最终的多注意力层的输出。

4. 利用残差思想residual connection得到最终结果。



### 损失函数设计

totalloss = auxiliary loss + main loss
其中main loss为`CrossEntropyLoss` ,auxiliary loss在17年CVPR有提及

https://openaccess.thecvf.com/content_cvpr_2017/papers/Zhao_Pyramid_Scene_Parsing_CVPR_2017_paper.pdf

### 实验

[详看这个的翻译](https://blog.csdn.net/Acmer_future_victor/article/details/115789573)

- **数据集**

在Cityscapes[1]、ADE20K[2]以及PASCAL Context[3]这三个数据集上进行实验评估；

- **实现细节**

基于mmsegmentation框架里面默认的设置（如数据增强和训练策略）：

(1) 先以0.5或2的比例随机resize原图，然后随机裁剪成768、512和480分别应用于上述三个数据集，紧接着执行随机的水平翻转；

(2) 对于Cityscapes数据集，采用的batch size为8；而两外两个数据集ADE20K和PASCAL 则分别采用batch size为8和16的大小迭代训练160k和80k次；

(3) 采用多项式的学习率衰减策略并基于SGD进行训练和优化，其中Momentum和Weight decay分别设置为0.9和0；

(4) 最后，对于上述三个数据集的初始学习率分别设置为0.01、0.001以及0.01.

- **辅助损失**

同PSPNet一样，作者在这里也引入了辅助损失。即监督不同的的层级输出：

(1) Naive upsampling——(Z10; Z15; Z20)；

(2) Progressive UPsampling——(Z10; Z15; Z20; Z24)；

(3) Multi-Level feature Aggregation——(Z6; Z12; Z18; Z24)；

在PSPNet中最终的损失是<img src="E:\MarkDown\picture\image-20210908212932756.png" alt="image-20210908212932756" style="zoom: 67%;" />，这里没有说加权应该是全部直接相加然后计算了。这里采用的是一个2层的（3×3 conv + Synchronized BN + 1×1 conv)网络进行中间层的输出。

- **多尺度测试**

首先将输入图像缩放到一个统一的尺寸，然后执行多尺度的缩放以及随机的水平翻转，尺度缩放因子分别为（0.5，0.75，1.0，1.25，1.5，1.75），紧接着采用滑动窗口的方式进行重叠的切片预测，最后再合并测试结果。如果移动的步长不足以得到一张完整的切片，那么以短边为例保持同等的aspect ratio。其中，由于采用的是多卡的分布式训练，因此Synchronized  BN也被用于解码器和辅助损失头的训练过程中。为了简化训练过程，作者这里并没采用在线困难样本挖掘（OHEM）[4]之类的trick用于模型的训练。

- **基准模型**

采用mmsegmentation中自带的dilated FCN和Semantic FPN。注意到，考虑到计算的瓶颈，最终的FCN是8倍上采样回去，而本文所提出的SETF是进行16倍上采样。

- **SETR变体**

SETR-Naive,  SETR-PUP和SETR-MLA对应上述三种解码器。另外，对于编码器来说，采用的是M层的Transformer，这里根据M的大小划分为"T-Small"和"T-Large"，分别对应12和24层。除非特别说明，本文默认采用的是24层的TF（这样一来就有3*2=6种组合）。初次之外，作者还涉及了一种结合CNN+TF的混合模型，即采用ResNet-50作为预编码器用于初步的特征提取，然后将所提取特征喂入SETR进行进一步的特征提取。为了降低GPU的计算开销，这里ResNet-50将原始输入图像下采样16倍，同时采用SETR-Naive-S的组合。

- **预训练**

作者将**ViT**训练出来的权重用于SETR的编码器进行权重初始化。额，说白了就是把它照搬过来微调了下（白嫖？）。值得注意，这里非常关键的一点是随机初始化和带ViT的预训练权重效果差别这么大：

![img](E:\MarkDown\picture\v2-fd5dcbc4a1523a37a296e484ce15fcea_1440w.jpg)

- **可视化**

![image-20210908213022541](E:\MarkDown\picture\image-20210908213022541.png)

可以看出，在第1层的时候便可以捕获到全局的特征，越往后所提取到的特征越抽象。这足以证明Transformer建立长距离依赖的能力。

### 三、总结

总的来说，本文将Pure  Transformer在自然图像的语义分割任务上进行了首次尝试，整体来说取得的效果是相当不错的。知乎貌似有许多人对其开炮，质疑其创新点不足或者没有放出参数量计算量等亦或是没跟基于自注意力的方法如CC-Net和EMA-Net等比较。然而，我个人的观点的是论文本身可以分为两种，一种是精度型，一种是探索型。大家纠结的原因就是将其归纳为前者，当然这里与作者反复强调在ADE-20k数据集上取得xx成绩也有关，很容易把节奏带进去。为了弥补，作者在题目又强调时Rethinking，即本文只是尝试可以这样做。且不论这个创新性有多强，这其实更应该被当成一篇实验性论文，告诉大家这条路可以走得通。其实，当看到这篇文章的时候，我最关注的地方并不是整体的结构，而是作者是如何将其训练到work的？毕竟这种结构我想绝大多数人都试到吐了，通过整篇文章读下来，才发现要训好这个网络步骤原来这么繁琐，难道笔者基于同样的结构训练一轮下来被直接摁在地下摩擦。最后，很好奇SETF是基于什么样的硬件设施下进行实验的？ 









4.1 

<img src="E:\MarkDown\picture\image-20210909205618303.png" alt="image-20210909205618303" style="zoom:50%;" />

表2  .比较在不同训练前策略和主干上的SETR变体。所有实验都在批次大小为8的城市景观训练精细集上进行训练，并在城市景观验证集上使用单规模测试方案以平均IoU (%)率进行评估。

从表2中，我们可以得出以下结论:(1)逐步对要素地图进行上采样，SETR-PUP在城市景观的所有变量中取得了最佳性能。SETR-MLA性能较差的一个可能原因是，不同transformer层的特征输出不像特征金字塔网络(FPN)那样具有分辨率金字塔的优势(见图5)。然而，SETR-MLA的性能比SETR-PUP稍好，并且比在ADE20K值集上将transformer输出特性一次上采样16倍的变体SETRNaive优越得多(表3和表4)。(2)如预期的那样，使用“T-Large”的变异体(如SETR-MLA和SETR-Naive)优于它们的“T-Base”对应体，即SETR-MLA-Base和SETR-Naive-Base。(3)虽然我们的SETR-PUP-Base比Hybrid-Base表现更差，但当用更多的迭代(80k)训练时，它表现出色(78.02)。这表明FCN编码器的设计可以在语义分割中替代，进一步验证了我们模型的有效性。(4)预培训对我们的模式至关重要。随机初始化的SETR-PUP只给出42.27%的城市景观。在ImageNet-1K上用DeiT [44]预训练的模型在城市景观上的性能最好，略好于在ImageNet-21K上用ViT [16]预训练的模型。(5)为了研究预训练的力量并进一步验证我们建议方法的有效性，我们对表3中的预训练策略进行了消融研究。为了与FCN基线进行公平的比较，我们首先在Imagenet-21k数据集上对ResNet-101进行分类任务的预训练，然后在ADE20K或Cityscapes上对语义分割任务采用扩展FCN训练的预训练权重。

![image-20210909211357801](E:\MarkDown\picture\image-20210909211357801.png)

表3显示，与在ImageNet-1k上预先训练的变体相比，在ImageNet-21k上预先训练的FCN基线有了明显的改善。然而，我们的方法在很大程度上优于FCN的同类方法，验证了我们方法的优势很大程度上来自于提出的序列到序列建模策略，而不是更大的预训练数据。



**4.3 Comparison to state-of-the-art**

<img src="E:\MarkDown\picture\image-20210909213438209.png" alt="image-20210909213438209" style="zoom: 50%;" />

表4显示了我们在更具挑战性的ADE20K数据集上的结果。我们的SETR-MLA在单尺度(SS)推理下获得了48.64%的优越MIoU值。当采用多尺度推理时，我们的方法达到了一个新的水平，MIoU达到了50.28%

![image-20210910200421113](E:\MarkDown\picture\image-20210910200421113.png)

表5.PASCAL Context数据集的最新比较。报告了不同模型变体的性能。SS：单量表推理。MS：多尺度推理。





**Results on Cityscapes**
 表6和表7分别显示了城市景观验证集和测试集的比较结果



<img src="E:\MarkDown\picture\image-20210910200516520.png" alt="image-20210910200516520" style="zoom:67%;" />

<img src="E:\MarkDown\picture\image-20210910200552646.png" alt="image-20210910200552646" style="zoom:67%;" />



### 评价

[原文](https://www.zhihu.com/question/437479751/answers/updated)

全文没一处提及计算量，参数量及速度。不过好在直接继承VIT，根据FB最新的deit里给的数据，在imagenet小图224的数据下

![image-20210910214008615](E:\MarkDown\picture\image-20210910214008615.png)

![image-20210910214036540](E:\MarkDown\picture\image-20210910214036540.png)



## 复现

### 1)官方mmcv代码

[官方code(基于MMSegmentation框架)](https://github.com/fudan-zvg/SETR) 

[模型试跑](https://blog.csdn.net/qq_33642342/article/details/118734271?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163116874416780265472402%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163116874416780265472402&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v29_ecpm-14-118734271.pc_search_ecpm_flag&utm_term=SETR&spm=1018.2226.3001.4187)  [预训练权重仓库](https://github.com/open-mmlab/mmsegmentation/tree/master/configs/setr)

[mmcv讲解](https://zhuanlan.zhihu.com/p/126725557)   [2](https://zhuanlan.zhihu.com/p/336081587)

pip install mmcv-full==1.3.2 -f https://download.openmmlab.com/mmcv/dist/{cu_version}/{torch_version}/index.html

安装出现问题，版本不对



### 2)第三方代码无训练程序

[第三方复现](https://github.com/gupta-abhay/setr-pytorch)       [采用Segmentation Transformer（SETR）（Pytorch版本）训练CityScapes数据集步骤](https://blog.csdn.net/qq_41964545/article/details/116542528)



# 二、其他transformor网络

## 1.TransUNet

也是transformor的语义分割，结合了Unet，用于医学领域

作者单位：JHU, 电子科大, 斯坦福大学等s
代码：[Beckschen/TransUNet](https://link.zhihu.com/?target=https%3A//github.com/Beckschen/TransUNet)
论文：[https://arxiv.org/abs/2102.04306](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2102.04306)

[训练步骤](https://blog.csdn.net/qq_20373723/article/details/115548900)

![image-20210909215003452](E:\MarkDown\picture\image-20210909215003452.png)



## 2.Swin-Unet

Swin-Unet: Unet-like Pure Transformer for Medical Image Segmentation

单位：慕尼黑工业大学, 复旦大学, 华为(田奇等人)
代码：[https://github.com/HuCaoFighting/Swin-Unet](https://link.zhihu.com/?target=https%3A//github.com/HuCaoFighting/Swin-Unet)
论文下载链接：[https://arxiv.org/abs/2105.05537](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2105.05537)

![img](E:\MarkDown\picture\v2-a97af0fff600f13cda5a5dd97f4eeada_b.jpg)









![image-20210910100226054](E:\MarkDown\picture\image-20210910100226054.png)

