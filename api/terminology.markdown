---
layout: default
title: "概念和术语"
---

- [空间（Bucket）](#Bucket)
- [键（Key）](#Key)
- [应用服务器（App-Server）](#App-Server)
- [应用客户端（App-Client）](#App-Client)
- [用户凭证（Access Key）](#Access-Key)
- [签名密钥（Secret Key）](#Secret-Key)
- [授权（Authentication）](#Authentication)
- [令牌（Token）](#Token)
- [云处理（FOP）](#FOP)
  - [异步云处理（AsyncOp，预处理）](#FOP-Async)
- [魔法变量（Magic Variable）](#Magic-Variable)
- [自定义变量（xVariable）](#Selfdef-Variable)
- [Callback（回调）、Return（返回）和重定向（Redirect）](#Callback-Return)


<a name="Bucket"></a>

### 空间（Bucket）

Bucket是容纳数据对象的容器。Bucket可以存放任意数量的数据对象，但一个数据对象只能存放在一个Bucket中。每个帐号可以拥有多个Bucket，但每个用户可以创建的Bucket数量存在上限。Bucket的名字在一个用户帐号内唯一，不能重复创建。

用户可以将Bucket设置为公有或私有。公有的Bucket，任何人都可以直接访问读取。私有的Bucket中，数据对象的访问则必须获得拥有者的授权才能访问。但无论是公有的，还是私有的Bucket，数据的上传和管理（移动、删除、复制等等）都必须获得拥有者的授权。

<a name="Key"></a>

### 键（Key）

Key是一个数据对象的名字，用以在Bucket中标识该数据对象。Key在Bucket中唯一。当用户使用一个已经存在的Key向Bucket中写入新数据对象后，原有的数据对象会被覆盖。

<a name="App-Server"></a>

### 应用服务器（App-Server）

App-Server是指使用七牛云存储的应用的业务服务器。这些服务器为七牛用户所有，运行了七牛用户的业务系统。App-Server可以是Web服务器，业务服务器等等。App-Server负责向App-Client（最终用户使用的客户端）数据对象的访问授权，并生成访问七牛云存储的HTTP请求。

<a name="App-Client"></a>

### 应用客户端（App-Client）

App-Client是指使用七牛云存储的应用的客户端（最终用户）。这些客户端的使用者并非七牛云存储的用户，没有直接访问七牛云存储的权限。他们对七牛云存储的访问需要客户应用向其颁发数据访问授权。

<a name="Access-Key"></a>

### 用户凭证（Access Key）

Access Key是七牛云存储颁发给用户的标识符。用户将Access Key放入访问请求，以便七牛云存储识别访问者的身份。Access Key和Secret Key成对颁发，不会重复。一个用户可以拥有多个Access Key/Secret Key，用于不同的访问。

<a name="Secret-Key"></a>

### 签名密钥（Secret Key）

Secret Key是七牛云存储颁发给用户，用于对访问请求签名的字串。用户使用Secret Key对访问请求的核心要素进行不可逆签名，获得请求认证token。用户将此token随同访问请求一起发送至七牛云存储服务，七牛云存储将对此token进行校验，以确认用户请求的合法性。

*** 注意 ： Secret Key是七牛云存储对用户访问安全验证的核心要素，用户必须妥善保管，不能泄露给第三方，亦不可置于最终用户使用的客户端中。如发生泄露，请立刻更换Access Key和Secret Key。 ***

<a name="Authentication"></a>

### 授权（Authentication）

七牛云存储的用户是应用或在线服务，最终用户并非七牛云存储的直接用户，七牛云存储只允许应用和在线服务访问私有的数据。但最终用户使用的客户端需要访问七牛云存储。因此，七牛云存储的用户需要向最终用户颁发私有数据访问的授权。用户构造HTTP请求，对其签名，获得token，并将其发送至最终用户使用的客户端。客户端通过带签名的HTTP请求向七牛云存储发起数据访问。

<a name="Token"></a>

### 令牌（Token）

Token是用户访问七牛云存储时，进行身份验证的凭证。当用户将一个Bucket设置为私有后，在访问七牛云存储时，必须通过身份验证。用户将访问请求中的一些要素整合起来，用Secret Key对其加密，得到token。然后将token随同请求一起发送至七牛云存储。用户可以在token中指定请求的时效，防止请求被非法使用。

*** 注： 七牛云存储服务统一使用UTC时间计算token有效期 ***

七牛云存储使用三种token：

##### 上传令牌（UploadToken）

用于上传数据对象。将Bucket、Key、失效时间等请求内容序列化成jason格式，使用hmac-sha1算法和Secret Key加密，并转换成url-safe的base64编码。

##### 下载令牌（DownloadToken）

用于下载数据对象。用户首先以query string的格式构造数据下载的url，然后加上请求时效参数，最后对该url执行hmac-sha1算法和Secret Key加密，进行url-safe的base64编码，生成token。

##### 管理令牌（AccessToken）

用于数据对象管理的身份验证token。其算法同DownloadToken类似：构造完管理操作的url，然后用Secret Key对其做hmac加密，进行base64编码，产生token。

<a name="FOP"></a>

### 云处理（FOP）

云处理是七牛云存储提供的数据处理功能。用户可以对存放在七牛云存储进行一系列的数据处理。云处理包括图片处理、音视频处理、文档转换等功能。用户的数据无需离开七牛云存储的数据中心，便可以进行各种类型的数据处理。方便用户，也节省了费用。

<a name="FOP-Async"></a>

#### 异步云处理（AsyncOp，预处理）

用户有时需要在一个数据对象上传完成后，对其进行某种数据处理，比如上传完成一张图片后，对其进行缩放。异步云处理便是为了方便这种用况。在七牛云存储的上传参数中，用户可以通过AsyncOp参数驱动七牛云存储，在完成数据对象上传后，启动相应的异步数据处理操作。所以，异步云处理也被称为“预处理”。“异步”的含义在于：数据对象上传完成后随即反馈用户，告知上传结果。同时，发起数据处理操作，但其结果将不再通知用户。

<a name="Magic-Variable"></a>

### 魔法变量（Magic Variable

魔法变量是七牛云存储提供的服务端的一些信息。魔法变量主要用于数据上传完成后，七牛服务端向用户反馈相关的信息。用户在数据上传时在ReturnBody，或者CallbackBody参数中，指定所需的魔法变量。上传完成后，服务器会填充相应的变量，然后在HTTP Response Body中，以JSON格式返回。

<a name="Selfdef-Variable"></a>

### 自定义变量（xVariable）

自定义变量是用户的App-Client同App-Server之间交换信息途径。主要用于数据上传。App-Client通过POST中的<x:...>参数携带自定义的变量信息。七牛云存储服务会根据CallbackBody参数中的设定，将请求中的自定义变量填充入发送至App-Server的Callback请求。

<a name="Callback-Return"></a>

### Callback（回调）、Return（返回）和Redirect（重定向）

七牛云存储有两种方式向使用者返回数据上传结果：Callback、Return和Redirect。

Callback是指七牛云存储服务端在完成数据上传完成后，向用户指定的url（App-Server）发送结果，App-Server可趁此机会进行一些处理，然后可以将一些信息反馈给七牛云存储服务端，七牛会再将这些结果反馈给App-Client。

Return是指七牛云存储服务端直接向App-Client返回结果。

Redirect是用于文件上传成功后，七牛云存储反馈301，引导浏览器跳转至用户指定的URL
