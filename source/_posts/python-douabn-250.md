---
title: 爬取豆瓣电影版Top250
date: 2017-05-21 21:27:23
author: kongwz
tags:
  - Python
categories:
  - Python
comments: true
---

### 使用工具

>python 3.6
>bs4 BeautifulSoup
>xlwt (用于保存到本地Excel)
>Chrome

### 代码
<!--more-->

```bash
import requests
from bs4 import BeautifulSoup
import xlwt

workbook=xlwt.Workbook(encoding='utf-8')
booksheet=workbook.add_sheet('Sheet 1',cell_overwrite_ok=True)

def init_url():
	DATA=(('电影名字','评分','是否可以播放','链接','标题'),)
	for i in range(0,250,25):
		url = 'https://movie.douban.com/top250?start={}&filter='.format(i) 
		print(url)
		res = requests.get(url)
		res.encoding = 'utf-8'
		soup = BeautifulSoup(res.text,'html.parser')
		DATA += get_url_data(soup)
	create_exl(DATA)
		
def get_url_data(soup):
	newdata=()
	for data in soup.select('.info'):
		if len(data.select('.hd')) > 0:
			name =data.select('a')[0].text.split(' ')[0].split('\n')[1]
			pingfen = data.select('.rating_num')[0].text.strip() 
			canplay = ''
			if len(data.select('.playable')) > 0:
				canplay = data.select('.playable')[0].text.rstrip()		
			link = data.select('a')[0]['href']
			biaoti = ''
			if len(data.select('.inq')) > 0:
				biaoti = data.select('.inq')[0].text
			newdata += ((str(name),str(pingfen),str(canplay),str(link),biaoti),)
	return newdata	
	
def create_exl(DATA):
	for i,row in enumerate(DATA):
		for j,col in enumerate(row):
			booksheet.write(i,j,col)
	workbook.save('douban_Top250.xls')

print(init_url())

```
### 效果
![爬取结果](https://blogimages-1253307164.cos.ap-shanghai.myqcloud.com/douban1.png)