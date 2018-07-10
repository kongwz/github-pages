---
title: 吃鸡手游小地图开发
date: 2018-06-21 17:42:18
author: kongwz
tags:
  - Unity
categories:
  - Unity
comments: true
---
### 需求：*做一个吃鸡游戏的小地图，包含放大缩小，拖动，标记点，自己位置与标记位置连线，显示周围敌人的声音（脚步，开火等）*
**这里包含一个需求，就是再地图放大的时候要以自己人物的位置为基准点，也就是说放大到最后需要吧自己的人物移动到屏幕的中间，当然再缩小，还是按照这个思路来，如果放大后发生拖动，则以拖动后地图显示区域的中心为基准点缩放。**有点绕……看一下最终实现效果图。

![最终效果](http://ophmqxrq8.bkt.clouddn.com/5.gif)


<!--more-->

### 下面 是全部的代码，至于其他需求，拖动边缘检测、标记、连线、声音等就不在这里一一细说了。

```bash
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;


public class Test003 : MonoBehaviour , IDragHandler, IPointerDownHandler, IPointerUpHandler , IBeginDragHandler , IEndDragHandler
{


    public Slider _slider;
    public Image _spr;
    public GameObject obj;
    private  Vector2 _rolePos = new Vector3(100, 100, 0);
    private Vector2 _CurrSprCenterPos = Vector2.zero;
    Vector2 endSprPos = Vector2.zero;
    // Use this for initialization
    void Start()
    {
        _CurrSprCenterPos = _rolePos;
        originalPos = _spr.GetComponent<RectTransform>().anchoredPosition;
        _slider.onValueChanged.AddListener(delegate
        {
            SliderValueChanged();
        });
        Debug.LogError(_rolePos);
    }

    // Update is called once per frame
    void Update()
    {

    }
    Vector2 originalPos = Vector2.zero;
    public void SliderValueChanged()
    {
        _spr.transform.localScale = new Vector3(4 * _slider.value + 1,4* _slider.value + 1, 4* _slider.value + 1);
        SetSprPosition();
    }

    public void SetSprPosition() {
        float tmpX = _spr.transform.localScale.x * _spr.GetComponent<RectTransform>().sizeDelta.x / 2 - _spr.GetComponent<RectTransform>().sizeDelta.x / 2;
        float tmpY = _spr.transform.localScale.y * _spr.GetComponent<RectTransform>().sizeDelta.y / 2 - _spr.GetComponent<RectTransform>().sizeDelta.y / 2;
        
        float tmpx = (originalPos.x - _rolePos.x) * _spr.transform.localScale.x;
        float tmpy = (originalPos.y - _rolePos.y) * _spr.transform.localScale.x;
        if (tmpX < Mathf.Abs(tmpx) && tmpY < Mathf.Abs(tmpy))
        {
            tmpX = originalPos.x >= _rolePos.x ? tmpX : tmpX * -1;
            tmpY = originalPos.y >= _rolePos.y ? tmpY : tmpY * -1;
            _spr.GetComponent<RectTransform>().anchoredPosition = originalPos + new Vector2(tmpX, tmpY);
        }
        else if (tmpY < Mathf.Abs(tmpy) && tmpX >= Mathf.Abs(tmpx))
        {
           
            tmpY = originalPos.y >= _rolePos.y ? tmpY : tmpY * -1;
            _spr.GetComponent<RectTransform>().anchoredPosition = new Vector2(((originalPos.x - _rolePos.x) * _spr.transform.localScale.x + originalPos.x), originalPos.y + tmpY);
        }
        else if (tmpY >= Mathf.Abs(tmpy) && tmpX < Mathf.Abs(tmpx))
        {
            tmpX = originalPos.x >= _rolePos.x ? tmpX : tmpX * -1;
            _spr.GetComponent<RectTransform>().anchoredPosition = new Vector2(originalPos.x + tmpX , ((originalPos.y - _rolePos.y) * _spr.transform.localScale.y + originalPos.y));
        }
        else
        {
            _spr.GetComponent<RectTransform>().anchoredPosition = new Vector2(((originalPos.x - _rolePos.x) * _spr.transform.localScale.x + originalPos.x), ((originalPos.y - _rolePos.y) * _spr.transform.localScale.y + originalPos.y));
        }
        _CurrSprCenterPos = new Vector2(originalPos.x - (_spr.GetComponent<RectTransform>().anchoredPosition.x - originalPos.x) / _spr.transform.localScale.x, originalPos.y - (_spr.GetComponent<RectTransform>().anchoredPosition.y - originalPos.y) / _spr.transform.localScale.y);
        endSprPos = _spr.GetComponent<RectTransform>().anchoredPosition;
    }


    #region 拖拽
    Vector2 offset = Vector2.zero;

    public void OnDrag(PointerEventData eventData)
    {
        Vector2 tmpPos = Vector2.zero;
        if (RectTransformUtility.ScreenPointToLocalPointInRectangle(this.transform.parent.GetComponent<RectTransform>(), eventData.position, eventData.enterEventCamera, out tmpPos))
        {
            if (_slider.value > 0)
            {
                _spr.GetComponent<RectTransform>().anchoredPosition = offset + tmpPos;
                _CurrSprCenterPos = new Vector2(originalPos.x - (_spr.GetComponent<RectTransform>().anchoredPosition.x - originalPos.x) / _spr.transform.localScale.x, originalPos.y - (_spr.GetComponent<RectTransform>().anchoredPosition.y - originalPos.y) / _spr.transform.localScale.y);
            }

        }
    }

    public void OnPointerDown(PointerEventData eventData)
    {
        Vector2 tmpPos = Vector2.zero;
        if (RectTransformUtility.ScreenPointToLocalPointInRectangle(this.transform.parent.GetComponent<RectTransform>(), eventData.position, eventData.enterEventCamera, out tmpPos))
        {
            offset = tmpPos;
            Debug.LogError("点击 位置    " + tmpPos);
        }
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        offset = Vector2.zero;
    }
    Vector2 beginDragPos = Vector2.zero;
    public void OnEndDrag(PointerEventData eventData)
    {
        Debug.LogError("************END*************");
        beginDragPos = _spr.GetComponent<RectTransform>().anchoredPosition;
        _rolePos = _CurrSprCenterPos;
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        Debug.LogError("************START*************");
        beginDragPos = _spr.GetComponent<RectTransform>().anchoredPosition;
        offset = beginDragPos - offset;
    }

    #endregion

}

```

-------

### 2.小地图人物移动，对应地图图片发生位移主要代码

```bash

int _sceneWidth = 1500;
    int _sceneHigth = 1500;

    float widthrate = _mapImage.rectTransform.sizeDelta.x / _sceneWidth

    float hightrate = _mapImage.rectTransform.sizeDelta.y / _sceneHigth

    float RadarMapX = gloable.role.position.x * widthrate * -1;
    float RadarMapY = gloable.role.position.y * hightrate * -1;

    _mapImage.rectTransform.anchoredPosition = Vector2.New(RadarMapX + _mapImage.rectTransform.sizeDelta.x / 2 ,RadarMapY + _mapImage.rectTransform.sizeDelta.y / 2)

```