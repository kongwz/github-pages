---
title: Unity-Shader-小记
date: 2018-06-25 15:18:33
tags:
  - Unity Shader
categories:
  - Unity
comments: true
---

- 初学Shader记录一下 **float4 tex2D(sampler2D samp, float2 s)**
	
2D纹理采样，CG内置函数。 
内部实现分为以下几步： 
1. 用图片的宽高度乘以uv数值，得到像素坐标。widthPixel=samp.x*s.x;heightPixel=samp.y*s.y; 
2. 因为取到的数值基本上都是带有小数点的，也就是说不是一个整数，这个时候，需要看图片的过滤设置了。也就是Unity3d的图片设置中的Filter Mode。 
3. Filter Mode是Point，不过滤，就会取像素点最靠近的整数，也就是四舍五入，得到像素点的坐标，然后出去图片中，这个坐标的颜色。 
4. 双线性过滤，会取目标像素的附近4个像素，然后进行插值计算，得到平均颜色值，作为最终颜色。适合纹理由小放大过程中，出现的“马赛克”。 
5. 三线性过滤，在双线性过滤的基础上考虑到了深度LOD，会进行两次双线性过滤，来使不同的LOD等级纹理中，更加平滑的过渡。