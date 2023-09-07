配置文件

![image-20221201165755405](E:\MarkDown\picture\image-20221201165755405.png)

## demo

mul_top_down_img_demo_with_mmdet为自写的输入文件夹的演示。这里默认隐藏了box

```cpp
python demo/mul_top_down_img_demo_with_mmdet.py     demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py     checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth     configs/top_down/hrt/coco/hrt_small_coco_256x192.py     checkpoints/hrt_small_coco_256x192.pth     --img-root test3/     --out-img-root res2
```

训练要加横杠，测试时不加

![image-20221123231142769](E:\MarkDown\picture\image-20221123231142769.png)

![image-20221123231229617](E:\MarkDown\picture\image-20221123231229617.png)

![image-20221123231128701](E:\MarkDown\picture\image-20221123231128701.png)



基于HRFormer的demo演示

标注的关键点和连接线：

mmpose/apis/inference.py/def vis_pose_result函数中可更改

gpu在这改

![image-20221123230622983](E:\MarkDown\picture\image-20221123230622983.png)

box颜色粗细在这改

![image-20221123221426950](E:\MarkDown\picture\image-20221123221426950.png)

box在这隐藏

![image-20221123221915618](E:\MarkDown\picture\image-20221123221915618.png)

关键点半径颜色在这改

![image-20221123225130564](E:\MarkDown\picture\image-20221123225130564.png)





显式的阈值：

只有这里改有用

改高了后更容易出现关键点缺失。默认是0.3

![image-20221123215331444](E:\MarkDown\picture\image-20221123215331444.png)

这些地方没用

![image-20221123214001786](E:\MarkDown\picture\image-20221123214001786.png)

![image-20221123214049338](E:\MarkDown\picture\image-20221123214049338.png)



![image-20221123210903323](E:\MarkDown\picture\image-20221123210903323.png)

## 一、论文代码

### RSN

exps->RSN18.coco->network.py
class Bottleneck(nn.Module)中包含了RSB模块

### UDP

deep-high-resolution-net.pytorch中yaml
yaml首先可以将全部参数都设置一个默认值，比如网络的层数，激活函数用哪个等等，大多是模型内相关的参数以及train和test使用的数据的地址；argparse通常设置几个train和test时经常更改的参数，比如训练的epoch，batch_size，learning_rate...



target_type = 'GaussianHeatmap'



#### mmpose中HRNet加上UDP后区别

```python
1、20行，高斯heatmap
target_type = 'GaussianHeatmap'

2、1行，将model部分中的shift_heatmap改为False，增加target_type=target_type，增加use_udp=True

3、113行，将train_pipeline中的 dict(type='TopDownGenerateTarget', sigma=2),改为dict(

​    type='TopDownGenerateTarget',

​    sigma=2,

​    encoding='UDP',

​    target_type=target_type),

4、相应改了val_pipeline
```

dark与UDP

```
UDP多了个target_type = 'GaussianHeatmap'

74行，dark:post_process='unbiased',

dark110
dict(type='TopDownGenerateTarget', sigma=2, unbiased_encoding=True),
udp:
dict(
        type='TopDownGenerateTarget',
        sigma=2,
        encoding='UDP',
        target_type=target_type),
```









### HRNet

——experiments

​    ——coco

​        ——hrnet
​        w32_256x192_adam_lr1e-3.yaml，存放GPU、Model、训练、测试等配置信息

——lib
    ——config
    default.py，存放了些关于训练和测试的默认参数，有“_c.”前缀
    models.py，resnet和high_resolution_net的一些默认参数

​    ——core

​        ——evaluate.py，计算的是PCK？？？？

​        ——function.py，训练和验证一个batch的核心代码 。validate函数中all_preds的shape为(样本数、关节点数17、3)，all_boxes为(样本数、6)（（（注意，这里数据集提供的为矩形框的左上角坐标和矩形框的长宽，所以为什么为6？

output

其中从valid_dataset获得i、(input, target, target_weight, meta)等数据

​        ——[inference.py](https://zhuanlan.zhihu.com/p/141716039?from_voters_page=true)，测试与后处理（关节点坐标解码）的代码

​        ——loss.py，损失函数MSE Loss

​    ——dataset

​        ——coco.py //COCO API的一些代码 class COCODataset(JointDataset) 

​        ——JointsDataset.py //class JointsDataset(Dataset)

​    ——models

​        ——pose_hrnet.py：完成功能构建hrnet网络架构

├── tools

├── test.py //测试代码 

├── train.py //训练代码

├── visualization

└── plot_coco.py //结合COCO API进行pose可视化



包含
1.基础框架：
1）resnet网络的两种形式：class BasicBlock(nn.Module)
                       class Bottleneck(nn.Module)
2）关键部分：高分辨率模块：class HighResolutionModule(nn.Module)
            该模块的重要函数：def _make_one_branch
                            def _make_branches
                            def _make_fuse_layers
2.关键点预测模块【完整的网络】： class PoseHighResolutionNet(nn.Module)
            该模块的重要函数：def _make_transition_layer
                            def _make_layer
                            def _make_stage

 

## 二、mmdetection

[官方文档](https://mmdetection.readthedocs.io/zh_CN/v2.21.0/)

[中文解读文案汇总](https://github.com/open-mmlab/mmdetection/blob/master/docs/zh_cn/article.md)

安装

git clone https://github.com/open-mmlab/mmdetection.git
cd mmdetection

pip install -r requirements/build.txt
pip install -v -e .  or "python setup.py develop"

## 三、mmpose



### 安装指定版本mmcv-full
先在[这里](https://download.openmmlab.com/mmcv/dist/cu102/torch1.8.0/index.html)下载指定版本
再运行pip install /dataset/wh/wh_code/HRFormer-main/pose/mmcv_full-1.3.9-cp37-cp37m-manylinux1_x86_64.whl



MMpose、mmcv、mmdet关系

需要先安装mmcv，之后再安装mmpose，mmdet是可选项

OpenMMLab 系列是专为计算机视觉不同方向建立统一而开放的代码库。目前已经开源了多个算法框架，如物体检测的 MMDetection、姿态估计的 MMPose，而 MMCV 是上述一系列上层框架的基础支持库，目前实现的功能也是非常丰富。

mmcv安装的时候，pip install mmcv-full -f https://download.openmmlab.com/mmcv/dist/cu101/torch1.11.0/index.html，注意查看环境中的torch版本。

[mmpose安装](https://zhuanlan.zhihu.com/p/341722982)

[mmpose基础教程](https://mmpose.readthedocs.io/zh_CN/latest/getting_started.html#)

[mmpose github官网](https://github.com/open-mmlab/mmpose)

[mmdetection github](https://github.com/open-mmlab/mmdetection)



conda activate open-mmlab
cd mmcv/mmpose

### 数据集
放在mmcv/mmpose/data下，现放入coco，其中train2017和val2017为img，annotations为标记，person_detection_results为人体检测框

### 测试：
在mmcv/mmpose/checkpoints文件夹内放入.pth文件[模型池](https://mmpose.readthedocs.io/zh_CN/latest/topics/body%282d%2Ckpt%2Csview%2Cimg%29.html)，在configs/body里找到对应数据集、对应模型的.py文件，对下列语句修改后运行

测试自己训练的：
./tools/dist_test.sh configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w32_coco_256x192_udp.py work_dirs/hrnet_w32_coco_256x192_udp/epoch_60.pth 3 --eval mAP

```
./tools/dist_test.sh configs/body/2d_kpt_sview_rgb_img/associative_embedding/coco/res50_coco_512x512.py checkpoints/res50_coco_512x512-5521bead_20200816.pth 4 --eval mAP
```

具体细节见[基础教程](https://mmpose.readthedocs.io/zh_CN/latest/getting_started.html)(如更换数据集的话--eval的指标要改成别的)



```
/home/celia/anaconda3/envs/open-mmlab/lib/python3.7/site-packages/torch/distributed/launch.py:186: FutureWarning: The module torch.distributed.launch is deprecated
and will be removed in future. Use torchrun.

  FutureWarning,
WARNING:torch.distributed.run:


UserWarning: __floordiv__ is deprecated, and its behavior will change in a future version of pytorch. It currently rounds toward 0 (like the 'trunc' function NOT 'floor'). This results in incorrect rounding for negative values. To keep the current behavior, use torch.div(a, b, rounding_mode='trunc'), or for actual floor division, use torch.div(a, b, rounding_mode='floor').
  y = ind // W


上述UserWarning目前没找到解决方法，若想不报错就加上这句
import warnings
warnings.filterwarnings("ignore")


[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>] 5000/5000, 4.7 task/s, elapsed: 1055s, ETA:     0s/home/celia/anaconda3/envs/open-mmlab/lib/python3.7/site-packages/json_tricks/encoders.py:367: UserWarning: json-tricks: numpy scalar serialization is experimental and may work differently in future versions
  warnings.warn('json-tricks: numpy scalar serialization is experimental and may work differently in future versions')
```

### demo

**测试单张图片，用了mmdet先提取检测框，再用自己训练的模型进行测试**

```
python demo/top_down_img_demo_with_mmdet.py \
    demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py \
    checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
    configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w32_coco_256x192_udp.py \
    work_dirs/hrnet_w32_coco_256x192_udp/epoch_60.pth \
    --img-root tests/data/wh/ \
    --img 11.jpg \
    --out-img-root vis_results
```

下载模型测试单张图片

```
python demo/top_down_img_demo_with_mmdet.py \
    demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py \
    https://download.openmmlab.com/mmdetection/v2.0/faster_rcnn/faster_rcnn_r50_fpn_1x_coco/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
    configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w48_coco_256x192.py \
    https://download.openmmlab.com/mmpose/top_down/hrnet/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth \
    --img-root tests/data/coco/ \
    --img 000000196141.jpg \
    --out-img-root vis_results
```



**测试单张图片**：(这里用了人工标注的人体框test_coco.json作为输入)

```
python demo/top_down_img_demo.py \
    configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w48_coco_256x192.py \ https://download.openmmlab.com/mmpose/top_down/hrnet/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth \
    --img-root tests/data/coco/ --json-file tests/data/coco/test_coco.json \
    --out-img-root vis_results
```

文件会自动下载到/home/celia/.cache/torch/hub/checkpoints/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth

**测试视频**：（用full_frame整个框

```bash
python demo/top_down_video_demo_full_frame_without_det.py configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w48_coco_256x192.py checkpoints/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth --video-path demo/VID20211224121627.mp4 --out-video-root vis_results
```

**mmdet视频演示**（用mmdet检测框

```
python demo/top_down_video_demo_with_mmdet.py \
    demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py \
    https://download.openmmlab.com/mmdetection/v2.0/faster_rcnn/faster_rcnn_r50_fpn_1x_coco/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
    configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w48_coco_256x192.py \
    https://download.openmmlab.com/mmpose/top_down/hrnet/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth \
    --video-path demo/resources/demo.mp4 \
    --out-video-root vis_results
```

> （注意，这些例子都是自上而下的



### 训练

训练参数在hrnet_w48_coco_256x192.py 中更改
<img src="E:\MarkDown\picture\image-20220320143729284.png" alt="image-20220320143729284" style="zoom:70%;" />
训练好的文件放在mmcv/mmpose/work_dirs/中
--resume-from work_dirs/hrnet_w48_coco_256x192/latest.pth为加载模型权重文件，加载 model weights 和 optimizer status, and the epoch 也在这个checkpoint里. 一般用于恢复先前中断的训练.
`load-from` 只加载 model weights，training epoch 从 0 开始. 一般用于 finetuning.



















#### UDP训练

./tools/dist_train.sh configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w32_coco_256x192_udp.py 3 --resume-from work_dirs/hrnet_w32_coco_256x192_udp/latest.pth

第一个epoch10分钟

UDP源代码中训练参数：

WORKERS: 12
BATCH_SIZE_PER_GPU: 32
LR: 0.001

```
python tools/analysis/print_config.py configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w32_coco_256x192_udp.py
```



![image-20220320141147969](E:\MarkDown\picture\image-20220320141147969.png)



epoch60结果
![image-20220321111600283](E:\MarkDown\picture\image-20220321111600283.png)



### Benchmark

可以使用下面命令，获得平均推导时间；但不包括 IO time 和 pre-processing time 。

python tools/benchmark_inference.py configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w32_coco_256x192_udp.py



### error

1、[Torch 1.11版本在用mmpose进行训练时，报错：AttributeError: 'MMDistributedDataParallel' object has no attribute '_sync_params'](https://github.com/open-mmlab/mmcv/pull/1816/commits/f6cc614b57b9949fa48533000d6942f82b6dfd4a)

2、训练时报错：RuntimeError: The server socket has failed to listen on any local network address. The server socket has failed to bind to [::]:29500 (errno: 98 - Address already in use). The server socket has failed to bind to 0.0.0.0:29500 (errno: 98 - Address already in use).

因为有人在用gpu训练，所以报错。

3、client_loop: send disconnect: Connection reset

> 这个方法还没有试过，要是再次出现这个问题再试试

通过`ssh`连上服务器后，一段时间不操作，就会自动中断，并报错。

配置`~/.ssh/config`文件，增加以下内容即可：

```text
Host *         
         # 断开时重试连接的次数         
         ServerAliveCountMax 5          
         # 每隔5秒自动发送一个空的请求以保持连接         
         ServerAliveInterval 60
```

添加在`/etc/ssh/ssh_config`应该也是可以的。



# 不错的代码

累加器

```python
class Accumulator:
    #在n个变量上累加
    def __init__(self, n):
        self.data = [0.0] * n  # 根据传进来的n的大小来创建n个空间，且初始化全部为0.0
    def add(self, *args):
        for a, b in zip(self.data, args):  # zip() 函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。这里将n个为0的空间与输入对应的像素打包为一个个元组。然后用a、b一个元组一个元组的返回
        self.data = [a + float(b) for a,b in zip(self.data, args)]
        # 把原来类中对应位置的data和新传入的args做 a + float(b)加法操作然后重新赋给该位置的data。从而达到累加器的累加效果。
 
    def reset(self):
        self.data = [0.0] * len(self.data)# 重新设置空间大小并初始化
 
    def __getitem__(self, idx):
        return self.data[idx]
```

使用实例：2

```python
def evaluate_loss(net,data_iter,loss):
    metric=d2l.Accumulator(2)
    for X,y in data_iter:
        out=net(X)
        y=y.reshape(out.shape)
        l=loss(out,y)
        metric.add(l.sum(),l.numel())
    return metric[0]/metric[1]
```

​         











