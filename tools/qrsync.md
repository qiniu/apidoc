---
title: qrsync 命令行辅助同步工具使用文档 | 七牛云存储
---

# qrsync 命令行辅助同步工具使用文档

## 简介

qrsync 是一个根据七牛云存储API实现的简易命令行辅助上传同步工具，支持断点续上传，增量同步，它可将用户本地的某目录的文件同步到七牛云存储中，同步或上传几百GB甚至上TB的文件毫无鸭梨。

qrsync 命令行辅助工具下载地址：[https://github.com/qiniu/devtools/tags](https://github.com/qiniu/devtools/tags)

## 用法

先建立一个 task 文件，比如叫 foo.task，内容可参考 default.task，大体如下：

	{
		"access_key": "<Please apply your access key>",
		"secret_key": "<Dont send your secret key to anyone>",
		"bucket": "<Bucket name on qiniu resource storage>",
		"sync_dir": "<Local directory to upload>",
		"debug_level": 1
	}

其中，access_key 和 secret_key 在七牛云存储平台上申请。步骤如下：

1. [开通七牛开发者帐号](https://dev.qiniutek.com/signup)
2. [登录七牛开发者自助平台，查看 Access Key 和 Secret Key](https://dev.qiniutek.com/account/keys) 。

bucket 是你在七牛云存储上希望保存数据的 Bucket 名（类似于数据库的表），这个自己选择一个合适的就可以，要求是只能由字母、数字、下划线等组成。

sync_dir 是本地需要上传的目录。这个目录中的所有内容会被同步到指定的 bucket 上。

在建立 task 文件后，就可以运行 qrsync 程序进行同步：

	qrsync foo.task

需要注意的是，qrsync 是增量同步的，如果你上一次同步成功后修改了部分文件，那么再次运行 qrsync 时只同步新增的和被修改的文件。当然，如果上一次同步过程出错了，也可以重新运行 qrsync 程序继续同步。


