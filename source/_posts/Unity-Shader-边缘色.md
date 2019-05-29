---
layout: post
title: Unity-Shader-边缘色
date: 2018-06-25 18:04:54
author: kongwz
tags:
  - Unity Shader
categories:
  - Unity
comments: true
---
## 边缘着色。代码控制改变颜色和控制开关

先看代码 

Shader 代码


<!--more-->
```

Shader "Custom/bianyuancolor" 
{
	Properties
	{
		_MainTex("【文理】Texture" , 2D) = "white"{}
		_BumpMap("【凹凸文理】Bumpmap" , 2D) = "bump"{}
		_RimColor("【边缘颜色】Rim Color",Color) = (0.17, 0.36 , 0.81 ,0.0)
		_RimPower("【边缘颜色强度】Rim Power" , Range(0.6 , 9.0)) = 1.0
		_ShowRimColor("【是否显示边缘颜色】,Show Rim Color" , Int) = 1
	}

	SubShader
	{
		Tags{"RanderType"="Opaque"}
		CGPROGRAM

		#pragma surface surf Lambert

		struct Input
		{
			float2 uv_MainTex;
			float2 uv_BumpMap;
			float3 viewDir; //观察方向

		};

		sampler2D _MainTex;
		sampler2D _BumpMap;
		float4 _RimColor;
		float _RimPower;
		int _ShowRimColor;

		void surf(Input IN , inout SurfaceOutput o)
		{
			//2D 文理查询
			o.Albedo = tex2D(_MainTex , IN.uv_MainTex).rgb;

			o.Normal = UnpackNormal(tex2D(_BumpMap , IN.uv_BumpMap));
			half rim = 1.0 - saturate(dot(normalize(IN.viewDir) , o.Normal));
			if(_ShowRimColor == 1)
				o.Emission = _RimColor.rgb * pow(rim , _RimPower);
		}
		ENDCG
	}

	Fallback "Diffuse"
}

```

saturate函数（saturate(x)的作用是如果x取值小于0，则返回值为0。如果x取值大于1，则返回值为1。若x在0到1之间，则直接返回x的值

C# 代码


```bash

  void Update()
    {
        if (Input.GetKeyDown(KeyCode.K)) {
            obj.GetComponent<MeshRenderer>().material.SetInt("_ShowRimColor", 0);
        }
        if (Input.GetKeyUp(KeyCode.K))
        {
            obj.GetComponent<MeshRenderer>().material.SetInt("_ShowRimColor", 1);
        }
        if (Input.GetKeyDown(KeyCode.R))
        {
            obj.GetComponent<MeshRenderer>().material.SetColor("_RimColor", Color.red);
        }
        if (Input.GetKeyDown(KeyCode.Y))
        {
            obj.GetComponent<MeshRenderer>().material.SetColor("_RimColor", Color.yellow);
        }
        if (Input.GetKeyDown(KeyCode.B))
        {
            obj.GetComponent<MeshRenderer>().material.SetColor("_RimColor", Color.blue);
        }
        if (Input.GetKeyDown(KeyCode.G))
        {
            obj.GetComponent<MeshRenderer>().material.SetColor("_RimColor", Color.green);
        }
    }

```

### 先看一下关闭打开效果图

![效果](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/bianyuanse.gif)


### 代码控制颜色
![效果](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/bianyuanse1.gif)
