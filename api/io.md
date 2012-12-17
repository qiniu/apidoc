---
title: 云存储接口 | 七牛云存储
---

# 云存储接口

**注意**：初次阅读本篇文档前，您需要理解一些常用术语比如 Entry, EntryURI, EncodedEntryURI 等。关于七牛云存储API术语您可以查阅另一篇专门解释术语的文档：[**理解常用术语**](/v3/api/words/)

在请求七牛云存储各个接口之前，需要充分理解授权操作，可以参阅文档：[应用接入与认证授权](/v3/api/auth/) 。


## 目录

- [创建空间](#mkbucket)
- [上传文件](#upload)
    - [生成上传授权凭证](#upload-token)
        - [算法](#upload-token-algorithm)
        - [样例](#upload-token-examples)
    - [授权直传文件](#upload-file)
        - [API（multipart/form-data）](#upload-file-by-multipart)
        - [HTML表单形式直传](#upload-file-by-html-form)
    - [回调处理](#callback-after-uploaded)
        - [回调逻辑](#callback-logic)
        - [作为代理](#callback-as-proxy)
- [断点续上传](#resumable-upload)
    - [术语](#resumable-upload-keywords)
    - [工作模型](#resumable-upload-model)
    - [流程](#resumable-upload-workflow)
    - [API](#resumable-upload-api)
        - [授权](#resumable-upload-authorization)
        - [创建分割块（Block）并上传第一个数据块（Chunk）](#resumable-upload-mkblk)
        - [上传分割块（Block）中的数据块（Chunk）](#resumable-upload-bput)
        - [合并文件](#resumable-upload-mkfile)
    - [样例](#resumable-upload-examples)
- [下载文件](#download)
    - [动态获取文件授权后的临时下载链接](#get)
    - [直接绑定域名为文件创建公开外链](#publish)
    - [解除域名绑定取消文件的公开外链](#unpublish)
    - [断点续下载](#download-by-range-bytes)
    - [针对下载出现 404 NotFound 智能化处理](#download-if-notfound)
    - [设置源文件(比如原图)保护](#set-protected)
        - [为指定的存储空间设置保护模式](#pub-access-mode)
        - [设置友好URL访问中的连接符](#pub-separator)
        - [设置URL友好的风格样式名](#pub-style)
        - [取消URL友好风格的样式名访问](#pub-unstyle)
    - [防盗链设置](#anti-theft-chain)
        - [设置防盗链模式](#uc-antiLeechMode)
        - [更新防盗链记录值](#uc-referAntiLeech)
- [查看文件基本属性信息](#stat)
- [删除指定文件](#delete)
- [删除所有文件（整个空间/Bucket）](#drop)
- [批量操作](#batch)
    - [批量获取文件基本属性信息](#batch-stat)
    - [批量获取文件授权后的临时下载链接](#batch-get)
    - [批量删除文件](#batch-delete)
- [清除服务端缓存](#refresh-bucket)

<a name="mkbucket"></a>

### 1. 创建空间

      POST http://rs.qbox.me/mkbucket/<Bucket>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <ACCESS_TOKEN>
      }

      HTTP/1.1 200 OK

**参数**

`<Bucket>`
: 具体的空间名称，仅限[a-zA-Z0-9_]组合。

如果您不想通过 API 创建空间，也可以直接在七牛云存储开发者网站上直接 [新建空间](https://dev.qiniutek.com/buckets/new)。

<a name="upload"></a>

### 2. 上传文件

要上传一个文件，首先需要获得上传授权，七牛云存储通过 `uploadToken` 的方式实现上传授权操作。`uploadToken` 可以根据 `accessKey` 和 `secretKey` 对一组数据进行数字签名生成。在文件上传时，该 `uploadToken` 作为文件上传流中 multipart/form-data 的一部分进行传输，也可以附带在上传请求的 HTTP Headers 中传输。

<a name="upload-token"></a>

#### 2.1 生成上传授权凭证

<a name="upload-token-algorithm"></a>

##### 2.1.1 算法　

`uploadToken` 算法如下：

    // 步骤1：组织元数据
    authInfo = {
        scope: <targetBucket string>,
        deadline: <UnixTimestamp int64>,
        callbackUrl: <callbackUrl string>,
        callbackBodyType: <callbackBodyType string>,
        customer: <EndUserId string>,
        escape: <0|1>
    }

    // 步骤2：编码元数据
    authInfoEncoded = urlsafe_base64_encode(json_encode(authInfo))

    // 步骤3：将编码后的元数据经过私钥签名，提取摘要值
    authDigest = hmac_sha1(authInfoEncoded, secretKey)

    // 步骤4：将公钥、摘要值、已编码的元数据进行字符串拼接，生成上传授权凭证
    uploadToken = accessKey:authDigest:authInfoEncoded

**步骤1**

`authInfo` 各个字段详解：

字段名 | 类型 | 是否必须 | 说明
----- | --- | ------- | ----
scope | string | 可选 | 一般指定文件要上传到的目标存储空间（Bucket）
deadline | int64 | 必须 | 定义 uploadToken 的失效时间，Unix时间戳，精确到秒
callbackUrl | string | 可选 | 定义文件上传完毕后，云存储服务端执行回调的远程URL
callbackBodyType | string | 可选 | 为执行远程回调指定Content-Type，比如可以是：application/x-www-form-urlencoded
customer | string | 可选 | 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据终端用户标识给图片打水印
escape | int | 可选 | 可选值 0 或者 1，缺省为 0。值为 1 表示 callback 传递的自定义数据中允许存在转义符号 `$(VarExpression)`，参考 [VarExpression](/v3/api/words/#VarExpression)。

<a name="escape-expression"></a>

当 `escape` 的值为 `1` 时，常见的转义语法如下：

- 若 `callbackBodyType` 为 `application/json` 时，一个典型的自定义回调数据（[CallbackParams](#CallbackParams)）为：

    `{foo: "bar", size: $(fsize), etag: $(etag), w: $(imageInfo.width), h: $(imageInfo.height), exif: $(exif)}`

- 若 `callbackBodyType` 为 `application/x-www-form-urlencoded` 时，一个典型的自定义回调数据（[CallbackParams](#CallbackParams)）为：

    `foo=bar&size=$(fsize)&etag=$(etag)&w=$(imageInfo.width)&h=$(imageInfo.height)&exif=$(exif)`

authInfo 中的 `scope` 字段还可以有更灵活的定义：

- 若为空，表示可以上传到任意Bucket（仅限于新增文件）
- 若为"Bucket"，表示限定只能传到该Bucket（仅限于新增文件）
- 若为"Bucket:Key"，表示限定特定的文档，可新增或修改文件


**步骤2**

- `authInfo` 数据结构首先必须转成标准的 [JSON](http://json.org/) 格式。
- 将该JSON标准格式的元数据经过 [urlsafe_base64_encode](/v3/api/words/#URLSafeBase64Encode) 编码。

**步骤3**

- 通过 `hmac_sha1` 签名算法，将将编码后的元数据经过私钥（secretKey）进行计算签名

**步骤4**

- 将公钥（accessKey）、签名后的摘要值（authDigest）、已编码的元数据（authInfoEncoded）用冒号（:）进行字符串拼接，生成上传授权凭证 `uploadToken`。

<a name="upload-token-examples"></a>

##### 2.1.2 样例

生成 uploadToken 样例代码可参考：

- Python - <https://github.com/qiniu/python-sdk/blob/master/qbox/uptoken.py>

- Ruby - <https://github.com/qiniu/ruby-sdk/blob/master/lib/qiniu/tokens/upload_token.rb>

- PHP - <https://github.com/qiniu/php5-sdk/blob/master/qbox/authtoken.php>

<a name="upload-file"></a>

#### 2.2 直传文件

<a name="upload-file-by-multipart"></a>

##### 2.2.1 API（multipart/form-data）

请求包，`multipart/form-data` 格式：

    POST http://up.qbox.me/upload
    Content-Type: multipart/form-data; boundary=<Boundary>

    <Boundary>
    Content-Disposition: form-data; name="auth"

    <UploadToken>
    <Boundary>
    Content-Disposition: form-data; name="action"

    <PutAction>
    <Boundary>
    Content-Disposition: form-data; name="file"; filename="<LocalFileName>"
    Content-Type: <ContentType>

    <FileContent>
    <Boundary>
    Content-Disposition: form-data; name="params"

    <CustomData>

返回包(JSON)：

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Response Body: {
        hash: <FileETag string>
    }

**请求参数详解**

此 `multipart/form-data` 请求包需传递三个必要参数，他们分别是 `auth`、`action` 和 `file` 。

**auth**

`auth` 字段的值即 [上传授权凭证(uploadToken)](#upload-token) 的值：`accessKey:authDigest:authInfoEncoded`。

<a name="upload-action"></a>

**action**

`action=<PutAction>` 是要执行的上传行为，表示向具体的资源表里新建一个条目。具体规格如下：

    action="/rs-put/<EncodedEntryURI> \
            /mimeType/<EncodedMimeType> \
            /meta/<EncodedCustomMeta> \
            /crc32/<FileCRC32Checksum> \
            /rotate/<Rotate>"

反斜杠（\）因排版换行需要，实际情况下请忽略。

若尖括号包裹的字段（`<...>`）不传入，相应的前缀也不必传入。比如当不传入 `<EncodedCustomMeta>`, `<FileCRC32Checksum>` 和 `<Rotate>` 时：

    action="/rs-put/<EncodedEntryURI>/mimeType/<EncodedMimeType>"

`action` 字段的值至少是 `/rs-put/<EncodedEntryURI>` ，其他字段可选。以下为各个字段的详细解释。


- [EncodedEntryURI](/v3/api/words/#EncodedEntryURI) 定义了具体的 Bucket 和 Key，必须项。

注意：同一存储空间（Bucket）下，已存在与当前上传条目名称（[EncodedEntryURI](/v3/api/words/#EncodedEntryURI) 元素中的 [Key](/v3/api/words/#EntryURI)）相同的条目时，若当前上传文件内容与原有文件内容一致，则返回成功响应；若文件内容不一致，则上传失败，并返回失败响应。

- [EncodedMimeType](/v3/api/words/#EncodedMimeType) 表明文件的 MIME 类型，缺省情况下为 `application/octet-stream`，可选项。

- [EncodedCustomMeta](/v3/api/words/#EncodedCustomMeta) 文件备注信息，可选项，一般不传入。

- [FileCRC32Checksum](/v3/api/words/#FileCRC32Checksum) 文件的 crc32 校验值，十进制整数，可选项。若不传此参数则不执行数据校验。

- `Rotate` 上传图片时专用，特别适合移动设备拍照后直传。`<Rotate>` 值为 0 ：表示根据图像EXIF信息自动旋转；值为 1 : 右转90度；值为 2 :右转180度；值为 3 : 右转270度。可结合 [生成上传授权凭证 uploadToken 之 escape 参数详解](#escape-expression) 搭配使用。


**file**

`file=<FileContent>` 是要上传的具体文件内容，`<FileContent>` 则是要上传文件的二进制内容。

<a name="CallbackParams"></a>

**params**

`params` 用于文件上传成功后执行回调，七牛云存储服务器会向客户方的业务服务器 POST 这些指定的参数。一般可用于回传跟上传文件相关的具体信息，这样客户方的业务服务器会知道一个文件上传成功后以某条目名称记录到了七牛云存储的哪个空间（Bucket）里。

比如，若生成 [uploadToken](#upload-token) 中 `callbackBodyType` 为 `application/json` 时，`params` 可以是如下字符串的表达形式：

    {bucket: "<BucketName>", key: "<FileUniqKey>", uid: "<customer>"}

若生成 [uploadToken](#upload-token) 中 `callbackBodyType` 为 `application/x-www-form-urlencoded` 时，`params` 可以是如下字符串的表达形式：

    bucket=<BucketName>&key=<FileUniqKey>&uid=<customer>

以上尖括号包裹的字段（`<...>`）代表具体的变量名称。

七牛云存储允许文件上传成功后执行指定的 API 回调操作，参考 [生成上传授权凭证 uploadToken 之 escape 参数详解](#escape-expression) 。

<a name="upload-file-by-html-form"></a>

##### 2.2.2 HTML表单形式直传

如果您觉得这个接口理解起来有难度，不妨以一种更简单的HTML Form结构来理解，以上接口描述等价于如下 HTML Form:

    <form method="POST" enctype="multipart/form-data" action="http://up.qbox.me/upload">
        <input type="hidden" name="auth" value="{uploadToken}" />
        <input type="hidden" name="action" value="/rs-put/{URLSafeBase64Encode({bucket}:{key})}" />
        <input type="hidden" name="params" value="bucket={bucketName}&key={fileUniqKey}&k1=v1&k2=v2..." />
        <input type="hidden" name="return_url" value="http://DOMAIN/PATH?QUERY_STRING" />
        <input name="file" type="file" />
        <input type="submit" value="Upload File" />
    </form>

如果是采用如上 HTML 表单直传文件，倘若有入 `return_url` 字段，七牛云存储会在文件上传成功后执行301跳转，跳转的 URL 即 `return_url` 指定的值。


<a name="callback-after-uploaded"></a>

#### 2.3 回调处理

<a name="callback-logic"></a>

##### 2.3.1 回调逻辑

七牛云存储允许一个文件直传成功后向指定的远程服务器执行回调。在 [生成上传授权凭证(uploadToken)](#upload-token) 这一过程中，元数据（authInfo）包含的 `callbackUrl` 字段即用于指定远程回调地址。在 [直传文件](#upload-file) 这一过程中，`multipart/form-data` 请求包中的 `params` 字段提供了回调请求的具体数据。如果 `callbackUrl` 和 `params` 都有指定的情况下，那么文件上传成功后，七牛云存储服务端即会将 `params` 字段的值以 HTTP POST 的方式发送到指定的 `callbackUrl`。如果 [生成上传授权凭证(uploadToken)](#upload-token) 这一过程中有指定 `callbackBodyType`，七牛云存储服务端执行远程回调发送 HTTP 请求的 Content-Type 将会是 `callbackBodyType` 提供的值，一般会是 `Content-Type: application/x-www-form-urlencoded`。

<a name="callback-as-proxy"></a>

##### 2.3.2 作为代理

七牛云存储的回调处理还有代理作用。执行远程回调后，如果客户方的业务服务器返回的 HTTP Response Body 为标准的 [JSON](http://json.org/) 格式，并且 `HTTP Headers["Content-Type"] = 'application/json'`，那么七牛云存储服务端会将其 Response Body 按照 JSON 格式返回给客户方的客户端程序。

<a name="resumable-upload"></a>

### 3. 断点续上传

七牛云存储提供断点续上传功能。该功能将单个文件分割成数个固定大小的块并发上传，可以在实现断点续传的同时加快上传速度（并发上传）。

<a name="resumable-upload-keywords"></a>

#### 3.1 术语

1. 上传服务器（Up-Server）：提供断点续上传功能的服务器，负责启动新的上传过程、接受上传内容、合并生成最终上传文件。

2. 业务服务器（Biz-Server）：七牛云存储的客户的业务服务器，负责上传操作鉴权、分配操作策略、生成UpToken、驱动上传端启动上传。

3. 上传授权凭证（UpToken）：由业务服务器使用AccessKey和SecretKey，对操作策略进行数字签名防伪而生成的上传凭证。参考：[生成上传授权凭证](#upload-token)

4. 操作策略（Policy）：由业务服务器填写、由上传服务器执行的操作信息。参考：[生成上传授权凭证](#upload-token)
    4.1. 操作域（Scope）：1）空，表示可以上传到任意Bucket（仅限于新增文件）；2) "Bucket"，表示限定只能传到该Bucket（仅限于新增文件）；3) "Bucket:Key"，表示限定特定的文档，可新增或修改文件;
    4.2. 超时时限（DeadLine）：上传授权凭证的有效时间，单位是秒;
    4.3. 回调URL（CallbackURL）：如果指定，在合并文件后，由上传服务器调用此URL，以通知业务服务器做相应处理;
    4.4. 返回URL（ReturnURL）：如果指定，在合并文件后，由上传服务器重定向到此URL，以通知客户端继续表单处理流程。

5. 上传端（Up-Client）：七牛云存储的客户的业务终端，负责提出、实施上传。

6. 分割块（Block）：分割块是以指定大小（一般为4MB）为单位分割待传文件得到的内容块。最后一个分割块可以小于4MB。不同分割块可以乱序并行上传。所有分割块上传完毕后由服务器进行内容排序合并。

7. 上传块（Chunk）：上传块是对分割块的进一步切分，可以由用户自行设定大小，以适应不同网络环境的限制。上传块必须顺序上传，同时根据需求保存上传过程中的上下文信息、同一分割块已传部分的校验值，以便在断点续传时恢复操作环境。

8. 上下文信息（Context）：服务器成功保存上传块后返回的操作环境信息，可保存在上传端本地，以便恢复操作环境。上传开始后，每个分割块都有自己的上下文信息。上传端不能修改接收到的上下文信息。

9. 校验值（CheckSum）：服务器成功保存上传块后返回的、当前分割块的已传部分的校验值，可保存在上传端本地，用于最后合并文件。上传开始后，每个分割块都有自己的校验值。上传端不能修改接收到的校验值。

<a name="resumable-upload-model"></a>

#### 3.2 工作模型

                                                        |
      CUSTOM                                            | QINIU
                                                        |
            +------------+                              |     +------------+
            |            |    6.1 Invoke CallbackURL    |     |            |
            |            | <----------------------------+---- |            |
            | Biz-Server |                              |     | Up-Server  |
            |            | -----------------------------+---> |            |
            |            |    6.2 Return Result         |     |            |
            +------------+                              |     +------------+
                 ^  |                                   |       ^ |    ^ |
                 |  |                                   |       | |    | |
                 |  |                                   |       | |    | |
      1 Request  |  | 2 Make Policy/                    |       | |    | |
        Upload   |  |   UpToken                         |       | |    | |
                 |  |                                   |       | |    | |
      -----------+--+-----------                        |       | |    | |
                 |  |                                   |       | |    | |
      END-USER   |  |                                   |       | |    | |
                 |  V                                   |       | |    | |
            +------------+    4.1 Upload Blocks         |       | |    | |
            |            | -----------------------------+-------/ |    | |
            |            | <----------------------------+---------/    | |
            |            |    4.2 Return Context/       |              | |
            |            |        CheckSum              |              | |
            | Up-Client  |                              |              | |
            |            |                              |              | |
            |            |    5   Make File             |              | |
      3 Split File       | -----------------------------+--------------/ |
            |            | <----------------------------+----------------/
            +------------+    6.3 Return Result         |
                                                        |
                                                        |

<a name="resumable-upload-workflow"></a>

#### 3.3 流程

1. 请求断点续上传（Request Upload）：由上传端发起，向业务服务器申请执行断点续上传;

2. 生成操作策略/上传凭证（Make Policy/UpToken）：业务服务器对上传端进行鉴权/签名上传凭证/授权;

3. 分割文件（Split File）：上传端获得授权后，以指定块大小（一般为4MB）为单位，将待传文件分割为数个分割块;

4. 上传分割块（Upload Blocks）：上传端将单个分割块至上传服务器（可以并发上传不同的分割块，加快上传速度）。每个分割块的上传过程必须顺序完成（串行上传每个上传块）。上传服务器会针对接受到的上传块，返回对应分割块的已上传部分的上下文信息和校验码;

5. 合并文件（Make File）：所有分割块均成功上传完毕后，由上传端通知上传服务器将其合并成原上传对象文件;

6. 若指定CallbackURL，上传服务器在合并文件后会调用此URL，通知业务服务器做相应处理; 否则返回响应结果。

注：在第4步的任何节点均可终止（Abort）上传分割块，或在断点处根据上下文信息恢复上传分割块。


<a name="resumable-upload-api"></a>

#### 3.4 API

<a name="resumable-upload-authorization"></a>

##### 3.4.1 授权

授权信息在 HTTP 头部表现如下：

    Authorization UpToken <UploadToken>

授权操作需要在 HTTP Headers 中新增一个名为 Authorization 的字段，并传入 [UploadToken](#upload-token) 作为值。

上述授权格式等价于：

    Authorization UpToken accessKey:authDigest:authInfoEncoded

`<UploadToken>` 的细节可以参考文档：[生成上传授权凭证](#upload-token)

<a name="resumable-upload-mkblk"></a>

##### 3.4.2 创建分割块（Block）并上传第一个数据块（Chunk）

      HTTP/1.1
      POST http://up.qbox.me/mkblk/<BlockSize>
      Content-Type: application/octet-stream
      Request Headers: {
          Authorization: UpToken <UploadToken>
      }
      Request Body: <First-Chunk-Binary>

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store
      Response Body: {
          ctx: <BlockCtx string>,
          checksum: <BlockChecksum string>,
          crc32: <ChunkCrc32 int>,
          host: <SelectedUpHost string> // 后续的 bput, rs-mkfile 等请求要求发到此 host
      }

<a name="resumable-upload-bput"></a>

##### 3.4.3 上传分割块（Block）中的数据块（Chunk）

      HTTP/1.1
      POST <SelectedUpHost>/bput/<BlockCtx>/<Offset>
      Content-Type: application/octet-stream
      Request Headers: {
          Authorization: UpToken <UploadToken>
      }
      Request Body: <Next-Chunk-Binary>

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store
      Response Body: {
          ctx: <BlockCtx string>,
          checksum: <BlockChecksum string>,
          crc32: <ChunkCrc32 int>,
          host: <SelectedUpHost string>
      }

<a name="resumable-upload-mkfile"></a>

##### 3.4.4 合并文件

      HTTP/1.1
      POST <SelectedUpHost>/rs-mkfile/<EncodedEntryURI>/fsize/<Fsize> \
                                      /mimeType/<EncodedMimeType> \
                                      /meta/<EncodedCustomMeta> \
                                      /customer/<CustomerId> \
                                      /params/<EncodedCallbackParams> \
                                      /rotate/<Rotate>
      Content-Type: text/plain
      Request Headers: {
          Authorization: UpToken <UploadToken>
      }
      Request Body: <Ctx-Array> // 以 “,” 分隔的 ctx string 列表，注意 Content-Type: text/plain

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store
      Response Body: {
          hash: <FileEtag string>
      }

其中反斜杠（\）因排版换行需要，实际情况请忽略。

若尖括号包裹的字段（`<...>`）不传入，相应的前缀也不必传入。比如当不传入 `<EncodedCustomMeta>`, `<CustomerId>` 和 `<Rotate>` 时，POST 的 URL 则为：

    http://up.qbox.me/rs-mkfile/<EncodedEntryURI> \
                               /fsize/<Fsize> \
                               /mimeType/<EncodedMimeType> \
                               /params/<EncodedCallbackParams>

其中反斜杠（\）因排版换行需要，实际情况请忽略。

**URL参数详解**

- [EncodedEntryURI](/v3/api/words/#EncodedEntryURI) 定义了具体的 Bucket 和 Key，必须项。

注意：同一存储空间（Bucket）下，已存在与当前上传条目名称（[EncodedEntryURI](/v3/api/words/#EncodedEntryURI) 元素中的 [Key](/v3/api/words/#EntryURI)）相同的条目时，若当前上传文件内容与原有文件内容一致，则返回成功响应；若文件内容不一致，则上传失败，并返回失败响应。

- `Fsize` 文件大小，单位 Byte，必须项。

- [EncodedMimeType](/v3/api/words/#EncodedMimeType) 表明文件的 MIME 类型，缺省情况下为 `application/octet-stream`，可选项。

- [EncodedCustomMeta](/v3/api/words/#EncodedCustomMeta) 文件备注信息，可选项，一般不传入。

- `CustomerId` 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据终端用户标识给图片打水印时可以传入终端用户ID。该参数为可选项。

- `EncodedCallbackParams` 指定文件上传成功后七牛云存储向客户方的业务服务器执行回调发送的数据，详细解释可以参考 [CallbackParams](#CallbackParams)。该参数的值需经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码。该参数为可选项。

- `Rotate` 上传图片时专用，特别适合移动设备拍照后直传。`<Rotate>` 值为 0 ：表示根据图像EXIF信息自动旋转；值为 1 : 右转90度；值为 2 :右转180度；值为 3 : 右转270度。可结合 [生成上传授权凭证 uploadToken 之 escape 参数详解](#escape-expression) 搭配使用。该参数为可选项。


<a name="resumable-upload-examples"></a>

#### 3.5 样例

样例程序：

- C/C++ - <https://github.com/qiniu/c-sdk/blob/master/qbox/up.c>
- Java - <https://github.com/qiniu/java-sdk/blob/master/src/main/java/com/qiniu/qbox/up/UpService.java>
- Perl - <https://github.com/qiniu/perl-sdk/blob/master/lib/QBox/UP.pm>
- Ruby - <https://github.com/qiniu/ruby-sdk/blob/master/lib/qiniu/rs/up.rb>

参考资料：

- [C/C++ SDK 使用指南——断点续上传](/v3/sdk/c/#rs-put-blocks)

<a name="download"></a>

### 4. 下载文件

七牛云存储以下几种权限方式的下载文件：

1. 生成带下载授权凭证的URL，文件可公开临时匿名下载，但链接有生命周期，可自定义该有效期。
2. 绑定域名，以公开外链的方式下载。
3. 限制直接访问源文件，只限访问预处理后的目标文件。比如限制访问原图，只可访问打水印后的缩略图。
4. 设置黑名单/白名单，仅限/或拒绝特地区域的用户访问下载。

<a name="get"></a>

#### 4.1 生成带下载授权凭证的URL

生成带下载授权凭证的URL即表示为私有资源下载。私有(private)是bucket的一个属性，一个私有bucket中的资源为私有资源。私有资源不可匿名下载。

新创建的bucket默认为私有，也可以将某个bucket设为公有，公有bucket中的资源为公有资源，公有资源可以匿名下载。

可以使用以下命令将<bucket>设为私有：

    qboxrsctl private <bucket> 1

可以使用以下命令将<bucket>设为公开：

    qboxrsctl private <bucket> 0

**私有资源的下载方式**

私有资源无法匿名下载。下载私有资源需要带有下载授权，下载授权以download token的方式表达。

假设某bucket为私有，则

- `http://<bucket>.qiniudn.com/x.jpg` 无法下载。
- `http://<bucket>.qiniudn.com/x.jpg?token=<token>`，在 `<token>` 合法的条件下，可以正常下载。

**下载授权凭证（downloadToken）的构成**

downloadToken 由三部分组成：

    downloadToken = "<access_key>:<checksum>:<scope>"

注意：尖括号“<>”表示要替换的内容，不可在实际字符串中出现。

checksum 的构成为：

    checksum = "urlsafe_base64_encode(<hmac>)"
    hmac = "sha1_hmac(<secret_key>, <scope>)"

scope的构成为：

    scope = "urlsafe_base64_encode(<json_scope>)"

`urlsafe_base64_encode()` 函数的规格参考：[URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode)

json_scope 是一个 JSON 标准格式的字符串：

    {"E": <deadline>, "S": <pattern>}

`deadline` 为一个时间点，数值是1970年1月1日0点到此时间点的秒数，这个时间点之后，资源无法下载。

`pattern` 为URL匹配模式，下载URL匹配该模式不成功的话，资源无法下载。

注意：`pattern` 匹配完整的URL，而不是URL的子串。

pattern 详解：

模式 | 说明 | 示例
-----|-----|------
`*` | 匹配所有不含"/"字符的子串 | `http://dl.example.com/*-small.jpg`
`?` | 匹配所有非"/"的字符 | `http://dl.example.com/a?.jpg`
`abc` | 匹配字符串 "abc", ('*', '?', '\\', '[' 除外) | `http://dl.example.com/abc.jpg`
`abc\\?d` | 匹配字符串 "`abc?d`", 双斜杠表示转义，可以包含特殊字符 | `http://dl.example.com/\\abc?d=xxx`
`[abc]` | 匹配字符 a, b 或者 c | `http://dl.example.com/[abc].jpg`
`[^abc]` | 匹配除 a, b 或者 c 以外的字符 | `http://dl.example.com/[^abc].jpg`
`[a-z]` | 匹配 a-z 范围以内的任意字符 | `http://dl.example.com/[a-zA-Z0-9].jpg`
`[^a-z]` | 匹配 a-z 范围以外的任意字符 | `http://dl.example.com/[^a-z].jpg`
`[abc\\?d]` | 匹配字符 a, b, c, `?` 或者 d | `http://dl.example.com/[\\abc?d]`

<a name="publish"></a>

#### 4.2 直接绑定域名为文件创建公开外链

将一个存储空间 `<Bucket>` 里边的所有 `<Key>` 以静态外链的形式发布到某个指定域名 `<Domain>` 下。

生成的外链地址为：`http://<Domain>/<key>` （注意 <key> 首字符是不用带斜杠的）

      HTTP/1.1
      POST http://rs.qbox.me/publish/<EncodedDomain>/from/<Bucket>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

**请求参数**

`<EncodedDomain>`
: 需要绑定的目标域名，例如：cdn.example.com，需经过 [URLSafeBase64Encode](/v3/api/words/#EncodedEntryURI)。

`<Bucket>`
: 空间名称

`<Domain>` 可以是真实域名，真实的 `<Domain>` 需要在 DNS 管理里边 CNAME 到 iovip.qbox.me 。

由于众所周知的原因，使用 publish 接口绑定的域名若是您自己（或您公司）的域名，必须是已经备案过的。您需要将该域名备案信息发送到我们的邮箱 <support@qiniutek.com>，我们的工作人员会完成后续处理。

您也可以在 [七牛云存储开发者自助网站上进行域名绑定操作](https://dev.qiniutek.com/buckets)。

<a name="unpublish"></a>

#### 4.3 解除域名绑定取消文件的公开外链

      HTTP/1.1
      POST http://rs.qbox.me/unpublish/<EncodedDomain>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

您也可以在 [七牛云存储开发者自助网站上进行域名解绑操作](https://dev.qiniutek.com/buckets)。

<a name="download-by-range-bytes"></a>

#### 4.4 断点续下载

断点续下载协议标准参考：<http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.35>

七牛云存储按以上标准支持断点续下载，只需在 HTTP 请求下载链接的头部附带 `Range` 字段即可。

    Range: bytes=<first-byte-pos>-<last-byte-pos>

**参数**

`<first-byte-pos>`
: 起始位置，从数字零（0）开始计算。

`<last-byte-pos>`
: 断点续下载中，最后一个字节所在的位置。

<a name="download-if-notfound"></a>

#### 4.5 针对下载出现 404 NotFound 智能化处理

您可以上传一个应对HTTP 404出错处理的文件，当您 [绑定域名创建公开外链](#publish) 后，若公开的外链找不到该文件，即可使用您上传的“自定义404文件”。要这么做，您只须在名为 action 的表单域中将 [EncodedEntryURI](/v3/api/words/#EncodedEntryURI) 元素中的 `<Key>` 设置为固定字符串类型的值 `errno-404` 即可。

<a name="set-protected"></a>

#### 4.6 设置源文件(比如原图)保护

您可以在 [七牛云存储开发者自助网站上为指定的存储空间设置源文件保护](https://dev.qiniutek.com/buckets)。

设置原图(或源文件)保护后，其原图(或源文件)为私有。不能通过如下方式公开访问：

- http://绑定域名/文件key或相对路径
- http://绑定域名/文件key或相对路径?操作符/操作符参数

而只能通过如下方式进行公开访问（其中加号“+”忽略）：

- http://绑定域名/文件key或相对路径 + 已设定的连接符 + 已设定的操作符规格别名

其中，已设定的连接符可以是缩略图间隔标志符，已设定的操作符规格别名可以是缩略图样式名。

    http://<Domain>/<Key><Separator><StyleName>

<a name="pub-access-mode"></a>

##### 4.6.1 为指定的存储空间设置保护模式

      HTTP/1.1
      POST http://pu.qbox.me:10200/accessMode/<Bucket>/mode/<ProtectedMode>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

**请求参数**

`<Bucket>`
: 指定目标存储空间名称，必须，字符串类型。

`<ProtectedMode>`
: 是否开启保护，可选值为数字 0 或者 1 。

<a name="pub-separator"></a>

##### 4.6.2 设置友好URL访问中的连接符

以下操作在存储空间 `<ProtectedMode>=1` 时有效。

      HTTP/1.1
      POST http://pu.qbox.me:10200/separator/<Bucket>/sep/<EncodedSeparator>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

**请求参数**

`<Bucket>`
: 指定目标存储空间名称，必须，字符串类型。

`<EncodedSeparator>`
: 分割符，字符串类型类型，比如可以是 感叹号!、地址符（@）、美元符（$）、中划线（-）、下划线（_）、点（.）等。经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码。

<a name="pub-style"></a>

##### 4.6.3 设置URL友好的风格样式名

以下操作在存储空间 `<ProtectedMode>=1` 时有效。

      HTTP/1.1
      POST http://pu.qbox.me:10200/style/<Bucket>/name/<EncodedStyleName>/style/<EncodedStyle>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

**请求参数**

`<Bucket>`
: 指定目标存储空间名称，必须，字符串类型。

`<EncodedStyleName>`
: 风格样式名，字符串类型，经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码。

`<EncodedStyle>`
: 样式定义，字符串类型，经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码。

若请求成功，文件可以如下链接形式进行访问：

    [GET] http://<Domain>/<Key><Separator><StyleName> HTTP/1.1 200 OK

<a name="pub-unstyle"></a>

##### 4.6.4 取消URL友好风格的样式名访问

以下操作在存储空间 `<ProtectedMode>=1` 时有效。

      HTTP/1.1
      POST http://pu.qbox.me:10200/unstyle/<Bucket>/name/<EncodedStyleName>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

**请求参数**

`<Bucket>`
: 指定目标存储空间名称，必须，字符串类型。

`<EncodedStyleName>`
: 风格样式名，字符串类型，经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码。

若请求成功，文件以如下链接形式进行访问将会出现404：

    [GET] http://<Domain>/<Key><Separator><StyleName> HTTP/1.1 404 Not Found

<a name="anti-theft-chain"></a>

#### 4.7 防盗链设置

您可以在 [七牛云存储开发者自助网站上为指定的存储空间进行防盗链设置](https://dev.qiniutek.com/buckets)。

<a name="uc-antiLeechMode"></a>

##### 4.7.1 设置防盗链模式

      HTTP/1.1
      POST http://uc.qbox.me/antiLeechMode
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }
      Request Body: {
          bucket: <BucketName string>,
          mode : <Mode int> // 0:n/a; 1:white list; 2:black list
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

<a name="uc-referAntiLeech"></a>

##### 4.7.2 更新防盗链记录值

      HTTP/1.1
      POST http://uc.qbox.me/referAntiLeech
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }
      Request Body: {
          bucket: <BucketName string>,
          mode : <Mode int> // 0:n/a; 1:white list; 2:black list
          action: <Action string> // add | del
          pattern: <Pattern string> // *.example.com | 12.34.56.*
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

<a name="stat"></a>

### 5. 查看文件基本属性信息

      HTTP/1.1
      POST http://rs.qbox.me/stat/<EncodedEntryURI>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store
      Response Body: {
          hash: <FileETag string>,
          fsize: <FileSize int>,
          putTime: <PutTime int64>, // 文件上传时候的七牛云存储服务器时间
          mimeType: <MimeType string>
      }

**请求参数**

`<EncodedEntryURI>`
: 指定的具体文件，必填。参考：[EncodedEntryURI](/v3/api/words/#EncodedEntryURI)

<a name="delete"></a>

### 6. 删除指定文件

      HTTP/1.1
      POST http://rs.qbox.me/delete/<EncodedEntryURI>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

<a name="drop"></a>

### 7. 删除所有文件（整个空间/Bucket）

      HTTP/1.1
      POST http://rs.qbox.me/drop/<Bucket>
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

<a name="batch"></a>

### 8. 批量操作

**请求**

    POST http://rs.qbox.me/batch
    Content-Type: application/x-www-form-urlencoded
    RequestBody: op=<Operation>&op=<Operation>&...

其中 op=<Operation> 是一个操作指令。例如 `/get/<EncodedEntryURI>`, `/stat/<EncodedEntryURI>`, `/delete/<EncodedEntryURI>`, …

<a name="batch-stat"></a>

#### 8.1 批量获取文件基本属性信息

    POST http://rs.qbox.me/batch
    Content-Type: application/x-www-form-urlencoded
    RequestBody: op=/stat/<EncodedEntryURI>&op=/stat/<EncodedEntryURI>&...

<a name="batch-get"></a>

#### 8.2 批量获取文件授权后的临时下载链接

    POST http://rs.qbox.me/batch
    Content-Type: application/x-www-form-urlencoded
    RequestBody: op=/get/<EncodedEntryURI>&op=/get/<EncodedEntryURI>&...

<a name="batch-delete"></a>

#### 8.3 批量删除文件

    POST http://rs.qbox.me/batch
    Content-Type: application/x-www-form-urlencoded
    RequestBody: op=/delete/<EncodedEntryURI>&op=/delete/<EncodedEntryURI>&...

**响应**

    200 OK [
        <Result1>, <Result2>, ...
    ]
    298 Partial OK [
        <Result1>, <Result2>, ...
    ]

    <Result> 是 {
        code: <HttpCode int>,
        data: <Data> 或 error: <ErrorMessage string>
    }

<a name="refresh-bucket"></a>

### 9. 清除服务端缓存

      HTTP/1.1
      POST http://uc.qbox.me/refreshBucket
      Content-Type: application/x-www-form-urlencoded
      Request Headers: {
          Authorization: QBox <AccessToken>
      }
      Request Body: {
          bucket: <BucketName string>
      }

      HTTP/1.1 200 OK
      Content-Type: application/json
      Cache-Control: no-store

您可以在 [七牛云存储开发者自助网站上为指定的存储空间一键清除缓存](https://dev.qiniutek.com/buckets)。
