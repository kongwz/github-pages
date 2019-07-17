---
title: Unity 加载网络图片解决方案
date: 2019-07-17 11:52:42
author: kongwz
tags:
  - hexo
categories:
  - Unity
comments: true
---

>写在前面

在游戏开发过程中难免会遇到各种活动的配图频繁更换的情况，因为我们的游戏活动配图是存在客户端bundle中的，所以要频繁更新bundle，在后期开新服的时候造成了很大的不便。所以考虑把活动的图放在网上。

之前设想使用WebView实现这个功能，但是我们游戏内部的WebView使用的是一个开元的插件（这里附上[地址](https://github.com/gree/unity-webview)），作者回复说WebView只能显示在ui上层，无法设置WebView的层级关系，这样会导致游戏中的弹窗提示被遮挡，所以这个方案就放弃了

所以决定使用从服务器读取图片并赋值给客户端Image的方式


<!--more-->

### 方案
主要步骤：

 1. 从服务器获取图片地址
 2. 检测本地有没有下载过这张图片
 3. 如果下载过 就从本地读取
 4. 没有下载过的话使用WWW将图片下载下来缓存到本地
 5. 赋值给Image
 6. 关闭界面时 删除本地缓存图片

### 代码

```
using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using System.IO;
using System.Collections.Generic;

public class AsyncImageDownload : MonoBehaviour
{
    private Sprite placeholder;
    private static List<string> files = null;
    public void Start()
    {
        if (!Directory.Exists(Application.persistentDataPath + "/ImageCache/"))
        {
            Directory.CreateDirectory(Application.persistentDataPath + "/ImageCache/");
        }
    }

    public void SetAsyncImage(string url, Image image)
    {
        //开始下载图片前，将UITexture的主图片设置为占位图
        //image.sprite = placeholder;

        //判断是否是第一次加载这张图片
        if (!File.Exists(path + url.GetHashCode()))
        {
            if (files != null)
            {
                files.Add(url.GetHashCode().ToString());
            }
            else {
                files = new List<string>();
                files.Add(url.GetHashCode().ToString());
            }
            
            //如果之前不存在缓存文件
            StartCoroutine(DownloadImage(url, image));
        }
        else
        {
            StartCoroutine(LoadLocalImage(url, image));
        }
    }

    IEnumerator DownloadImage(string url, Image image)
    {
        Debug.Log("downloading new image:" + path + url.GetHashCode());//url转换HD5作为名字
        WWW www = new WWW(url);
        yield return www;

        Texture2D tex2d = www.texture;
        //将图片保存至缓存路径
        byte[] pngData = tex2d.EncodeToPNG();
        File.WriteAllBytes(path + url.GetHashCode(), pngData);

        Sprite m_sprite = Sprite.Create(tex2d, new Rect(0, 0, tex2d.width, tex2d.height), new Vector2(0, 0));
        image.sprite = m_sprite;
        image.SetNativeSize();
    }

    IEnumerator LoadLocalImage(string url, Image image)
    {
        string filePath = "file:///" + path + url.GetHashCode();

        Debug.Log("getting local image:" + filePath);
        WWW www = new WWW(filePath);
        yield return www;

        Texture2D texture = www.texture;
        Sprite m_sprite = Sprite.Create(texture, new Rect(0, 0, texture.width, texture.height), new Vector2(0, 0));
        image.sprite = m_sprite;
        image.SetNativeSize();
    }

    public string path
    {
        get
        {
            //pc,ios //android :jar:file//
            return Application.persistentDataPath + "/ImageCache/";

        }
    }

    public void OnDisable()
    {
        //CleanFiles();
    }

    public void OnDestroy()
    {
        CleanFiles();
    }

    public void CleanFiles() {
        if (files != null && files.Count > 0) {
            for (int i = files.Count - 1; i >= 0; i--) {
                if (!Directory.Exists(path + files[i])) {
                    Debug.LogError("delete file path " + path + files[i]);
                    File.Delete(path + files[i]);
                    files.RemoveAt(i);
                }
            }
        }
    }
}
```
 
