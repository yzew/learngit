[qkv理解](https://www.zhihu.com/question/325839123/answer/2718310467?utm_campaign=&utm_medium=social&utm_oi=864942345865560064&utm_psn=1574947170024837120&utm_source=qq)

self-attention，计算几个输入之间的相关性

将query和每个key进行相似度计算得到权重，然后softmax函数归一化

![image-20221028153956707](E:\MarkDown\picture\image-20221028153956707.png)

![image-20221028153647736](E:\MarkDown\picture\image-20221028153647736.png)

![image-20221027214552862](E:\MarkDown\picture\image-20221027214552862.png)

multi-head attention就相当于去做一个模型融合



Transformer架构中混合两种不同嵌入序列的注意机制

一个序列作为输入的Q，定义了输出的序列长度，另一个序列提供输入的K&V