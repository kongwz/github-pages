---
title: unity shader入门语法
date: 2017-05-17 17:14:52
author: kongwz
tags:
  - unity shader
categories:
  - unity shader
comments: true
---

# 前言
从毕业一直从事游戏行业有两年多了吧，却从没真正的接触过游戏渲染这块。一直打算学shader却（总是没时间），再不学就要被淘汰咯~
## 先说一下资料吧
我看的是这个书，讲的挺细的，从数学基础到shader案例教程都很详细
>Unity Shader 入门精要

需要的自行京东
好了不说了，上菜
<!--more-->
# Shader语法

## Properties语义块支持的属性类型
| 属性类型        | 默认值的定义语法           | 例子  |
| ------------- |:-------------:| -----:|
| Int      | number | _Int("Int",Int)=2 |
| Float      | number      |   _Float("Float",Float)=1.5 |
| Range(min,max) | number      |    _Range("Range".Range(0.0,5.0))=3.0 |
| Color | (number,number,number,number)      |    _Color("Color",Color)=(1,1,1,1) |
| Vector | (number,number,number,number)      |    _Vector("Vector",Vector)=(2,3,6,1) |
| 2D | "defaulttexture"{}      |    _2D("2D",2D)=""{} |
| Cube | "defaulttexture"{}      |    _Cube("Cube",Cube)="white"{} |
| 3D | "defaulttexture"{}      |    _3D("3D",3D)="black"{} |

## 示例代码(语义属性)
``` bash
Shader "TastShader"{
    Properties{
        //Numbers and Shiders
        _Int ("Int",Int)=2
        _Float ("Float",Float) = 1.5
        _Range ("Range",Range(0.0,5.0)) = 3.0
        //Color and Vectors
        _Color ("Color",Color) = (1,1,1,1)
        _Vector ("Vector",Vector) = (2,3,6,1)
        //Textures
        _2D ("2D",2D) = ""{}
        _Cube ("Cube",Cube) = "white"{}
        _3D ("3D",3D) = "black"{}
    }
    FallBack "Diffuse"
}
```

## SubShader语义块中包含的定义
```bash
SubShader{
    //可选的
    [Tags]
    //可选的
    [RanderSetup]
    Pass{

    }
    //Other Passes
}
```
SubShader 中定义了一系列Pass以及可选的**状态([RenderSetup])** 和**标签([Tags])**设置。每个Pass定义了一次完整的渲染流程，但如果Pass的数目过多，往往会造成渲染性能下降，所以要尽量降低Pass的使用数目
### 常见的渲染状态设置选项

| 状态名称        | 设置指令           | 解释  |
| ------------- |:-------------:| -----:|
| Cull      | Cull Back、Front、Off | 设置剔除模式：剔除背面/正面/关闭剔除 |
| ZTest      | ZTest Less Greater、LEqual、GEqual、NotEqual、Always | 设置自身测试时使用的函数 |
| Zwrite      | Zwrite On、Off | 开启/关闭深度写入 |
| Blend      | Blend SrcFactor DstFactor | 开启并设置混合模式 |

未完待续……