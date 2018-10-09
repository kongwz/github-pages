---
title: 说说Unity中的宏
date: 2018-10-09 15:06:54
author: kongwz
tags:
  - Unity
categories:
  - Unity
comments: true
---

唉，刚过完十一长假，上班各种不适，年前除了生日、元旦没有其他盼头了，想想就蓝廋

### 1、#region

这个是在开发中很常用的宏，它可以将我们的代码分块，有利于梳理思路，我在开发中也会经常用到，来看一下是怎么使用的。

```
public class Test : MonoBehaviour {
   
    #region 自定义宏
 
    int index = 0;
    void Fun()
    {
        
    }
 
    #endregion
}
```
还有一个好处是，加上这个宏后会在这个宏的地方出现类似方法前的“+ -”号，可以把这个代码块收缩或者展开。

<!--more-->

### 2、Unity自带的宏

先看一下代码：

```
#if UNITY_IPHONE
        Debug.Log("UNITY_IPHONE");
#elif UNITY_ANDROID
        Debug.Log("UNITY_ANDROID");
#elif UNITY_EDITOR
        Debug.Log("UNITY_EDITOR");
#endif
```

首先这些宏是Unity定义的，我们可以使用，这里是判断平台来执行不同的逻辑，这个在做移动开发的时候是经常用到的。也不多说了

### 3、自定义宏

如果我们有这样的需求，比如：当在条件A情况下执行代码段m，在条件B情况下执行代码段n，那么我们可以定义一个宏，来代表条件，在代码中判断条件是是否满足。

这样在每个需要进行条件对比的时候判断这个宏就可以了，那么怎么定义自己的宏呢？

```
public class Test : MonoBehaviour {
 
    string str = "";  
 
    void Start()
    {
 
    #if CONDITION_1
        str = "Hello";
    #elif CONDITION_2
        str = "Hello world";
    #endif
 
        Debug.Log(str);
    }
}
```

这里宏的名字可以根据自己的需求定义，最好是起个大多数人能看懂的名字。通过 #if 来使用。

那么我们定义好了宏也写了怎么使用，那么我们应该在哪里触发它呢。

这里我们首先选择当前项目的平台。打开“BuildSetting”，先在“Platform”中选择这个项目的平台，然后点击“Switch Platform”，当切换完平台后，再次点击BuildSetting，再点击“PlayerSetting”，然后在“OtherSetting”中的“Scripting Define Symbols”输入我们定义的宏（如果有多个宏，需要用分号隔开），比如这里如果输入了CONDITION_2 则会输出Hello world

### 4、进阶

在游戏开发中难免会有很多情况会遇到自己定义宏的时候，这个时候我们会发现在“PlayerSetting”中设置“Scripting Define Symbols”是一个很费劲的事。

之所以说费劲是，有可能十几二十个宏，都填在一个输入框中，难免有时候会出错，所以解决办法是，新建一个txt文件来储存宏的状态。

思路：
- 将自己定义的宏全部列在该文本文档中
- 定义在宏前面加上注释符号的算是不启用，反之启用
- 编写打包APK的脚本插件，在打包之前去读取这个文本文档，取出所有没有注释的宏并打包


当然如果不是在打包的时候用的 大家还是要主动书写进去的。

defines.txt
定义：
```
//自定义宏
BUILD_RELEASE
//自定义宏
//UNITY_DEBUG
//自定义宏
//SERVER_MOCK

//自定义宏
BUILD_STAGING_ROM
//BUILD_REGION_JP
BUILD_REGION_CN
//BUILD_REGION_TW
//BUILD_REGION_KR
//BUILD_REGION_EN
```

取出defines.txt中的宏，这里有一个替换是为了打包的版本替换掉相对应的宏，如果使用的时候可以忽略，直接读取defines.txt中的宏就行了。

```
private readonly string DEFINES_TXT_PATH = Application.dataPath + "/defines.txt";
public TemporaryRegexReplaceInDefinesTxt(string pattern, string replacement, bool rebuild = true) {
	m_old_defines_txt = System.IO.File.ReadAllText(DEFINES_TXT_PATH);
	var defines_txt = m_old_defines_txt;
	defines_txt = System.Text.RegularExpressions.Regex.Replace(defines_txt, pattern, replacement);
	System.IO.File.WriteAllText(DEFINES_TXT_PATH, defines_txt, System.Text.Encoding.Unicode);
	if (rebuild) {
		EditorApplication.ExecuteMenuItem("DeveloperTools/BuildTools/RebuildScripts/Development");
	}
}
public void Dispose() {
	System.IO.File.WriteAllText(DEFINES_TXT_PATH, m_old_defines_txt, System.Text.Encoding.Unicode);
}
```

下面这个代码是根据打包的平台设置ScriptingDefineSymbols，主要方法是SetScriptingDefineSymbolsForGroup（），这里会提前获取对应平台的设置数据GetScriptingDefineSymbolsForGroup（）


```
private class TemporaryAddScriptingDefineSymbols : System.IDisposable 
{
	public TemporaryAddScriptingDefineSymbols(BuildTargetGroup target_group, string define_symbol) 
	{
		m_target_group = target_group;
		m_old_scripting_define_symbols = PlayerSettings.GetScriptingDefineSymbolsForGroup(m_target_group);
		PlayerSettings.SetScriptingDefineSymbolsForGroup(m_target_group, m_old_scripting_define_symbols + " " + define_symbol);
	}
	public void Dispose() 
	{
		PlayerSettings.SetScriptingDefineSymbolsForGroup(m_target_group, m_old_scripting_define_symbols);
	}
	private BuildTargetGroup m_target_group;
	private string m_old_scripting_define_symbols;
}
```

这样设置以后就可以开始打包了，这样的目的是在打包的时候实现脚本划，且在打包的时候在脚本中设置一系列的属性，简化打包流程，而且也方便实现使用Jenkins实现打包自动化。

### 5、最后

最近很喜欢研究一些Unity开发中的一些简化工作流程的工具，从前面的[Unity客户端脚本自动化](http://kongwz.cn/2018/08/06/Unity%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%84%9A%E6%9C%AC%E8%87%AA%E5%8A%A8%E5%8C%96/)，到后来的[Unity插件之-更新SVN](http://kongwz.cn/2018/08/10/Unity%E6%8F%92%E4%BB%B6%E4%B9%8B-%E6%9B%B4%E6%96%B0SVN/)

前几天我把公司的打包流程使用[Jenkins](https://jenkins.io/)实现了自动化，本来想写个博客做个笔记，但是隔了几天后就不想写了，主要原因是不知道怎么写了，觉得没什么可以写的，所以以后还是要及时写点东西，拖延的后果就是放弃。

