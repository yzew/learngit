官方代码基于mmcv

## Swin Transformer

Swin Transformer: Hierarchical Vision Transformer using Shifted Windows

使用移动窗口的分层版本的Transformer

微软亚洲研究院，三月发布

[paper](https://arxiv.org/abs/2103.14030v1?ref=hackernoon.com)

[大师兄详解](https://zhuanlan.zhihu.com/p/361366090)  [2](https://zhuanlan.zhihu.com/p/401661320)  [3](https://zhuanlan.zhihu.com/p/367111046)

Swin  Transformer的最大贡献是提出了一个可以广泛应用到所有计算机视觉领域的**backbone**，并且大多数在CNN网络中常见的超参数在Swin  Transformer中也是可以人工调整的，例如可以调整的网络块数，每一块的层数，输入图像的大小等等。
在Swin  Transformer之前的**ViT**和**iGPT**，它们都使用了**小尺寸**的图像作为输入，这种直接resize的策略无疑会损失很多信息。与它们不同的是，Swin Transformer的输入是图像的**原始尺寸**，例如ImageNet的224*224。另外Swin  Transformer使用的是CNN中最常用的层次的网络结构，在CNN中一个特别重要的一点是随着网络层次的加深，节点的感受野也在不断扩大，这个特征在Swin Transformer中也是满足的。Swin  Transformer的这种层次结构，也赋予了它可以像FPN，U-Net等结构实现可以进行分割或者检测的任务。





