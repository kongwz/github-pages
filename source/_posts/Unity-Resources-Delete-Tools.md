---
title: Unity工具之——查找废弃资源并删除
date: 2018-09-19 16:50:34
tags:
  - Unity Tools
categories:
  - Unity
comments: true
---


刚刚接手了一个日本游戏项目，刚拿到手的时候看了一下项目大小，着实被吓了一跳，一个卡牌游戏的项目十个G...赶紧喝口枸杞水压压惊。

拿到项目以后，自然是好奇哪里占用了资源是吧，就发现一个很奇葩的事情，可能是为了做国际化，带文字的图片 不同语言的版本各出一张，呀呀呀，这还了得，同一张图就得搞出来四五套，所以有了以下的需求。。

> 删除非大陆版本的其他资源

我大概看了以下，这些资源怎么着都得有两三千个，要手动一个一个删除？ 想了想，毕竟年纪大了手脚不好使还是算了吧（主要是懒），本着能不动手就不动手的原则，那就写个脚本来帮我干活吧。

<!--more-->

> 工具思路

1. 选中一个文件夹，然后找到所有文件
2. 还好其他版本的文件是有标识的，这里的标识是文件名字后缀@XX，比如@hk是香港
3. 识别所有文件的名字，凡是带有@字符，且@后面两位是hk、jp、tw、、等等的文件，就加进一个字典中，这个字典储存文件的路径和asset
4. 第一个循环查找文件夹下所有文件，并识别
5. 第二个循环把上个循环拿到的文件进行进一步删除处理
6. 最后刷新资源。
7. 完事，再喝一口枸杞水。

> 搜集资料

首先是获取文件夹下所有文件的方法，如果有现成的最好了，没有的话那只能自己写了，无非就是个递归查找。

当然让我找到了  呐

```
DirectoryInfo dirinfo = new DirectoryInfo(rootDir);
FileInfo[] fs = dirinfo.GetFiles("*.*", SearchOption.AllDirectories);
```

这里的rootDir 是选择的文件夹的路径。

文件找到了，还有一个是，删除文件的方法，当然也是有现成的，  呐

```
AssetDatabase.DeleteAsset("Assets/folderName2/mat1.mat"); //删除资源
AssetDatabase.Refresh(); //刷新资源视图
```

> 开始干

思路说完了，API 也找到了，那就直接上代码吧

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;

public class ResourceTools : Editor {
    static Dictionary<string, Object> assetInfoDict = new Dictionary<string, Object>();

    private static string curRootAsset = string.Empty;
    private static float curProgress = 0f;


    [MenuItem("ResourceTools / DeleteOtherResources")]
    static void DeleteOtherResources()
    {
        string path = GetSelectedAssetPath();
        if (path == null)
        {
            Debug.LogWarning("请先选择目标文件夹");
            return;
        }
        ResourceTools.GetAllAssets(path);

    }
  

    public static void GetAllAssets(string rootDir)
    {
        assetInfoDict.Clear();

        DirectoryInfo dirinfo = new DirectoryInfo(rootDir);
        FileInfo[] fs = dirinfo.GetFiles("*.*", SearchOption.AllDirectories);
        int ind = 0;
        foreach (var f in fs)
        {
            curProgress = (float)ind / (float)fs.Length;
            curRootAsset = "搜寻中...：" + f.Name;
            EditorUtility.DisplayProgressBar("正在查询其他版本资源", curRootAsset, curProgress);
            ind++;
            int index = f.FullName.IndexOf("Assets");
            if (index != -1)
            {
                string assetPath = f.FullName.Substring(index);
                Object asset = AssetDatabase.LoadMainAssetAtPath(assetPath);
                string upath = AssetDatabase.GetAssetPath(asset);
                //Debug.Log(upath);
                if (assetInfoDict.ContainsKey(assetPath) == false
                    && assetPath.StartsWith("Assets")
                    && !(asset is MonoScript)
                    && !(asset is LightingDataAsset)
                    && asset != null
                    )
                {
                    
                    if (upath.EndsWith("@jp.png") ||
                        upath.EndsWith("@en.png") ||
                        upath.EndsWith("@tw.png") ||
                        upath.EndsWith("@kr.png") ||
                        upath.EndsWith("@hk.png")) {
                        assetInfoDict.Add(upath, asset);
                    }
                }
                EditorUtility.UnloadUnusedAssetsImmediate();
            }
            EditorUtility.UnloadUnusedAssetsImmediate();
        }
        EditorUtility.ClearProgressBar();

        int setIndex = 0;
        foreach (KeyValuePair<string, Object> kv in assetInfoDict)
        {
            EditorUtility.DisplayProgressBar("正在删除...", kv.Key, (float)setIndex / (float)assetInfoDict.Count);
            setIndex++;
            //这里 开始删除 资源
            AssetDatabase.DeleteAsset(kv.Key);
            Debug.Log(kv.Key);
        }
        EditorUtility.ClearProgressBar();
        EditorUtility.UnloadUnusedAssetsImmediate();
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();
    }

    static string GetSelectedAssetPath()
    {
        var selected = Selection.activeObject;
        if (selected == null)
        {
            return null;
        }
        Debug.Log(selected.GetType());
        if (selected is DefaultAsset)
        {
            string path = AssetDatabase.GetAssetPath(selected);
            Debug.Log("选中路径： " + path);
            return path;
        }
        else
        {
            return null;
        }
    }


}

```

> 写在后面

这里是写的查找png图片，如果想查找其他类型的资源并删除的话，可以在**GetAllAssets（string rootDir ， string type）** 方法中加一个类型，然后在筛选的时候做个判断就行了。

来看一下，现在它正给我干活呢。

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/resourceDelete.png)

**就先写到这里吧，枸杞水喝完了，去接水了...下次再聊**