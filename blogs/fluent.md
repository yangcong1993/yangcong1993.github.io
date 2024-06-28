---
layout: page
permalink: /blogs/fluent/index.html
title: 一文说清楚fluent初始化
---
一文说清楚Fluent初始化操作(标准+混合初始化+Patch+UDF)
====================================

[blog.csdn.net](https://blog.csdn.net/weixin_45560646/article/details/135759355)成就一亿技术人!

{#t0}**0. 前言**
--------------

本文3000多字(3363字)，可以说是全网对于[Fluent](https://so.csdn.net/so/search?q=Fluent&spm=1001.2101.3001.7020)初始化介绍最详细的文章了。主要介绍了初始化作用及Fluent初始化方法(标准初始化和混合初始化)，基本上把所有的参数及其设置意义都介绍了。

虽然但是，还是有很多没有顾及到讲解的，后面会补充上去，主要有：

1. Patch方法

2. 通过[UDF](https://so.csdn.net/so/search?q=UDF&spm=1001.2101.3001.7020)初始化

希望大家多多点赞、分享，再看呀

{#t1}**1. 初始化的概念**
------------------

### {#t2}**1.1 初始化概念**

什么是初始化？？很容易理解，所谓初始化就是给方程一个初始值。我们使用的所以数值模拟软件包括Fluent都是通过迭代的方式对方程组进行求解的，那么给出迭代的第一组值就被称为初始化。

比如超越方程

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Ff306dcc9e8eca762e0ef3c4cd528ea1c.png&valid=false)

上述方程是没有解析解的，我们想要求解方程的解，需要用迭代的方法求解。比如牛顿迭代或者高斯迭代等，都需要先给一个初值。给出初值的过程就是初始化的过程。

只不过我们计算的是场的多个物理量如温度、速度等，因此需要给整个场多个物理量的初值

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F2948ffc6dce217aa5b0fc23145cc9444.png&valid=false)

### {#t3}**1.2 初始化的重要性**

初始化重不重要呢？？既重要又不重要，为什么这样说呢？

重要性：因为初始化的值会直接影响到我们的收敛过程。设想，如果非常巧我们恰好初始化给出的值就是收敛后的稳定值，那我们可能只需要迭代一步就可以达到收敛了。

而相反如果我们给的初始化值非常离谱，完全违背物理学规律，那我们可能要进行很多次迭代才能收敛，甚至最后会发散。对于瞬态计算，初始化更是非常重要

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F096016093929259943e11394c467b71b.png&valid=false)

不重要性：但是我们99.99%的情况都不可能直接给出收敛结果作为初始化参数，相反我们并不知道结果是什么样？

因此我们只能猜测什么情况会比较接近收敛情况，或者我们干脆就以inlet参数作为初始化参数，以all-zones参数作为初始化参数。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F2bfbeff808cfb632d43feffb673cef0f.png&valid=false)

虽然初始化会影响到收敛过程，但是毕竟是电脑计算的，多几百次的迭代步似乎也可以接受。但还是之前的原则，初始化参数不能太离谱，否则可能直接发散了。

注：

1. 对于稳态计算来说，只要初始化值别太离谱，选择什么样的初始化值都不会影响最终的收敛结果。无非是收敛快慢的影响

2. 对于瞬态计算来说，初始化非常重要，不能够随便给出。瞬态计算的结果是随着时间变化的，初始化就是你物理模型的初始值，必须严格给出。

比如我们想模拟溃坝过程，那么初始时水就应该被堤坝挡住。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F81e63148509627aa0348a357bf8966b5.png&valid=false)

{#t4}**2. Fluent初始化界面**
-----------------------

Fluent提供了几种不同的初始方法

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F96aaafb8aa2ac2c21a2f7bb73f848306.png&valid=false)

### {#t5}**2.1 Standard Initialization标准初始化**

a. Standard Initialization：标准初始化。最简单的[初始化方法](https://so.csdn.net/so/search?q=%E5%88%9D%E5%A7%8B%E5%8C%96%E6%96%B9%E6%B3%95&spm=1001.2101.3001.7020)，也是默认的初始化方法。只有选择标准初始化才会出现上图中的各个选项。

所谓标准初始化，就是自定义初始化值，比如静压Gauge Pressure、湍动能Turbulent Kinetic Energy等值。

b. Initial Values：各物理量的初始化值，可以直接在Initial Values下面的各物理量栏中，输入我们想要初始化的值。但是需要一个一个的输入，可能会比较麻烦。

c. Compute from：为了方便输入初始化的值，可以直接从Compute from选择以各边界参数的值作为初始化值。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F62877d46f3fe2ce92e1d48d8bbe06395.png&valid=false)

常见的选择入口inlet作为初始化值。当从Compute from选择边界或者计算域后，Initial Values中的值会自动改变为边界参数的值。

比如当inlet设置流速为5m/s、wet_air h2o设置为0.01454，Initial Values值会自动设置为此值。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F98384ef50ed76912477e92f87c847da7.png&valid=false)

注：

1. 如果Compute from选择all_zones，表示以整个计算域的平均值作为初始化值

2. 如果边界没有设置某些物理量，那该物理量就会保持之前的设置。

3. 设置原则：如果是稳态计算，初始化值不那么重要，一般Compute from选择inlet即可；如果是瞬态计算，则需要根据初始情况自己更改Initial Values的值。

4. 所谓初始化，就是给流场赋值。如果计算到一半又进行初始化，那之前的计算值会被初始化值覆盖掉，相当于重新计算；如果计算一半，没有点击初始化，又重新计算，那就会接着之前的结果继续算。

d. Reference Frame参考系

Relative to Cell Zone：表示初始速度是相对于cell zone的运动速度。

Absolute：表示初始速度是绝对速度。只有当问题涉及运动参考系或滑移网格时即cell发生运动时，这个选项才有意义，否则这两个选项是等效的。

如果你的计算域是旋转的，使用Relative to Cell Zone更好。一般忽略即可。

以上是关于标准初始化的设置，只有选择标准初始化Standard Initialization才会出现Initial Values和Compute from等设置。

e. 优缺点

Ø 优点：标准初始化的好处是我们可以自定义一些初始的物理量场，这可以满足对初始场有要求的情况

Ø 缺点：缺点是整个场的初始值是均匀的，比如初始的温度值是300K，那么整个流场的温度都是300K，这肯定是不合理的，因此需要通过计算加强收敛性。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fb15874a6a3ccdea78dd92670d606624f.png&valid=false)

### {#t6}**2.2 Hybrid Initialization混合初始化**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe31b7ab42b3ad179d7cd2578c722e18f.png&valid=false)

a. 特点

Hybrid Initialization：混合初始化。它求解拉普拉斯方程来确定速度场和压力场。所有其他变量，如温度、湍流、物种分数和体积分数，都将根据计算域的平均值或插值方法。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F7466523ee280414468a16dff86171313.png&valid=false)

注：

1. 使用混合初始化，控制台会弹出上图的信息，下面的10个迭代步就是在求解拉普拉斯方程。有时候会弹出警告信息，大概意思就是求解拉普拉斯方程没有收敛，这种警告忽略即可。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Ff2408cbc1bccae4127214a34907a8c89.png&valid=false)

2. 单相稳态流的默认初始化方法是混合初始化方法。对于其他流类型，如多相流或瞬态计算，默认的初始化方法是标准初始化方法。

这是什么意思呢？？？

就是说选择混合初始化后，速度场和压力场已经被计算了一定的次数，更加接近收敛后的值。但是其他的变量仍然是初始值。这样做的好处就是压力场和速度场会更容易收敛。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe66240546057f76284b45ecf632a6711.png&valid=false)

b. 优缺点：

优点：速度场和压力场已经被计算了一定的次数，更加接近收敛后的值。压力场和速度场会更容易收敛。

缺点：无法自定义一些物理量，比如我们想让初始时整个计算域温度全是300K，无法直接通过混合初始化完成，需要Patch才能实现。但标准初始化可以。

### {#t7}**2.3 混合初始化More Settings**

选择Hybrid Initialization后，大多数情况下不需要设置任何参数，直接点击Initialize即可。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe31b7ab42b3ad179d7cd2578c722e18f.png&valid=false)

如果想进行更多的设置，需要点击more settings

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe5c03e9fa1faa73f21888cfce9ea9b66.png&valid=false)

1. General Settings

Ø Number of Iterations：迭代次数默认值为10，也就是上面控制台输出10次迭代步的原因。即求解拉普拉斯方程以初始化速度和压力的迭代次数。

对于比较复杂的模型，如果迭代次数达不到1e-06残差(控制台输出的残差)，可适当增加此值。一般默认即可。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fdd03a5f4f1db0630024964d61f43e266.png&valid=false)

Ø Explicit Under-Relaxation Factor显式下松弛因子，默认值为1。方程1表示速度场，方程2表示压力场。用于求解拉普拉斯方程。若残差达不到1e-06，可减小松弛因子来增加初始化的收敛性。

Ø Reference Frame：和标准初始化的Reference Frame参考系相同

Ø Use Specified Initial Pressure on Inlet：如果勾选此项，表示inlet设置的压力Supersonic/Initialization Gauge Pressure会被应用在入口作为初始化压力。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Faf6fa9f183de1e5a890a1a8bad1e9c2d.png&valid=false)

勾选Use Specified Initial Pressure on Inlet后的初始化压力场

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fabea04ab7125454b168e2c0dbfb7def2.png&valid=false)

Ø Use External-Aero Favorable Settings：对于外流场问题如机翼、汽车等，勾选此项会加速初始化速度场的收敛

Ø Maintain Constant Velocity Magnitude：勾选此值，表示通过求解方程-0获得的流动方向，但流速保持相同。对于一些不可压缩的外部流动问题和多孔介质问题，此选项有帮助。

勾选此选项后的速度场和之前不同

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F27dc5cb4d577b3dde882e96e904f9474.png&valid=false)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F918b3ae78f477a9c0815a9fd7f200d3a.png&valid=false)

2. Turbulence Settings：默认是勾选的，使用湍流参数的计算域平均值。如果不勾选，则表示使用可变的湍流参数。当这个选项被禁用时，表示使用局部流参数来计算湍流参数，如湍动能和耗散能。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fd49a3d2ae6309dc0569bf07c6b05d956.png&valid=false)

3. Species Settings：默认情况下不勾选Specify Species Parameters，表示初始化计算域大部分组分的质量分数为0。勾选并设置值，则会按照设置的值进行初始化，类似与标准初始化指定值。

为什么说大部分区域质量分数为0？这是因为混合初始化是按照插值方式进行初始化，壁面处的组分质量分数是按照壁面边界条件进行初始化的。

比如勾选后设置为0.1，则壁面处仍是按边界条件初始化，但主流区域则为0.1

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fc3e12003eb03cacc2b7415b4ad533cf3.png&valid=false)

{#t8}**3. 初始化原则**
-----------------

我们把两种初始化方法比较详细的讲解了一下。在一些情况下初始化有一些技巧。

### {#t9}**3.1 稳态初始化**

标准初始化和混合初始化均可。选择混合初始化，收敛速度会更快。选择标准初始化，一般都是compute from选择inlet。初始化值不会影响最终的计算结果，可能会影响收敛速度。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F2b34a299f27b037e07058dfbcaf5762e.png&valid=false)

### {#t10}**3.2 瞬态初始化**

对于比较难收敛的瞬态问题，可能需要一些就技巧。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F8d45c8fe53120fff25e779ce5ae10f8b.png&valid=false)

1. 先稳态计算，把能够稳态的场计算完成后，比如模拟蒸发问题，把速度场、温度场等先稳态计算完成。但要禁用掉瞬态方程，也就是只算部分方程。

2. 然后在此基础之上，再进行瞬态计算。但不要初始化，而是把稳态计算的场当作初始化值。类似与混合初始化先把速度场和压力场计算几步。

3. 为什么禁用瞬态方程？？比如蒸发过程一定是个瞬态过程，那么水蒸汽组分的质量分数和水的体积分数都是和蒸发过程直接相关的。如果先用稳态计算，必须先禁用这两个方程，否则收敛性一定很差。

4. 禁用方法：

a. 方法1：直接关掉这个模型，比如蒸发问题，关掉蒸发模型。待稳态计算完成后再打开。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F477bf7c2ef023547c1fb5fac477fe392.png&valid=false)

b. 方法2：不必关模型，只需要在solution controls---Equations不勾选相关方程即可。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fc1cbda15b47ec8f5ecc82f92b755efce.png&valid=false)

c. 比如对于蒸发问题，不勾选Volume Fraction和air h2o就不会计算这两个方程。而只会计算流动、能量和湍流方程。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fb7944f36ef89572a7f4712334202afbf.png&valid=false)

{#t11}**1. Patch操作**
--------------------

### {#t12}**1.1 概念介绍**

上篇文章我们详细的介绍了标准初始化和混合初始化[七十五、Fluent初始化操作详解](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247486459&idx=1&sn=a901bb3b683968a9eaba51528406a23e&chksm=c0ba515bf7cdd84d614b8f48e85834efe75d725e43786fa7a4f35605faae47abd3169638d011&scene=21#wechat_redirect "七十五、Fluent初始化操作详解")，但一些时候这两种初始化是不够的，还需要辅助的操作---patch操作

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fd71b0cfebc1c2695faa6719c6e0abc05.png&valid=false)

什么是Patch操作呢？英文翻译叫做"补丁"，它确实是补丁的意思。类似于给衣服打补丁，Patch操作就是为流场打上修补，即修改原始的正常流场。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe266deabbfe1711726bec47f0d551be6.png&valid=false)

这可以包括对初始化的速度场、温度场、体积分数场等进行调整，也可以是对经过计算后的各物理量场进行修正。Patch操作的目的是在流场中选择性地改变某些部分的特定物理量数值。

比如下图是在计算后的温度场基础上进行的patch操作，第二张图明显有"打补丁"的感觉。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F82e9e6bc47a87ce9f9ee4cfd8fe5c47e.png&valid=false)

注：虽然可以在计算结果后进行patch操作，但是一般是不这样做的，Patch主要还是修改初始化的流场，在初始化后紧接着进行patch操作。可以认为patch操作是一种高级的初始化。

### {#t13}**1.2 Patch操作的适用性**

什么时候需要"打补丁"呢？什么情况下需要patch操作？

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F77569372e5a9d7a29b39c13b1ea05cb1.png&valid=false)

一般来说，稳态计算不需要patch操作，而瞬态初始化才会用到patch操作。为什么呢？

因为稳态计算的最终结果不依赖初始化，那么初始化后的patch操作只是改变了迭代步数，对最终结果没有影响。

而瞬态计算不同，瞬态计算的最终结果依赖初始化，patch操作相当于改变了初始化值，会对最终的计算结果产生影响。比如溃坝问题，初始化后需要patch液态水的体积分数，来构建蓄水区域。

### {#t14}**1.3 Patch操作步骤**

下面我们以溃坝模型为例来详细说明patch操作。溃坝模型很简单，两相流，水刚开始被存储在蓄水池中，然后堤坝打开，水流出。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F13d8d34b6145b8d10f5284566cdb0aae.png&valid=false)

#### {#t15}a. 标准初始化

先进行标准初始化，我们将water Volume Fraction设置为0，也就是整个计算域都是空气，没有水。但这显然不是我们想要的初始化结果，因此必须打补丁，也就是进行patch操作，将蓄水池装满水。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F2dbc020507e5aff4d0c7a093444a0397.png&valid=false)

#### {#t16}b. Patch界面

点击patch后会弹出patch界面

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F46ecde6e62fd4cc81186107ebb0d5feb.png&valid=false)

Patch界面可以分为三部分，第一部分是选择需要修改的物理量；第二部分是设置需要修改的值；第三部分是选择需要修改物理量的区域。我们详细讲解：

Ø 第一部分：

Reference Frame：参考系，默认即可，不必理会，具体意义可参考上篇文章。[七十五、Fluent初始化操作详解](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247486459&idx=1&sn=a901bb3b683968a9eaba51528406a23e&chksm=c0ba515bf7cdd84d614b8f48e85834efe75d725e43786fa7a4f35605faae47abd3169638d011&scene=21#wechat_redirect "七十五、Fluent初始化操作详解")

Phase和Variable：Phase表示需要修改哪一相的物理量，对于两相流，可以设置mixture和次相。Variable表示物理量。这两者是互相配合的，Phase下可以选择不同的相，可以设置不同的物理量。

对于本例，我们需要设置water的体积分数，因此phase需要选择water，Variable需要选择体积分数Volume Fraction。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F1ef49e2bc4b0b6f9ab51c530db57b4bd.png&valid=false)

Volume Fraction patch options:只有VOF 和 Eulerian 模型需要patch体积分数时才可用。一般保持默认即可。

Patch Reconstructed Interface：patch界面重构

我们patch的体积分数界面和实际的相界面，通过线性插值的方式重构，也就是界面过渡的地方是线性过渡的，而不是直接突变的。

Volumetric Smoothing

通过相邻网格体积分数的平均值来使体积分数比较平滑。启用此选项后，可以单击"Smooth"，对整个域中的体积分数进行平滑。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F26df0d99e010812bb54a3dd6141684b6.png&valid=false)

Smoothing Relaxation Factor

和Volumetric Smoothing搭配使用的，用来控制体积分数的平滑程度。在0（无平滑）和1（最大平滑）之间。

Ø 第二部分：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F950d0cc36efefe250f855bc667d550f4.png&valid=false)

Vaule：需要修改物理量的值，在本例即1，也就是将water的体积分数修改为1。也可以通过表达式来定义，详细可参考文章：[七十一、Fluent表达式进阶实例](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247486233&idx=1&sn=51cf7bbdd2e2cd57b2b90fe5d19f410b&chksm=c0ba51b9f7cdd8af151459476894a75bfe814deddcff650f6a18bcdd7bcec668dd8768dab82d&scene=21#wechat_redirect "七十一、Fluent表达式进阶实例")

Use Field Function：通过Field Function的形式来确定修改的物理量。就是对一些物理量进行计算，比如动压=1/2\*密度\*速度\^2。一般不需要这样设置。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fdcbef975ef83a82b8438ecfac3d5065b.png&valid=false)

Field Function：勾选Use Field Function后可用，选择通过Field Function定义的场函数。比如下图定义的动压

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F7b579979a0d8ac3a5097c4746f16ddbe.png&valid=false)

Patch界面就会出现刚才定义的场函数dy-pre

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F9e6cb03f11002559abee9d252d235d1f.png&valid=false)

Ø 第三部分：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fb6a83efb189a461f7486674f11fe2693.png&valid=false)

Zones to Patch：选择需要patch的zone计算域，当存在多个计算域时，需要对某一个计算域进行patch，可以在此处选择。比如我们可以将蓄水池单独划分成一个计算域，那么这里就可以选择这个计算域。但是如果没有单独划分计算域，这个不需要进行选择。

Registers to Patch：选择需要patch的Registers区域。Registers to Patch和Zones to Patch只能选择一个。Registers区域是指通过Fluent单独定义一个区域。这样可以在不划分计算域的情况下，仍然mark一部分区域。

Registers区域的具体操作可参考文章：[五十四、Fluent网格自适应详细操作](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247485285&idx=1&sn=a4e0bf21564996ad979fcc27674f2c09&chksm=c0ba5dc5f7cdd4d35a21ba1978afae315426f220a9a60fbac9f6adc134091f5b97eb81bbbc12&scene=21#wechat_redirect "五十四、Fluent网格自适应详细操作")

本例需要mark的区域如下图

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F9d78f5548da49cd8467f64a57c337965.png&valid=false)

点击save/display后可看到被标记的区域。详细操作参考文章[五十四、Fluent网格自适应详细操作](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247485285&idx=1&sn=a4e0bf21564996ad979fcc27674f2c09&chksm=c0ba5dc5f7cdd4d35a21ba1978afae315426f220a9a60fbac9f6adc134091f5b97eb81bbbc12&scene=21#wechat_redirect "五十四、Fluent网格自适应详细操作")

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F1dcd7ac9b0269151d12610072a5128f8.png&valid=false)

回到patch界面，Registers to Patch会出现刚才定义的区域pool。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F17bb19009ef5badf8e6fda355ce121df.png&valid=false)

点击patch后，可在后处理查看体积分数云图，满足了我们的初始要求。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F397cd8063f4e348595d97087f5644f0c.png&valid=false)

溃坝过程模拟

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Faf1516da676910b4d64a942defc257af.gif&valid=false)

{#t17}**2. UDF初始化**
-------------------

### {#t18}**2.1 UDF初始化概念**

Patch操作可以辅助标准初始化，给标准初始化打个补丁。但并不能满足所有的要求。有些时候我们的初始物理场并不是打个补丁那么简单。

比如我们知道温度随着海拔高度而降低，此时温度场和高度有关，也就是和y轴坐标有关。无论使用标准初始化还是patch操作都比较难初始化这样的温度场。当然还有更加复杂的符合你的实际情况的初始化场。这时候就需要用UDF来完成初始化。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F5ff82b220cbe2dbe6f9e926bf102bb5c.png&valid=false)

### {#t19}**2.2 DEFINE_INIT 初始化宏**

DEFINE_INIT (name, d)只有两个参数，其用法和ADJUST宏类似，参考文章[五十八、Fluent UDF调节宏ADJUST](http://mp.weixin.qq.com/s?__biz=MzkwMTAyNTc0Mw==&mid=2247485385&idx=1&sn=8faea1819ef916dd4ecf49e9b8c39cbd&chksm=c0ba5d69f7cdd47f298e72a7a2b765ac9f2434a0495ec0cdca6097982786695a7c310a574010&scene=21#wechat_redirect "五十八、Fluent UDF调节宏ADJUST")

a. name:DEFINE_INIT宏的名称，可以是任意的

b. d：计算域的指针。对于多相流来说，d指针是指向混合域的。由于没有传递cell和thread，因此为了对网格物理量进行操作，需要对计算域d下的线程thread和thread下的网格cell进行遍历。可参考文章：

c. 此函数无返回值

d. DEFINE_INIT函数每次初始化执行一次，只要选择初始化后，会自动执行。

举个例子：让温度场是y轴坐标的函数

T=273+20\*y


​      
​    1. 


​            
​               #include "udf.h"


​                                                        

               DEFINE_INIT(my_init_func, d)


​                     

               {


​                            

                   cell_t c;


​             
​              
​                 


​          
​             
​    8. 


​             


​          
​             
​    9. 


​             
​              
​                 
​              
​               
​                  
​                   Thread *t;


​             
​              
​                 


​          
​             
​    10. 


​              


​          
​             
​    11. 


​              
​               
​                  
​               
​                
​                   
​                    real xc\[ND_ND\];


​              
​               
​                  


​          
​             
​    12. 


​              


​          
​             
​    13. 


​              
​               
​                  
​               
​                
​                   /* 遍历计算域d内的所有线程t */


​              
​               
​                  


​          
​             
​    14. 


​              


​          
​             
​    15. 


​              
​               
​                  
​               
​                
​                   
​                    thread_loop_c(t, d)


​              
​               
​                  


​          
​             
​    16. 


​              


​          
​             
​    17. 


​              
​               
​                  
​               
​                
​                   
​                    {


​              
​               
​                  


​          
​             
​    18. 


​              


​          
​             
​    19. 


​              
​               
​                  
​               
​                
​                   /* 遍历t下的所有网格 */


​              
​               
​                  


​          
​             
​    20. 


​              


​          
​             
​    21. 


​              
​               
​                  
​               
​                
​                   
​                        begin_c_loop_all(c, t)


​              
​               
​                  


​          
​             
​    22. 


​              


​          
​             
​    23. 


​              
​               
​                  
​               
​                
​                   
​                        {


​              
​               
​                  


​          
​             
​    24. 


​              


​          
​             
​    25. 


​              
​               
​                  
​               
​                
​                   
​                            C_CENTROID(xc, c, t);


​              
​               
​                  


​          
​             
​    26. 


​              


​          
​             
​    27. 


​              
​               
​                  
​               
​                
​                   
​                            C_T(c, t) = 273.+20*xc\[1\];


​              
​               
​                  


​          
​             
​    28. 


​              


​          
​             
​    29. 


​              
​               
​                  
​               
​                
​                   
​                        }


​              
​               
​                  


​          
​             
​    30. 


​              


​          
​             
​    31. 


​              
​               
​                  
​               
​                
​                   end_c_loop_all(c, t)


​              
​               
​                  


​          
​             
​    32. 


​              


​          
​             
​    33. 


​              
​               
​                  
​               
​                
​                   
​                    }


​              
​               
​                  


​          
​             
​    34. 


​              


​          
​             
​    35. 


​              
​               
​                  
​               
​                
​                   
​                }


​              
​               
​                  


​         
​            
​       ![](https://image.cubox.pro/cardImg/4ltmj5e47kvkh1oplrhrw19mt57u6ftjbpvp4w52wssoh4urxu.png?imageMogr2/quality/90/ignore-error/1)


​       
{#code}

### {#t20}**2.3 加载DEFINE_INIT**

编译型UDF界面如下图，上面有两个框Source Files和Header Files，Source Files表示源文件，就是编写好的UDF文件；

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fbdc25df5847eae321b3ee44a69b1db59.png&valid=false)

Header Files表示头文件，只有当UDF很复杂，为了使UDF模块化才需要从这里导入头文件。UDF自带了很多头文件如udf.h，但是这些头文件不需要从这里导入。

首先点击Add，选中编写好的UDF后导入，然后点击Build，如果UDF没有问题，则不会出现任何报错信息（只要控制界面有error，则说明有问题）。

在没有报错的前提下，点击Load，则UDF加载成功。关于UDF报错问题，建议大家看看文章四十九、五十和五十一。如果没有报错，控制台应该会显示下面的信息，其中就有各种DEFINE宏的name

和ADJUST宏类似，DEFINE_INIT需要hook使用。如果没有hook，即使加载成功，也不能调用。

点击Function Hooks，会弹出所有需要hooks界面

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fa2bf81ca08bf19ced0ad5898dd7dcb81.png&valid=false)

下面的图中包含很多宏，即当使用这些DEFINE宏时，都必须hook才能正常使用。比如DEFINE_EXECUTE_AT_END、DEFINE_INIT等，对于DEFINE_INIT宏，需要先点击Initialization宏的Edit进行设置界面

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Feed8455a8712060fa1b4f3efb690fc39.png&valid=false)

选中编写好的UDF宏名称，点击Add，宏名称将从左栏转入到右栏，单击OK，则表示hook成功。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F660f9b893477b7ac94db3b572ba549e3.png&valid=false)

hook成功后，我们还是使用刚才的例子，打开能量方程，进入初始化界面，点击标准初始化

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fa94f582d4565be213011320790ccf4bd.png&valid=false)

查看温度云图是否按照UDF的方式初始化成功

温度云图

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F6a4a4bf3b8ee9e0e9be710ca44b08214.png&valid=false)

X=2.5m处温度曲线图

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F33beeab9d577ed513f12531ed416cc8c.png&valid=false)

初始化成功，这里提供本文所需的所有文件，包括mesh、cas、dat和udf文件等

原文链接

[一文说清楚Fluent初始化操作(标准+混合初始化+Patch+UDF)什么是初始化？？很容易理解，所谓初始化就是给方程一个初始值。我们使用的所以数值模拟软件包括Fluent都是通过迭代的方式对方程组进行求解的，那么给出迭代的第一组值就被称为初始化。![](https://image.cubox.pro/cardImg/6d6r72ounbhy70a5vtwhutqav30sopviatkymy9wdtpsqk4b0a.png?imageMogr2/quality/90/ignore-error/1)https://mp.weixin.qq.com/s/I6F4XWZruEo_VeToomBodg](https://mp.weixin.qq.com/s/I6F4XWZruEo_VeToomBodg "一文说清楚Fluent初始化操作(标准+混合初始化+Patch+UDF)")

[跳转到 Cubox 查看](https://cubox.pro/my/card?id=7204796670482056827)
