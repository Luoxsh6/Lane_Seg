## 简介
数据集来自百度2019年的[车道线识别挑战赛](https://aistudio.baidu.com/aistudio/competition/detail/5)，参考了初赛第一名的作者的[paddlepaddle版代码](https://github.com/gujingxiao/Lane-Segmentation-Solution-For-BaiduAI-Autonomous-Driving-Competition)（后面称为`baseline`）。

在训练集上划分出来的验证集上，最后得到的miou是0.684。

## 特色
1. 对unet做了一些抽象，可以较容易地替换backbone成其他的，比如不同版本的resnet。见`nets/unet.py`。
2. 针对unet的输出尺寸跟输入尺寸不一致的问题做了微调，使其一致。好处是，跑出来的结果可以直接用，不必像baseline那样在最后要对结果做人工修正。
3. 利用训练的过程中miou的异常下降筛查出一部分有问题的训练数据，并将这些脏数据去掉后，重新训练，效果得到比较大的提升。相关的逻辑可以在代码里搜`LOG_SUSPICIOUS_FILES`。脏数据主要包括：标签错误，标签缺失，原图过曝（百度比赛说明里提到，车道线的标记是用其他方式得到的，并非在原图上得到，所以会出现原图压根看不见车道线，但标签里却标出来了）。
4. 参考了baseline的思路，对原图裁掉了上侧小半部分，因为这些地方都是天空和树木这种背景，裁掉后还可以减小图片，使得在有限的显存下能训练得下去。
5. 参考了baseline的思路，先后将裁剪后的原图缩小到3种分辨率（768x256, 1024x384, 1536x512）进行训练，比如在768x256下训练了十来个epoch后，将最后的权重作为预训练权重，在1024x384的分辨率上重新训练。由于本人训练时的显存只有15G，所以对应这三种分辨率的batchsize分别是8/4/2。
6. 对原图进行分辨率缩放时，采用的是双线性插值模式（cv2.INTER_LINEAR）。但对于标签图进行缩放时，采用的是最近邻模式（cv2.INTER_NEAREST），因为标签的像素值有特殊的含义，如果用双线性插值的话，缩放后会出现约定范围之外的像素值。相应地，对于推断结果，即预测的标签图，需要恢复成原图尺寸，也要采用最近邻模式。见`utils/data.py`


## 文件说明

1. 配置在`config.py`
2. data_list/bad.csv 是初赛训练数据里要剔除的数据
3. data_list/train.csv 是初赛训练数据里作为训练集使用的
4. data_list/val.csv 是初赛训练数据里作为验证集使用的

## 预测效果
[![3TwsrF.md.png](https://s2.ax1x.com/2020/03/05/3TwsrF.md.png)](https://imgchr.com/i/3TwsrF)
[![3Tw25R.md.png](https://s2.ax1x.com/2020/03/05/3Tw25R.md.png)](https://imgchr.com/i/3Tw25R)
[![3TwIKO.md.png](https://s2.ax1x.com/2020/03/05/3TwIKO.md.png)](https://imgchr.com/i/3TwIKO)
