---
title: 仿真技巧 \| ANSYS 接触设置详解-Interface Treatment
date: 2024-07-01
categories: [ansys]
tags: [Workbench]
comments: true
---



仿真技巧 \| ANSYS 接触设置详解-Interface Treatment
========================================

[www.fangzhenxiu.com](https://www.fangzhenxiu.com/post/5460744/?uri=205_b0GAvhE5Qs0&code=001NSu1w3w6kV23uhl3w3c94oP3NSu1V&state=afterWeixin)安世亚太1年前浏览3331关注


本文通过一个案例详细介绍Geometric Modification中Interface Treatment的选项。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594021320_1rlb1b.jpg%3FimageView2%2F0&valid=true)

**1、案例几何模型**

一个圆柱体和一个长方块，初始间隙为1mm左右，如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594026218_x56fn8.jpg%3FimageView2%2F0&valid=true)

**2、网格划分**

有限元模型如下所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594016903_3mobpi.jpg%3FimageView2%2F0&valid=true)

**3、手动定义接触对**

手动选择接触对象，定义接触类型为摩擦接触，接触算法为Normal Lagrange，其余选项皆为默认。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594028069_8gl7o8.jpg%3FimageView2%2F0&valid=true)

**4、非线性接触诊断**

Interface Treatment属性一共包含7个选项。默认选项为Add Offset，No Ramping。其中选项Adjust to touch选项会忽略初始间隙和初始穿透，在计算开始建立了实际的接触关系，有利于求解顺利完成，但过盈配合分析不能选择该选项。

* **当选项为Add Offset,No Ramping，如下所示：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594031646_vs7lmk.jpg%3FimageView2%2F0&valid=true)

接触状态如下，Status为Near Open，Geometric Gap 为1 mm，Gap也为1mm。这种选项下，软件认为接触对存在初始间隙，当圆柱体和长方体相对运动消除这个间隙后，才会建立实际的接触关系。这种初始不接触，后续才建立接触关系的场景对软件算法是一个挑战，所以需要设置一定阻尼系数，否则可能无法成功建立接触关系。（时间步也建议设置小一点）

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594018747_5hzm8e.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594022901_7ijr6g.jpg%3FimageView2%2F0&valid=true)

如果不设置一定阻尼系数，计算无法完成，报错信息如下:  

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594018398_ielmhu.jpg%3FimageView2%2F0&valid=true)

* **当选项为Add Offset,No Ramping，如下所示：**

**![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594021688_ofwzhd.jpg%3FimageView2%2F0&valid=true)**

接触状态如下，Status为Closed，Geometric Gap 为1 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594026499_pwkf32.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：  

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594028374_s556f0.jpg%3FimageView2%2F0&valid=true)

* **当选项为Add Offset, Ramped Effects，如下所示：**

****![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594024030_j9x2pq.jpg%3FimageView2%2F0&valid=true)****

接触状态如下，Status为Closed，Geometric Gap 为1 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594031961_ydmr6y.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594018024_t6eos5.jpg%3FimageView2%2F0&valid=true)

* **当选项为Offset Only,No Ramping，如下所示：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594025930_cv5ksk.jpg%3FimageView2%2F0&valid=true)

接触状态如下，Status为Closed，Geometric Gap 为1 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594032281_45f0hg.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594026797_l0pze8.jpg%3FimageView2%2F0&valid=true)

* **当选项为Offset Only,Ramped Effects，如下所示：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594019782_ojnkzg.jpg%3FimageView2%2F0&valid=true)

接触状态如下，Status为Closed，Geometric Gap 为1 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594022243_j13fe9.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594023317_eibmnc.jpg%3FimageView2%2F0&valid=true)

* **当选项为Offset Only,Ignore Initial Status, No Ramping，如下所示：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594019342_3f32os.jpg%3FimageView2%2F0&valid=true)

接触状态如下，Status为Closed，Geometric Gap 为1.0374 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594019015_2pb9kx.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594024345_k1ef99.jpg%3FimageView2%2F0&valid=true)

* **当选项为Offset Only,Ignore Initial Status,Ramped Effects，如下所示：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594020443_p61l0l.jpg%3FimageView2%2F0&valid=true)

接触状态如下，Status为Closed，Geometric Gap 为1.8244 mm，Gap为0mm。这种选项下，软件会忽略初始间隙，在计算开始建立了实际的接触关系。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594017614_qd2cxi.jpg%3FimageView2%2F0&valid=true)

位移，接触应力，接触压力，接触穿透结果如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpublic.fangzhenxiu.com%2FfixComment%2FcommentContent%2Fimgs%2F1665594029469_p3j6h7.jpg%3FimageView2%2F0&valid=true)

[跳转到 Cubox 查看](https://cubox.pro/my/card?id=7198216302807221379)
