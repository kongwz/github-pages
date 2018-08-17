---
title: Unity插件之-更新SVN
date: 2018-08-10 10:49:52
author: kongwz
tags:
  - 插件
categories:
  - Unity
comments: true
---

SVN作为版本控制，团队协作，代码托管，是一个很好的工具，当然别的工具 也很强大，比如Git等，不说哪个强大，这里只是针对Unity做一个一键更新SVN的插件，这样免去了要打开文件夹-右击-更新SVN的繁琐步骤，只需要在Unity界面按一下快捷键就可以了，这里我设置的快捷键是Alt+s，这个快捷键是自定义的，如果觉得不喜欢可以更改。

原理很简单，就是在Unity里执行SVN的更新命令，并设置一些更新时的属性配置，就行了。

<!--more-->

看一下代码吧

```bash
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.Diagnostics;
using System.IO;

public class SVNUpdate : Editor {

    [MenuItem("Tools/SvnUpdate &s")]
    public static void UpdateSVN() {
        ProcessCommand("TortoiseProc.exe", "/command:update /path:. /closeonend:0");
        AssetDatabase.Refresh();
        EditorUtility.UnloadUnusedAssetsImmediate();
        System.GC.Collect();
    }

    public static void ProcessCommand(string command, string argument) {
        ProcessStartInfo start = new ProcessStartInfo(command);
        start.Arguments = argument;
        start.CreateNoWindow = false;
        start.ErrorDialog = true;
        start.UseShellExecute = true;
        if (start.UseShellExecute)
        {
            start.RedirectStandardOutput = false;
            start.RedirectStandardError = false;
            start.RedirectStandardInput = false;
        }
        else {
            start.RedirectStandardOutput = true;
            start.RedirectStandardError = true;
            start.RedirectStandardInput = true;
            start.StandardOutputEncoding = System.Text.UTF8Encoding.UTF8;
            start.StandardErrorEncoding = System.Text.UTF8Encoding.UTF8;
        }

        Process p = Process.Start(start);
        if (!start.UseShellExecute) {
            PrintOutPut(p.StandardOutput);
            PrintOutPut(p.StandardError);
        }
        p.WaitForExit();
        p.Close();
    }

    public static void PrintOutPut(StreamReader reader) {
        string line = reader.ReadLine();
        while (!reader.EndOfStream) {
            UnityEngine.Debug.Log(line);
        }
        reader.Close();
    }

}


```

---

> 最后

代码已经全部附上了，就不做演示了，直接复制代码过去就可以使用

周五了今天，预祝周末愉快