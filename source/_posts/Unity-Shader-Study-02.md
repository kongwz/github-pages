---
title: Unity-Shader(二)漫反射光照模型
date: 2018-08-30 15:03:36
author: kongwz
tags:
  - Unity Shader
categories:
  - Unity
comments: true
---

先来看一下基本光照模型中的漫反射部分的计算公式。
![慢反射公式](http://ophmqxrq8.bkt.clouddn.com/manfanshe.png)

## 逐顶点光照实现

在Pass 中指明 光照模式

```
Tags{"LightMode" = "ForwardBase"}
```

<!--more-->

**LightMode标签支持的渲染路径设置选项**

标签名 | 描述
---|---
Always | 不管使用那种渲染路径，该pass总是会被渲染，但不会计算任何光照
ForwardBase | 用于**前向渲染**。该pass会计算环境光、最重要的平行光、逐顶点/SH光源和Lightmaps
ForwardAdd | 用于**前向渲染**。该pass会计算额外的逐像素光源，每个pass对应一个光源。
Deferred | 用于**延迟渲染**。该Pass会渲染G缓冲（G-buffer）
ShadowCaster | 把物体的深度信息渲染到阴影映射文理（shadowmap）或一张深度文理中
PrepassBase | 用于**遗留的延迟渲染**。该pass会渲染法线和高光反射的指数部分。
PrepassFinal | 用于**遗留的延迟渲染**。该pass通过合并文理、光照和自发光来渲染得到最后的颜色
Vertex、VertexLMRGBM 和 VertexLM | 用于**遗留的顶点照明渲染**

我们这里主要是来看一下 如何实现逐顶点的漫反射光照，看一下顶点着色器
```
v2f vert(a2v v)
{
	v2f o ;
	o.pos = UnityObjectToClipPos( v.vertex);
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	fixed3 worldNormal = normalize(mul(v.normal , (float3x3)unity_WorldToObject));
	fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLight));
	o.color = ambient + diffuse;
	return o;
}
```

顶点着色器最基本的任务就是把顶点位置从模型空间转换到裁剪空间中，因此我们使用UNITY_MATRIX_MVP Unity内置的模型*世界*投影矩阵来完成坐标转换，这里因为我的版本比较高，unity 会自动

Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'。

然后使用UNITY_LIGHTMODEL_AMBIENT得到环境光部分。

计算漫反射光照我们需要四个参数

现在我们已知材质的漫反射颜色（_Diffuse）以及顶点法线v.normal，所以 我们还需要知道光源颜色和强度以及方向。

Unity提供了一个内置变量_LightColor0来访问该pass 处理的光源颜色和强度信息。（注意：要想得到正确的值需要定义合适的LightMode标签）

光源方向可以通过_WorldSpaceLightPos0来得到。（这里只考虑只有一个光源，且是平行光）

我们计算法线和光源方向的点积时，需要保证他们在同一个坐标系下，这里a2v中的顶点法线是模型空间下的，我们选择世界空间坐标系，所以需要先转换一下坐标系，将法线的坐标系转换到 世界空间坐标系下，我们可以使用**顶点变换矩阵的逆转置矩阵对法线**进行相同的变换，因此我们首先得到模型空间到世界空间的变换矩阵的逆矩阵**_World2Object**等同于新版本的**unity_WorldToObject**，然后通过调换它在mul函数中的位置，得到和转置矩阵相同的矩阵乘法。

得到世界空间中的法线和光源方向，然后进行归一化，且要防止是负值，因此使用了**saturate**函数，作用是可以吧参数截取到[0，1]的范围内

```
fixed3 worldNormal = normalize(mul(v.normal , (float3x3)unity_WorldToObject));
fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

saturate(dot(worldNormal , worldLight))
```

然后 再与光源颜色和强度以及材质的漫反射颜色相乘得到最终的漫反射光照。

```
fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLight));
```

最后，把环境光和漫反射光部分相加，得到最终效果

```
o.color = ambient + diffuse;
return o;
```
全部代码：

```
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unlit/Study_2"
{
	Properties
	{
		_Diffuse("Diffuse",Color) = (1.0,1.0,1.0,1.0)
	}
	SubShader
	{
		Pass{
			Tags{"LightMode" = "ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Diffuse;
			struct a2v{
				float4 vertex : POSITION;
				float2 normal : NORMAL;
			};

			struct v2f{
				float4 pos : SV_POSITION;
				float3 color : COLOR;
			};

			v2f vert(a2v v){
				v2f o ;
				o.pos = UnityObjectToClipPos( v.vertex);

				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(mul(v.normal , (float3x3)unity_WorldToObject));
				fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLight));
				o.color = ambient + diffuse;
				return o;
			}

			float4 frag(v2f i): SV_Target{
				return fixed4(i.color , 1.0);
			
			}
			
			ENDCG
		}
		
	}
	Fallback "Diffuse"
}

```

![](http://ophmqxrq8.bkt.clouddn.com/diffuse1.png)


通过上面的效果可以看到，会有一些锯齿

对于细分程度较高的模型，逐顶点光照可以表现出来较好的效果，但是对于一些细分程度较低的模型，会有一些视觉问题，如锯齿。

## 逐像素光照

还是拿上面的代码做一下修改。

```
struct v2f{
	float4 pos : SV_POSITION;
	float3 worldNormal : TEXCOORD0;
};
```

顶点着色器不需要计算光照模型，所以只需要将世界空间下的法线传递给片元着色器即可：

```
v2f vert(a2v v){
	v2f o ;
	o.pos = UnityObjectToClipPos( v.vertex);
	o.worldNormal = mul(v.normal , (float3x3)_World2Object);
	return o;
}
```

片元着色器需要计算漫反射光照模型：

```
float4 frag(v2f i): SV_Target{
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
	fixed3 color = ambient + diffuse;
	return fixed4(color,1.0);			
}
```

全部代码：

```
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'

Shader "Unlit/Study_3"
{
	Properties
	{
		_Diffuse("Diffuse",Color) = (1.0,1.0,1.0,1.0)
	}
	SubShader
	{
		Pass{
			Tags{"LightMode" = "ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Diffuse;
			struct a2v{
				float4 vertex : POSITION;
				float2 normal : NORMAL;
			};

			struct v2f{
				float4 pos : SV_POSITION;
				float3 worldNormal : TEXCOORD0;
			};

			v2f vert(a2v v){
				v2f o ;
				o.pos = UnityObjectToClipPos( v.vertex);
				o.worldNormal = mul(v.normal , (float3x3)unity_WorldToObject);
				return o;
			}

			float4 frag(v2f i): SV_Target{
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
				fixed3 color = ambient + diffuse;
				return fixed4(color,1.0);			
			}
			
			ENDCG
		}
	}
}

```

![](http://ophmqxrq8.bkt.clouddn.com/diffuse2.png)

通过效果图可以看出，现在表现的比较平滑，并没有锯齿了，但是可以法线，在光无法到达的地方模型外观是全黑的，没有任何明暗变化，所以下面一起看一下 **半兰伯特光照模型**

## 半兰伯特光照模型

> 公式

![](http://ophmqxrq8.bkt.clouddn.com/diffuse4.png)


使用 逐像素中的片元着色器代码，套用半兰伯特公式

```
float4 frag(v2f i): SV_Target{
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	fixed halfLambert = dot(worldNormal , worldLightDir) * 0.5 + 0.5;
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * halfLambert;
	fixed3 color = ambient + diffuse;
	return fixed4(color , 1.0);
	//fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
	//fixed3 color = ambient + diffuse;
	//return fixed4(color,1.0);			
}			
```

对比一下效果图，左一是逐顶点漫反射光照，中间是逐像素漫反射光照，最右边是半兰伯特光照模型

![](http://ophmqxrq8.bkt.clouddn.com/diffuse3.png)


*本文所写内容参考《UnityShader 入门精要》。*