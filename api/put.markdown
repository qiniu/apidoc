---
layout: default
title: "上传接口"
---

- [上传流程](#workflow)
    - [本地上传](#local-upload) 
    - [客户端加速直传](#ugc-upload) 
        - [普通上传](#normal-upload)
        - [高级上传 - 重定向功能](#upload-with-redirect)
        - [高级上传 - 回调功能](#upload-with-callback)
- [上传文件](#upload)
    - [上传接口 - API](#upload-api)
    - [上传凭证 - uploadToken](#uploadToken)
        - [生成算法](#uploadToken-algorithm)
        - [参数说明](#uploadToken-args)
        - [使用普通上传，App-Client 接收来自 Qiniu-Cloud-Storage 的 Response Body](#uploadToken-returnBody)
        - [使用高级上传的重定向功能，实现 HTML Form 上传后跳转](#upload-with-redirect-appclient)
        - [使用高级上传的回调功能，App-Client 接收来自 App-Server 的 Response Body](#upload-with-callback-appserver)
        - [音视频上传预转 - asyncOps](#uploadToken-asyncOps)
        - [样例代码](#uploadToken-examples)
- [附录](#dictionary)
    - [魔法变量 - MagicVariables](#MagicVariables)
    - [自定义变量 - xVariables](#xVariables)
    - [错误码](#error-code)

<a name="workflow"></a>

## 上传流程

<a name="local-upload"></a>

### 本地上传

若您需要上传已存在您电脑或者是服务器上的文件到七牛云存储，可以直接使用七牛提供的上传工具：

| 名称                                         | 使用   | 适用平台                     | 说明                                 |
|----------------------------------------------|--------|------------------------------|--------------------------------------|
| [qrsync](/tools/qrsync.html)                 | 命令行 | Linux,Windows,MacOSX,FreeBSD | 手动同步文件/文件夹到七牛云存储      |
| [qiniu-autosync](/tools/qiniu-autosync.html) | 命令行 | Linux                        | 自动同步指定文件夹内的新增或改动文件 |


如果是需要通过网站(Web)或是移动应用(App)等客户端上传文件，则可以参考[客户端加速直传](#ugc-upload)部分的流程。

<a name="ugc-upload"></a>

### 客户端加速直传

<a name="upload-without-callback"></a>

#### 普通上传

<div class="imgwrap"><img src="img/normal_upload.png" alt="普通上传"/></div><br/>

流程简述：

1. App-Client 向 App-Server 请求上传文件
2. App-Server 根据[算法](#uploadToken-algorithm)（也可使用我们提供的 [SDK](/sdk/index.html)）生成上传凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传凭证（UploadToken）后，将文件直传到最近的存储节点
4. 文件上传成功后，Qiniu-Cloud-Storage 返回给 App-Client 上传结果（可包含相应的文件信息）
5. App-Client 将文件上传结果及相关信息汇报给 App-Server，App-Server 可执行后续的业务逻辑

<a name="upload-with-redirect"></a>

#### 高级上传 - 重定向功能

重定向功能可以让我们的服务器在文件上传结束后通知业务客户端（App-Client）进行301跳转操作。特别适用于浏览器端的文件上传操作。

<div class="imgwrap"><img src="img/redirect_upload.png" alt="重定向上传"/></div><br/>

流程简述：

1. App-Client 向 App-Server 请求上传文件
2. App-Server 根据[算法](#uploadToken-algorithm)（也可使用我们提供的 [SDK](/sdk/index.html)）生成上传凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传凭证（UploadToken）后，将文件直传到最近的存储节点
4. 文件上传成功后，Qiniu-Cloud-Storage 将返回状态码为 301 的重定向 HTTP Response （可包含相应的文件信息）给上传者 App-Client
5. App-Client 访问并跳转到重定向的页面。

<a name="upload-with-callback"></a>

#### 高级上传 - 回调功能

回调功能可以让我们的服务器在文件上传结束后回调一个指定的URL以通知业务服务器（App-Server）。

<div class="imgwrap"><img src="img/callback_upload.png" alt="回调上传"/></div><br/>

流程简述：

1. App-Client 向 App-Server 请求上传文件
2. App-Server 根据[算法](#uploadToken-algorithm)（也可使用我们提供的 [SDK](/sdk/index.html)）生成上传凭证（UploadToken），并颁发给 App-Client
3. App-Client 取得上传凭证（UploadToken）后，将文件直传到最近的存储节点
4. 文件上传成功后，Qiniu-Cloud-Storage 以 HTTP POST 方式告知 App-Server 上传结果（可包含相应的文件信息）
5. App-Server 可执行相应的业务逻辑，然后经 Qiniu-Cloud-Storage 中转返回给 App-Client 它想要的信息
6. Qiniu-Cloud-Storage 作为代理，原封不动地将回调 App-Server 的返回结果回传给 App-Client

**带回调功能的高级上传相对于普通上传的优势，体现在以下几方面**:

1. App Client 无需向 App-Server 发送通知，全部统一由 Qiniu-Cloud-Storage 发送 Callback，当存在多种终端（比如Web/iOS/Android）都需要上传文件时，每个终端不需要各自处理 Callback 业务逻辑。
2. Callback 环节加速，七牛云存储的就近节点能比 App-Client 以更优异的网络回调 App-Server 。
3. 只要文件上传成功，App-Server 必然知情。即使 App-Server 回调失败，App-Client 还是会得到完整的回调数据，可自定义策略进行异步处理。

**注意**

- 以上所有的上传流程中，步骤(1)和步骤(2)中 App-Client 获取上传凭证（UploadToken）不用重复频繁获取，UploadToken 可通过 `deadline` 选项设置有效期，在设定的有效期内可多次复用。后续 [上传凭证 - uploadToken 算法说明](#uploadToken-algorithm) 会解释各选项的具体作用。


<a name="upload-api"></a>

## 上传API

HTML Form API

    <form method="post" action="http://up.qiniu.com/" enctype="multipart/form-data">
      <input name="key" type="hidden" value="{FileID}">
      <input name="x:custom_field_name" type="hidden" value="{SomeVal}">
      <input name="token" type="hidden" value="{UploadToken}">
      <input name="file" type="file" />
    </form>

参数

名称        | 类型   | 必须 | 说明
------------|--------|------|-------------------------------------
key         | string | 否   | 标识文件的索引，所在的存储空间内唯一。key可包含斜杠， **但不能以斜杠开头** ，比如 `a/b/c.jpg` 是一个合法的key。若不指定 key，缺省使用文件的 etag（即上传成功后返回的hash值）作为key；此时若 UploadToken 有指定 returnUrl 选项，则文件上传成功后跳转到 `returnUrl?query_string`, query_string 包含`key={FileID}`
x:custom_field_name | string | 否 | [自定义变量](#xVariables)，必须以 `x:` 开头命名，不限个数。可以在 uploadToken 的 `callbackBody` 选项中使用 `$(x:custom_field_name)` 求值。
token       | string | 是   | 上传凭证 - UploadToken
file        | file   | 是   | 文件内容本身

该 HTML Form API 还可以用如下 `multipart/form-data` 形式表达。

    POST http://up.qiniu.com/
    Content-Type: multipart/form-data; boundary=<Boundary>

    <Boundary>
    Content-Disposition: form-data; name="key"

    <FileID>

    <Boundary>
    Content-Disposition: form-data; name="x:custom_field_name"

    <SomeVal>

    <Boundary>
    Content-Disposition: form-data; name="token"

    <UploadToken>

    <Boundary>
    Content-Disposition: form-data; name="file"; filename="<FileName>"
    Content-Type: <MimeType>

    <FileContent>

上传完毕后，Qiniu-Cloud-Storage 会向 App-Client 返回如下信息：

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Response Body: {
        hash: <FileETag string>,
        ...
    }


<a name="uploadToken"></a>

## 上传凭证 - uploadToken

相关术语说明：

术语        | 说明
------------|-------------------------------
AccessKey   | 公钥，可用于识别七牛云存储帐号
SecretKey   | 密钥，可用于签名过程中进行加密
uploadToken | 令牌，也称上传凭证

uploadToken 有 3 个作用:

1. 识别 App-Client 的身份是否合法
2. 识别 App-Client 的请求是否合法
3. 双重识别合法的情况下，可以根据 uploadToken 元数据针对上传行为个性化处理


<a name="uploadToken-algorithm"></a>

### 生成算法

uploadToken 算法如下：

    // 步骤1：组织元数据（JSONString）
    Flags = {
        scope: <Bucket string>,
        deadline: <UnixTimestamp int64>,
        endUser: <EndUserId string>,
        returnUrl: <RedirectURL string>,
        returnBody: <ResponseBodyForAppClient string>,
        callbackBody: <RequestBodyForAppServer string>
        callbackUrl: <RequestUrlForAppServer string>,
        asyncOps: <asyncProcessCmds string>
    }

    // 步骤2：将 Flags 进行安全编码
    EncodedFlags = urlsafe_base64_encode(JSONString(Flags))

    // 步骤3：将编码后的元数据混入私钥进行签名
    Signature = hmac_sha1(EncodedFlags, SecretKey)

    // 步骤4：将签名摘要值进行安全编码
    EncodedSign = urlsafe_base64_encode(Signature)

    // 步骤5：连接各字符串，生成上传凭证
    uploadToken = AccessKey:EncodedSign:EncodedFlags

**注意**

- `Flags` 数据必须为标准的 [JSON](http://json.org/) 字符串
- `Flags` 各字段里边的尖括号“`<Variable Type>`”内容表示要被替换掉的“变量”，“变量”的数据类型已在括号内指定
- `urlsafe_base64_encode(string)` 函数按照标准的 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 实现，开发者可以参考 <https://github.com/qiniu> 上各SDK的样例代码。
- `AccessKey:EncodedSign:EncodedFlags` 这里的冒号是字符串，仅作为连接分隔符使用，最终连接组合的 UploadToken 也是一个字符串（String）。
- `callbackUrl` 与 `returnUrl` 不可同时指定，两者只可选其一。
- `callbackBody` 与 `returnBody` 不可同时指定，两者只可选其一。


<a name="uploadToken-args"></a>

### 参数说明

uploadToken 参数详解：

 字段名       | 必须 | 说明
--------------|------|-----------------------------------------------------------------------
 scope        | 是   | 用于指定文件要上传到的目标Bucket和Key。格式为：\<bucket name\>\[:\<key\>\]。若只指定Bucket名，表示文件上传至该Bucket。若同时指定了Bucket和Key（\<bucket name\>:\<key\>），表示上传文件限制在指定的Key上。两种形式的差别在于，前者是 **“新增”** 操作：如果所上传文件的Key在Bucket中已存在，上传操作将失败。而后者则是 **“新增或覆盖”** 操作：如果Key在Bucket中已经存在，将会被覆盖；如不存在，则将文件新增至Bucket中。
 deadline     | 否   | 定义 uploadToken 的失效时间，Unix时间戳，精确到秒，缺省为当前时间 3600 秒之后的Unix时间戳
 endUser      | 否   | 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据终端用户标识给图片或视频打水印
 returnUrl    | 否   | 设置用于浏览器端文件上传成功后，浏览器执行301跳转的URL，一般为 HTML Form 上传时使用。文件上传成功后会跳转到 returnUrl?query_string, query_string 会包含 returnBody 内容。注意：returnUrl 不可与 callbackUrl 同时使用。
 returnBody   | 否   | 文件上传成功后，自定义从 Qiniu-Cloud-Storage 最终返回給终端 App-Client 的数据。支持 [魔法变量](#MagicVariables)，不可与 callbackBody 同时使用。
 callbackBody | 否   | 文件上传成功后，Qiniu-Cloud-Storage 向 App-Server 发送POST请求的数据。支持 [魔法变量](#MagicVariables) 和 [自定义变量](#xVariables)，不可与 returnBody 同时使用。
 callbackUrl  | 否   | 文件上传成功后，Qiniu-Cloud-Storage 向 App-Server 发送POST请求的URL，必须是公网上可以正常进行 POST 请求并能响应 HTTP Status 200 OK 的有效 URL 
 asyncOps     | 否   | 指定文件（图片/音频/视频）上传成功后异步地执行指定的预转操作。每个预转指令是一个API规格字符串，多个预转指令可以使用分号“;”隔开


<a name="uploadToken-returnBody"></a>

### 使用普通上传，App-Client 接收来自 Qiniu-Cloud-Storage 的 Response Body

如果开发者使用普通上传模型，App-Client 上传一张图片到 Qiniu-Cloud-Storage 后，App-Client 想知道该图片的一些信息比如 Etag, EXIF 等信息，那么此时即可在 uploadToken 中使用 **`returnBody`** 参数。 

App-Client 想求值得到的这些 Etag, EXIF 等信息我们称之为魔法变量（[MagicVariables](#MagicVariables)）。

returnBody 赋值可以把 魔法变量（[MagicVariables](#MagicVariables)）的求值结果以 `Content-Type: application/json` 形式返回給 App-Client。

<a name="use-returnBody"></a>

一个典型的包含 MagicVariables 的 returnBody 字段声明如下（returnBody 必须是一个JSON字符串）：

    Flags["returnBody"] = `{
        "foo": "bar",
        "name": $(fname),
        "size": $(fsize),
        "type": $(mimeType),
        "hash": $(etag),
        "w": $(imageInfo.width),
        "h": $(imageInfo.height),
        "color": $(exif.ColorSpace.val)
    }`

假使如上，当一个用户在 iOS 端用包含该 returnBody 字段的 uploadToken 成功上传一张图片，那么该 iOS 端程序将收到如下一段 HTTP Response 应答：

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Response Body: {
        "foo": "bar",
        "name": "gogopher.jpg",
        "size": 214513,
        "type": "image/jpg",
        "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
        "w": 640,
        "h": 480,
        "color": "sRGB"
    }

可用的魔法变量列表参考：[MagicVariables](#MagicVariables)

<a name="upload-with-redirect-appclient"></a>

### 使用高级上传的重定向功能实现 HTML Form 上传后跳转

如果在 uploadToken 中指定了 `returnUrl` 和 `returnBody` 选项，那么就可以使用带重定向功能的上传模型，在文件上传成功后，Qiniu-Cloud-Storage 会返回如下应答：

    HTTP/1.1 301 Moved Permanently
    Location: returnUrl?upload_ret={returnBodyEncoded}

即跳转到指定的 `returnUrl` 并附带上经过 `urlsafe_base64_encode()` 编码过的 `returnBody` 。 `urlsafe_base64_encode(string)` 函数按照标准的 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 实现，开发者可以参考 <https://github.com/qiniu> 上各SDK的样例代码。可以通过逆向还原 `returnBody` 。

若上传失败，且上传失败的原因不是由于 uploadToken 无效造成的，Qiniu-Cloud-Storage 会返回如下应答：

    HTTP/1.1 301 Moved Permanently
    Location: returnUrl?code={error_code}&error={error_message}


<a name="upload-with-callback-appserver"></a>

### 使用高级上传的回调功能，App-Client 接收来自 App-Server 的 Response Body

如果在 uploadToken 中指定了 `callbackUrl` 和 `callbackBody` 选项，那么就可以使用带回调功能的高级上传模型，App-Client 使用该 uploadToken 将文件上传成功后，Qiniu-Cloud-Storage 会向指定的 `callbackUrl` 以 HTTP POST 方式将 `callbackBody` 的值以 `application/x-www-form-urlencoded` 的形式发送给 App-Server。App-Server 接收请求后，返回 `Content-Type: "application/json"` 形式的 Response Body, 该段 JSON 数据会原封不动地经由 Qiniu-Cloud-Storage 返回给 App-Client 。

**callbackUrl** 必须是公网上可以公开访问的 URL  

**callbackUrl** 若指定， **callbackBody** 也必须指定，且两者的值都不能为空  

**callbackBody** 必须是 `a=1&b=2&c=3` 这种形式的字符串。当包含 [魔法变量](#MagicVariables) 时，可以是这样一种形式：`a=1&key=$(etag)&size=$(fsize)&uid=$(endUser)` 。当包含 [自定义变量](#xVariables) 时，可以是这样一种形式：`test=$(x:test)&key=$(etag)&size=$(fsize)&uid=$(endUser)`，其中 `x:test` 是一个自定义变量。自定义变量命名必须以 `x:` 开头，且在 `multipart/form-data` 上传流中存在。

Qiniu-Cloud-Storage 在回调 App-Server 成功后，App-Server 必须返回如下格式的 Response:

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Response Body: {
        "foo": "bar",
        "name": "gogopher.jpg",
        "size": 214513,
        "type": "image/jpg",
        "w": 640,
        "h": 480
    }

其中，Response Body 部分 App-Server 随意，只需为标准的 [JSON](http://json.org) 格式即可。

参考：

- [魔法变量 - MagicVariables](#MagicVariables)
- [自定义变量 - xVariables](#xVariables)

### callback 失败处理

如果文件上传成功，但是 Qiniu-Cloud-Storage 回调 App-Server 失败，Qiniu-Cloud-Storage 会将回调失败的信息连同 `callbackBody` 数据本身返回给 App-Client, App-Client 可选自定义策略进行相应的处理。

<a name="uploadToken-asyncOps"></a>

### 音视频上传预转 - asyncOps

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
2. 以此生成带有预转功能的上传凭证（UploadToken）
3. 向七牛云存储上传一个 aac 格式的音频文件
4. 上传成功后，服务器会对这个 aac 音频文件异步地做如下两个预转操作
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

- [图像处理接口](/api/image-process.html)
- [音频/视频/流媒体处理](/api/audio-video-hls-process.html) 


<a name="uploadToken-examples"></a>

### 生成 uploadToken 的样例代码

生成 uploadToken 的样例代码可以参考: 

- C - <https://github.com/qiniu/c-sdk/blob/develop/qiniu/rs.c>
- Go - <https://github.com/qiniu/api/blob/develop/rs/token.go>


<a name="dictionary"></a>

## 附录

<a name="MagicVariables"></a>

### 魔法变量 - MagicVariables

文件上传成功后，Qiniu-Cloud-Storage 返回给 App-Client 的 Response Body 可以包含该文件的一些属性信息，这些属性信息通常需要上传成功后询问 Qiniu-Cloud-Storage 得知，然后作为 Response Body 的一部分返回给 App-Client。

例如 App-Client 成功上传一张图片到 Qiniu-Cloud-Storage，App-Client 想知道该图片的一些信息像是 Etag, EXIF 等信息，App-Client 想求值得到的这些 Etag, EXIF 等信息我们可以通过魔法变量（MagicVariables）的方式获取。 

MagicVariables 是一组规定的 API Call，可以使用 `$(APIName)` 或者是 `$(APIName.FieldName)` 的形式进行求值。主要用在 [uploadToken 的 returnBody 选项](#use-returnBody) 中。

可用 MagicVariables 列表:

API 名称  | 子项 | 说明
----------|------|-------------------------------------------
etag      | 无   | 文件上传成功后的 etag，上传前不指定 key 时，etag 等同于缺省的 key
fname     | 无   | 原始文件名
fsize     | 无   | 文件大小，单位: Byte
mimeType  | 无   | 文件的资源类型，比如 .jpg 图片的资源类型为 `image/jpg`
imageInfo | 有   | 获取所上传图片的基本信息，支持访问子字段
exif      | 有   | 获取所上传图片EXIF信息，支持访问子字段
endUser   | 无   | 获取 uploadToken 中指定的 endUser 选项的值，即终端用户ID

MagicVariables 支持同 [JSON](http://json.org/) 对象一样的 `{Object}.{Property}` 访问形式，比如：

- {API名称}
- {API名称}.{子项}
- {API名称}.{子项}.{下一级子项}

其中花括号部分（“`{…}`”）实际情况下需要用具体的 API 及其子项代替。

MagicVariables 求值示例：

- `$(etag)` - 获取当前上传文件的 etag
- `$(fname)` - 获取原始文件名
- `$(fsize)` - 获取当前上传文件的大小
- `$(mimeType)` - 获取当前上传文件的资源类型
- `$(imageInfo)` -  获取当前上传图片的基本属性信息
- `$(imageInfo.width)` - 获取当前上传图片的原始高度
- `$(imageInfo.height)` - 获取当前上传图片的原始高度
- `$(imageInfo.format)` -  获取当前上传图片的格式
- `$(endUser)` - 获取 uploadToken 中指定的 endUser 选项的值，即终端用户ID

imageInfo 接口返回的 JSON 数据可参考：<http://qiniuphotos.qiniudn.com/gogopher.jpg?imageInfo>

- `$(exif)` - 获取当前上传图片的 EXIF 信息
- `$(exif.ApertureValue)` - 获取当前上传图片所拍照时的光圈信息
- `$(exif.ApertureValue.val)` - 获取当前上传图片拍照时的具体光圈值

exif 接口返回的 JSON 数据可参考：<http://qiniuphotos.qiniudn.com/gogopher.jpg?exif>


<a name="xVariables"></a>

### 自定义变量 - xVariables

已知 [上传API](#upload-api) 结构如下

HTML Form API

    <form method="post" action="http://up.qiniu.com/" enctype="multipart/form-data">
      <input name="key" type="hidden" value="{FileID}">
      <input name="x:custom_field_name" type="hidden" value="{SomeVal}">
      <input name="token" type="hidden" value="{UploadToken}">
      <input name="file" type="file" />
    </form>

自定义变量即是其中的 `x:custom_field_name`，Qiniu-Cloud-Storage 允许在 form 或 `multipart/form-data` 流中添加任意以 `x:` 开头的自定义字段，不限个数，例如：

	<input name="x:uid" value="xxx">
	<input name="x:album_id" value="yyy">

这样，开发者在 uploadToken 的 `callbackBody` 选项里面就可以通过 `$(x:album_id)` 引用此自定义字段的值。例如此时 `callbackBody` 可以设置为 `key=$(etag)&album=$(x:album_id)&uid=$(x:uid)`，App-Server 通过此设置即可得到 App-Client 端文件上传成功后附带的自定义变量的值。


<a name="error-code"></a>

### 错误码

HTTP 状态码 | 错误说明
------------|-----------------------------------------------------------------
400         | 请求参数错误
401         | 认证授权失败，可能是 AccessKey/SecretKey 错误或 UploadToken 无效
405         | 请求方式错误，非预期的请求方式
579         | 文件上传成功，但是回调（callback app-server）失败
599         | 服务端操作失败
614         | 文件已存在
631         | 指定的存储空间（Bucket）不存在
701         | 上传数据块校验出错
