---
title: 七牛测试域名回收解决方案
date: 2019-05-29 18:44:07
author: kongwz
tags:
  - hexo
categories:
  - Unity
comments: true
---


很久没有更新博客了，本来想上去看看之前写的一写笔记，发现我之前使用七牛图床存的所有图都不能正常显示了。然后去网上查了一下，原来是七牛回收了所有的测试域名，而且也给我发送了回收提醒邮件（当时完全没有当回事...）我所有在七牛上储存的图片都不能正常查看了，也不能获取外链……我在网上查了一下解决办法：

> 解决办法
    
- 在七牛云上填写自己的域名（需要备案）
- 使用其他图床（前提需要把七牛上储存的图片下载下来）

第一个方案我嫌域名备案太麻烦，所以这里直接选择第二套方案

<!--more-->

> 第一步：把七牛云原来储存的图片下载到本地

由于测试域名已经被回收了，没有下载入口，不过还好七牛提供了工具。 下载地址、使用指南及常用命令

参见 [下载地址](https://developer.qiniu.com/kodo/tools/1302/qshell) 

根据系统下载命令行工具(qshell)，以win7系统为例进行详细说明，避免入坑：

1. 下载命令行工具后，为了便于使用，将与系统对应的exe文件重命名为qshell.exe。
2. 打开系统命令行工具，将qshell.exe文件所在目录设置为当前工作目录，才可以使用七牛云命令行工具(qshell)。如果不设置前两步而直接运行EXE文件则闪退。
3. 设置秘钥。秘钥可在“个人面板/秘钥管理”页面查看

> 执行命令 设置秘钥

```
$ qshell account AccessKey SecretKey //设置秘钥
```
AccessKey 和SecretKey 需要自己去个人面板去查

> 获取仅有文件名的文件列表

批量复制batchcopy命令，需要用到仅有文件名的文件列表。

```
$ qshell listbucket bucketname list.txt
```

需要注意的是，将list.txt 中所有除文件名字以外的信息删掉包括空格，只保留文件名。


> 批量复制文件至新存储空间

在此之前 要新建一个储存空间，需要确保和你之前测试域名失效的储存空间是在同一个储存区域（我的是华东），所以再创建一个华东区域的储存空间。



然后就开始复制吧~

```
/* 
--force 加入此选项不需要输入验证码
<SrcBucket> 原存储空间名称
<DestBucket> 新建存储空间名称
<SrcDestKeyMapFile> 需要复制的文件列表，即上述得到的name.txt
*/
$ qshell batchcopy --force <SrcBucket> <DestBucket> <SrcDestKeyMapFile>
```

比如我域名失效的储存空间叫 photo  新建的储存空间名字叫 newphoto ，我刚刚获取的所有photo储存空间下的文件名字文件叫 list.txt 

```
$ qshell batchcopy --force photo newphoto list.txt
```

复制完之后可以进入新的存储空间进行确认复制是否成功

然后我们就可以批量下载了，这就解决了之前的储存空间，域名被回收后不能下载的情况


> 批量将七牛云图片下载到本地

这里需要用到下载命令qdownload 首先需要我们手动配置一个 下载配置文件 qdisk_down.conf  [参考](https://github.com/qiniu/qshell/blob/master/docs/qdownload.md)

```
{
	"dest_dir"	:	"/Users/jemy/Temp7/backup",
	"bucket"	:	"qdisk",
	"cdn_domain"    :      "if-pbl.qiniudn.com",
	"prefix"	:	"movies/",
	"suffixes"	:	".mp4"
}
```
上面的配置内容中
- dest_dir 本地数据备份路径，为全路径
- bucket 空间名称 （这里是指的你要下载的 储存空间的名字，也就是新创建的储存空间名字）
- cdn_domain 设置下载的CDN域名，默认为空表示从存储源站下载，【该功能默认需要计费，如果希望享受10G的免费流量，请自行设置cdn_domain参数，如不设置，需支付源站流量费用，无法减免！！！】 这里用新创建的储存空间的域名即可
- prefix 只同步指定前缀的文件，默认为空
- suffixes 只同步指定后缀的文件，默认为空


下面是我的配置示例：

```
{
	"dest_dir"	:	"F:\\qiniu",
	"bucket"	:	"newphoto",
	"cdn_domain"    : "ps9c56ppe.bkt.clouddn.com",
	"prefix"	:	"",
	"suffixes"	:	".png,.gif,.jpg"
}
```

下载命令：

```
$ qshell qdownload [-c <ThreadCount>] <LocalDownloadConfig>
```
- LocalDownloadConfig 本地下载的配置文件，内容包括要下载的文件所在空间，文件前缀等信息，具体参考配置文件说明
- ThreadCount 其中 `ThreadCount** 表示支持同时下载多个文件
- c 选项 -c ThreadCount ==> 下载的并发协程数量, 大小必须在1-2000，如果不在这个范围内，默认为5


我的下载命令示例：

```
$ qshell qdownload -c 54 qdisk_down.conf
```
跟七牛有关的操作就结束了，可以跟七牛说拜拜了！

> 上传腾讯云储存

我因为着急使用就临时采用了腾讯云储存，大家可以自己选择其他的图床

腾讯云的免费使用只有六个月，到期后应该是需要付费的。

其他的就没啥了 说一下注意事项吧。

> 注意事项

1. 获取的文件列表储存在txt 中后，注意删掉文件名字后面的所有内容，包括空格，保持每行只有一个文件名
2. 新创建的七牛云储存空间要和已经被回收测试域名的空间在一个储存区域，比如都是华东，或者华北
3. 保持每天好心情哟~