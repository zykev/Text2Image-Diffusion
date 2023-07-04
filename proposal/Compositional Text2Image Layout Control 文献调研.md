# Compositional Text2Image Layout Control 文献调研

## 背景介绍

一些stable diffusion的生成缺陷是多物体场合下出现的忽视物体 (neglect object) 和属性错位 (attribute binding) 问题:

e.g. prompt:  a cat on the back of a lion

![00101-2183694867](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00101-2183694867.png)

方位错误 (location insensitivity) 问题：e.g. prompt: a cat on top of a microwave

![00127-2513660327](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00127-2513660327.png)

remark: 所有图片均由[stable diffusion webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 生成。

现有工作的主要解决思路：引入bounding box 信息进行layout control， 具体方法主要是围绕调整U-Net结构里的cross attention map 展开。

* train based: 通过添加新的attention模块学习额外给出的bounding box位置信息

[ReCo](https://arxiv.org/abs/2211.15518): 把bounding box坐标信息引入text prompts, 学习regional embedding

[LayoutDiffusion](https://arxiv.org/abs/2303.17189): 设计新的layout attention模块加入U-Net中

[GLIGEN](https://arxiv.org/abs/2301.07093): 提出新的gated self-attention模块加入U-Net中，并统一了给定spatial prompts为bounding box, pose structure, segmentation等等下的学习框架

train based methods 优缺点分析：、

pros: 可以处理较为复杂的多个物体（3个及以上）位置控制，每个物体可以通过text prompts额外指定更细节的特征

cons: finetune 需要更多计算资源，数据集要求更高，需要冻结部分层以免出现小数据集上的pre trained 知识遗忘问题

* train-free: 不需要训练网络，修改采样过程引导图片的生成方式

方向一：直接修改cross attention map特定位置的权重，引导采样时更加注意通过bounding box强调的物体， 例如 [direct diffusion](https://arxiv.org/abs/2302.13153), forward method in [layout-guidance](https://github.com/silent-chen/layout-guidance)

方向二：通过attention map梯度信息修改采样过程 （效果更好），例如[attend and excite](https://arxiv.org/abs/2301.13826), backward method in layout-guidance

pros: 更少的计算消耗

cons: 主要展示了两个物体的位置控制生成，多个物体下效果是否下降

* combine: 不引入bounding box信息，直接从prompt中利用一个额外网络预测出bounding box，之后再使用train-free 方法 [BoxNet](https://arxiv.org/abs/2305.13921)

## 问题介绍

现有工作已经很好解决了diffusion model在多物体prompt下位置信息不敏感和遗漏物体等问题，但每个物体作为个体相对独立，对于物体间的交互没有很好的控制手段。Direct diffusion中提出limitation: 能否进行物体的orientation control。 通过2个物体间orientation的控制是达到交互动作生成的必要条件，而交互动作对于画面表现感，场景生成，故事性表现感都具有重要意义。

考虑普通的stable diffusion对于如下prompts的生成图：

A cat catches a flying ball

![00106-4151909253](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00106-4151909253.png)



![00113-4151909260](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00113-4151909260.png)

希望控制猫以什么样的角度扑向球，例如右斜上，左斜上等等

A line of expedition caravans rode camels into the depths of the desert

![00132-943955865](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00132-943955865.png)

![00123-943955856](https://github.com/zykev/Text2Image-Diffusion/blob/main/proposal/images/images/00123-943955856.png)

希望控制骆驼商队以什么样的角度进入沙漠
