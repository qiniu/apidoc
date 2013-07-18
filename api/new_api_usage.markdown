# 七牛云存储API使用手册

## 1. 基本概念和术语

### 1.1. Bucket(空间)

Bucket是容纳数据对象的容器。Bucket可以存放任意数量的数据对象，但一个数据对象只能存放在一个Bucket中。每个帐号可以拥有多个Bucket，但每个用户可以创建的Bucket数量存在上限。Bucket的名字可以用于构成域名，因此在整个七牛云存储服务中唯一，一旦一个Bucket的名字被占用，就无法重复创建同名Bucket。

用户可以将Bucket设置为公有或私有。公有的Bucket，任何人都可以直接访问读取。私有的Bucket中，数据对象的访问则必须获得拥有者的授权才能访问。但无论是公有的，还是私有的Bucket，数据的上传和管理（移动、删除、复制等等）都必须获得拥有者的授权。

### 1.2. Key

Key是一个数据对象的名字，用以在Bucket中标识该数据对象。Key在Bucket中唯一。当用户使用一个已经存在的Key向Bucket中写入新数据对象后，原有的数据对象会被覆盖。

### 1.3. 域名和自定义域名

七牛云存储提供一组域名对外提供服务。比如：rs.qbox.me、up.qiniu.me分别用于数据对象管理和数据上传。数据下载域名包含Bucket名称：<Bucket>.qiniudn.com，比如：mybucket.qiniudn.com。
用户也可以使用自己的域名绑定到七牛云存储的服务入口，比如：www.myblog.com。通过自定义的域名，用户可以更加方便地使用七牛云存储的服务。

### 1.4. 上传

上传是指用户将数据对象保存至七牛云存储服务。七牛云存储将为用户永久保存这些数据，直到用户将其删除。用户可以采用直接上传、回调上传等多种模式向七牛云存储上传数据对象。同时还可以在上传的过程中对数据对象进行各种处理操作。

### 1.5. 下载

下载是指用户从七牛云存储服务获取指定数据对象。用户可以采用直接下载，或者授权下载的模式下载数据对象。在下载数据对象的同时，还可以对数据进行多种数据处理操作。

### 1.6. App-Server

App-Server是指使用七牛云存储的应用的业务服务器。这些服务器为七牛用户所有，运行了七牛用户的业务系统。App-Server可以是Web服务器，业务服务器等等。App-Server负责向App-Client（最终用户使用的客户端）数据对象的访问授权，并生成访问七牛云存储的HTTP请求。

### 1.7. App-Client

App-Client是指使用七牛云存储的应用的客户端（最终用户）。这些客户端的使用者并非七牛云存储的用户，没有直接访问七牛云存储的权限。他们对七牛云存储的访问需要客户应用向其颁发数据访问授权。

### 1.8. Access Key

Access Key是七牛云存储颁发给用户的标识符。用户将Access Key放入访问请求，以便七牛云存储识别访问者的身份。Access Key和Security Key成对颁发，不会重复。一个用户可以拥有多个Access Key/Security Key，用于不同的访问。

### 1.9. Security Key

Security Key是七牛云存储颁发给用户，用于对访问请求签名的字串。用户使用Security Key对访问请求的核心要素进行不可逆签名，获得请求认证token。用户将此token随同访问请求一起发送至七牛云存储服务，七牛云存储将对此token进行校验，以确认用户请求的合法性。

*** 注意 ： Security Key是七牛云存储对用户访问安全验证的核心要素，用户必须妥善保管，不能泄露给第三方，亦不可置于最终用户使用的客户端中。如发生泄露，请立刻更换Access Key和Security Key。 ***

### 1.10. 授权

七牛云存储的用户是应用或在线服务，最终用户并非七牛云存储的直接用户，七牛云存储只允许应用和在线服务访问私有的数据。但最终用户使用的客户端需要访问七牛云存储。因此，七牛云存储的用户需要向最终用户颁发私有数据访问的授权。用户构造HTTP请求，对其签名，获得token，并将其发送至最终用户使用的客户端。客户端通过带签名的HTTP请求向七牛云存储发起数据访问。

### 1.11. Token

Token是用户访问七牛云存储时，进行身份验证的凭证。当用户将一个Bucket设置为私有后，在访问七牛云存储时，必须通过身份验证。用户将访问请求中的一些要素整合起来，用Security Key对其加密，得到token。然后将token随同请求一起发送至七牛云存储。用户可以在token中指定请求的时效，防止请求被非法使用。

*** 注： 七牛云存储服务统一使用东八区时间（即北京时间） ***

七牛云存储使用三种token：

##### UploadToke

用于上传数据对象。将Bucket、Key、失效时间等请求内容序列化成jason格式，使用hmac-sha1算法和Security Key加密，并转换成url-safe的base64编码。

##### DownloadToken

用于下载数据对象。用户首先以query string的格式构造数据下载的url，然后加上请求时效参数，最后对该url执行hmac-sha1算法和Security Key加密，进行url-safe的base64编码，生成token。

##### AccessToken

用于数据对象管理的身份验证token。其算法同DownloadToken类似：构造完管理操作的url，然后用Security Key对其做hmac加密，进行base64编码，产生token。

### 1.12. 云处理

云处理是七牛云存储提供的数据处理功能。用户可以对存放在七牛云存储进行一系列的数据处理。云处理包括图片处理、音视频处理、文档转换等功能。用户的数据无需离开七牛云存储的数据中心，便可以进行各种类型的数据处理。方便用户，也节省了费用。

#### 1.12.1. 异步云处理（预处理）

用户有时需要在一个数据对象上传完成后，对其进行某种数据处理，比如上传完成一张图片后，对其进行缩放。异步云处理便是为了方便这种用况。在七牛云存储的上传参数中，用户可以通过AsyncOp参数驱动七牛云存储，在完成数据对象上传后，启动相应的异步数据处理操作。所以，异步云处理也被称为“预处理”。“异步”的含义在于：数据对象上传完成后随即反馈用户，告知上传结果。同时，发起数据处理操作，但其结果将不再通知用户。

### 1.13. 魔法变量

魔法变量是七牛云存储提供的服务端的一些信息。魔法变量主要用于数据上传完成后，七牛服务端向用户反馈相关的信息。用户在数据上传时在ReturnBody，或者CallbackBody参数中，指定所需的魔法变量。上传完成后，服务器会填充相应的变量，然后在HTTP Response Body中，以JSON格式返回。

### 1.14. 自定义变量

自定义变量是用户的App-Client同App-Server之间交换信息途径。主要用于数据上传。App-Client通过POST中的<x:...>参数携带自定义的变量信息。七牛云存储服务会根据CallbackBody参数中的设定，将请求中的自定义变量填充入发送至App-Server的Callback请求。

### 1.15. Callback和Return

七牛云存储有两种方式向使用者返回数据上传结果：Return和Callback。Return是指七牛云存储服务端直接向App-Client返回结果。Callback是指七牛云存储服务端在完成数据上传完成后，向用户指定的url（App-Server）发送结果，App-Server可趁此机会进行一些处理，然后可以将一些信息反馈给七牛云存储服务端，七牛会再将这些结果反馈给App-Client。

## 2. 服务访问基础

### 2.1. 服务入口

七牛云存储以HTTP的形式对外提供服务，主要使用GET、POST指令。GET请求主要用于数据对象下载、云处理。POST请求用于数据上传和数据管理。

不同的服务使用不同的域名：

1. 数据上传：up.qiniu.com
2. 数据下载、云处理：\<bucket>.qiniudn.com
3. 数据对象管理：rs.qbox.me
4. 高级管理功能：rsf.qbox.me

### 2.2. 服务功能

七牛云存储服务基本功能包括两大类：数据存储和云处理。数据存储包含上传、下载和管理三个部分。云处理则依托数据存储，为用户提供进一步的数据处理服务。

#### 2.2.1. 上传

数据对象的上传是用户使用七牛云存储的起点。数据上传有诸多来源。如果用户是移动应用开发商，那么上传的数据来自移动客户端的图片、视频、音频等等数据；如果用户是个网站，那么可能是最终用户上传的文件和数据；如果用户仅仅是使用七牛云存储进行数据备份，那么他可能更多地是简单地从自己的服务器直接向七牛云存储上传数据。

这些不同的用况，需要用户采用不同的方式使用七牛云存储。因此，七牛云存储提供三种使用模式，满足各种不同的使用需求。这三种模式分别为：本地上传（直接上传），普通上传（客户端上传）和高级上传（回调上传）。

** 本地上传 ** 是七牛云存储的用户从他们自己的计算机上，发起上传操作。由于是在用户自己的计算机，不存在授权问题，可以直接使用Security Key。最简便的方法是使用qrsync或qiniu-autosync工具进行手动或自动的上传。如果用户希望在自己的软件系统中进行上传操作，那么可以使用七牛云存储SDK，或者直接通过HTTP访问七牛云存储。

基本流程如下：

              *************
          ****             ****
        **                     **
      **                         **
      *    Qiniu-Cloud-Storage    *
      **                         **
        **                     **
          ****             ****
              *************
                  ^  |
                  |  |
                  |  |
      (1) Request |  |
           (put)  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  | (2) Response
                  |  |
                  |  |
                  |  v
       +-------------------------+
       |                         |
       |                         |
       |                         |
       | User Server or Computer |
       |                         |
       |                         |
       |                         |
       +-------------------------+

1. 用户使用自己的服务器或计算机访问文件 URL 发起数据上传
2. Qiniu-Cloud-Storage 将上传结果响应用户，无论成功或者失败。

这种模式通常用于上传存放在用户服务器的数据对象。比如某些数据在用户服务器上计算和处理完成后，再上传至七牛云存储。或者将一些存档文件备份到七牛云存储。

有时用户为了简单，往往让App-Client将数据上传到用户自己的服务器，然后再转发到七牛云存储。我们不推荐这种使用方式，因为数据的反复出入，成倍增加用户服务器的流量。并且在大量数据上传的情况下，用户服务器不得不承受巨大的数据访问压力。



** 普通上传 ** 有更广泛的应用场景。当七牛用户是应用开发商、网站、或者服务提供商的时候，会需要从大量的最终用户的客户端上传数据对象。由于最终用户不是七牛云存储的直接用户，所以他们无法直接发起上传操作。因此，需要使用七牛云存储的用户向最终用户（客户端）授权上传。授权通过UploadToken实现。七牛用户的服务器构造上传请求，然后计算出UploadToken，整合成完整的上传请求，发送至App-Client。App-Client在需要的时候，使用上传请求向七牛云存储上传数据对象。

基本流程如下：


                                       *************
                                   ****             ****
                                 **                     **
                               **                         **
                               *    Qiniu-Cloud-Storage    *
                               **                         **
                                 **                     **
                               ^   ****             ****
                              /   /    *************
                             /   /
                            /   /
                           /   /
                          /   /
                         /   /
                        /   / (4) Return Result
                       /   /
      (3) Upload File /   /
                     /   /
                    /   /
                   /   /
                  /   /
                 /   /
                /   /
               /   /
              /   v
      +------------------+                                        +------------------+
      |                  |                                        |                  |
      |                  |    (1) Request Upload (can be once)    |                  |
      |                  |--------------------------------------->|                  |
      |    App-Client    |                                        |    App-Server    |
      |                  |<---------------------------------------|                  |
      |                  |    (2) Make Policy / UploadToken       |                  |
      |                  |                                        |                  |
      +------------------+                                        +------------------+
               |                                                           ^
               |              (5) Callback                                 |
               +-----------------------------------------------------------+

1. App-Client 向 App-Server 请求上传文件
2. App-Server 使用 Qiniu-SDK 生成上传授权凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传授权许可（UploadToken）后，使用 Qiniu-Client-SDK 直传文件到最近的存储节点
4. 文件上传成功后，Qiniu 返回给 App-Client 上传结果（可包含相应的文件信息）
5. App-Client 将文件上传结果及相关信息汇报给 App-Server，App-Server 可写表做记录等操作



** 高级上传 ** 为了满足一些更复杂的用况：使用七牛云存储的应用使用移动客户端，或者web之类的瘦客户端。但又是对于上传数据对象后反馈的结果处理，需要服务端进行。使用普通上传，用户只能由客户端接收结果，然后再传送回应用自己的服务器。这个过程繁琐，而且易错。因此，七牛云存储允许用户在上传数据对待时，提供一个可回调的url。当七牛云存储完成数据上传操作后，回调这个url，将结果直接反馈给用户的服务器。而且，用户还可以在回调的响应中携带额外的信息，七牛云存储会将这些信息传递给客户端，进一步简化客户端的逻辑。

                                           *************
                                       ****             ****
                                     **                     **
                                   **                         **
                                   *    Qiniu-Cloud-Storage    *
                                   **                         **
                                     **                     **
                                   ^   ****             ****    \
                                  /   /    *************     ^   \
                                 /   /                        \   \
                                /   /                          \   \
                               /   /                            \   \
                              /   /                              \   \
                             /   /                                \   \
                            /   /                                  \   \ (4) Callback
                           /   /                                    \   \
          (3) Upload File /   /                                      \   \
                         /   /                                        \   \
                        /   / (6) Return Result                        \   \
                       /   /                                            \   \
                      /   /                            (5) Return Result \   \
                     /   /                                                \   \
                    /   /                                                  \   \
                   /   /                                                    \   \
                  /   v                                                      \   v
          +------------------+                                        +------------------+
          |                  |                                        |                  |
          |                  |    (1) Request Upload                  |                  |
          |                  |--------------------------------------->|                  |
          |    App-Client    |                                        |    App-Server    |
          |                  |<---------------------------------------|                  |
          |                  |    (2) Make Policy / UploadToken       |                  |
          |                  |                                        |                  |
          +------------------+                                        +------------------+


1. App-Client 向 App-Server 请求上传文件
2. App-Server 使用 Qiniu-SDK 生成上传授权凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传授权许可（UploadToken）后，使用 Qiniu-Client-SDK 直传文件到最近的存储节点
4. 文件上传成功后，Qiniu 以 HTTP POST 方式告知 App-Server 上传结果（可包含相应的文件信息）
5. App-Server 可写表做记录等操作，然后经 Qiniu 中转返回给 App-Client 它想要的信息
6. Qiniu 作为代理，原封不动地将回调 App-Server 的返回结果回传给 App-Client

**高级上传相对于普通上传的优势体现在以下几方面**:

1. App Client 无需向 App-Server 发送通知，全部统一由 Qiniu 发送 Callback，当存在多种终端（比如Web/iOS/Android）都需要上传文件时，每个终端不需要各自处理同业务服务器进行数据交换的业务逻辑。

2. Callback 环节加速，七牛云存储的就近节点能比 App-Client 以更优异的网络回调 App-Server 。

3. 只要文件上传成功，App-Server 必然知情。即使 App-Server 回调失败，App-Client 还是会得到完整的回调数据，可自定义策略进行异步处理。


#### 2.2.2. 下载

下载相对上传简单很多。分为共有资源下载和私有资源下载。

#### 2.2.3. 数据管理

#### 2.2.4. 云处理

### 2.3. 访问模式

#### 2.3.1. 直接访问

#### 2.3.2. 公开资源访问

#### 2.3.3. 授权访问

### 2.4. 授权和签名

#### 2.4.1. 签名算法

#### 2.4.2. Token的生成

##### UploadToken

##### DownloadToken

##### AccessToken

## 3. 服务使用

### 3.1. 上传

#### 3.1.1. 本地上传
若您需要上传已存在您电脑或者是服务器上的文件到七牛云存储，可以直接使用七牛提供的上传工具：

| 名称                                         | 使用   | 适用平台                     | 说明                                 |
|----------------------------------------------|--------|------------------------------|--------------------------------------|
| [qrsync](/tools/qrsync.html)                 | 命令行 | Linux,Windows,MacOSX,FreeBSD | 手动同步文件/文件夹到七牛云存储      |
| [qiniu-autosync](/tools/qiniu-autosync.html) | 命令行 | Linux                        | 自动同步指定文件夹内的新增或改动文件 |


如果是需要通过网站(Web)或是移动应用(App)等客户端上传文件，则可以参考如下 UGC (User Generated Content) 上传流程。


#### 3.1.2. 普通上传

```
                                       *************
                                   ****             ****
                                 **                     **
                               **                         **
                               *    Qiniu-Cloud-Storage    *
                               **                         **
                                 **                     **
                               ^   ****             ****
                              /   /    *************
                             /   /
                            /   /
                           /   /
                          /   /
                         /   /
                        /   / (4) Return Result
                       /   /
      (3) Upload File /   /
                     /   /
                    /   /
                   /   /
                  /   /
                 /   /
                /   /
               /   /
              /   v
      +------------------+                                        +------------------+
      |                  |                                        |                  |
      |                  |    (1) Request Upload (can be once)    |                  |
      |                  |--------------------------------------->|                  |
      |    App-Client    |                                        |    App-Server    |
      |                  |<---------------------------------------|                  |
      |                  |    (2) Make Policy / UploadToken       |                  |
      |                  |                                        |                  |
      +------------------+                                        +------------------+
               |                                                           ^
               |              (5) Callback                                 |
               +-----------------------------------------------------------+
```

1. App-Client 向 App-Server 请求上传文件
2. App-Server 使用 Qiniu-SDK 生成上传授权凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传授权许可（UploadToken）后，使用 Qiniu-Client-SDK 直传文件到最近的存储节点
4. 文件上传成功后，Qiniu 返回给 App-Client 上传结果（可包含相应的文件信息）
5. App-Client 将文件上传结果及相关信息汇报给 App-Server，App-Server 可写表做记录等操作

##### ReturnUrl

##### ReturnBody

#### 3.1.3. 高级上传（回调）


                                           *************
                                       ****             ****
                                     **                     **
                                   **                         **
                                   *    Qiniu-Cloud-Storage    *
                                   **                         **
                                     **                     **
                                   ^   ****             ****    \
                                  /   /    *************     ^   \
                                 /   /                        \   \
                                /   /                          \   \
                               /   /                            \   \
                              /   /                              \   \
                             /   /                                \   \
                            /   /                                  \   \ (4) Callback
                           /   /                                    \   \
          (3) Upload File /   /                                      \   \
                         /   /                                        \   \
                        /   / (6) Return Result                        \   \
                       /   /                                            \   \
                      /   /                            (5) Return Result \   \
                     /   /                                                \   \
                    /   /                                                  \   \
                   /   /                                                    \   \
                  /   v                                                      \   v
          +------------------+                                        +------------------+
          |                  |                                        |                  |
          |                  |    (1) Request Upload                  |                  |
          |                  |--------------------------------------->|                  |
          |    App-Client    |                                        |    App-Server    |
          |                  |<---------------------------------------|                  |
          |                  |    (2) Make Policy / UploadToken       |                  |
          |                  |                                        |                  |
          +------------------+                                        +------------------+


1. App-Client 向 App-Server 请求上传文件
2. App-Server 使用 Qiniu-SDK 生成上传授权凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传授权许可（UploadToken）后，使用 Qiniu-Client-SDK 直传文件到最近的存储节点
4. 文件上传成功后，Qiniu 以 HTTP POST 方式告知 App-Server 上传结果（可包含相应的文件信息）
5. App-Server 可写表做记录等操作，然后经 Qiniu 中转返回给 App-Client 它想要的信息
6. Qiniu 作为代理，原封不动地将回调 App-Server 的返回结果回传给 App-Client


                                           *************
                                       ****             ****
                                     **                     **
                                   **                         **
                                   *    Qiniu-Cloud-Storage    *
                                   **                         **
                                     **                     **
                                   ^   ****             ****    \
                                  /   /    *************     ^   \
                                 /   /                        \   \
                                /   /                          \   \
                               /   /                            \   \
                              /   /                              \   \
                             /   /                                \   \
                            /   /                                  \   \ (4) Callback
                           /   /                                    \   \
          (3) Upload File /   /                                      \   \
                         /   /                                        \   \
                        /   / (6) Return Result                        \   \
                       /   /                                            \   \
                      /   /                            (5) Return Result \   \
                     /   /                                                \   \
                    /   /                                                  \   \
                   /   /                                                    \   \
                  /   v                                                      \   v
          +------------------+                                        +------------------+
          |                  |                                        |                  |
          |                  |    (1) Request Upload                  |                  |
          |                  |--------------------------------------->|                  |
          |    App-Client    |                                        |    App-Server    |
          |                  |<---------------------------------------|                  |
          |                  |    (2) Make Policy / UploadToken       |                  |
          |                  |                                        |                  |
          +------------------+                                        +------------------+


1. App-Client 向 App-Server 请求上传文件
2. App-Server 使用 Qiniu-SDK 生成上传授权凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传授权许可（UploadToken）后，使用 Qiniu-Client-SDK 直传文件到最近的存储节点
4. 文件上传成功后，Qiniu 以 HTTP POST 方式告知 App-Server 上传结果（可包含相应的文件信息）
5. App-Server 可写表做记录等操作，然后经 Qiniu 中转返回给 App-Client 它想要的信息
6. Qiniu 作为代理，原封不动地将回调 App-Server 的返回结果回传给 App-Client

**其中模型2相对于模型1更为高级，体现在以下几方面**:

1. App Client 无需向 App-Server 发送通知，全部统一由 Qiniu 发送 Callback，当存在多种终端（比如Web/iOS/Android）都需要上传文件时，每个终端不需要各自处理 Callback 业务逻辑。

2. Callback 环节加速，七牛云存储的就近节点能比 App-Client 以更优异的网络回调 App-Server 。

3. 只要文件上传成功，App-Server 必然知情。即使 App-Server 回调失败，App-Client 还是会得到完整的回调数据，可自定义策略进行异步处理。


**注意**

- 以上两种上传模型中，步骤(1)和步骤(2)中 App-Client 获取上传授权凭证（UploadToken）不用重复频繁获取，UploadToken 可通过 `deadline` 选项设置有效期，在设定的有效期内可多次复用。后续 [上传授权凭证 - uploadToken 算法说明](#uploadToken-algorithm) 会解释各选项的具体作用。


| 适用平台                                                              |
| --------------------------------------------------------------------- |
| APP - 移动端应用（iOS、Android、WindowsPhone、BlackBerry、Symbian 等）|
| WEB - 浏览器网页                                                      |


##### CallbackUrl

##### CallbackBody

#### 3.1.4. 上传数据预处理（异步数据处理）

七牛云存储的云处理API（图像/音视频在线处理）满足如下规格:

    url?<fop>

即基于文件的 URL 通过问号传参来实现即时云处理，`<fop>` 表示 API 指令及其所需要的参数，是 File Operation 概念的缩写。

例如,

- [http://apitest.b1.qiniudn.com/sample.wav?avthumb/mp3/ar/44100/aq/3](http://apitest.b1.qiniudn.com/sample.wav?avthumb/mp3/ar/44100/aq/3)

其中,

- `url = http://apitest.b1.qiniudn.com/sample.wav`
- `fop = avthumb/mp3/ar/44100/aq/3`

表示将原 wav 格式的音频转换为 mp3 格式，并指定动态码率（VBR）参数为3，采样频率为 44100 进行输出。

由于音视频文件一般都比较大，转换也是一个比较耗时的操作，故七牛云存储提供上传异步预转功能，即文件上传完毕后执行异步转换处理，这样在访问时即可获取到已经转换好了的目标文件。

可以在上传时候指定预转选项，只需在生成 uploadToken 时对 **asyncOps** 赋值相应的 `<fop>` 指令即可。可同时异步执行多个预转指令：

    asyncOps = <fop>[;<fop2>;<fop3>;…;<fopN>]

**asyncOps** 预转示例参见如下说明。

**上传**

1. 设定 `asyncOps = "avthumb/mp3/ar/44100/ab/32k;avthumb/mp3/aq/6/ar/16000"`
2. 以此生成带有预转功能的上传授权凭证（UploadToken）
3. 向七牛云存储上传一个 aac 格式的音频文件
4. 传成功后，服务器会对这个 aac 音频文件异步地做如下两个预转操作
    - `avthumb/mp3/ar/44100/ab/32k`
    - `avthumb/mp3/aq/6/ar/16000`

**下载**

可以通过 `http://<domain>/<key>` 的形式下载：

- `http://<bucket>.qiniudn.com/<key>?avthunm/mp3/ar/44100/ab/32k`
- `http://<bucket>.qiniudn.com/<key>?avthumb/mp3/aq/6/ar/16000`

如果有为 `<fop>` 定义 `<style>`, 那么也可以用友好URL风格进行访问。

我们先来熟悉 [qboxrsctl](/tools/qboxrsctl.html) 的两个命令行，

    // 定义 url 和 fop 之间的分隔符为 separator 
    qboxrsctl separator <bucket> <separator>

    // 定义 fop 的别名为 aliasName
    qboxrsctl style <bucket> <aliasName> <fop>

例如:

    qboxrsctl separator <bucket> "."
    qboxrsctl style <bucket> "mp3" "avthumb/mp3/aq/6/ar/16000"

那么，以下两个 URL 则等价:

- `http://<bucket>.qiniudn.com/<key>?avthumb/mp3/aq/6/ar/16000`
- `http://<bucket>.qiniudn.com/<key>.mp3`


访问以上链接，如果之前上传已经成功做完预转，那么此次请求就不需要再转换，将会直接下载预转后的结果文件。

图片、视频预转类似，开发者需要熟悉七牛云存储的更多 `<fop>` 指令，参考:


#### 3.1.5. 断点续上传 ？？？


### 3.2. 下载

#### 3.2.1. 本地下载

#### 3.2.2. 公开数据下载

**格式**

    [GET] http://<bucket>.qiniudn.com/<key>

**或者**

    [GET] http://<domain>/<key>

**或者**

    [GET] http://<domain>/<key>?<fop>/<params>

Qiniu-Cloud-Storage 的云处理（图片/音频/视频）API 满足 `url?<fop>/<params>` 这种格式，`<fop>` 表示具体的云处理指令，例如：`url?imageView/1/w/480/h/320` 表示一个缩略图预览的URL，其中 `imageView` 是具体的 `<fop>`, `1/w/480/h/320` 是该 fop 的 `<params>`。


**参数**

名称   | 说明
-------|---------------------------------------------------
bucket | 空间名称
key    | 上传时 App-Client 端指定的文件ID，在指定空间内唯一
domain | bucket 绑定的自定义域名

**流程**

              *************
          ****             ****
        **                     **
      **                         **
      *    Qiniu-Cloud-Storage    *
      **                         **
        **                     **
          ****             ****
              *************
                  ^  |
                  |  |
                  |  |
      (1) Request |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  |
                  |  | (2) Response
                  |  |
                  |  |
                  |  v
       +-------------------------+
       |                         |
       |                         |
       |                         |
       |        App-Client       |
       |(Web/iOS/Android/etc,...)|
       |                         |
       |                         |
       |                         |
       +-------------------------+

1. App-Client 访问文件 URL 请求下载资源
2. Qiniu-Cloud-Storage 响应 App-Client, 命令距离 App-Client 物理距离最近的 IO 节点输出文件内容
3. Response 过程中支持 [断点续下载](#download-by-range)


#### 3.2.3. 授权下载

**格式**

    [GET] http://<bucket>.qiniudn.com/<key>?e=<deadline>&token=<downloadToken>

**或者**

    [GET] http://<domain>/<key>?e=<deadline>&token=<downloadToken>

**或者**

    [GET] http://<domain>/<key>?<fop>/<params>&e=<deadline>&token=<downloadToken>


**参数**

名称          | 说明
--------------|-------------------------------------------------------------------------------------------------------------
bucket        | 空间名称
key           | 上传时 App-Client 端指定的文件ID，在指定空间内唯一
domain        | bucket 绑定的自定义域名
deadline      | 失效期，标准的Unix timestamp形式，从1970年1月1日（UTC/GMT的午夜）开始到失效期所经过的秒数，过了这个时间点之后，后续请求无效
downloadToken | 下载授权凭证，由 App-Server 根据 [downloadToken 签名算法](#download-token-algorithm) 生成并颁发给 App-Client

**流程**

                                         *************
                                     ****             ****
                                   **                     **
                                 **                         **
                                 *    Qiniu-Cloud-Storage    *
                                 **                         **
                                   **                     **
                                 ^   ****             ****
                                /   /    *************
                               /   /
                              /   /
        (3) Request Download /   /
                            /   /
                           /   /
                          /   /
                         /   /  (4) Return Result
                        /   /
                       /   /
                      /   /
                     /   /
                    /   /
                   /   /
                  /   /
                 /   /
                /   v
        +------------------+                                        +------------------+
        |                  |                                        |                  |
        |                  |    (1) Request (can be once)           |                  |
        |                  |--------------------------------------->|                  |
        |    App-Client    |                                        |    App-Server    |
        |                  |<---------------------------------------|                  |
        |                  |    (2) Return DownloadToken            |                  |
        |                  |                                        |                  |
        +------------------+                                        +------------------+

1. App-Client 向 App-Server 请求下载授权
2. App-Server 根据 [downloadToken 签名算法](#download-token-algorithm) 生成 downloadToken, 并颁发给 App-Client
3. App-Client 拿到 downloadToken 后，向 Qiniu-Cloud-Storage 请求下载文件
4. Qiniu-Cloud-Storage 在校验 downloadToken 成功后，输出文件内容。如果校验失败，返错误信息（401 bad token）
5. Qiniu-Cloud-Storage 输出文件内容过程中支持 [断点续下载](#download-by-range)


#### 3.2.4. 在下载中执行云处理

#### 3.2.5. 断点续下载

断点续下载协议标准参考：<http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.35>

七牛云存储按以上标准支持断点续下载，只需在 HTTP 请求下载链接的头部附带 `Range` 字段即可。

    Range: bytes=<first-byte-pos>-<last-byte-pos>

**参数**

名称               | 说明
-------------------|-------------------------------------
`<first-byte-pos>` | 起始位置，从数字0开始计算
`<last-byte-pos>`  | 断点续下载中，最后一个字节所在的位置


#### 3.2.6. 下载SaveAs

    [GET] url?download/<saveAsFriendlyName>

如上 API 规格，`download` 是一个 `fop` 指令，`<saveAsFriendlyName>` 是该 fop 指令的参数，表示下载资源保存在本地的文件名称，一般建议带上文件后缀。

公有资源下载

    [GET] http://<domain>/<key>?download/<saveAsFriendlyName>

私有资源下载

    [GET] http://<domain>/<key>?download/<saveAsFriendlyName>&e=<deadline>&token=<downloadToken>

**示例**

- <http://qiniuphotos.qiniudn.com/gogopher.jpg?download/test.jpg>


#### 3.2.7. 自定义404

七牛云存储支持自定义 404 Not Found 处理，当一个资源不存在，可以让该请求命中一个缺省的资源。该资源可以是一张图片，也可以是一段HTML等。

要实现自定义 404 Not Found，只需向指定的 bucket (存储空间) 上传一个 key (FileID) 为 `errno-404` 的文件即可。


### 3.3. 管理操作

#### 3.3.1. 列出文件

请求某个存储空间（bucket）下的文件列表，如果有前缀，可以按前缀（`prefix`）进行过滤；如果前一次返回`marker`就表示还有资源，下一步请求需要将`marker`参数填上。

    HTTP/1.1
    POST http://rsf.qbox.me/list?bucket=<BucketName>&
                                 marker=<Marker>&
                                 limit=<Limit>&
                                 prefix=<Prefix>
    Content-Type: application/x-www-form-urlencoded
    Request Headers: {
        Authorization: QBox <AccessToken>
    }

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Response Body: {
        marker: <Marker> //如果没有剩余条目，服务器返回这项为空
        items: [
            {
                key：<Key>,
                time: <FilePutTime>,
                hash: <FileETag>,
                fsize: <FileSize>,
                mimeType: <MimeType>,
                customer: <EndUserId>
            },
            ...
        ]
    }

**请求参数**

名称   | 必填项 | 说明
-------|--------|-------------------------------------------
bucket | 是     | 存储空间名称
marker | 否     | 为服务器上次导出时返回的标记，没有可以不填
limit  | 否     | 单次查询返回的最大条目数，最大不超过1000
prefix | 否     | 指定要过滤出来的前缀


#### 3.3.2. 单文件操作

#### 3.3.3. 批量操作

### 3.4. 云处理

#### 3.4.1. 图像处理

##### 图片信息

**请求**

    GET <ImageDownloadURL>?imageInfo

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        format: <ImageType> // "png", "jpeg", "gif", "bmp", etc.
        width: <ImageWidth>
        height: <ImageHeight>
        colorModel: <ImageColorModel> // "palette16", "ycbcr", etc.
    }


##### EXIF

**请求**

    GET <ImageDownloadURL>?exif

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        // ...EXIF Data...
    }


##### 缩略图

    [GET] <ImageDownloadURL>?imageView/<mode>
                             /w/<Width>
                             /h/<Height>
                             /q/<Quality>
                             /format/<Format>


**响应**

    200 OK
    <ImageBinaryData>

**请求参数详解**

参数名称    | 说明
------------|-------------------------------------------------------------------
`<mode>`    | 图像缩略处理的模式
`<Width>`   | 指定目标缩略图的宽度，单位：像素（px）
`<Height>`  | 指定目标缩略图的高度，单位：像素（px）
`<Quality>` | 指定目标缩略图的图像质量，取值范围 1-100
`<Format>`  | 指定目标缩略图的输出格式，取值范围：jpg, gif, png, webp 等图片格式


其中 `<mode>` 分为如下几种情况：

模式         | 说明
-------------|------------------------------------------------------------------------------------------------
`<mode>=1` | 表示限定目标缩略图的宽度和高度，放大并从缩略图中央处裁剪为指定 `<Width>x<Height>` 大小的图片。
`<mode>=2` | 指定 `<Width>` 和 `<Height>`，表示限定目标缩略图的长和宽，将缩略图的大小限定在指定的宽高矩形内。
`<mode>=2` | 指定 `<Width>` 但不指定 `<Height>`，表示限定目标缩略图的宽度，高度等比缩略自适应。
`<mode>=2` | 指定 `<Height>` 但不指定 `<Width>`，表示限定目标缩略图的高度，宽度等比缩略自适应。

**示例**

示例1：针对原图进行缩略，并从缩略图的中央部位裁剪为 200x200 的缩略图：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200

![200x200](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200)

示例2：针对原图进行缩略，并限定目标缩略图的长边为 200 px，短边自适应，缩略图宽和高都不会超出 200px：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200/h/200

![限定长边为 200px，短边自适应，宽和高都不会超出 200px](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200/h/200)

示例3：针对原图进行缩略，并限定目标缩略图的宽度为 200px，高度等比缩略自适应：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200

![限定宽度为 200px, 高度等比缩略自适应](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200)

示例4：针对原图进行缩略，并限定目标缩略图的高度为 200px，宽度等比缩略自适应：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200

![限定高度为 200px, 宽度等比缩略自适应](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200)


##### 水印

    [GET] <ImageDownloadURL>?watermark/<Mode>/xxx

其中，`<ImageDownloadURL>` 必须返回一张图片。

`<Mode>` = 1 时，表示图片水印：

    [GET] <ImageDownloadURL>?watermark/1
                             /image/<EncodedImageURL>
                             /dissolve/<Dissolve>
                             /gravity/<Gravity>
                             /dx/<DistanceX>
                             /dy/<DistanceY>

`<Mode>` = 2 时，表示纯文本水印：

    [GET] <ImageDownloadURL>?watermark/2
                             /text/<EncodedText>
                             /font/<EncodedFontName>
                             /fontsize/<FontSize>
                             /fill/<EncodedTextColor>
                             /dissolve/<Dissolve>
                             /gravity/<Gravity>
                             /dx/<DistanceX>
                             /dy/<DistanceY>


**参数**

名称                 | 必填 | 说明
---------------------|------|-----------------------------------------------------
`<EncodedImageURL>`  | 是   | 水印图片，使用图片水印时需指定用于水印的远程图片URL。`EncodedImageURL = urlsafe_base64_encode(ImageURL)`
`<EncodedText>`      | 是   | 水印文本，文字水印时必须。`EncodedText = urlsafe_base64_encode(Text)`
`<EncodedFontName>`  | 否   | 字体名，若水印文本为非英文字符（比如中文）构成，则必须。`EncodedFontName = urlsafe_base64_encode(FontName)`
`<FontSize>`         | 否   | 字体大小，0 表示默认，单位: 缇，等于 1/20 磅。
`<EncodedTextColor>` | 否   | 字体颜色。`EncodedTextColor = urlsafe_base64_encode(TextColor)`。RGB格式，可以是颜色名称（比如 `red`）或十六进制（比如 `#FF0000`），参考 [RGB颜色编码表](http://www.rapidtables.com/web/color/RGB_Color.htm)
`<Dissolve>`         | 否   | 透明度，取值范围 1-100，默认值 `100`，即表示 100%（不透明）。
`<Gravity>`          | 否   | 位置，默认值为 `SouthEast`（右下角）。可选值：`NorthWest`, `North`, `NorthEast`, `West`, `Center`, `East`, `SouthWest`, `South`, `SouthEast` 。
`<DistanceX>`        | 否   | 横向边距，单位：像素（px），默认值为 10。
`<DistanceY>`        | 否   | 纵向边距，单位：像素（px），默认值为 10。

`urlsafe_base64_encode(string)` 函数的实现符合 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 标准，开发者可以参考 <https://github.com/qiniu> 上各SDK的样例代码。

**示例**

图片水印样例

 - 水印图片: <http://www.b1.qiniudn.com/images/logo-2.png>
     - `ImageURL = "http://www.b1.qiniudn.com/images/logo-2.png"`
     - `EncodedImageURL = urlsafe_base64_encode(ImageURL)`
 - 水印透明度: 50% (`dissolve=50`)
 - 水印位置: 右下角 (`gravity=SouthEast`)
 - 横向边距: 20px
 - 纵向边距: 20px

![图片水印](http://qiniuphotos.qiniudn.com/gogopher.jpg?watermark/1/image/aHR0cDovL3d3dy5iMS5xaW5pdWRuLmNvbS9pbWFnZXMvbG9nby0yLnBuZw==/dissolve/50/gravity/SouthEast/dx/20/dy/20)

右键获取以上图片获得链接可以查看水印生成的具体规格参数。


文字水印样例

- 水印文本：`七牛云存储`
- 水印文本字体：`宋体`
- 水印文本字体大小：`1000`
- 水印文本颜色：`white`
- 水印文本透明度：15% (`dissolve=85`)
- 水印文本位置：右下脚 (`gravity=SouthEast`)

![文字水印](http://qiniuphotos.qiniudn.com/gogopher.jpg?watermark/2/text/5LiD54mb5LqR5a2Y5YKo/font/5a6L5L2T/fontsize/1000/fill/d2hpdGU=/dissolve/85/gravity/SouthEast/dx/20/dy/20)

右键获取以上图片获得链接可以查看水印生成的具体规格参数。

**优化建议**

- 1.图片上传完毕后，可异步进行水印预转，这样不必在初次访问时进行水印处理，访问速度更快。
    - 参考 [uploadToken 之 asyncOps](put.html#uploadToken-asyncOps) 。

- 2.给图片链接中的水印规格添加别名，使得URL更加友好。
    - 设置别名，可使用 [qboxrsctl style 命令](/tools/qboxrsctl.html#style)
    - 查看别名规格，可使用 [qboxrsctl bucketinfo 命令](/tools/qboxrsctl.html#bucketinfo)

  示例

    qboxrsctl login <email> <password>

    qboxrsctl style <bucket> watermarked.jpg watermark/2/text/<EncodedText>

    qboxrsctl separator <bucket> -

  此时，如下两个 URL 等价:

    http://<Domain>/<Key>?watermark/2/text/<EncodedText>

    http://<Domain>/<Key>-watermarked.jpg


- 3.设置原图保护，仅限使用缩略图样式别名的友好URL形式来访问目标图片。
    - 设置原图保护，可使用 [qboxrsctl protected 命令](/tools/qboxrsctl.html#protected)
    - 也可在 <http://www.qiniu.com/>  上进行可视觉化操作。

  设置原图保护后，原图不能访问：

    http://<Domain>/<Key>

  同时也禁止根据图像处理API对原图进行参数枚举：

    http://<Domain>/<Key>?watermark/2/text/<EncodedText>

  此时只能访问指定规格的图片资源：

    http://<Domain>/<Key>-watermarked.jpg


##### 高级处理

除了对图像进行缩略有单独的处理接口，七牛云存储还提供了比较高级的图像处理接口，包含缩略、裁剪、旋转等一系列的功能，该接口规格如下：

**请求**

    [GET] <ImageDownloadURL>?imageMogr/auto-orient
          /thumbnail/<ImageSizeGeometry>
          /gravity/<GravityType>
          /crop/<ImageSizeAndOffsetGeometry>
          /quality/<ImageQuality>
          /rotate/<RotateDegree>
          /format/<DestinationImageFormat> =jpg, gif, png, webp, etc.

注意：

- `/auto-orient` 参数是和图像处理顺序相关的，一般建议放在首位（根据原图EXIF信息自动旋正）。


**响应**

    200 OK
    <ImageBinaryData>

示例：

    [GET] http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/auto-orient
                /thumbnail/!256x256r
                /gravity/center
                /crop/!256x256
                /quality/80
                /rotate/45

![高级图像处理](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/auto-orient/thumbnail/!256x256r/gravity/center/crop/!256x256/quality/80/rotate/45)

您可能留意到部分参数以 ! 开头，这是参数被转义的标识。为了方便阅读，我们采用了特殊的转义方法。以下是转义符号列表：

    p => % (percent)
    r => ^ (reverse)
    a => + (add)

也就是 !50x50r 其实代表 50x50^ 这样一个字符串。!50x50 代表 50x50 这样一个字符串（实际上这个字符串不需要转义）。`<ImageSizeAndOffsetGeometry>` 中的 OffsetGeometry 部分可以省略，缺省为 +0+0。也就是 /crop/50x50 等价于 /crop/!50x50a0a0，执行 -crop 50x50+0+0 语义。

如下是 `/thumbnail/<ImageSizeGeometry>` 和 `/crop/<ImageSizeAndOffsetGeometry>` 参数规格详解。

指定图片缩略或裁剪后的尺寸：

  size | 规格说明 | 样例
-------| -------- | ------------------------------------------------------
  scale% | 基于原图大小，按照指定的百分比进行缩放。 | [50%](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/!50p)
  scale-x%xscale-y% | 以百分比的形式指定缩略图的宽或高，另一边自适应等比缩放，只能使用一个 % 限定。 | [50%x](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/!50p) [x50%](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/!x50p)
  width | 限定缩略图宽度，高度等比自适应。 | [200](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/200)
  xheight | 限定缩略图高度，宽度等比自适应。 | [x100](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/x100)
  widthxheight | 限定长边，短边自适应，将缩略图的大小限定在指定的宽高矩形内。若指定的宽度大于指定的高度，以指定的高度为基准，宽度自适应等比缩放；若指定的宽度小于指定的高度，以指定的宽度为基准，高度自适应等比缩放。 | [100x200](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/100x200) [200x100](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/200x100)
  widthxheight^ | 限定短边，长边自适应，目标缩略图大小会超出指定的宽高矩形。若指定的宽度大于指定的高度，以指定的宽度为基准，高度自适应等比缩放；若指定的宽度小于指定的高度，以指定的高度为基准，宽度自适应等比缩放。 | [100x200^](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/100x200^) [200x100^](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/200x100^)
  widthxheight! | 限定缩略图宽和高。缩略图按照指定的宽和高强行缩略，忽略原图宽和高的比例，可能会变形。 | [100x200!](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/100x200!) [200x100!](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/200x100!)
  widthxheight> | 当原图尺寸超出给定的宽度或高度时，按照给定的 widthxheight 规格进行缩略。 | [100x200>](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/100x200%3E) [1000x2000>](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/1000x2000%3E)
  widthxheight< | 当原图尺寸低于给定的宽度和高度时，按照给定的 widthxheight 规格进行拉伸。 | [100x200<](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/200x100<) [1000x2000<](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/2000x1000<)
  area@ | 缩略图按原始图片高宽比例等比缩放，但缩放后的宽乘高的总分辨率不超过给定的总像素。 | [20000@](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageMogr/thumbnail/20000@)

指定图片缩略或裁剪前相对于原图的起始坐标：

{size}{offset}   | 指定偏移量 (缺省是 +0+0)，{size} 代表上述表格中的任意规格
-----------------|-------------------------------------------------------------
{size}{+-}x{+-}y | 指定子图片相对于源图片的坐标，x代表横轴，y代表纵轴，单位像素。偏移量会受 gravity 参数的影响，但不受 {size} 操作符比如 % 的影响。

x 为正数时为从源图区域左上角的横坐标，为负数时，左上角坐标为0，然后从截出的子图片右边减去x象素宽度。
y 为正数时为从源图区域左上角的纵坐标，为负数时，左上角坐标为0，然后从截出的子图片上边减去y象素高度。

例如，

`/crop/!300×400a10a10` 表示从源图坐标为 x:10 y:10 截取 300×400 的子图片。
`/crop/!300×400-10a10` 表示从源图坐标为 x:0  y:10 截取 290×400 的子图片。

这个源图可以是 `/thumbnail/<ImageSizeGeometry>` 参数处理过后的图片，意味着 `thumbnail` 和 `crop` 之间的操作可以链式处理。


##### 持久化

也可以将高级图像处理接口处理过的图片进行云端持久化，即将一个存储在七牛云存储的图片进行缩略、裁剪、旋转和格式转化处理后的缩略图作为一个新文件持久化存储到七牛云存储服务器上，这样就可以供后续直接使用而不用每次都传入参数进行图像处理。

图像处理持久化的接口规格如下：

**请求**

    [POST] <ImageDownloadURL>?imageMogr/auto-orient
           /thumbnail/<ImageSizeGeometry>
           /gravity/<GravityType>
           /crop/<ImageSizeAndOffsetGeometry>
           /quality/<ImageQuality>
           /rotate/<RotateDegree>
           /format/<DestinationImageFormat> =jpg, gif, png, webp, etc.
           /save-as/<EncodedEntryURI>

注意：

- `/auto-orient` 参数是和图像处理顺序相关的，一般建议放在首位（根据原图EXIF信息自动旋正）。


**参数**

和之前的流式图像处理接口 `imageMogr` 相比，持久化的图像处理接口多了一个 `save-as` 参数，指定了处理过后的缩略图存放的目标路径。

名称                | 说明
--------------------|--------------------------------------------------------------------------------
`<EncodedEntryURI>` | 指定目标缩略图存放的地址。`EncodedEntryURI = urlsafe_base64_encode(bucket:key)`

`urlsafe_base64_encode(string)` 函数的实现符合 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 标准，开发者可以参考 [github.com/qiniu](https://github.com/qiniu) 上各SDK的样例代码。

该 POST 请求需要进行签名认证才能调用，参考 [授权认证 - AccessToken](file-handle.html#digest-auth)。

**响应**

    200 OK
    {"hash" => "FrOXNat8VhBVmcMF3uGrILpTu8Cs"}


#### 3.4.2. 音视频处理

##### 音频转换

**请求**

    [GET] <AudioDownloadURL>?avthumb/<Format>
                             /ab/<BitRate>
                             /aq/<AudioQuality>
                             /ar/<SamplingRate>

参数释义参考: [音视频API参数详解](#args)

**响应**

    HTTP/1.1 200 OK
    Body: <AudioBinaryData>

**示例**

示例1：将 wav 音频格式转换为 mp3 格式：

    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3

示例2：将 wav 音频格式转换为 mp3 格式，并指定比特率为 192k：

    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3/ab/192k

示例3：将 wav 音频格式转换为 mp3 格式，并指定 VBR 参数为3，采样频率为44100：

    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3/ar/44100/aq/3

**支持的格式**

支持转换的音频格式详见：<http://ffmpeg.org/general.html#Audio-Codecs>

支持的音频 Codec 有：libmp3lame，libfaac，libvorbis 。

**优化建议**

为了保证良好的用户体验，请配合上传预转机制使用。参考: [上传预转](#upload-fop)


##### 视频转换

**请求**

    [GET] <VideoDownloadURL>?avthumb/<Format>
                             /r/<FrameRate>
                             /vb/<VideoBitRate>
                             /vcodec/<VideoCodec>
                             /acodec/<AudioCodec>
                             /ab/<BitRate>
                             /aq/<AudioQuality>
                             /ar/<SamplingRate>

参数释义参考: [音视频API参数详解](#args)

**响应**

    HTTP/1.1 200 OK
    Body: <VideoBinaryData>

**示例**

示例1：将 mp4 视频格式转换为 flv 格式，帧率为 24，使用 x264 进行视频编码：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/flv/r/24/vcodec/libx264

示例2：将 mp4 视频格式转换为 avi 格式，使用 mp3 进行音频编码，且音频比特率为64k：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/avi/ab/64k/acodec/libmp3lame

示例3：将 mp4 视频格式转换为 flv 格式，帧率 30，视频比特率 256k，使用 x264 进行视频编码，音频采样频率 22050，音频比特率 64k，使用 mp3 进行音频编码：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/flv
                                                     /r/30
                                                     /vb/256k
                                                     /vcodec/libx264
                                                     /ar/22050
                                                     /ab/64k
                                                     /acodec/libmp3lame

示例4：将 mp4 视频格式转换为 ogv 格式，帧率 30，视频比特率 1800k，使用 libtheora 进行视频编码，音频采样频率 44100，音频比特率 128k，使用 libvorbis 进行音频编码：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/ogv
                                                     /r/30
                                                     /vb/1800k
                                                     /vcodec/libtheora
                                                     /ar/44100
                                                     /ab/128k
                                                     /acodec/libvorbis

**支持的格式**

支持转换的视频格式详见：<http://ffmpeg.org/general.html#File-Formats>

支持的视频 Codec 有：libx264，libvpx，libtheora，libxvid 。

支持的音频 Codec 有：libmp3lame，libfaac，libvorbis 。

**优化建议**

为了保证良好的用户体验，请配合上传预转机制使用。参考: [上传预转](#upload-fop)


##### 视频缩略图

**请求**

    GET <VideoDownloadURL>?vframe/<Format>
                           /offset/<Second>
                           /w/<Width>
                           /h/<Height>

**响应**

    HTTP/1.1 200 OK
    Body: <ImageBinaryData>

**请求参数详解**

参数   | 说明
-------|--------------------------------------
Format | 要输出的目标缩略图格式，支持 jpg，png
Second | 取视频的第几秒
Width  | 缩略图宽度，范围为 1 ~ 1920
Height | 缩略图高度，范围为 1 ~ 1080

**示例**

示例：取视频第 7 秒的截图，图片格式为 jpg，宽度为 480px，高度为 360px：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg
                                                     /offset/7
                                                     /w/480
                                                     /h/360

上述示例效果如下：

![Go——基于连接与组合的语言](http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360)


##### HLS

HTTP Live Streaming 是由 Apple 提出的基于 HTTP 的流媒体传输协议。
它将一整个音视频流切割成可由 HTTP 下载的一个个小的音视频流，并生成一个播放列表（M3U8），客户端只需要获取资源的 M3U8 播放列表即可播放音视频。

以下用 HLS 代指 HTTP Live Streaming 。

###### 使用七牛提供的 HLS 服务

HLS 必须使用友好风格的 URL，可以使用命令行工具 `qboxrsctl` 配置 HLS 友好风格。

下载 [qboxrsctl](/tools/qboxrsctl.html)

下载 qboxrsctl 之后，我们需要先了解该命令行工具的3个指令。

注意:

 - qboxrsctl 工具需在命令行模式下使用
 - 尖括号注明的参数是需要自行替换的内容

指令1，登录授权:

    qboxrsctl login <注册邮箱> <登录密码>

指令2，设置友好风格的 URL 分隔符:

    qboxrsctl separator <空间名称> <分隔符字符>

指令3，设置 API 规格别名:

    qboxrsctl style <空间名称> <API规格别名> <API规格定义字符串>

示例

    // 设置分隔符为点号（“.”) 
    qboxrsctl separator <空间名称> .

    // 设置风格名为 m3u8_audio，代表音频的 HLS, 码率为32k
    qboxrsctl style <空间名称> m3u8_audio avthumb/m3u8/preset/audio_32k

    // 设置风格名为 m3u8_video，代表视频的 HLS, 长宽比为16x9，码率为150k
    qboxrsctl style <空间名称> m3u8_video avthumb/m3u8/preset/video_16x9_150k


已知文件上传到七牛后，下载方式如下:

公有资源

    [GET] http://<Domain>/<Key>

私有资源

    [GET] http://<Domain>/<Key>?token=<DownloadToken>


上述示例设置完之后就可以用以下 URL 来访问 HLS 资源：

    // 公有资源
    [GET] http://<Domain>/<Key>.m3u8_audio
    [GET] http://<Domain>/<Key>.m3u8_video

    // 私有资源（m3u8私有资源访问暂不支持）
    [GET] http://<Domain>/<Key>.m3u8_audio?token=<DownloadToken>
    [GET] http://<Domain>/<Key>.m3u8_video?token=<DownloadToken>

    HTTP/1.1 200 OK
    Content-Type: application/x-mpegurl
    Body: <M3U8File>

###### 上传预转

由于在线音视频频转换或将音视频切割成多个小文件并生成 M3U8 播放列表是一个相对耗时的操作，为了保证良好的用户体验，需要配合上传预转机制使用。实际上，七牛官方推荐音视频在线编解码都通过上传预转的方式进行。

上传预转参考文档：[音视频上传预转 - asyncOps](put.html#uploadToken-asyncOps)

接上述示例，已知 `m3u8_audio` 的 API 规格定义，将其作为上传授权凭证（`uploadToken`）预转参数（`asyncOps`）的值即可。

    asyncOps = "http://example.qiniudn.com/$(key)?avthumb/m3u8/preset/audio_32k"

可以设置多个预转指令，用分号“;”隔开即可:

    asyncOps = "http://example.qiniudn.com/$(key)?avthumb/m3u8/preset/audio_32k;
                http://example.qiniudn.com/$(key)?avthumb/m3u8/preset/audio_48k"

实际情况下，`example.qiniudn.com` 换成存储空间（bucket）绑定的域名即可。

同样，视频预转的操作方式也一样。

设置预转后，当文件上传完毕即可异步执行预转指令操作。第一次访问该资源时，就无需再转换了，访问到的即已经转换好的资源。


#### 3.4.3. 文档转换

##### Markdown转HTML

Markdown 转 HTML 接口的规格如下

    [GET] url?md2html/<mode>/css/<EncodedCSSURL>

`url` 获取可以参考 [下载接口](get.html)

**参数**

名称          | 类型   | 必须 | 说明
--------------|--------|------|------------------------------------------------------------------------------
mode          | int    | 否   | `0` 表示转为完整的 HTML(head+body) 输出; `1` 表示只转为HTML Body，缺省值：`0`
EncodedCSSURL | string | 否   | CSS 样式的URL，`EncodedCSSURL = urlsafe_base64_encode(CSSURL)`

`urlsafe_base64_encode(string)` 函数的实现符合 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 标准，开发者可以参考 <https://github.com/qiniu> 上各SDK的样例代码。


#### 3.4.4. 其他操作

##### 生成二维码

**格式**

    url?qrcode

`url` 代表存储在七牛云存储上的资源，获取url可以参考 [下载接口](get.html) 。

例如，通向本篇文档的二维码图片地址是：

    http://docs.qiniudn.com/api/qrcode.html?qrcode

![通向本篇文档的二维码图片地址](http://docs.qiniudn.com/api/qrcode.html?qrcode)

###### API 规格

**请求**

    [GET] url?qrcode/<Mode>/level/<Level>

**响应**

    HTTP/1.1 200 OK
    Body: <QRcodeImageBinary>

**请求参数详解**

参数  | 必须 | 说明
------|------|------
Mode  | 否   | 可选值`0`或`1`，缺省为`0`。`0`表示以当前url生成二维码，`1`表示以当前URL中的数据生成二维码。
Level | 否   | 冗余度，可选值 `L`、`M`、`Q`，或 `H`，缺省为 `L`

**Level**

值 | 冗余度
---|-------
L  | 7%
M  | 15%
Q  | 25%
H  | 30%

L 是最低级别的冗余度，H 最高，冗余度越高，生成的图片体积越大。具体参见 [维基百科](http://en.wikipedia.org/wiki/QR_code#Error_correction)


###### 样例

示例1: Mode=0 时，基于 URL 生成二维码

- <http://docs.qiniudn.com/api/qrcode.html?qrcode>

示例2: Mode=1 时，基于 URL 的内容生成二维码

- <http://qrcode.qiniudn.com/qiniu.vcard?qrcode/1>

示例3: 分别用不同的冗余度生成不同尺寸的二维码

- <http://docs.qiniudn.com/api/qrcode.html?qrcode/0/level/L>
- <http://docs.qiniudn.com/api/qrcode.html?qrcode/0/level/H>

以上两个二维码图片尺寸不同，但表示的内容相同。


###### 高级用法

二维码+Logo，可以使用七牛云存储的 [Pipeline API](pipeline.html) 和 [图像水印接口](image-process.html#watermark) 操作实现。例如，

![QRCode+Logo](http://qrcode.qiniudn.com/qiniu.vcard?qrcode/1/level/M|watermark/1/image/aHR0cDovL3FyY29kZS5xaW5pdWRuLmNvbS93ZWlib2xvZ282LnBuZz9pbWFnZU1vZ3IvdGh1bWJuYWlsLzMyeDMy/gravity/center/dx/0/dy/0)

可以右键查看该二维码图片的URL


### 二维码中的内容

二维码中的内容实际上是文本，却可存储多种类型的内容，具体用例可见:

- <https://code.google.com/p/zxing/wiki/BarcodeContents>


#### 3.4.5. 云处理的Pipeline

已知，七牛云存储的云处理API满足如下规格:

    [GET] url?<fop>

即基于文件的 URL 通过问号传参来实现即时云处理，`<fop>` 表示 API 指令及其所需要的参数，是 File Operation 的缩写，表示文件处理。

那么，将一个资源经由多个 `<fop>` 链式处理，各 `<fop>` 之间用竖线（`|`）分割，我们称之为 Pipeline API。也称之为管道操作，熟悉 Linux 命令行的开发者可能会有更透彻的理解。

Pipeline API 规格如下

	[GET] url?<fop1>|<fop2>|<fop3>|<fopN>

`url` 获取可以参考 [下载接口](get.html)


##### 样例

###### 例1: 将一个原图缩略，然后在缩略图上打上另外一个图片作为水印

- 原图
	- <http://qiniuphotos.qiniudn.com/gogopher.jpg>
- 基于原图生成缩略图
	- <http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200>
- 在生成的缩略图之上打水印
	- <http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200|watermark/1/image/aHR0cDovL3d3dy5iMS5xaW5pdWRuLmNvbS9pbWFnZXMvbG9nby0yLnBuZw==>

###### 例2: 从视频中提取某一帧生成缩略图，然后基于该缩略图打水印

- 视频
	- <http://open.qiniudn.com/thinking-in-go.mp4>
- 提取视频帧并生成缩略图
	- <http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360>
- 在生成的缩略图之上打水印
	- <http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360|watermark/1/image/aHR0cDovL3d3dy5iMS5xaW5pdWRuLmNvbS9pbWFnZXMvbG9nby0yLnBuZw==>


##### 别名

如果觉得 `url?<fop1>|<fop2>|<fop3>|<fopN>` 这样的形式够冗长，还可以为这些串行的 `<fop>` 集合定义一个友好别名。如此一来，就可以用友好URL风格进行访问。

我们先来熟悉 [qboxrsctl](/tools/qboxrsctl.html) 的两个命令行，

    // 定义 url 和 fop 之间的分隔符为 separator 
    qboxrsctl separator <bucket> <separator>

    // 定义 fop 的别名为 aliasName
    qboxrsctl style <bucket> <aliasName> <fop>

例如:

    qboxrsctl separator <bucket> "."
    qboxrsctl style <bucket> "jpg" "vframe/jpg/offset/7/w/480/h/360|watermark/1/image/aHR0cDovL3d3dy5iMS5xaW5pdWRuLmNvbS9pbWFnZXMvbG9nby0yLnBuZw=="

那么，以下两个 URL 则等价:

原始URL:

- <http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360|watermark/1/image/aHR0cDovL3d3dy5iMS5xaW5pdWRuLmNvbS9pbWFnZXMvbG9nby0yLnBuZw==>

友好风格URL:

- <http://open.qiniudn.com/thinking-in-go.mp4.jpg>


### 3.5. 工具

#### 3.5.1. qrsync

#### 3.5.2. qboxrsctl

#### 3.5.3. qiniu-qutosync