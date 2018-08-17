---

layout: post
title: UGUI事件顺序
date: 2018-06-05 22:20:52
author: kongwz
tags:

  -  Ugui
categories:
  -  Unity
comments: true

---
### 由于刚刚换了公司，所以现在正在接触新框架Entitas和UGUI，因为以前都是在使用NGUI，所以，现在每天感觉都在接触新的东西，可谓是每天都充满了挑战呀，哈哈。

### 今天做一个功能，类似与吃鸡游戏的小地图和大地图，功能大概包括*周围人物怪物标识，场景物资标识，玩家和队友再地图上的标点和 自身与标点的划线、缩放、拖拽等*

因为设计点击和拖拽，所以需要单独监听这两个事件，但是我发现UGUI 的事件执行顺序是这样的
<!--more-->
点击一下的事件执行顺序
```bash
OnPointerDown
OnInitializePotentialDrag
OnPointerEnter
OnPointerUp
OnPointerClick
OnPointerExit
```
当我按下并拖动的事件执行顺序
```bash
OnPointerDown
OnInitializePotentialDrag
OnPointerEnter
OnBeginDrag
OnDrag
OnPointerUp
OnPointerClick
OnEndDrag
OnPointerExit
```

可以看出 当同一个物体我们同时监听它的点击事件和拖动事件的时候，是不能直接通过OnPointerClick或者OnPointerDown来得到的，因为当我们拖动的时候也会执行这两个事件，所以用以下解决方案。

- 设置一个bool变量
- OnPointerDown事件把变量设为true
- OnDrag设为false，或者OnBeginDrag
- 在OnPointerUp事件触发的时候判断这边变量是否为true，如果是true 则认为是点击，否则是拖动

很简单，代码就不写了。
写一下测试代码吧。后续有机会再写一下关于Entitas的笔记。

```bash
using UnityEngine;
using UnityEngine.EventSystems;

public class EventTest : MonoBehaviour,
    IPointerClickHandler,
    IPointerEnterHandler,
    IPointerExitHandler,
    IPointerDownHandler,
    IPointerUpHandler,
    IBeginDragHandler,
    IDragHandler,
    IInitializePotentialDragHandler,
    IEndDragHandler,
    IDropHandler,
    IUpdateSelectedHandler,
    ISelectHandler,
    IDeselectHandler,
    IScrollHandler,
    IMoveHandler,
    ISubmitHandler,
    ICancelHandler
{
    #region 鼠标指针类
    //鼠标进入时响应
    public void OnPointerEnter(PointerEventData eventData)
    {
        Debug.Log("OnPointerEnter");
    }

    //鼠标离开时响应
    public void OnPointerExit(PointerEventData eventData)
    {
        Debug.Log("OnPointerExit");
    }

    //鼠标按下时响应
    public void OnPointerDown(PointerEventData eventData)
    {
        Debug.Log("OnPointerDown");
    }

    //鼠标释放时响应
    public void OnPointerUp(PointerEventData eventData)
    {
        Debug.Log("OnPointerUp");
    }

    //鼠标点击时响应
    public void OnPointerClick(PointerEventData eventData)
    {
        Debug.Log("OnPointerClick");
    }
    #endregion


    #region 拖拽类
    //初始化拖拽
    public void OnInitializePotentialDrag(PointerEventData eventData)
    {
        Debug.Log("OnInitializePotentialDrag");
    }

    //开始拖拽
    public void OnBeginDrag(PointerEventData eventData)
    {
        Debug.Log("OnBeginDrag");
    }

    //拖拽中
    public void OnDrag(PointerEventData eventData)
    {
        Debug.Log("OnDrag");
    }

    //拖拽结束
    public void OnEndDrag(PointerEventData eventData)
    {
        Debug.Log("OnEndDrag");
    }

    //拖拽释放
    public void OnDrop(PointerEventData eventData)
    {
        Debug.Log("OnDrop");
    }
    #endregion


    #region 点选类
    //当物体被选中时每帧触发
    public void OnUpdateSelected(BaseEventData eventData)
    {
        Debug.Log("OnUpdateSelected");
    }

    //选中物体
    public void OnSelect(BaseEventData eventData)
    {
        Debug.Log("OnSelect");
    }

    //未选中物体
    public void OnDeselect(BaseEventData eventData)
    {
        Debug.Log("OnDeselect");
    }
    #endregion

    #region 输入类
    //鼠标中轮滚动
    public void OnScroll(PointerEventData eventData)
    {
        Debug.Log("OnScroll");
    }

    //移动物体
    public void OnMove(AxisEventData eventData)
    {
        Debug.Log("OnMove");
    }

    //提交
    public void OnSubmit(BaseEventData eventData)
    {
        Debug.Log("OnSubmit");
    }

    //取消
    public void OnCancel(BaseEventData eventData)
    {
        Debug.Log("OnCancel");
    }
    #endregion
}
```