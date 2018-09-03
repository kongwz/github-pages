---
title: Unity-Shader(一)顶点着色去与片段着色器的通讯&语义
date: 2018-08-24 15:27:24
author: kongwz
tags:
  - Unity Shader
categories:
  - Unity
comments: true
---

每次说想学东西，却没有时间，嫌工作忙，加班..最近有时间了，还是没能学下去。其实不是没时间，不是忙，只是没有真正的想过怎么去学习，或者说自学能力差，自控能力差。

索性就从最基础的学吧，做笔记，内容很简单，但能坚持才是重点。

## Shader 顶点着色器与片元着色器之间的通讯

> 结构体

这里顶点着色器与片元着色器的通讯，是通过结构体来传递的，那么就先说一下结构体的定义吧

```
struct structName{
	Type Name : Semantic;
	Type Name : Semantic;
	...
};
```
Semantic 语义是不可以省略的，结构体结束后要有 分号 “；”

<!--more-->

下面定义一下 今天使用的 两个结构体

第一个 从模型中获取数据

```
//结构体的名字 a2v a是应用，v 是顶点着色器
struct a2v{
	//POSITION 语义告诉Unity 使用模型空间的顶点坐标填充 vertex变量
	float4 vertex : POSITION;
	//NORMAL 语义告诉Unity 使用模型空间的法线方向 填充 normal变量	
	float3 normal : NORMAL;
	//TEXCOORD0 语义告诉Unity 使用模型的第一套文理坐标填充texcoord变量
	float4 texcoord : TEXCOORD0;
};
```

第二个 顶点着色器和片元着色器的通讯结构体

```
//定义 顶点着色器与片段着色器之间的通讯 结构体 用于 顶点着色器的输出，和 片段着色器的输入
			//需要注意 变量后面的语义，分别代表的它储存的信息
			struct v2f {
				//SV_POSITION 语义告诉Unity pos 里包含顶点在剪裁空间中的位置信息
				float4 pos : SV_POSITION;
				//COLOR 语义用于储存 颜色信息
				fixed3 color : COLOR;
			};
```

这里需要注意一下，在顶点着色器和片元着色器的通讯结构体中，必须包含一个变量，它的语义是SV_POSITION，否则渲染器将无法得到裁剪空间中的顶点坐标，也就无法把顶点渲染到屏幕上，这里的SV 代表的是**系统数值**。

至于后面的COLOR语义的变量，可以有也可以没有，因为颜色是可以自定义的。


结构体定义好了 ，下面就是使用了，跟通常一样，把结构体当做参数或者返回值来用就好了，上代码...

```

// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

/*
	顶点着色器与片段着色器之间的通讯 v2f

*/
Shader "Unlit/Study_1"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	SubShader
	{
		Tags { "RenderType"="Opaque" }
		LOD 100

		Pass
		{
			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag
			#include "unityCG.cginc"
			
			struct a2v{
				float4 vertex : POSITION;
				float3 normal : NORMAL;
				float4 texcoord : TEXCOORD0;
			};

			//定义 顶点着色器与片段着色器之间的通讯 结构体 用于 顶点着色器的输出，和 片段着色器的输入
			//需要注意 变量后面的语义，分别代表的它储存的信息
			struct v2f {
				//SV_POSITION 语义告诉Unity pos 里包含顶点在剪裁空间中的位置信息
				float4 pos : SV_POSITION;
				//COLOR 语义用于储存 颜色信息
				fixed3 color : COLOR;
			};

			v2f vert(a2v a) {
				v2f o ;
				o.pos = UnityObjectToClipPos( a.vertex);
				o.color = a.normal * 0.5 + fixed3(0.5,0.5,0.5);
				return o;
			}

			fixed4 frag(v2f i ) : COLOR{
				return fixed4(i.color , 1.0);
			}


			ENDCG
		}
	}
}

```

去看下一篇吧 ，写完了。。。




好吧再来点...

----------
## 语义
#### 从应用阶段传递模型数据给顶点着色器时的常用语义

语义 | 描述
---|---
POSITION | 模型空间中的顶点位置，通常是float4
NORMAL | 顶点法线，通常是float3类型
TANGENT | 顶点切线，通常是float4类型
TEXCOORDn | 该顶点的文理坐标，第一组，第二组..通常是float2或float4类型
COLOR | 顶点颜色，通常是float4或者fixed4类型

#### 从顶点着色器传递给片元着色器时的常用语义
语义 | 描述
---|---
SV_POSITION | 裁剪空间中的顶点坐标，结构体中必须包含一个用该语义修饰的变量。等同于Direct9中的POSITION
COLOR0 | 通常用于输出第一组顶点颜色，不是必须的
COLOR1 | 通常用于输出第二组顶点颜色，不是必须的
TEXCOORD0~TEXCOORD7 | 通常用于输出文理坐标，不是必须的

#### 片元着色器输出时的常用语义
语义 | 描述
---|---
SV_Target | 输出值将会存储到渲染目标（render target）中，等同于Direct9中的COLOR



*本文所写内容参考《UnityShader 入门精要》。*