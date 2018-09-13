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

> 本文所有内容针对Unity版本：5.x以后，我用的是2018.2.2f1

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


### 打包策略

Unity5.x 已经更新了 很多有关打包相关的功能，其中就有一项，可以自动收集依赖打包，所以在打包的时候就可以有两种策略
1. 给每一个资源设置AssetBundleName，这样每个资源都会生成一个AssetBundle文件
2. 将资源归类设置AssetBundleName，这样打包会生成依赖相对集中的AssetBundle文件

第一种方式，会造成很多文件，担心会造成加载速度变慢（联想一下电脑在拷贝同样大小的一千个小文件Or一个压缩包，自然是压缩包速度快），所以针对依赖打包 需要根据游戏定制一套策略。

- Shader文件、字体，单独打包，进游戏加载并常住内存
- 将音频/动画控制器/动画/大的背景贴图、图集等不需要实例化的Prefab、存储配置设置/数据等相关的Prefab单独打包成AssetBundle。
- 依赖打包

对于依赖打包，如果有多个资源引用相同的一个资源，那么最好将这个被引用的资源单独打包，这样可以避免这个资源被同时打包到不同AssetBundle中造成资源冗余。

1. 根据文件夹打一个整体Bundle，特殊的文件夹内所有的内容打成一个Bundle，比如Shader文件夹，策划配置文件，模型组合等
2. 文件夹内所有文件各设置一个AssetBundleName，然后各自打一个包
3. 

**注意**：为了避免资源冗余需要写的一些工具
- 给该文件夹下所有文件设置单独的AssetBundleName工具
- 检查资源冗余的工具（主要是检查资源的引用情况）当我们给特定文件夹下的文件都单独打Bundle的时候，有可能存在资源冗余，在打完Bundle后可以用来检查，如果父类被引用了两次，子类也被引用了两次的话那么子类可以不单独打包。    *比如只有模型A和模型B都用了材质C，材质C只用了图片D。这时候，我们检查所有依赖时，会发现材质C和图片D的引用次数都是2，因为检查依赖的方法会把依赖拆分得最细，材质C和图片D都单独列出来了。实际上我们不想把图片D拆分出来，只需要单独打包材质C包含图片D就够了。于是在获得了大于1的所有资源列表之后，我们还需要做一个遍历，看看其中有没有像刚才材质C和图片D一样互相包含的情况，如果有，剔除掉。*


> 针对打包，需要根据游戏需求来设计，比如有的团队几乎把所有资源都打成了Bundle，有的只把美术资源和Shader还有策划的配置文件打包了， 而游戏里的UIPrefab 并没有打包成Bundle（可能是没有做热更新）。


### 用脚本设置资源的AssetBundleName

这里使用了网友[漫漫之间n](https://blog.csdn.net/u012740992/article/details/79371986)的轮子，如果有兴趣可以改一下，实现用来筛选打包后资源是否存在冗余的情况。
思路主要是遍历文件夹下所有文件，然后查找它的引用，并设置一个引用数量的阈值，主要分为以下的情况：
- 引用个数为0--->直接设置AssetBundleName
- 引用个数n大于零小于阈值--->不设置AssetBundleName（会有n-1个资源冗余，为了减少AssetBundle的数量，酌情定制阈值）
- 引用数量大于阈值--->给资源单独设置AssetBundleName，避免出现资源出现大量冗余的情况。

代码就不贴出来了，上面[漫漫之间n](https://blog.csdn.net/u012740992/article/details/79371986)已经写的很清楚了，有兴趣的话可以看看。


## 最后

网上的AssetBundle相关的文章真的是一堆一堆的，我想主要是涉及到Unity的AssetBundle打包机制随着版本的更新，也发生了很大的变化，我现在就想写一套最新的AssetBundle打包加载的流程，所以最近看了很多相关的资料，因为我暂时还没有机会去在工作中接触AssetBundle相关的任务，所以自己慢慢摸索吧，争取早日写出来，不至于以后工作中用到的话忙手忙脚。

写的不对的地方，希望大家不吝赐教。