---
title: Unity-Shader(三)高光反射&Blinn-Phong光照模型
date: 2018-09-03 14:37:57
author: kongwz
tags:
  - Unity Shader
categories:
  - Unity
comments: true
---

先看一下基本光照模型中的高光反射部分的计算公式：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular1.png)

## 逐顶点光照实现 高光反射

<!--more-->

> 分析

在Properties语义块中声明了三个属性：
```
Properties
{
	_Diffuse ("Diffuse" , Color) = (1,1,1,1)
	_Specular ("Specular" , Color) = (1,1,1,1)
	_Gloss ("Gloss" , Range(8.0,256)) = 20
}
```
_Specular用来控制材质的高光反射的颜色

_Gloss用于控制高光区域的大小

其他的代码基本和往常一样，不细说了

```
Tags { "LightMode" = "ForwardBase" }

CGPROGRAM
#pragma vertex vert
#pragma fragment frag
#include "Lighting.cginc"

fixed4 _Diffuse;
fixed4 _Specular;
float _Gloss;

struct a2v{
	float4 vertex : POSITION;
	float3 normal : NORMAL;
};
struct v2f{
	float4 pos : SV_POSITION;
	float3 color : COLOR;
};
```

在顶点着色器中计算了包含高光反射的光照模型

```
v2f vert(a2v v){
	v2f o;
	o.pos = UnityObjectToClipPos( v.vertex);
	//得到环境光
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	//将法线从模型空间变换的世界空间
	fixed3 worldNormal = normalize(mul(v.normal , (float3x3)unity_WorldToObject));
	//在世界空间中光照方向
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
	fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(unity_ObjectToWorld , v.vertex).xyz);
	fixed3 specular = _LightColor0.rgb * _Specular.rgb *pow(saturate(dot(reflectDir , viewDir)),_Gloss);
	o.color = ambient + diffuse + specular;
	return o;
}
```

首先计算了入射光线防线关于表面法线的反射方向reflectDir。犹豫Cg的reflect函数的入社方向要求是由光源指向交点处的，因此需要对worldLightDir取反再传给reflect函数。

然后通过——WorldSpaceCameraPos得到世界空间中的摄像机位置，再把顶点位置从模型空间变换得到世界空间下，再通过——WorldSpaceCameraPos相减即可得到世界空间下的视角方向。

最后 

```
fixed4 frag(v2f i) : SV_Target{
	return fixed4(i.color , 1.0);
}
```

> 效果

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular2.png)

**使用逐顶点着色器的方法得到的高光效果有比较大的问题，可以从效果图中看到有明显的不平滑，因为高光反射部分的计算是非线性的，而在顶点着色器中计算光照在进行插值的过程是线性的，破换了原计算的非线性关系。**
> 代码

```
// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unlit/Specular_1"
{
	Properties
	{
		_Diffuse ("Diffuse" , Color) = (1,1,1,1)
		_Specular ("Specular" , Color) = (1,1,1,1)
		_Gloss ("Gloss" , Range(8.0,256)) = 20
	}
	SubShader
	{
		Pass{
			Tags { "LightMode" = "ForwardBase" }

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

			struct a2v{
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};
			struct v2f{
				float4 pos : SV_POSITION;
				float3 color : COLOR;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos = UnityObjectToClipPos( v.vertex);
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(mul(v.normal , (float3x3)unity_WorldToObject));
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
				fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(unity_ObjectToWorld , v.vertex).xyz);
				fixed3 specular = _LightColor0.rgb * _Specular.rgb *pow(saturate(dot(reflectDir , viewDir)),_Gloss);
				o.color = ambient + diffuse + specular;
				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				return fixed4(i.color , 1.0);
			}


			ENDCG
		}
		
	}
	Fallback "Specular"
}

```

## 逐像素 实现高光反射模型

> 分析

在原来代码的基础上进行修改

修改顶点着色器的输出结构v2f

```
struct v2f{
	float4 pos : SV_POSITION;
	float3 worldNormal : TEXCOORD0;
	float3 worldPos : TEXCOORD1;
};
```

修改顶点着色器

```
v2f vert(a2v v){
	v2f o;
	o.pos = UnityObjectToClipPos( v.vertex);
	o.worldNormal = mul(v.normal , (float3x3)unity_WorldToObject);
	o.worldPos = mul(unity_ObjectToWorld , v.vertex).xyz;
	return o;
}
```

修改片元着色器：

```
fixed4 frag(v2f i) : SV_Target{
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
	fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
	fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir , viewDir)) , _Gloss);
	return fixed4(ambient + diffuse + specular , 1.0);
}
```

> 效果：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular3.png)

可以看出，使用逐像素实现高光反射，使得高光效果更加平滑

> 全部代码：

```
// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unlit/Specular_2"
{
	Properties
	{
		_Diffuse ("Diffuse" , Color) = (1,1,1,1)
		_Specular ("Specular" , Color) = (1,1,1,1)
		_Gloss ("Gloss" , Range(8.0,256)) = 20
	}
	SubShader
	{
		Pass{
			Tags { "LightMode" = "ForwardBase" }

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

			struct a2v{
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};
			struct v2f{
				float4 pos : SV_POSITION;
				float3 worldNormal : TEXCOORD0;
				float3 worldPos : TEXCOORD1;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos = UnityObjectToClipPos( v.vertex);
				o.worldNormal = mul(v.normal , (float3x3)unity_WorldToObject);
				o.worldPos = mul(unity_ObjectToWorld , v.vertex).xyz;
				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
				fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
				fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir , viewDir)) , _Gloss);
				return fixed4(ambient + diffuse + specular , 1.0);
			}


			ENDCG
		}
		
	}
	Fallback "Specular"
}

```

## Blinn-Phong 光照模型

> 公式：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular6.png)


修改逐像素实现高光反射的片元着色器的代码：

```
fixed4 frag(v2f i) : SV_Target{
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
	fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
	fixed3 halfDir = normalize(worldLightDir + viewDir);
	fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(worldNormal , halfDir)) , _Gloss);
	return fixed4(ambient + diffuse + specular , 1.0);
}
```

> 效果：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular4.png)

> 全部代码

```
// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unlit/Specular_3"
{
	Properties
	{
		_Diffuse ("Diffuse" , Color) = (1,1,1,1)
		_Specular ("Specular" , Color) = (1,1,1,1)
		_Gloss ("Gloss" , Range(8.0,256)) = 20
	}
	SubShader
	{
		Pass{
			Tags { "LightMode" = "ForwardBase" }

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

			struct a2v{
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};
			struct v2f{
				float4 pos : SV_POSITION;
				float3 worldNormal : TEXCOORD0;
				float3 worldPos : TEXCOORD1;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos = UnityObjectToClipPos( v.vertex);
				o.worldNormal = mul(v.normal , (float3x3)unity_WorldToObject);
				o.worldPos = mul(unity_ObjectToWorld , v.vertex).xyz;
				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal , worldLightDir));
				fixed3 reflectDir = normalize(reflect(-worldLightDir , worldNormal));
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
				fixed3 halfDir = normalize(worldLightDir + viewDir);
				fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(worldNormal , halfDir)) , _Gloss);
				return fixed4(ambient + diffuse + specular , 1.0);
			}


			ENDCG
		}
		
	}
	Fallback "Specular"
}

```

## 最后

 三种类型高光反射的效果对比
从左到右分别是 逐顶点高光反射、逐像素高光反射、Blinn-Phong光照模型

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/specular5.png)

可以看出，Blinn-Phong模型的高光反射部分看起来更大更亮，在实际渲染中，绝大多数情况也都会选择Blinn-Phong光照模型

### 划重点

**使用 normalize(_WorldSpaceLightPos0.xyz);来得到光源方向**

**使用 normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);来得到视角方向**

*本文所写内容参考《UnityShader 入门精要》。*