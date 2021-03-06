---
layout: post
title:  深度学习（十九）——Mask R-CNN
category: DL 
---

# 语义分割的展望

俗话说，“没有免费的午餐”（“No free lunch”）。基于深度学习的图像语义分割技术虽然可以取得相比传统方法突飞猛进的分割效果，但是其对数据标注的要求过高：不仅需要海量图像数据，同时这些图像还需提供精确到像素级别的标记信息（Semantic labels）。因此，越来越多的研究者开始将注意力转移到弱监督（Weakly-supervised）条件下的图像语义分割问题上。在这类问题中，图像仅需提供图像级别标注（如，有“人”，有“车”，无“电视”）而不需要昂贵的像素级别信息即可取得与现有方法可比的语义分割精度。

另外，示例级别（Instance level）的图像语义分割问题也同样热门。该类问题不仅需要对不同语义物体进行图像分割，同时还要求对同一语义的不同个体进行分割（例如需要对图中出现的九把椅子的像素用不同颜色分别标示出来）。

![](/images/article/Instance_level.jpg)

此外，基于视频的前景／物体分割（Video segmentation）也是今后计算机视觉语义分割领域的新热点之一，这一设定其实更加贴合自动驾驶系统的真实应用环境。

最后，目前使用的分割模型在对分割注释有限的大型概念词汇的识别方面表现欠佳。原因在于它们忽略了所有概念的固有分类和语义层次。例如，长颈鹿、斑马和马同属于有蹄类动物，这个大类描绘了它们的共同视觉特征，使得它们很容易与猫/狗区分开来。对人来说，即使你没见过斑马，但也不会把它错误的认成猫/狗。但目前的DL模型在这方面的能力还很薄弱。

参考：

https://mp.weixin.qq.com/s/palhFeMnWOZj-T2cqQN7tw

新型语义分割模型：动态结构化语义传播网络DSSPN

# Mask R-CNN

Mask R-CNN虽然挂着R-CNN的名头，但却是一个对象实例分割（不仅要分出对象的类别，连同一类对象的不同实例也要分出来）的网络。它是何恺明2017年的新作。

论文：

《Mask R-CNN》

代码：

官方Pytorch版本：

https://github.com/facebookresearch/maskrcnn-benchmark

TensorFlow版本：

https://github.com/hillox/TFMaskRCNN

MXNet版本：

https://github.com/TuSimple/mx-maskrcnn

![](/images/img2/mask_rcnn.png)

上图是Mask R-CNN的网络结构图。它实际上就是在Faster R-CNN的基础上添加了一个FCN。

![](/images/img3/Mask_R-CNN.png)

上图也是Mask R-CNN的网络结构图，但它对Faster R-CNN中与本主题无关的部分做了省略。

Mask R-CNN的要点主要有：

- **RoI Align**

RoI Align避免对RoI的边界或者块（bins）做任何量化，例如直接使用x/16代替[x/16]。

然而这就引来一个问题：如果x/16不是整数该怎么采样呢？

答案：对临近的整数采样点，使用双线性插值（bilinear interpolation）拟合，得到非整数采样点的值。

![](/images/img3/ROI_Align.png)

- **独立的类别预测**

把loss由`tf.nn.softmax_cross_entropy_with_logits`换成`tf.nn.sigmoid_cross_entropy_with_logits`。参见[《深度目标检测（五）》](/deep object detection/2018/12/01/Deep_Object_Detection_5.html#YOLOv3)的YOLOv3一节。没错，YOLOv3借鉴了Mask R-CNN的这一设计思路。

- **对象实例分割**

Mask R-CNN只对RoI Align后的区域进行分割，而不像U-NET等会对全景进行分割。因此，更适合抠图之类的应用。

Mask R-CNN除了可以用于实例分割以外，还可用于关键点检测。这点在原始论文和FB的代码中有体现，但是在通常的介绍中往往被忽略。

![](/images/img3/Mask_R-CNN_keypoint.png)

keypoint branch的输出结果是一个keypoint的heatmap（每个keypoint都有自己的heatmap），显然，heatmap中值最大的点就是keypoint的所在。

当然ROI区域和原图，无论是坐标，还是尺寸，都有差异，需要通过插值恢复回去。Mask R-CNN使用的是bicubic插值，该方法计算量较大，因此实际中，多采用下文所述的基于Taylor展开的方法进行插值。

《Invariant Features from Interest Point Groups》

参考：

https://zhuanlan.zhihu.com/p/25954683

Mask R-CNN个人理解

https://mp.weixin.qq.com/s/E0P2B798pukbtRarWooUkg

Mask R-CNN的Keras/TensorFlow/Pytorch代码实现

https://zhuanlan.zhihu.com/p/30967656

从R-CNN到Mask R-CNN

https://www.zhihu.com/question/57403701

如何评价Kaiming He最新的Mask R-CNN?
