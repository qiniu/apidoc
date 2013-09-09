---
layout: default
title: "数据处理(持久化)"
---

- [持久化处理说明](#persistentOps-overall)
- [持久化处理机制](#persistentOps-method)
- [处理状态通知和查询](#persistentOps-status)
    - [通知](#notify)
    - [查询](#status)
    - [状态内容](#status-description)


<a name="persistentOps-overall"></a>

## 持久化处理说明  

一般的数据处理接口形式是：  

    [GET] http://<domain>/<key>?<fop>
    
比如：  
图片缩略

    http://cyj.qiniudn.com/22734/1359639667984p17i8ddoi31ara1ccp1njsq319s62.jpg?imageView/1/w/310/h/395/q/80

音频转码

    http://apitest.b1.qiniudn.com/sample.wav?avthumb/mp3/ab/192k
    
对于这类请求，服务端会实时对原资源进行处理、建立缓存，然后将结果返回给客户端。  
然而在实际应用中，用户可能会遇到缓存还未建立或已经失效的情况，需要等待服务端重新处理。如果是音视频文件的处理，耗时会相对较长，甚至可能长达数分钟。毫无疑问，这会影响用户体验。  
有音视频处理需求的用户可以使用数据持久化处理功能来避免上面提到的情况。此外，配合处理状态通知和查询功能，还能优化上传流程、提升用户体验。  

<a name="persistentOps-method"></a>
## 持久化处理机制  

### 上传  
用户使用持久化处理，需要在生成uploadToken时增加`persistentOps`和 `persistentNotifyUrl` 两个字段。  

字段 | 含义
----- | -------------
`persistentOps` | 需要进行的数据处理命令,可以指定多个命令，以`;`分隔。
`persistentNotifyUrl` | 用户接收视频处理结果的接口URL。

### 服务端处理  
用户使用指定了`persistentOps` 和 `persistentNotifyUrl` 的uploadToken上传一个音视频文件之后，服务端会生成此次处理的进程ID `persistentId`，并开始数据处理。  
`persistentId`可以用来获取处理的进度和结果。用户可以在`returnBody` 或 `callbackBody` 中使用魔法变量`$(persistentId)` 来得到该ID。  

### 下载  
服务端处理完成之后，用户即可通过  

    [GET] http://<domain>/<key>?p/1/<fop>  
    
这样形式的url访问处理结果。和普通的数据处理url不同，这里用`p/1`表明访问的是持久化处理的结果，不会出现重新处理耗费大量时间的情况。但需注意，如果访问一个没有在`persistentOps`中指定的处理结果，会直接返回404。  


<a name="persistentOps-status"></a>
## 状态通知和查询

<a name="notify"></a>
### 通知  
服务端完成所有的数据处理后，会以 HTTP POST 的方式将处理状态`<JsonStatusDescription>`以`application/json`的形式发送给用户指定的`persistentNotifyUrl`。  

    Content-Type: application/json
    Body: <JsonStatusDescription>
	
`<JsonStatusDescription>`的内容及含义参考： [状态内容](#persistentOps-status-description) 

<a name="status"></a>
### 查询  
用户也可以使用`persistentId`来主动查询数据处理的状态。查询的接口为：  

    [GET] http://api.qiniu.com/status/get/prefop?id=<persistentId>  

接口返回的内容及含义参考 [状态内容](#persistentOps-status-description)

<a name="status-description"></a>
### 状态内容  
用户获得的数据处理状态是一个JSON字符串，内容范例如下：  

	{
		"id": "16864pauo1vc9nhp12",
		"code": 0,
		"desc": "The fop was completed successfully",
		"items": [
			{
				"cmd": "avthumb/mp4/r/30/vb/256k/vcodec/libx264/ar/22061/ab/64k/acodec/libmp3lame",
				"code": 0,
				"desc": "The fop was completed successfully",
				"error": "",
				"hash": "FrPNF2qz66Bt14JMdgU8Ya7axZx-",
				"key": "v-PtT-DzpyCcqv6xNU25neTMkcc=/FjgJQXuH7OresQL4zgRqYG5bZ64x"
			},
			{
				"cmd": "avthumb/iphone_low",
				"code": 0,
				"desc": "The fop was completed successfully",
				"error": "",
				"hash": "FmZ5PbHMYD5uuP1-kHaLjKbrv-75",
				"key": "tZ-w8jHlQ0__PYJdiisskrK5h3k=/FjgJQXuH7OresQL4zgRqYG5bZ64x"
			},
			{
				"cmd": "avthumb/m3u8/r/30/vb/256k/vcodec/libx264/ar/22071/ab/64k/acodec/libmp3lame",
				"code": 0,
				"desc": "The fop was completed successfully",
				"error": "",
				"hash": "Fi4gMX0SvKVvptxfvoiuDfFkCuEG",
				"key": "8ehryqviSaMIjkVQDGeDcKRZ6qc=/FjgJQXuH7OresQL4zgRqYG5bZ64x"
			},
			{
				"cmd": "avthumb/m3u8/preset/video_16x9_440k",
				"code": 0,
				"desc": "The fop was completed successfully",
				"error": "",
				"hash": "FtuxnwAY9NVBxAZLcxNUuToR9y97",
				"key": "s2_PQlcIOz1uP6VVBXk5O9dXYLY=/FjgJQXuH7OresQL4zgRqYG5bZ64x"
			}
		]
	}


参数及含义  

参数 | 含义
---- | --------
`id` | 数据处理的进程ID，即前文中的`persistentId`。
`code` | 状态码，0 表示成功，1 表示等待处理，2 表示正在处理，3 表示处理失败，4 表示回调失败。
`desc` | 状态码对应的详细描述。
`items` | 列表，包含每个`fop`处理的完成情况。
`cmd` | 所执行的`fop`处理命令。
`error` | 如果处理失败，这个字段会给出详细的失败原因。
`hash` | 数据处理结果保存在服务端的唯一 hash 标识。
`key` | 数据处理结果的外链key。范例中，`avthumb/iphone_low`的处理结果可以通过`http://domian/tZ-w8jHlQ0__PYJdiisskrK5h3k=/FjgJQXuH7OresQL4zgRqYG5bZ64x`来直接访问。



## 处理实例  

1.  上传一个音频文件 **persistent.mp3** ，并设置处理命令为 `avthumb/mp3/aq/6/ar/16000` 和 `avthumb/mp3/ar/44100/ab/32k`。  
2.  通知接口收到的处理状态通知内容：  

    {  
        'code': 0,   
        'id': '168739cd2fn1g76f13',   
        'desc': 'The fop was completed successfully',  
        'items': [  
            {'code': 0, 'hash': 'FvvxM7gMI6WfiuXlBgKbkzU67Tpa', 'cmd': 'avthumb/mp3/ar/44100/ab/32k', 'key': 'sFhZ4dSjB1zvL3De1UBX2qZ_VR0=/lgxucMCQso_KOW_YDM-_KVIeX6o5', 'error': '', 'desc': 'The fop was completed successfully'},   
            {'code': 0, 'hash': 'FpSzDMYJtP_UY_6EMIyaBe4awXp3', 'cmd': 'avthumb/mp3/aq/6/ar/16000', 'key': '1G8-OWwP3jPLvi7O3qOf7yCl4YI=/lgxucMCQso_KOW_YDM-_KVIeX6o5', 'error': '', 'desc': 'The fop was completed successfully'}  
        ]  
    }  


3.  访问链接：  
[原文件](http://t-test.qiniudn.com/persistent.mp3)  
[处理1(avthumb/mp3/aq/6/ar/16000)结果](http://t-test.qiniudn.com/persistent.mp3?p/1/avthumb/mp3/aq/6/ar/16000)  
[处理2(avthumb/mp3/ar/44100/ab/32k)结果](http://t-test.qiniudn.com/persistent.mp3?p/1/avthumb/mp3/ar/44100/ab/32k)   


4.  用户也可以通过自定义样式来访问处理结果。我们设置样式 `persistent1` 和 `persistent2`，如图中示例。  

![persistent style](http://for-test.qiniudn.com/persistent_style.png)  

这样，用户可以通过带有样式的链接来访问处理结果：  
[样式'persistent1'](http://t-test.qiniudn.com/persistent.mp3-persistent1)  
[样式'persistent2'](http://t-test.qiniudn.com/persistent.mp3-persistent2)  

