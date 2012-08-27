---
title: 云存储接口 | 七牛云存储
---

# 云存储接口

## 目录

- [客户端直传](#rs-PutAuth)
    - [取得上传授权](#rs-PutAuth)
    - [上传文件](#rs-PutFile)
- [服务端直传](#server-upload)
- [获取指定资源信息](#rs-Stat)
- [获取指定资源内容（可下载）](#rs-Get)
- [发布公开资源](#rs-Publish)
- [取消资源发布](#rs-Unpublish)
- [删除指定资源](#rs-Delete)
- [删除整张资源表](#rs-Drop)
- [批量处理接口](#rs-Batch)
- [移动或重命名操作接口](#rs-Move)


## 说明

**注意**：初次阅读本篇文档前，您需要理解一些常用术语比如 Entry, EntryURI, EncodedEntryURI 等。关于七牛云存储API术语您可以查阅另一篇文档：[**理解常用术语**](/v3/api/words/)

在请求七牛云存储各接口之前，您的服务端应用程序需要完成 [应用接入与认证授权](/v3/api/auth/) 。

且确保您服务端应用程序向七牛云存储接口的请求中隐式地包含 [QBox Token](/v3/api/auth/#req-auth) ，如此，您的服务端应用程序无需每次都显式地在参数中传递 Access Token 了。

服务端返回给您应用程序的每个可操作的URL都是独立有效的，即返回的每个URL本身就是一个独立有效的凭证，无需关联Cookies等其他额外的授权操作。


## 协议

### 1. 客户端直传

<a name="rs-PutAuth"></a>

### 1.1 取得上传授权

**请求**

    POST http://iovip.qbox.me/put-auth/<ExpiresIn>/callback/<urlsafe-base64-encode-callback-url>

**参数**

\<ExpiresIn\>
: 生成一个指定有效期的上传URL，缺省情况下是3600秒。\<ExpiresIn\> 参数是用来限定此授权URL的有效时间，假设当前时间是
10:00:00 ，如果 \<ExpiresIn\> 设置为3600秒，那么返回的这个授权URL的过期时间就是 11:00:00 之后 ，在
10:59:59 这个时间点都还可以成功上传任意大小的文件，但过了 11:00:00，哪怕上传一个几Bytes的文件都将失败。此临时授权URL用来进行匿名上传，可以很巧妙地搭配使用 \<ExpiresIn\> 参数，比如可以让一个授权URL是一次性的，也可以是一段时间内长期有效的。

\<urlsafe-base64-encode-callback-url\>
: 该 callback 参数为可选，是一个经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode) 编码后的字符串。该 callback url 是客户方的业务服务端用于接收七牛云存储服务端向其发送POST请求的URL，POST 请求的数据即客户方客户端程序在上传文件时设定的回调参数。在一个直传过程中，客户方的客户端程序（终端用户）往七牛云存储直传文件，可以设定一些用于文件上传成功后向客户方的业务服务器发送回调请求的参数，七牛云存储在文件上传成功后会将这些参数原文返回给客户方的业务服务器（执行回调）。

如下是文件直传场景下的一个回调流程：

1. 客户方业务服务端程序拿到七牛云存储上传文件的授权URL
2. 客户方业务服务端程序将该上传文件的授权URL返回给其客户端程序
3. 客户端程序往七牛云存储直传文件（可设置回调参数）
4. 七牛云存储接收完毕
5. 如果有回调参数，七牛云存储向客户方业务服务器发送回调（HTTP POST Request）
6. 如果有发起回调，七牛云存储接收客户方业务服务器的返回信息（HTTP Status Code, HTTP Response Body）
7. 七牛云存储返回给客户方客户端程序 HTTP Response；如果客户方的业务服务器返回的 HTTP Response Body 为 JSON 并且 `HTTP Headers["Content-Type"] = 'application/json'`，七牛云存储服务端会将其 Response Body 按照 JSON 格式返回给客户方的客户端程序。

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        url: <UploadURL>, // 生成的 URL 为：http://iovip.qbox.me/upload/<UploadHandle>
        expiresIn: <RealExpiresIn>
    }

返回的 UploadURL 在有效期内支持同时并发上传多个文件。当然，同一时间获取多个UploadURL进行上传也是支持的。


<a name="rs-PutFile"></a>

### 1.2 上传文件

**请求**

    POST http://io-node-n.qbox.me/upload/<UploadHandle>
    Content-Type: multipart/form-data; boundary=<Boundary>

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

该请求POST的地址即 [取得上传授权](#rs-PutAuth) 成功后返回的用于文件上传的临时有效URL 。

注意：该协议接口不需要传递 Access Token ，因为 POST 的目标URL已经是一个带有凭证的短期有效地址。

此请求传递三个必要参数，他们分别是 `action`，`file` 和 `params` 。

**action**

`action=<PutAction>` 是要执行的上传行为，表示向具体的资源表里新建一个条目。具体规格如下：

    action="/rs-put/{EncodedEntryURI}/mimeType/{EncodedMimeType}/meta/{EncodedCustomMeta}/crc32/{FileCRC32Checksum}"

{[EncodedEntryURI](/v3/api/words/#EncodedEntryURI)} 定义了具体的资源表和条目，必须项。

{[EncodedMimeType](/v3/api/words/#EncodedMimeType)} 表明文件的 MIME 类型，缺省情况下为 `application/octet-stream`，可选项。

{[EncodedCustomMeta](/v3/api/words/#EncodedCustomMeta)} 文件备注信息，可选项。

{[FileCRC32Checksum](/v3/api/words/#FileCRC32Checksum)} 文件的 crc32 校验值，十进制整数，可选项。若不传此参数则无数据校验功能。

`action` 的值至少是 `/rs-put/{EncodedEntryURI}` ，其他可选。

注意：同一资源表下，已存在与当前上传条目名称（{[EncodedEntryURI](/v3/api/words/#EncodedEntryURI)} 元素中的 <[Key](/v3/api/words/#EntryURI)\>）相同的条目时，若当前上传文件内容与原有文件内容一致，则返回成功响应；若文件内容不一致，则上传失败，并返回失败响应。

**file**

`file=<FileContent>` 是要上传的具体文件内容，`<FileContent>` 则是要上传文件的二进制内容。

**params**

params 用于文件上传成功后执行回调，七牛云存储服务器会向您应用的ClientId关联的业务服务器POST这些指定的参数。一般用于回传 [EntryURI](/v3/api/words/#EntryURI)，这样客户方的业务服务器会知道一个文件上传成功后以某条目名称记录到了七牛云存储的哪个资源表。

如果您觉得这个接口理解起来有难度，不妨以一种更简单的HTML Form结构来理解，以上接口描述等价于如下 HTML Form:

    <form enctype="multipart/form-data" action="http://io-node-n.qbox.me/upload/{UploadHandle}" method="POST">
        <input type="hidden" name="action" value="/rs-put/{EncodedEntryURI}/mimeType/{EncodedMimeType}/meta/{EncodedCustomMeta}/crc32/{FileCRC32Checksum}" />
        <input type="hidden" name="params" value="filename={thisFileName}&fileKey={thisFileKey}&tbname={ResourceTableName}"
        Choose a file to upload: <input name="file" type="file" />
        <input type="submit" value="Upload File" />
    </form>

**响应**

上传成功，七牛云存储服务端返回 HTTP Status Code 为 200：

    HTTP/1.1 200 OK


<a name="server-upload"></a>

### 2. 服务端直传

如果上传文件并非您的终端用户（即非用户创造内容情况下），而是直接从您的服务端直接向七牛云存储服务器上传。可以参考如下规格：

**请求**

    POST <IO_HOST>/rs-put/<EncodedEntryURI>/mimeType/<MimeType>/meta/<CustomMeta>
    Content-Type: application/octet-stream
    Body: <Data>

其中 `<IO_HOST>` 一般为 http://iovip.qbox.me 。Body 部分的 `<Data>` 为二进制文件流。

**响应**

返回包(JSON)：

    200 OK {
        hash: <FileETag>
    }


<a name="rs-Put-NotFound-File"></a>

**NotFound 处理**

您可以上传一个应对HTTP 404出错处理的文件，当您 [发布公开资源](#rs-Publish) 后，若公开的外链找不到该文件，即可使用您上传的“自定义404文件”。要这么做，您只须在名为 action 的表单域中将 {[EncodedEntryURI](/v3/api/words/#EncodedEntryURI)} 元素中的 <[Key](/v3/api/words/#EntryURI)\> 设置为固定字符串类型的值 "errno-404" 即可。如果您是使用七牛的 SDK ，调用SDK提供的 PutFile() 方法时，向参数 key 传值字符串 "errno-404" 即可。


<a name="rs-Stat"></a>

### 3. 获取指定资源信息

**请求**

    POST http://rs.qbox.me:10100/stat/<EncodedEntryURI>

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        hash: <FileETag>,
        fsize: <FileSize>,
        putTime: <PutTime>, // 文件上传时候的七牛云存储服务器时间
        mimeType: <MimeType>
    }

<a name="rs-Get"></a>

### 4. 获取指定资源内容（可下载）

**请求**

    POST http://rs.qbox.me:10100/get/<EncodedEntryURI>/base/<BaseHash>/attName/<AttName>/expires/<Seconds>

**参数**

- attName=\<AttachmentName\> 是下载时的名称，可选。如果未指定，直接是文件名。
- base=\<BaseHash\> 是文件内容的基版本，可选。如果指定了base，那么当服务器端文件已经发生变化时，返回失败。
- expires=\<Seconds\> 是希望返回的下载url的生命周期，精度为秒。如果未指定，默认 1 小时（3600）。

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        url: <DownloadURL>
        hash: <FileETag>
        fsize: <FileSize>
        mimeType: <MimeType>
        expires:<Seconds>    //下载url的实际生命周期，精度为秒。
    }

<a name="rs-Publish"></a>

### 5. 发布公开资源

将一个资源表 `<TableName>` 里边的所有 `<key>` 以静态外链的形式发布到某个指定域名 `<Domain>` 下。

`<Domain>` 可以是真实域名，真实的 `<Domain>` 需要在 DNS 管理里边 CNAME 到 iovip.qbox.me 。

生成的外链地址为：`http://<Domain>/<key>` （注意 `<key>` 首字符是不用带斜杠的）

若想让外链无法访问的那个资源有个文件可以替代，类似于自定义404页面，可以参考 [文件上传之 NotFound 处理](#rs-Put-NotFound-File)

**特别注意**

由于众所周知的原因，使用 `publish` 接口公开的 `<Domain>` 若是您自己（或您公司）的域名，必须是已经备案过的。您需要将该域名备案信息发送到我们的邮箱 [support@qiniutek.com](mailto:support@qiniutek.com)，我们的工作人员会完成后续处理。

**请求**

    POST http://rs.qbox.me:10100/publish/<EncodedDomain>/from/<TableName>

**参数**

\<EncodedDomain\>
: 资源发布的目标域名，例如：cdn.example.com，需经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode)。

\<TableName\>
: 指定要发布的资源表名称

**响应**

    HTTP/1.1 200 OK

<a name="rs-Unpublish"></a>

### 6. 取消资源发布

将一个已经公开的外链资源集合取消发布状态。

**请求**

    POST http://rs.qbox.me:10100/unpublish/<EncodedDomain>

**参数**

\<EncodedDomain\>
: 已发布资源对应的目标域名，例如：cdn.example.com，需经过 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode)。

**响应**

    HTTP/1.1 200 OK

<a name="rs-Delete"></a>

### 7. 删除指定资源

**请求**

    POST http://rs.qbox.me:10100/delete/<EncodedEntryURI>

**响应**

    HTTP/1.1 200 OK

<a name="rs-Drop"></a>

### 8. 删除整张资源表

将指定资源表的条目及其数据全部删除，需谨慎操作。

**请求**

    POST http://rs.qbox.me:10100/drop/<TableName>

**响应**

    HTTP/1.1 200 OK

<a name="rs-Batch"></a>

### 9. 批量处理接口

**请求**

    POST http://rs.qbox.me:10100/batch
    Content-Type: application/x-www-form-urlencoded
    Body: op=<Operation>&op=<Operation>&...

其中 op=\<Operation\> 是一个操作指令。例如 /get/\<EncodedEntryURI\>, /stat/\<EncodedEntryURI\>, /delete/\<EncodedEntryURI\>, ...

**示例**

批量获取文件属性信息：

    POST http://rs.qbox.me:10100/batch?op=/stat/<EncodedEntryURI>&op=/stat/<EncodedEntryURI>&...

批量获取下载链接：

    POST http://rs.qbox.me:10100/batch?op=/get/<EncodedEntryURI>&op=/get/<EncodedEntryURI>&...

批量删除文件：

    POST http://rs.qbox.me:10100/batch?op=/delete/<EncodedEntryURI>&op=/delete/<EncodedEntryURI>&...

**响应**

    200 OK [
        <Result1>, <Result2>, ...
    ]
    298 Partial OK [
        <Result1>, <Result2>, ...
    ]
    <Result> 是 {
        code: <HttpCode>,
        data: <Data> 或 error: <ErrorMessage>
    }

当部分 op 执行失败时，返回 298（PartialOK）。

<a name="rs-Move"></a>

### 10. 移动或重命名操作接口

**请求**

    POST http://rs.qbox.me:10100/move/<EncodedSrcEntryURI>/<EncodedDestEntryURI>

**参数**

\<EncodedSrcEntryURI\>
: 原 [EncodedEntryURI](/v3/api/words/#EncodedEntryURI)。

\<EncodedDestEntryURI\>
: 目标 [EncodedEntryURI](/v3/api/words/#EncodedEntryURI)。

**响应**

    HTTP/1.1 200 OK
