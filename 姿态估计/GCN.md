[图卷积的一些推导](https://blog.csdn.net/weixin_45901519/article/details/106388964)

图是表示一些点之间的关系

![image-20221106195703385](E:\MarkDown\picture\image-20221106195703385.png)

 





![image-20221106200315815](E:\MarkDown\picture\image-20221106200315815.png)





图是一个稀疏的架构，动态结构的计算是比较难的；超参数的优化是比较难的





深度学习一直都是被几大经典模型统治着，常见的有CNN、RNN网络,它们在CV和NLP领域
都取得了优异的效果。但人们发现了很多CNN、RNN无法解决，或者效果不好的问题--图
结构数据，如社交网络I人物关系、分子结构等，所以就有了GNN网络(Graph Neural
Networks)。

图卷积神经网络(Graph Convolutional Network, GCN),是一类采用图卷积形式迭代的神经网
络，属于图神经网络GNN的一种。





![image-20221106213039464](E:\MarkDown\picture\image-20221106213039464.png)



![image-20221106213358329](E:\MarkDown\picture\image-20221106213358329.png)

邻接矩阵表示顶点之间的连通性，度矩阵表示一个节点和几个节点相连，也相当于邻接矩阵每一行的和；特征矩阵就是每个节点的向量

![image-20221106213735540](E:\MarkDown\picture\image-20221106213735540.png)

通过度矩阵，防止A~和X相乘后的矩阵无限变大

![image-20221106213932762](E:\MarkDown\picture\image-20221106213932762.png)

![image-20221106214116434](E:\MarkDown\picture\image-20221106214116434.png)





但存在一个问题，就是那些相连节点的影响都比较大，比如这里E的影响，因为和每个顶点都相连，每个顶点的计算中都会加上E，因此E占总体的比重很大，因此我们要标准化

![image-20221106214220656](E:\MarkDown\picture\image-20221106214220656.png)



![image-20221106214319783](E:\MarkDown\picture\image-20221106214319783.png)



![image-20221106214335036](E:\MarkDown\picture\image-20221106214335036.png)



![image-20221106214526712](E:\MarkDown\picture\image-20221106214526712.png)



ReLU里面是第一层，整体是第二层，W是我们网络要学习的东西

图结构是根据相连关系自己构建的，因此我们用GCN的时候需要根据结构关系先创建出个图模型来，这里注意比如双腿，可以设成关联的。创建图结构的库是networkxZ`

















