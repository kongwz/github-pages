---
title: Unity-ScriptableObject笔记
date: 2018-09-18 11:16:45
author: kongwz
tags:
  - Unity
categories:
  - Unity
comments: true
---


因为之前的项目中，都是使用了原来的数据存储方式 *在Excel中配置数据，使用工具打包成特定格式，然后在客户端中建立索引，进入游戏加载出全部数据，然后通过引用来获取...* 估计这也是大多数游戏使用的策略吧。

最近刚刚接触到今天要说的这个神器，也可能是我太菜，别人都用了好久了我才知道。。。但知道了就要研究一下，做个记录吧。

### 介绍

> ScriptableObject 是什么？

ScriptableObject是一个允许你存储大量独立于脚本实例的共享数据的类，它的数据存储在 asset 资源文件中，类似unity材质或纹理资源，也就是说，在游戏运行的时候是可以更改的，且在退出游戏的时候会保留更改的内容。

<!--more-->

> ScriptableObject 的优点是什么？

- 运行时可以更改数据，且可以保留
- 在实例化时是被引用，而不是复制，这样就不会造成占用更多的内存。可以节省内存。
- 可以在Inspector显示的看到数据内容，方便更改和检查。
- 类似其他资源，它可以被任何场景引用，即场景间共享
- 没有多余的Component

> ScriptableObject 缺点

- 首先是 回调函数比较少
- OnEnable
- OnDisable
- OnDestroy
- 还有就是因为它是真正意义上的共享，所以在别的地方更改了数据，它本身的数据也就被更改了。

### 使用

这么好的东西我居然才开始使用，是不是说以后策划就不用陪Excel表了？哈哈哈，仅从客户端的角度看是可以的，但是...服务器呢，估计会被气炸吧..so..酌情而定吧。

> ScriptableObject 使用

首先，创建一个脚本，这个脚本要继承ScriptableObject，不要再继承MonoBehaviour了。然后写一个公共的属性，比如：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(menuName = "CreateScriptableObj")]
public class ScriptableObjTest : ScriptableObject {

    public string str;
    private Color cc; //这个是不会在Inspector显示出来
    public  int num;
}
```
这里在创建资源的菜单中增加了创建ScriptableObject的选项，也就是这行代码：

```
[CreateAssetMenu(menuName = "CreateScriptableObj")]
```

还有一个创建的方法，使用代码创建：

```
[MenuItem("Create/ScriptableObject")]
    public static void CreateScriptableObj() {
        var asset = ScriptableObject.CreateInstance<ScriptableObjTest>();
    }
```

写一个方法，使用ScriptableObject.CreateInstance<ScriptableObjTest>();就可以，这里的ScriptableObjTest，是我创建的脚本名字，要注意 **脚本名字要和类名相同**

如果这个类里面要使用自定义类型，那么需要在自定义类上加上 **[System.Serializable]**

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

[CreateAssetMenu(menuName = "CreateScriptableObj")]
public class ScriptableObjTest : ScriptableObject {

    public string str;
    private Color cc; //这个是不会在Inspector显示出来
    public  int num;

    public List<ScriptableObjItem> itemList = new List<ScriptableObjItem>();
   
    [MenuItem("Create/ScriptableObject")]
    public static void CreateScriptableObj() {
        var asset = ScriptableObject.CreateInstance<ScriptableObjTest>();
    }

    public Type _type;

    public enum Type {
        NONE = 0,
        Monster = 1,
        Hero = 2,
    }
}

[System.Serializable]
public class ScriptableObjItem {
    public string name;
}

```

也可以结合Json使用，运行时读取json文件来创建ScriptableObject

```
[CreateAssetMenu]
class LevelData : ScriptableObject { ... }

LevelData LoadLevelFromFile(string path) {
    string json = File.ReadAllText(path);
    LevelData result = CreateInstance<LevelData>();
    JsonUtility.FromJsonOverwrite(result, json);
    return result;
}
```

简单的用法就先写到这里，后面再持续更新，搬砖去了...