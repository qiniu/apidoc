---
layout: default
title: "概念和术语"
---

- [Bucket](#Bucket)
- [Key](#Key)
- [域名和自定义域名](#domain-selfdef)
- [上传](#Upload)
- [下载](#Download)
- [App-Server](#App-Server)
- [App-Client](#App-Client)
- [Access Key](#Access-Key)
- [Security Key](#Security-Key)
- [授权](#Authertication)
- [Token](#Token)
- [云处理](#FOP)
  - [异步云处理（预处理）](#FOP-Async)
- [魔法变量](#Magic-Variable)
- [自定义变量](#Selfdef-Variable)
- [Callback和Return]（#Callback-Return）


<a name="Bucket"></a>

### Bucket(空间)

Bucket是容纳数据对象的容器。Bucket可以存放任意数量的数据对象，但一个数据对象只能存放在一个Bucket中。每个帐号可以拥有多个Bucket，但每个用户可以创建的Bucket数量存在上限。Bucket的名字可以用于构成域名，因此在整个七牛云存储服务中唯一，一旦一个Bucket的名字被占用，就无法重复创建同名Bucket。

用户可以将Bucket设置为公有或私有。公有的Bucket，任何人都可以直接访问读取。私有的Bucket中，数据对象的访问则必须获得拥有者的授权才能访问。但无论是公有的，还是私有的Bucket，数据的上传和管理（移动、删除、复制等等）都必须获得拥有者的授权。

<a href="Key"></a>

### Key

Key是一个数据对象的名字，用以在Bucket中标识该数据对象。Key在Bucket中唯一。当用户使用一个已经存在的Key向Bucket中写入新数据对象后，原有的数据对象会被覆盖。

<a href="domain-selfdef"></a>

### 域名和自定义域名

七牛云存储提供一组域名对外提供服务。比如：rs.qbox.me、up.qiniu.me分别用于数据对象管理和数据上传。数据下载域名包含Bucket名称：<Bucket>.qiniudn.com，比如：mybucket.qiniudn.com。
用户也可以使用自己的域名绑定到七牛云存储的服务入口，比如：www.myblog.com。通过自定义的域名，用户可以更加方便地使用七牛云存储的服务。

<a href="Upload"></a>

### 上传

上传是指用户将数据对象保存至七牛云存储服务。七牛云存储将为用户永久保存这些数据，直到用户将其删除。用户可以采用直接上传、回调上传等多种模式向七牛云存储上传数据对象。同时还可以在上传的过程中对数据对象进行各种处理操作。

<a href="Download"></a>

### 下载

下载是指用户从七牛云存储服务获取指定数据对象。用户可以采用直接下载，或者授权下载的模式下载数据对象。在下载数据对象的同时，还可以对数据进行多种数据处理操作。

<a href="App-Server"></a>

### App-Server

App-Server是指使用七牛云存储的应用的业务服务器。这些服务器为七牛用户所有，运行了七牛用户的业务系统。App-Server可以是Web服务器，业务服务器等等。App-Server负责向App-Client（最终用户使用的客户端）数据对象的访问授权，并生成访问七牛云存储的HTTP请求。

<a href="App-Client"></a>

### App-Client

App-Client是指使用七牛云存储的应用的客户端（最终用户）。这些客户端的使用者并非七牛云存储的用户，没有直接访问七牛云存储的权限。他们对七牛云存储的访问需要客户应用向其颁发数据访问授权。

<a href="Access-Key"></a>

### Access Key

Access Key是七牛云存储颁发给用户的标识符。用户将Access Key放入访问请求，以便七牛云存储识别访问者的身份。Access Key和Security Key成对颁发，不会重复。一个用户可以拥有多个Access Key/Security Key，用于不同的访问。

<a href="Security-Key"></a>

### Security Key

Security Key是七牛云存储颁发给用户，用于对访问请求签名的字串。用户使用Security Key对访问请求的核心要素进行不可逆签名，获得请求认证token。用户将此token随同访问请求一起发送至七牛云存储服务，七牛云存储将对此token进行校验，以确认用户请求的合法性。

*** 注意 ： Security Key是七牛云存储对用户访问安全验证的核心要素，用户必须妥善保管，不能泄露给第三方，亦不可置于最终用户使用的客户端中。如发生泄露，请立刻更换Access Key和Security Key。 ***

<a href="Authertication"></a>

### 授权

七牛云存储的用户是应用或在线服务，最终用户并非七牛云存储的直接用户，七牛云存储只允许应用和在线服务访问私有的数据。但最终用户使用的客户端需要访问七牛云存储。因此，七牛云存储的用户需要向最终用户颁发私有数据访问的授权。用户构造HTTP请求，对其签名，获得token，并将其发送至最终用户使用的客户端。客户端通过带签名的HTTP请求向七牛云存储发起数据访问。

<a href="Token"></a>

### Token

Token是用户访问七牛云存储时，进行身份验证的凭证。当用户将一个Bucket设置为私有后，在访问七牛云存储时，必须通过身份验证。用户将访问请求中的一些要素整合起来，用Security Key对其加密，得到token。然后将token随同请求一起发送至七牛云存储。用户可以在token中指定请求的时效，防止请求被非法使用。

*** 注： 七牛云存储服务统一使用东八区时间（即北京时间） ***

七牛云存储使用三种token：

##### UploadToke

用于上传数据对象。将Bucket、Key、失效时间等请求内容序列化成jason格式，使用hmac-sha1算法和Security Key加密，并转换成url-safe的base64编码。

##### DownloadToken

用于下载数据对象。用户首先以query string的格式构造数据下载的url，然后加上请求时效参数，最后对该url执行hmac-sha1算法和Security Key加密，进行url-safe的base64编码，生成token。

##### AccessToken

用于数据对象管理的身份验证token。其算法同DownloadToken类似：构造完管理操作的url，然后用Security Key对其做hmac加密，进行base64编码，产生token。

<a href="FOP"></a>

### 云处理

云处理是七牛云存储提供的数据处理功能。用户可以对存放在七牛云存储进行一系列的数据处理。云处理包括图片处理、音视频处理、文档转换等功能。用户的数据无需离开七牛云存储的数据中心，便可以进行各种类型的数据处理。方便用户，也节省了费用。

<a href="FOP-Async"></a>

#### 异步云处理（预处理）

用户有时需要在一个数据对象上传完成后，对其进行某种数据处理，比如上传完成一张图片后，对其进行缩放。异步云处理便是为了方便这种用况。在七牛云存储的上传参数中，用户可以通过AsyncOp参数驱动七牛云存储，在完成数据对象上传后，启动相应的异步数据处理操作。所以，异步云处理也被称为“预处理”。“异步”的含义在于：数据对象上传完成后随即反馈用户，告知上传结果。同时，发起数据处理操作，但其结果将不再通知用户。

<a href="Magic-Variable"></a>

### 魔法变量

魔法变量是七牛云存储提供的服务端的一些信息。魔法变量主要用于数据上传完成后，七牛服务端向用户反馈相关的信息。用户在数据上传时在ReturnBody，或者CallbackBody参数中，指定所需的魔法变量。上传完成后，服务器会填充相应的变量，然后在HTTP Response Body中，以JSON格式返回。

<a href="Selfdef-Variable"></a>

### 自定义变量

自定义变量是用户的App-Client同App-Server之间交换信息途径。主要用于数据上传。App-Client通过POST中的<x:...>参数携带自定义的变量信息。七牛云存储服务会根据CallbackBody参数中的设定，将请求中的自定义变量填充入发送至App-Server的Callback请求。

<a href="Callback-Return"></a>

### Callback和Return

七牛云存储有两种方式向使用者返回数据上传结果：Return和Callback。Return是指七牛云存储服务端直接向App-Client返回结果。Callback是指七牛云存储服务端在完成数据上传完成后，向用户指定的url（App-Server）发送结果，App-Server可趁此机会进行一些处理，然后可以将一些信息反馈给七牛云存储服务端，七牛会再将这些结果反馈给App-Client。
