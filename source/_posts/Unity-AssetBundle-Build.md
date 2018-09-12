---
title: Unity-AssetBundle打包
date: 2018-09-10 12:05:15
author: kongwz
tags:
  - Unity AssetBundle
categories:
  - Unity
comments: true
---

==本文所有内容针对Unity版本：5.x以后，我用的是2018.2.2f1==

## AssetBundle打包方式（一）

### 打包设置

在Project面板里选中一个资源，然后在Inspector面板中设置这个资源的AssetBundleName

![设置AssetBundleName](http://ophmqxrq8.bkt.clouddn.com/BuildAssetBundle1.png)

<!--more-->

这样以后，这个资源会被打包在一个名字叫Cube（这里是你设置的名字，随意的）AssetBundle中，注意：**如果想要将多个资源打包在一个bundle中，那么将多个资源的AssetBundleName设置成一样的即可**后面的Variant参数是设置Bundle的后缀名字，使用这个参数 可以更方便的进行多分辨率支持。

### 打包核心

打包核心代码：
```
BuildPipeline.BuildAssetBundles(Path, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows64);
```
> 参数
    
1. ##### outputPath:bundle目标文件存放目录
2. ##### BuildAssetBundleOptions 
    - **None**：默认的打包方式，采用LZMA压缩方式，会收集依赖。
    - **UncompressedAssetBundle**：打包不去采用任何压缩方式。
    - **CollectDependencies**：收集依赖的方式。
    - **CompleteAssets**：保证资源的完备性，会把该资源的和他所有的依赖打包到依噶AssetBundle中，默认开启。
    - **DisableWriteTypeTree**：不包含TypeTree类型信息（影响资源版本变化，可以让AssetBundle更小，加载更快；与4.x不同的是，对于移动平台，5.x下默认会将TypeTree信息写入AssetBundle）；
    - **DeterministicAssetBundle**：为资源维护固定ID（唯一标志AssetBundle，主要用来增量打包），unity5.x默认开启；
    - **ForceRebuildAssetBundle**:用于强制重打所有AssetBundle文件；
    - **IgnoreTypeTreeChanges**:判断AssetBundle更新时，是否忽略TypeTree的变化；
    - **AppendHashToAssetBundleName**：用于将Hash值添加在AssetBundle文件名之后，开启这个选项 可以直接通过文件名来判断哪些Bundle的内容进行了更新（4.x下普遍需要通过比较二进制等方法来判断，但在某些情况下即使内容不变重新打包，Bundle的二进制也会变化）；
    - **ChunkBasedCompression**：用于使用LZ4格式进行压缩，5.3新增，默认压缩格式为LZMA；
    - **StrictMode**：任何错误都将导致打包失败；
    - **DryRunBuild**：
    - **DisableLoadAssetByFileName**：不允许AB包通过文件名加载资源。
    - **DisableLoadAssetByFileNameWithExtension**：禁用通过使用扩展名加载AssetBundle。

3. ##### BuildTarget 选择打包平台，我这里是Windows所以选择的StandaloneWindows64，根据自己的情况选择。

### 打包代码

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class BundleTools : Editor {

    //根据设置的AssetBundleName将资源打包
    [MenuItem("AssetBundle/Create Bundele By Obj Name (Single)")]
    static void CreateBundleByObjName() {
        Caching.ClearCache();
        string Path = Application.dataPath + "/StreamingAssets";
        BuildPipeline.BuildAssetBundles(Path, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows64);
        AssetDatabase.Refresh();
    }
    //根据自己选中的资源，将他们打包到一个AssetBundle中
    [MenuItem("AssetBundle/Create Bundle By Select (ALL)")]
    static void CreateBundleBySelect() {
        string Path = Application.dataPath + "/StreamingAssets";
        AssetBundleBuild[] buildMap = new AssetBundleBuild[1];
        buildMap[0].assetBundleName = "enemyBundle";
        Object[] selects = Selection.GetFiltered(typeof(Object), SelectionMode.DeepAssets);
        string[] enemyAsset = new string[selects.Length];
        for (int i = 0; i < selects.Length; i++)
        {
            enemyAsset[i] = AssetDatabase.GetAssetPath(selects[i]);
        }
        buildMap[0].assetNames = enemyAsset;
        BuildPipeline.BuildAssetBundles(Path, buildMap, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows);
        AssetDatabase.Refresh();
    }
}

```

因为这个脚本是继承的Editor所以需要放在Editor目录下，如果没有这个目录，那就手动创建一个吧。

### 打包工具

使用AssetBundleBrowser可以判断是否有同一个资源被打包进了两个Bundle 中，可以方便的进行检查AssetBundle的依赖关系。
![](http://ophmqxrq8.bkt.clouddn.com/BuildAssetBundle2.png)

我这里的Cube和Capsule共同使用了一个材质球，然后把他们两个分别打包，并将材质球打包成newbundle，这样Cube和Capsule会共同依赖newbundle，可以看到途中Capsule的依赖关系。如果不将材质球单独打包的话会是这样的情况。
![](http://ophmqxrq8.bkt.clouddn.com/BuildAssetBundle3.png)

可以看到两个资源都会引用相同的材质球和贴图，导致资源重复。

出现这种情况可以，右键AssetBundle名字*Add Sibling > New bundle*然后选择有黄色叹号的bundle，将他们带黄色叹号的资源拖入新bundle中，就会发现使用相同资源的budle都会多了一个资源依赖
![](http://ophmqxrq8.bkt.clouddn.com/BuildAssetBundle4.png)


## 最后

网上的AssetBundle相关的文章真的是一堆一堆的，我想主要是涉及到Unity的AssetBundle打包机制随着版本的更新，也发生了很大的变化，我现在就想写一套最新的AssetBundle打包加载的流程，所以最近看了很多相关的资料，因为我暂时还没有机会去在工作中接触AssetBundle相关的任务，所以自己慢慢摸索吧，争取早日写出来，不至于以后工作中用到的话忙手忙脚。

写的不对的地方，希望大家不吝赐教。