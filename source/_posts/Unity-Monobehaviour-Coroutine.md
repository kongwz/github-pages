---
title: Unity--Monobehaviour生命周期 & 协程Coroutine
date: 2018-09-03 17:42:40
author: kongwz
tags:
  - Unity
categories:
  - Unity
comments: true
---


今天遇到一个问题，记录一下，顺便吧Monobehaviour生命周期 重新看了一遍。

> Q:当在monobehaciour中调用了StartCoroutine后（此时yield return new WaitForSeconds(5f)）monobehaciour脚本的enable置程false，那么协程后面的代码还会执行吗？

其实我的第一反应是不执行了，但是经过测试发现，我是错的。

<!--more-->

先看一下正常情况下，协程的执行顺序

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class test1 : MonoBehaviour {

    private bool startCall = true;
    private bool updateCall = true;
    private bool lateUpdateCall = true;

    // Use this for initialization
    void Start () {
        if (startCall) {
            Debug.LogError("Start Begin");
            StartCoroutine(TestStart());
            Debug.LogError("Start End");
            startCall = false;
        }        
    }    

    private void LateUpdate()
    {
        if (lateUpdateCall) {

            Debug.LogError("TestLaterUpdate Begin");
            StartCoroutine(TestLaterUpdate());
            Debug.LogError("TestLaterUpdate End");
            lateUpdateCall = false;
        }
    }

    // Update is called once per frame
    void Update () {
        if (updateCall) {
            Debug.LogError("Update Begin");
            StartCoroutine(TestUpdate());
            Debug.LogError("Update End");
            updateCall = false;
        }
	}

    IEnumerator TestStart()
    {
        Debug.LogError("TestStart  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestStart  After");
    }

    IEnumerator TestUpdate()
    {
        Debug.LogError("TestUpdate  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestUpdate  After");
    }

    IEnumerator TestLaterUpdate()
    {
        Debug.LogError("TestLaterUpdate  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestLaterUpdate  After");
    }
}

```

输出：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/201809031759.png)

那么现在改一下代码，在Start中调用完了协程后，将这个monobehaviour的enable置为false，看一下输出结果

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class test1 : MonoBehaviour {

    private bool startCall = true;
    private bool updateCall = true;
    private bool lateUpdateCall = true;

    // Use this for initialization
    void Start () {
        if (startCall) {
            Debug.LogError("Start Begin");
            StartCoroutine(TestStart());
            Debug.LogError("Start End");
            this.enabled = false;
            startCall = false;
        }        
    }    

    private void LateUpdate()
    {
        if (lateUpdateCall) {

            Debug.LogError("TestLaterUpdate Begin");
            StartCoroutine(TestLaterUpdate());
            Debug.LogError("TestLaterUpdate End");
            lateUpdateCall = false;
        }
    }

    // Update is called once per frame
    void Update () {
        if (updateCall) {
            Debug.LogError("Update Begin");
            StartCoroutine(TestUpdate());
            Debug.LogError("Update End");
            updateCall = false;
        }
	}

    IEnumerator TestStart()
    {
        Debug.LogError("TestStart  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestStart  After");
    }

    IEnumerator TestUpdate()
    {
        Debug.LogError("TestUpdate  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestUpdate  After");
    }

    IEnumerator TestLaterUpdate()
    {
        Debug.LogError("TestLaterUpdate  Before");
        yield return new WaitForSeconds(1);
        Debug.LogError("TestLaterUpdate  After");
    }
}

```

结果：

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/201809031803.png)

也就是说我们在将monobehaviour的enable置程false后并没能终止协程，协程还是在运行，但是如果我们将GameObject的SetActive(false)后协程和monobehaviour和协程都会停止。

所以monobehaviour的enable是对协程没有影响的

最后 看一下Unity Monobehaviour的生命周期图

![](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/o_exeOrder.png)