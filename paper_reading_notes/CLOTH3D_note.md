# CLOTH3D: Clothed 3D Humans

## Abstract

CLOTH3D，第一个大规模的合成3D穿着衣服的人体序列数据集。该数据集包括多样的服装类型，拓扑结构，形状，尺寸，松紧和布料。衣服在成千上万个不同姿态序列和身体体型上模拟生成逼真的动态衣服。

提供了数据集和用来生成衣服的生成模型。

提出条件变分自编码器基于图卷积来学习服装的浅层空间。

能在各种pose和shape的SMPL模型上生成3D服装。

## 1.Introduction

目前的建模，复现和生成衣服的方法主要聚焦在2D数据上。主要有两个原因：

第一，深度学习方法要大量数据，但现在没有足够的3D服装数据。

第二，服装在shape,size,topologies,fabrics,textures方面有很大的变化，增加了3D服装生成的复杂性。

为了生产生3D dressed human可以有三个策略：3D scans, 3D-from-RGB和synthetic generation.。

* 3D scans价格昂贵，最多能生产human + garments的单个mesh。
* 从RGB图像中推断衣服的三维几何形状的数据集不够准确的，不能正确地模拟衣服的动态表现。
* synthetic数据易于生成，无ground truth误差。用合成的数据来训练深度学习模型用在实际应用种被证明有效。

COLTH3D数据集特点：

* 第一个包含了成千上万个穿着高分辨率3D衣服的人体序列的合成的数据集
* 在服装，shape,pose变化方面unique，包括了超过2 million 3D samples。

开发了一个生成pipline，对每个序列在garment type, topology, shape, size, tightness and fabric方面生成一个独特的outfit。

提供了一个baseline模型，该模型能够能成dressed人体模型。将衣服编码作为连接到皮肤和布料的offsets，用SMPL作为人体模型。这就产生了数据的homogeneous dimensionality。用分割mask通过移动body vertices提取服装，该mask通过网络预测。

基于graph convolutions(GCVAE)，提出Conditional Variational Auto-Encoder(CVAE)来学习衣服的latent spaces.。

## 2.Related work

**3D grament datasets** 现在的文献关于3D服装缺少大规模可用的datasets。

一个策略是通过3D扫描，介绍了几个使用此技术产生的数据集的缺点。说CLOTH3D有多好。

**3D garment generation** 现在的3D clothing工作聚焦在生成dressed human。把相关的工作分成深度学习和非深度学习分别介绍。

非深度学习方法不赘述。

深度学习方法：有把身体和衣服当成不同的点云处理的，有把图VAE和GAN结合到SMPL模型上的。本文的方法编码衣服作为SMPL的offsets，然而，服装遵循人体拓扑结构的假设并不适用于裙子和连衣裙，这种情况另外解决。除此之外，本文的模型沿着offsets预测服装mask来生成layered models。

## 3.Dataset





