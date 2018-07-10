---

layout: post
title: Unity-Debug-封装-富文本
date: 2018-07-05 10:05:52
author: kongwz
tags:
  - Unity
categories:
  - Unity
comments: true

---

### Unity自带的日志输出可以说是很全面的，但是在实际开发中，往往会产生大量的log输出，自然影响效率是不用说的，而且有些时候我们需要根据log输出的时间来确定代码的执行顺序，这个时候难免会用到每个日志输出的时间。
#### Unity 自带的日志输出缺点
- 没有开关，可以直接关闭或者打开log
- 输出的日志不带有时间戳
- 颜色单一

<!--more-->

### 其实Unity 是支持富文本格式的，当然也适用于Debug.Log()，比如加粗、倾斜、改变某些内容的颜色
> 加粗 

```bash
We are <b>not</b> amused 
```
> 倾斜

```bash
We are <i>usually</i> not amused
```

> 更改字体大小

```bash
We are <size=50>largely</size> unaffected  
```

> 更改颜色

```bash
We are <color=green>green</color> with envy 
```

*参考资料*
  [http://www.ceeger.com/Manual/StyledText.html](http://www.ceeger.com/Manual/StyledText.html)

```bash
We are <color=green>green</color> with envy
```
### 我把几个常用的日志输出做了一下封装，添加了一个开关 *ShowLog*，并添加了日志输出的时间，稍微修改了一下颜色

> 效果

![运行效果](http://ophmqxrq8.bkt.clouddn.com/debug.png)

### 以下是具体封装代码

```bash
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameDebug  {
    public static bool ShowLog = true;
    public static void Log(object log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            Log(log.ToString());
        }
#endif
    }
    public static void Log(string log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            Debug.Log("<color=#FFFF00FF>[" + DateTime.Now.ToString("HH:mm:ss:ffff") + "]</color>  " + log);
        }
#endif
    }

    public static void LogObjs(params object[] objs)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            string str = "";
            foreach (var s in objs)
            {
                str += s.ToString() + "\t";
            }
            if (str.Length > 0)
            {
                str.Remove(str.Length - 1);
            }
            Log(str);
        }
#endif
    }

    public static void LogWarning(object log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            LogWarning(log.ToString());
        }
#endif
    }

    public static void LogWarning(string log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            Debug.LogWarning("<color=#FFFF00FF>[" + DateTime.Now.ToString("HH:mm:ss:ffff") + "]</color>  " + log);
        }
#endif
    }

    public static void LogError(object log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            LogError(log);
        }
#endif
    }

    public static void LogError(string log)
    {
#if UNITY_STANDALONE || UNITY_EDITOR
        if (ShowLog)
        {
            Debug.LogError("<color=#FFFF00FF>[" + DateTime.Now.ToString("HH:mm:ss:ffff") + "]</color>  " + log + "\n" + GetStackTrace());
        }
#endif
    }

    private static string GetStackTrace()
    {
        System.Diagnostics.StackTrace ss = new System.Diagnostics.StackTrace(true);
        return ss.ToString();
    }
}
```

### 调用代码
```bash
		GameDebug.Log("Hello World");
        GameDebug.LogWarning("Hello World");
        GameDebug.LogError("Hello World");
        GameDebug.ShowLog = false;
        GameDebug.LogError("Hello World");
```
输出结果是第一张图片，可以看到最后一条日志没有输出