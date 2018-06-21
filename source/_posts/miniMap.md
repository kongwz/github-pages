---
title: miniMap
date: 2018-06-21 17:42:18
tags:
---
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