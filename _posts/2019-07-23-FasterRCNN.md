---
title: Faster R-CNN阅读
date: 2019-07-23 22:00:00
categories: Object_Detection
tags: Object_Detection Faster_R-CNN
mathjax: true
---

**重拾Faster R-CNN**   
之前阅读过这篇文章，但是阅读的比较泛，也没有很好地去记录，最近需要用到Faster R-CNN，就想着再次阅读一次，主要是简单的翻译和总结，一来可以加深对文章某些细节的理解，二来可以做一些记录，方便以后查阅。


**文章地址：**    
地址1：arxiv：[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](https://arxiv.org/abs/1506.01497)   
地址2：nips：[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](http://papers.nips.cc/paper/5638-faster-r-cnn-towards-real-time-object-detection-with-region-proposal-networks.pdf)
代码：[https://github.com/rbgirshick/py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn)

# 摘要
先进的目标检测算法依赖于区域提取算法来假设目标的位置。SPPNet和Fast R-CNN等先进的算法降低了检测网络的运行时间，但是区域提取部分的计算也是一个瓶颈。本文中，我们提出了一个与检测网络共享卷积特征的区域提取网络(Region Proposal Network，RPN)，使得几乎无代价的区域提取成为可能。RPN是一个全卷积网络，同时预测每个位置的目标位置和置信度。RPN可以端到端的训练，可以产生高质量的候选区域，用来给Fast R-CNN进行检测。通过共享RPN和Fast R-CNN的基础网络，我们将这两个网络合并成一个统一的网络，使用最近流行的“注意力”机制，RPN告诉网络在何处寻找目标。对于使用VGG-16作为基础网络，我们的检测系统在GPU上的帧率为5fps，同时在PASCAL VOC 2007, 2012, and MS COCO数据集上获得了最先进的目标检测精度，我们的RPN网络仅仅提供300个候选区域。在ILSVRC和COCO 2015的比赛中，Faster R-CNN和RPN是在几个轨道上获得第一名的基础。代码已经公开。

# 1 引言
**这部分简读**    
（基于区域建议的方法的瓶颈）区域建议的方法和基于区域提取的CNN推动了目标检测领域的发展。现在区域提取部分是测试阶段的速度瓶颈。   
（一种解决方法）把算法从CPU迁移导致GPU上，但是这样做会忽略了对网络底层的特征的利用，错过了共享参数的机会。   
本文的方法：本文中，我们证明了基于深度卷积神经网络的区域提取方法是一个很好的解决方案，相对于检测网络的运行时间，其运行时间几乎是可以忽略的。区域提取的成本很小，10ms每幅图。   
共享卷积特征的可能性：可以在卷积特征图上加入了模块进行区域提取，不影响原来的结构。可以根据区域建议任务进行端到端的训练。   
（RPN的优点）RPN的设计是为了广泛提取具有不同尺寸和纵横比的候选区域，与常用的方法[8], [9], [1], [2]不同，它们使用图像金字塔或者滤波器金字塔等策略，我们引入了新的“anchor”框，可以作为多种尺度和纵横比的参考。我们的方案可以被认为是a pyramid of regression references，它避免了枚举多个尺度或纵横比的图像或过滤器。当使用单尺度图像进行训练和测试时，该模型性能良好，提高了运行速度。    
（训练策略）为了将RPN与Fast R-CNN统一起来，我们设计了一种训练方法，交替对RPN和Fast R-CNN进行微调，这种方法收敛速度快，可以让RPN与Fast R-CNN共享基础网络。     
（实验结果优秀）PASCAL VOC的结果精度和速度都最好。第一版文章发布后，基于RPN和Faster R-CNN，出现了很多优秀的算法，如3D object detection [13], part-based detection [14], instance segmentation [15], and image captioning [16]。其他比赛中也取得了优异的性能。这些结果表明，我们的方法不仅速度快，而且精度高。

# 2 相关工作
**这部分简读**    
**候选目标区域Object Proposals**   
关于候选目标区域有大量的文献资料，可以参考这几篇综述[19], [20], [21].广泛使用的候选区域提取方法基于grouping super-pixels (e.g., Selective Search [4], CPMC [22], MCG [23]) and sliding windows(e.g., objectness in windows [24], EdgeBoxes [6]).的方法。候选目标区域的方法作为独立于检测器的外部模块，比如选择性搜索（Selective Search）检测器，R-CNN和Fast R-CNN。   
**基于深度网络的目标检测**   
R-CNN的方法端到端地训练CNN，将候选区域分为目标或者背景类别。R-CNN主要作为一个分类器，不进行候选区域的预测，除了对边框进行精细回归。有文章[20]说明，它的性能主要取决于候选区域模块的性能。目前已经有一些文章[25], [9], [26], [27]使用深度网络来预测候选区域。在OverFeat[9]中，使用全连接层来为定位任务预测边框，假定图像中只有一个目标。全连接层后来转变成卷积层用来检测多类目标。在MultiBox方法[26],[27]中，通过网络最后一层全连接层同时预测多个类无关的候选边框，推广了OverFeat中的单一边框的方法。这些类别无关的边框可以被用作R-CNN的候选区域。DeepMask method [28] 也是基于我们的方法。   
共享卷积计算因其高效和高性能，在视觉识别领域受到了越来越广泛的关注。OverFeat使用图像金字塔进行检测。SPPNet中在共享的特征图上进行自适应的池化。


# 3 Faster R-CNN
我们的目标检测器称为Faster R-CNN，分为两个该部分。第一部分是全卷积网络，用来生成候选区域，第二部分是Fast R-CNN检测器。整个网络是一个独立统一的结构。3.1我们介绍RPN的设计和特性，3.2介绍训练共享卷积特征的两部分的训练算法。

**3.1 RPN**   
（RPN）RPN将整个图像输入网络，输出一系列矩形候选区域，以及它们的置信度。我们使用一个全卷积网络来实现这个过程。因为我们最终要将RPN与Fast R-CNN共享计算，因此我们假定两个网络共享一些卷积层。实验中，我们采用ZF Net作为基础网络，共享5个卷积层；使用VGG作为基础网络，共享13个卷积层。   
（RPN结构）为了生成候选区域，我们在最后一个卷积层输出的特征图上滑动一个小的网络，这个小网络将nxn大小的窗口作为输入的卷积特征图。通过小网络，每一个滑动窗口的特征都映射到一个低维的特征，每个特征被送入两个滑动的全连接层，一个边框回归层和一个边框分类层。考虑到这一层对应于输入图像的感受野较大，我们将n设置为3。图3表示了这个小网络的位置。因为这个小网络的计算是滑窗进行的，全连接层在不同位置上是共享的，所以这部分网络是通过一个nxn的卷积层和两个1x1的卷积层实现的。

**3.1.1 Anchors**   
（anchor个数，RPN输出节点个数）在每一个滑窗位置，我们同时预测多个候选区域，我们将每个位置最大可能的候选区域个数设置为k。因此回归层的输出节点数为4k，编码k个区域的位置；分类层的输出节点数为2k，用来预测每个候选区域是否为目标类。k个候选区域对应k个预选框，我们称之为anchor。anchor是以滑动窗口为中心，并且与一个尺寸和纵横比关联。我们默认设置了3个不同的尺寸和3个不同的纵横比，这样会在每个滑动窗口位置产生k=9个anchor。假设特征图的尺寸为WxH，那总共有WxHxk个anchor。   
* **Translation-Invariant Anchors**   
（平移不变的anchor）我们的方法的一个重要特性是平移不变性，包括anchor的计算和候选区域计算的相关函数。如果输入图像中的一个目标被移动了，那么候选区域也应该平移，相同的函数应该能计算不同位置的候选区域。相比之下，MultiBox方法使用k-means产生800个左右的候选区域，并不是具有平移不变性的。   
平移不变性也减少了网络的参数量。MultiBox具有(4+1)*800维度的全连接层输出层，我们的方法只需要(4+2)*9维度的卷积输出层。在网络参数量方面，我们的方法也具有。如果考虑到特征投影层，我们的方法依然比MultiBox方法的参数量少一个数量级。   
* **Multi-Scale Anchors as Regression References**   
（多尺度anchors作为回归参考）我们anchors的设计提出了一种新颖的解决多尺度和多纵横比的方案。如图1所示，多尺度预测主要有两种解决方案。第一种是基于图像/特征金字塔，比如 DPM [8] 和基于CNN的方法 [9], [1], [2]，这种方法将图像resize到不同的尺寸，通过网络得到不同尺寸的特征图，这种方法有效，但是十分耗时。第二种是在特征图上使用不同尺寸或者纵横比的窗口滑窗，不同尺寸的窗口分别使用不同尺寸的滤波器进行训练，这种可以理解成金字塔滤波器。第二种方法通常与第一种方法联合使用。    
与前面的方法不同，我们的方法建立在一个金字塔anchor上，在代价上更具有高效性。我们的方法参照多个尺度和纵横比的锚框对边界框进行分类和回归。只依赖一个尺度的输入图像和一个尺寸特征图，并且使用单一尺寸的滤波器。我们的实验结果说明了对多尺度问题的处理效果。   

**3.1.2 Loss Function**   
（赋予anchors标签）为了训练RPN，我们给每一个anchor分配一个二值标签，即是否是目标。在以下两种情况下，将anchor的标签赋为正类：（1）与某一个ground truth有最大重叠率（Intersection-over-Union，IoU）的anchor；（2）与任意一个GT bbox的IOU大于0.7的anchor。需要说明的是，一个GT bbox有可能为多个anchor分配正标签。通常上述第二个情况即可确定大部分anchor的标签。如果anchor与所有的GT bbox的IOU都小于0.3的话，将这个anchor的标签赋为负类。既不是正类也不是负类的anchor对训练没有帮助。   
（损失函数）有了如上的定义，我们遵循Fast R-CNN的多任务损失函数，设计RPN的损失函数如下：   

<img src="https://raw.githubusercontent.com/weidi1024/weidi1024.github.io/master/_posts/搜狗截图20190724222430.png" height="20%"/>

其中，i是一个mini-batch中anchor的索引，pi是anchor i是否为目标的概率。pi* 是GT label。ti是一个向量，表示预测bbox的4个参数坐标。ti * 是与正anchor相关的GTbbox的坐标。分类损失Lclc是在两个类别上的对数损失函数。对于回归损失函数，我们使用Lreg=R(ti-ti* )，其中R是一个鲁棒的损失函数，smooth L1，在[2]中有定义。pi* Lreg表示回归损失旨在anchor i为正类（pi* =1）的情况下才被激活，当pi*=0时，该项被忽略。分类层和回归层分别输出{pi}和{ti}。   
这两项通过Ncls和Nreg进行归一，然后通过平衡参数λ进行权重分配。在我们当前的版本中，Ncls=256，Nreg~2400，λ=10，因此分类和回归项的权重才能大致相等。我们通过实验表明，最终的结果对的值不敏感（表9）。我们发现上述的归一化不是必须的，可以被简化。   
对于边界回归，我们定义以下4个坐标的参数化： 

<img src="https://raw.githubusercontent.com/weidi1024/weidi1024.github.io/master/_posts/搜狗截图20190724231508.png" height="20%"/>


其中x，y，w，h表示anchor的中心坐标和宽长。变量x，xa，x*分别表示预测bbox，anchor bbox，GT bbox的中心横坐标。y，w，h也是相同。可以被认为是从anchor bbox到附近的GT bbox的回归。   
然而，我们的方法实现了边框回归，与其他方法不同[1] [2] 。[1]中，bbox回归通过在池化的特征上进行回归，并且不同尺寸的区域回归的权重相同。我们的方案中，用于回归的特征具有相同的尺寸，3x3。为了实现不同的尺寸，我们设置了k个bbox回归器。每一个回归器负责一种尺寸一种纵横比的bbox，k个回归器不共享参数。尽管使用相同尺寸的特征，预测不同尺寸的bbox还是有可能的，这要归功于anchors的设计。 所有anchor的损失函数可训练，但这样使得网络偏向于负样本，

**3.1.3  Training RPNs**   
（训练细节）RPN可以使用SGD进行端到端训练。每一个minibatch来自于一幅包含许多正负anchor的样本，但是这样会导致网络会偏向于负样本。**因此我们随机选择256个anchors来计算损失函数，其中正负样本的比例为1:1。** ==如果一个图像中有少于128个正样本，我们用负样本填充。==  
随机初始化，加载预训练模型，学习率的设置等。使用caffe。这部分就不详细翻译了。   

**3.2 RPN与Fast R-CNN共享特征**  
目前我们只叙述了如何训练RPN，没有利用这些候选区域进行检测。对于检测网络，我们使用Fast R-CNN。后面将叙述联合训练。   
RPN和Fast R-CNN可以独立训练，那么会使得卷积层的学习往不同的方向发展。因此我们需要开发一种技术运行在它们之间共享卷积层。我们讨论了三种不同的共享卷积的训练方法。   
1. 交替训练。在这部分，我们首先训练RPN，然后使用提取的候选区域训练Fast R-CNN。网络已经被Fast R-CNN调整了，我们使用调整过的网络去初始化RPN，这个过程可以交替进行。这是在本文中一直使用的方法。
2. 近似联合训练。在这部分，RPN和Fast R-CNN网络被整合成一个网络，反向传播对RPN和Fast R-CNN一起更新。这种方法容易实施，但是忽略了一个问题，候选区域既是网络输入，同时也是网络的响应。这个方法的实际训练时间比第一个交替训练方法缩短了20~25%。   
3. 非近似联合训练。因为上述的问题，理论上有效的反向传播求解器也应该包含bbox的梯度。在这部分中，我们需要一个RoI池层，它关于bbox坐标是可微分的。这是一个非常重要的问题，可以通过[15]中开发的“RoI翘曲”层给出解决方案，这超出了本文的讨论范围。   




**四步交替训练**   
在本文中，我们使用四步交替训练法学习共享特征。   
第一步，使用3.1.3的方法训练RPN。使用预训练模型初始化基础网络，使用区域提取任务进行端到端的训练。   
第二步，使用第一步产生的候选区域，训练一个分离的检测网络Fast R-CNN。网络同样使用预训练模型进行初始化。在这一步，两个网络并没有共享卷积层。   
第三步，我们还是要检测网络去初始化RPN，训练RPN，但是我们固定共享部分，只训练RPN独有的卷积层。现在两个网络共享卷积层。   
第四步，固定共享部分，只训练Fast R-CNN独有的部分。   
这样，两个网络共享相同的卷积层，形成了一个统一的网络。类似的迭代还可以运行多次，但是我们发现其改观是微乎其微的。   

**3.3 实施细节**  
（输入图像的尺寸）我们训练在单一尺寸的图像上对区域提取网络和检测网络进行训练。我们对图像进行尺寸调整使得短边为600个像素点。基于图像金字塔的多尺度特征提取也许可以提高精度，但是不能在速度和精度上实现很好的权衡。在重新缩放的图像上，ZF和VGG网在最后一个卷积层上的总步长为16像素，因此在调整大小之前，典型的PASCAL图像上的步长为∼10像素(∼500×375)。即使是如此大的步幅也能提供良好的结果，更小的步幅可能会进一步提高准确性。   
（anchors的设置）对于anchors，我们使用三个尺度的bbox，分别是128^2，256^2, 512^2；以及三个纵横比1:1,1:2,2:1。这些超参数不是为特定数据集精心准备的，下一节提供了消融实验来说明它们的影响。如前所述，我们的方法不需要图像金字塔或者滤波器金字塔来预测不同尺度的区域，节省了大量的运行时间。图3(右)显示了我们的方法在大范围尺度和纵横比下的能力。表1显示了使用ZF net获得的每个anchor的平均候选区域大小。注意到，我们的算法允许比潜在接受域更大的预测区域。这样的预测并非不可能——如果一个物体的中心是可见的，那么人们仍然可以粗略地推断出这个物体的范围。   
（关于anchors在图像边界上的问题）需要小心处理超出图像边界的anchor box。在训练期间，我们忽略了这些跨边界的anchors，所以它们对损失没有贡献。对于典型的1000×600的图片，大概有20000（60×40×9）个anchors。在训练期间，去掉跨边界的anchors，剩余大概有6000个。如果在训练期间没有忽略掉跨边界的anchors，那么它们在目标函数中引入大量难以纠正的错误项，训练损失不会收敛。在测试期间，我们仍在整个图像上应用RPN。这或许可以产生跨边界的候选区域，我们会对其裁剪到图像的边界。   
（候选区域去冗余）一些RPN之间也会有很高的IOU。为了减少冗余，我们在产生的候选区域上根据候选区域的置信度对其进行NMS。我们将IOU的阈值设置为0.7，这样每幅图像会产生大概2000个候选区域。正如我们展示的，NMS损害最终的检测精度，但是会显著减少候选区域的数量。NMS之后，我们使用前N个候选区域进行检测。后续，我们使用2000个RPN产生的候选区域训练Fast R-CNN，但是我们在测试时使用不同数量的候选区域（应该是300）。


# 4 实验
实验部分后续有时间再看



# 5 结论
略



# 一些细节    
Faster R-CNN如何保证训练过程中正负样本的均衡性？    

**1 RPN ：**       
**我们随机选择256个anchors来计算损失函数，其中正负样本的比例为1:1。**      
REFERENCE:    Faster R-CNN in Sec3.1.3    

**2 Fast R-CNN**      
**每次随机选择2幅图像，并从每幅图像对应的RoIs中选择64个RoIs，一共选择128个RoIs，具体方法为：其中选择前景(与GroundTruth的IOU大于0.5的RoIs）的25%作为训练使用的前景RoIs，剩下的RoIs从背景（与GroundTruth的IOU大于0.1小于0.5的RoIs）里面选择，作为训练使用的背景RoIS**    
**REFERENCE**:       Fast R-CNN in Sec2.3 Mini-batch sampling     
During fine-tuning, each SGD mini-batch is constructed from N = 2 images, chosen uniformly at random (as is common practice, we actually iterate over permutations of the dataset). We use mini-batches of size R = 128, sampling 64 RoIs from each image. As in R-CNN, we take 25% of the RoIs from object proposals that have intersection over union (IoU) overlap with a groundtruth bounding box of at least 0.5. These RoIs  comprise the examples labeled with a foreground object class, i.e. u ≥ 1. The remaining RoIs are sampled from object proposals that have a maximum IoU with ground truth in the interval [0.1: 0.5), following [11]. These are the background examples and are labeled with u = 0. The lower threshold of 0.1 appears to act as a heuristic for hard example mining [8]. During training, images are horizontally flipped with probability 0:5. No other data augmentation is used.    

实现过程较为简单粗暴，现在有一些较为新颖的方法比如OHEM、focal loss等

