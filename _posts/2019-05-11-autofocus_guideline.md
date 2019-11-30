---
layout:     post
title:      手机相机自动对焦(AF)
subtitle:   
date:       2019-05-11
author:     kinlin
header-img: img/theworld.jpg
catalog: true
tags:                            
    - 技术
---


### 什么是自动对焦(auto focus)

自动对焦是指手机中，利用sensor、控制软件和马达来实现改变成像距离，使sensor能获得清晰的照片的过程。在手机方案里，最重要的部分就是`自动`。想象一下，小时候都玩过的用放大镜来聚焦阳光取火的游戏。在阳光下移动放大镜，使得聚焦的光点最小，几秒钟就能点燃一张纸巾，这里那个`最亮的光点`就是焦点。

同样，手机摄像头结构上也是有感光元件和镜头组成，通过镜头移动来改变焦点以获得最清晰的图片。

### 来自生物学的礼物：人类的眼睛是如何对焦的？

手机摄像头的发展，很大程度上在吸取人眼的功能。比如，当我们扭头看向不同的物体时，对焦就已经完成，甚至不需要刻意去做什么； 能够适应不同的光照条件，当从正常光照条件下走向一间漆黑一片的房间后，经过很短时间的调节就能借助极其微弱的光照条件看清楚物体，倘若这时候拿出一部手机，很大可能获得的是黑乎乎一片加大片的噪点。 而且我们的眼睛天生就有防抖功能，比如我们在一个轻微抖动的车上，依然能看清车外的美景，但是如果相机没有防抖功能，那很大可能是无法完成对焦的。

那人眼的机构是什么样的呢？
![human_eye](/img/text_img/human_eye_anatomy.jpg)

> 人眼的组成部分有：
> Optical Nerve: 视觉神经
> Retina: 视网膜
> Lens: 晶状体
> Conenea: 
> Cilliary Muscle: 睫状肌
> Pupil: 瞳孔
> Iris: 虹膜

摄像头的结构：
![phone-camera](/img/text_img/phone-camera.jpg)
![phone-camera-2](/img/text_img/phone-camera_2.jpg)

可以看到，从结构上来说，与人眼是非常类似的。 人眼通过睫状肌调节来改变焦距，而手机通过音圈马达条件镜头实现对焦(需要注意的是早期摄像头不带马达，因此是定焦的)。人眼通过视网膜感光并通过视觉神经传输信号，而摄像头则是通过CMOS来感光。

但是人眼和器件的感光能力是有差异的。以下图为例. 从频谱上来说，CMOS的感光能力与人眼是有差距的。这虽然意味着从CMOS最终的成像其实与人眼所看到的是不一样的，但是有时候正好为我们所用，比如在黑暗环境下就能让我们看到更多东西。不过正常情况下，是需要滤光片过滤掉一些频谱的光源

> 某 CMOS 感光能力
![cmos_sense](/img/text_img/cmos_sense.png)

> 人眼感光能力
![eye_sense](/img/text_img/eye_sense.png)


而在CMOS的排布上似乎也是模仿人眼。人眼的视锥细胞是各有分工，有的对红色敏感，有的对绿色敏感。而在摄像头上，人们给每一个像素都加上了红色或绿色或蓝色的滤光片过滤，最终的图像是一个二维的，由单一红点、绿点、蓝点组成Bayer pattern。厂家宣传的几百万几千万像素指的就是单色的像素。如何形成我们看到的图像则需要经过debayer或者叫demosaic的过程插值还原。
demosaic可以参考一些文章，或者开源库。比较常规基础。
![bayer_pattern](/img/text_img/bayer_pattern.png)
![debayer](/img/text_img/debayer.png)


还得加上这张图，表示人眼的
* [](https://www.cnbeta.com/articles/tech/850243.htm)

### 那软件算法呢？
人眼对焦的时候，我们从来没有考虑过一个问题，就是`清晰`这个概念。对人类而言，清晰似乎就是一种感觉，但是当手机在拍照的时候，它是通过什么指标判断是否是清晰的呢？如果不解决这个问题，手机无法通过程序控制镜头移动到最清晰的位置。

> 怎么定义一张图片是否清晰？
>>ISP 内部，使用一个简单的filter kernel，对选择的roi进行逐行处理得到特定区域的FV值。根据horizontal和vertical方向区分为H1/V1, 当我们进行对焦过程中，这一计算过程是每一帧都有，所以每次lens move结束，都可以得到一个fv值，最简单的hill-climb 算法就是找出FV值最大的区域，就是最清晰的。此时可以结束对焦，不再移动lens。

但相机发展至今，使用FV值这种方式太慢了，后面随着sensor厂家的推陈出新（对焦的进步几乎就是器件和算法的综合），推出了相位对焦（PDAF）。PDAF可以直接得出距离，根据PD点的差值，这使得我们不再需要反复移动lens来找到这个FV而是直接得出最后的距离。

PD的引入是从iphone5开始的，至今已经有好几年了，sensor厂的pdsensor也从普通的mask形态的pd，改进为2x1的sparse PD，到dual-pd.到现在最新的2x2 pd imx689sensor, 每一步都是整个产业链的配合，使得我们的对焦能在0.1秒内就完成（3帧以内， 30fps）。

但从今年2019年的趋势看，pd几乎已经见顶了。因为现在的sensor pixel数量已经从当年的12M进化到48M甚至108M，这种恐怖的pixel点数，使得PD计算耗时倍增。但是为了对焦质量，减少PD点也是不可取的，而且对焦已经不仅仅是对准一个区域，而是需要准确识别遮挡，这对PD输出质量要求非常之高。于是
>一方面，PD的计算开始像当年的FV一样，从软件放到硬件去计算，ISP内部继承了专门的PD模块去处理巨量的PD点。

>另一方面，TOF开始逐渐盛行。因为激光测距有几个好处，一个是直接的物理测距没有什么复杂的计算，AF就是希望知道这个信息。另一个就是在激光测距范围内，不受低光照的影响，而PD在低光照情况下是受到影响的。



### 接下来还有哪些可以改进的？
玩法的创新：希区柯克变焦？




### 参考
* [autofocus  wiki](https://en.wikipedia.org/wiki/Autofocus)
* [physics-behind-accommodation-of-the-human-eye](https://tuitionphysics.com/2016-mar/physics-behind-accommodation-of-the-human-eye/)
* [知乎：人眼是如何对焦和防抖的](https://www.zhihu.com/question/34765747)
* [whats-the-difference-between-a-camera-and-a-human-eye](https://medium.com/photography-secrets/whats-the-difference-between-a-camera-and-a-human-eye-a006a795b09f)
