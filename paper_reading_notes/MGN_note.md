# Multi-Garment Net: Learning to Dress 3D People from Images

## Abstract

提出MGN网络，该网络从一段视频中的几帧（1-8帧）图像去预测body shape和堆叠在SMPL模型上的服装。几个实验证明这种表示相对于sigle mesh或者voxel representations of shape允许更高维的控制。该模型能预测服装的几何形状，并把它和体型联系起来，迁移到新的体型和姿势上。

为了训练MGN，作者利用一个包含712件对应的数字服装的数字衣橱，用一种新的方法将一组服装模板注册到一个数据集，该数据集对穿着不同服装和姿势的人进行了真实的3D扫描。来自“数字衣橱”的服装，或MGN预测的服装，可以用来为任何体型的人打扮成任意的姿势。

## 1 Introduction

最近有一些尝试重建穿着衣服的人的方式，但它们缺少逼真的效果以及不可控。这种限制很大程度上是由于它们用一个单一的surface(mesh or voxels)来表示衣服和身体。因此它们不能从从图像主题里分开捕获服装，更不用说把服装映射到新的体型上。

MGN能够直接从图片推断人的身体和堆叠在上面的服装作为分开的meshs。

相比于之前的工作，MGN能产生更高视觉质量的重建并允许有更多的控制：

* 1 能从一个主体推断3D服装并穿到另一个主体身上。
* 2 同类服装的纹理迁移：能细腻地把从图片捕获的服装纹理到映射到任何的同类的服装几何上。

为了实现这种级别的控制，作者主要解决两个挑战：

* 从穿着衣物的人体的3D scans学习每个服装的模型。
* 从图片学习重建服装的模型

作者定义了一组离散的服装模板（根据类别：长短袖，长短裤和外套），并为每个类别注册一个单独的模板到每个扫描实例，这些实例被自动分割为服装部分和皮肤部分。

因为一个类别内的服装几何变化得很明显，所以作者首先最小化模板和扫描边界的距离，同时尽量保持模板表面的拉普拉斯变换。这个初始化步骤只需要解一个线性系统，并很好地扩展和压缩全局模板，作者发现这对于后续的非刚性注册工作至关重要。利用这一点，作者设计了一个真实的3D服装的数字衣橱。从这些注册作者对每一个服装学习一个基于vertex的PCA模型。因为服装是很自然地和下层的SMPL身体模型联系在一起的，所以我们可以把它们转换成不同的身体姿势再重新穿到SMPL模型上。

给定一个人的一张或多张图片，MGN用数字衣橱来训练，输出预测这个人的身体姿势和体型参数，每个服装的PCA模型系数，以及PCA上的一个用于编码服装细节的位移区域。

在测试阶段，作者用一个新的自顶向下的目标函数来细化这个自底向上的估计，这个目标函数强制投影的服装和皮肤区域来解释输入的语义分割。这允许更细粒度的图像匹配相比于标准轮廓匹配。

主要贡献：

* 一个新的数据驱动的方式仅仅从图片（一个人在摄像机前旋转的少量RGB图片）去分离身体体型和服装model。
* 一个鲁棒的对于服装的3D扫描分割和注册的pipeline。目前还没有能够自动注册单个服装模板并设置为对穿着衣服的真人进行多次扫描的工作。
* 提出一个新的自顶向下的目标函数，该函数能使预测的服装和身体与输入的语义分割图像相匹配。
* 演示了几个之前不可能的应用，比如为替身穿上从图片预测的3D服装、服装几何和纹理的迁移等。
* 开源了用来从图片预测3D服装的MGN、数字衣橱数据集以及用来给SMPL模型穿上数字衣橱服装的代码。

## 2 Related Work

讨论了与该工作最相关的两个分支：

* 服装和身体体型的捕捉
* 数据驱动的服装模型

## 3 Method

为了直接从imgaes学习一个模型来预测body shaoe和garment geometry，作者处理一个有356个scans的人的数据集，这些人有不同的服装，姿势和体型。

数据预处理包含下面几步：

* SMPL注册到scans
* body aware scan segmentation
* 模板注册

对于每个scan，作者获取下层的body shape和注册到5个服装模板类别（长短袖，长短裤，外套）之一的服装。服装模板被定义为SMPL表面的区域，原始的外形遵循一个人的身体，但注册后它通过变形拟合每个扫描实例。

### 3.1 Data Pre-Processing: Scan Segmentation and Registration

*ClothCap*是register一个模板到一段4D scan sequence，而本文任务是通过instances注册单个模板，这些instances 有不同的styles, geometries,
body shapes和poses，registration和*ClothCap*相似，这里主要描述两者的区别：

**Body-Aware Scan Segmentation**  
首先把scans分割成三个区域：皮肤，上衣和裤子（对每个scan标注服装）。

因为甚至SOTA的图像语义分割都是不够准确的，因此简单地用到3D上不合适。这里，在non-rigid alignment之后，通过求解的在SMPL模型表面的UV-map上的MRF（马尔科夫随机场）来结合*body specific garment priors*和*segment scans*。

衡量服装和SMPL非重合度：为了惩罚outside the garment region的labelling vertices，garment prior（服装$g$）来自有可能和服装overlap的SMPL表面的labelling vertices的label $I_g$，比如服装和SMPLoverlap部分的verices的label为1，非overlap部分的verices的label为0。

衡量服装和SMPL重合度：相反地，对于in the garment region的labeling vertices，作者用不同于$g$的标签定义一个相似的penalty。

因为garment geometry的类内变化也很明显，作者定义一个cost，这个cost随着geodesic distance增加。

作为data terms，在*La*颜色空间中，作者合并CNN based semantic segmentation和的appearance terms based Gaussian Mixture Models。详见补充材料。

解完SMPL UV map的MRF后，把SMPL registration的标签迁移到scan上，就可以把scans分割成三个部分（皮肤，上衣，裤子）。

**Garment Template**
作者在SMPL+D模型上建立Garment Template。SMPL+D把人体模型表示为一个参数化的函数，用函数表示:

$$M(\beta, \theta, D) = W(T(\beta, \theta, D), J(\beta), \theta, W)$$

$$T(\beta,\theta, D) = T + B_s(\beta) + B_p(\theta) + D$$

参数|含义
:--:|:---
$M(.)$|a human body parametric function
$\theta$|human pose
$\beta$|human shape
$t$|global translation
$T$|human base mesh with n vertices in a T-pose
$T(.)$|deformed mesh
$W(.)$|standard skinning function
$W$|blend weights
$D$|optional per-vertex displacements
$J$|skeleton
$B_s(\beta)$|shape dependent deformations
$B_p(\theta)$|pose-dependent deformations of a skeleton

SMPL的基本原理是在一个bash mesh $T$上应用一系列的linear displacements，然后做standard skinning $W(.)$。

对每个服装种类$g$定义一个T姿势的template mesh $G^g$，使$I^g$作为一个indicator matrix，如果服装类$g$的vertx $i$和body shape vertex $j$有关联的话，就令$I_{i,j}^g=1$。

这样，$I^g$就是一个表示SMPL模型上哪些vertex是被服装覆盖的矩阵了。

特定shape的SMPL模型与服装的displacements即服装偏移身体的距离计算如下：

$$D^g = G^g - I^g T(\beta^g, 0_{\theta}, 0_{D})$$

即服装模板的mesh减去SMPL表面有服装的mesh得到偏移。

计算Unposed服装模型$T^g$（给定shape $\beta$和pose $\theta$新的人体SMPL模型）：

$$T^g(\beta, \theta, D^g) = I^gT(\beta,\theta,0) + D^g$$

计算Posed服装模型，用skinning function $W(.)$：

$$G(\beta, \theta, D^g) = W(T^g(\beta, \theta, D^g), J(\beta), \theta, W)$$

**Garment Registration**
在分割的scans上non-rigidly register身体和服装templates到scans，用multi-part alignment方式。

挑战在于不同的服装几何变化很大，导致multi-part registration失败。见补充材料。

因此，首先初始化，先将每个garment template的vertices根据SMPL的shape和pose变形，得到变形后的vertices $G_{init}^g$。

值得注意的是，由于定义每个garment template的vertices是固定的，所以初始化后的garment template边界会和scan的边界不匹配。为了globally变形template来匹配服装边界，用了基于*Laplacian deformation*的目标函数。

**Dressing SMPL**
给定garment $G^g$，用公示3，4，5给服装vertices变换姿势和蒙皮。

定义了函数$C(\theta, \beta, D)$返回posed和shaped vertices参数（见补充材料）。

### 3.2 From Images to Garments

### 3.3 Loss functions

### 3.4 Implemetation details

**Base Network($f_{\omega}^*$)**：用CNN将输入数据集${I， J}$映射到body shape，pose和garment latent spaces。

CNN结构：(2D 卷积 + max-pooling) * 5。

不幸的是，平移不变性导致了CNNs无法捕获特征的位置信息。

为了复现服装的3D细节，要充分利用2D images的2D特征和位置信息，把像素点的坐标接在每层CNN的输出上。把最后一个卷积feature map分成body shape，pose和garment information三个部分。三个部分被展平，将2D联合估计附加到pose部分。三个全连接层和average pooling作用在服装shape的浅层编码上，分别生成$l_{\beta}$，$l_{\theta}$和$l_{G}$，详见补充材料。

**Garment Network($M_w^g$)**：每个服装类训练单独的garment network。

主要包括两个分支：

第一个分支预测整个mesh shape，

第一个分支包含2个全连接层，从服装的浅层编码($l_G$)回归PCA系数。PCA bias和这些系数的点乘生成base garment mesh。

第二个分支增加高频细节。
用第二个全连接分支去回归在第一个分支预测的mesh上的位移，并限制这些位移在1cm以内，来保证整体shape是被PCA mesh解释而不是这些位移。

## 4 Dataset and Experiments

**Dataset**  
用356个人体3D scans，70个用来测试，剩下的训练。

对场景的限制是人要在镜头前转圈。

multi-mesh registration的scans允许data augmentation因为registered scans可以被re-posed和re-shaped。

### 4.1 Experiments

**Qualitative comparisons**
和[这篇论文](http://openaccess.thecvf.com/content_CVPR_2019/html/Alldieck_Learning_to_Reconstruct_People_in_Clothing_From_a_Single_RGB_CVPR_2019_paper.html)比较，本文的衣服建模有更小的disortions。

**Quantitative Comparison**
和[这篇论文](http://openaccess.thecvf.com/content_CVPR_2019/html/Alldieck_Learning_to_Reconstruct_People_in_Clothing_From_a_Single_RGB_CVPR_2019_paper.html)比较。计算一个symmetric error。

本文的平均vertex-to-surface误差是5.78mm，**octopus**的误差是5.72mm。

**octopus**表现更好是由于基于单一mesh的方式没有把顶点和语义信息结合起来，比如这些方式可以把mesh的任何顶点拉回来解释3D形变，但是本文的方法确保只有在语义上正确的顶点才能解释3D shape。

值得注意的是MGN把衣服作为潜层编码的线性函数（PCA系数）来预测，然而**octopus**用了GraphCNN。基于PCA的方式尽管容易处理但容易导致光滑的结果本文们的工作为进一步探索在固定拓扑上建立服装几何变化模型奠定了基础。

**GT vs Predicted pose**
3D顶点预测是一个pose和shape的函数。

### 4.2 Re-targeting

