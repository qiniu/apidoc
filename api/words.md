---
title: 理解常用术语 | 七牛云存储
---

# 理解常用术语

## 目录

- [Entry](#Entry)
- [EntryURI](#EntryURI)
- [EncodedEntryURI](#EncodedEntryURI)
- [URLSafeBase64Encode](#URLSafeBase64Encode)
- [MimeType](#MimeType)
- [EncodedMimeType](#EncodedMimeType)
- [CustomMeta](#CustomMeta)
- [EncodedCustomMeta](#EncodedCustomMeta)
- [FileCRC32Checksum](#FileCRC32Checksum)
- [VarExpression](#VarExpression)

<a name="Entry"></a>

### Entry

\<Entry\> 代指资源，一般我们说的资源即文件资源。

<a name="EntryURI"></a>

### EntryURI

用以定位资源（Entry）的 Key 称为 \<EntryURI\>，一般为文件路径的表示方法。每个 \<EntryURI\> 实际上是 \<Bucket\>:\<Key\> 的描述形式。其中 \<Bucket\> 是表名，\<Key\> 是资源的索引值，在同一个表中唯一。EntryURI 中可以包含 '/'，且 '/' 字符并无特殊意义，和普通字符一样对待。

<a name="EncodedEntryURI"></a>

### EncodedEntryURI

由于 \<EntryURI\> 中可能包含 URL 中不合适的字符，故此通常需要对其进行 Encode，我们称之为 \<EncodedEntryURI\>。Encode 方式由下面的 [URLSafeBase64Encode](#URLSafeBase64Encode) 定义。

<a name="URLSafeBase64Encode"></a>

### URLSafeBase64Encode

用于URL传递安全格式的Base64编码字符，该编码方式是将一字符串用base64编码，同时将编码后字符中的加号“+”换成中划线“-”，斜杠“/”换成下划线“_”。

假如您用PHP实现该函数，示例代码如下：

    function URLSafeBase64Encode($originalStr)
    {
        $find = array("+","/");
        $replace = array("-", "_");
        return str_replace($find, $replace, base64_encode($originalStr));
    }

<a name="MimeType"></a>

### MimeType

文件的 MIME type，即该资源的媒体类型。在七牛云存储各API里边，缺省的 MimeType 为 "application/octet-stream"。

<a name="EncodedMimeType"></a>

### EncodedMimeType

经过[URLSafeBase64Encode](#URLSafeBase64Encode) 编码过的 [MimeType](#MimeType)。

<a name="CustomMeta"></a>

### CustomMeta

自定义文本信息，可用于备注。

<a name="EncodedCustomMeta"></a>

### EncodedCustomMeta

经过[URLSafeBase64Encode](#URLSafeBase64Encode) 编码过的 [CustomMeta](#CustomMeta)。

<a name="FileCRC32Checksum"></a>

### FileCRC32Checksum

文件的 crc32 校验值，十进制整数。

<a name="VarExpression"></a>

### VarExpression

指定上传操作成功后回调执行指定的 API，要回调执行的 API 用 `$(VarExpression)` 表示。参考 [生成上传授权凭证 uploadToken 之 escape 参数详解](/v3/api/io/#escape-expression) 。

VarExpression 可以是如下几个具体的 API：

- `imageInfo` - [获取上传图片的基本信息](/v3/api/foimg/#fo-imageInfo)，支持访问 JSON 数据中的子字段
- `exif` - [获取图片EXIF信息](/v3/api/foimg/#fo-imageExif)，支持访问 JSON 数据中的子字段
- `fsize` - 获取文件大小，单位 Byte，没有子字段
- `etag` - 获取文件 etag 值，没有子字段

VarExpression 支持同 JSON 数据一样的 `<Object>.<Property>` 访问形式，比如：

- {API名称}
- {API名称}.{子字段}
- {API名称}.{子字段}.{下一级子字段}

其中花括号包裹的部分（“{...}”）实际情况下需要用具体的 API 及其字段替代。　

`$(VarExpression)` 示例：

- `$(fsize)` - 获取当前上传文件的大小
- `$(etag)` - 获取当前上传文件的 etag  g

- `$(imageInfo)` -  获取当前上传图片的基本属性信息
- `$(imageInfo.width)` - 获取当前上传图片的原始高度
- `$(imageInfo.height)` - 获取当前上传图片的原始高度
- `$(imageInfo.format)` -  获取当前上传图片的格式

imageInfo 接口返回的 JSON 数据可参考：<http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageInfo>

- `$(exif)` - 获取当前上传图片的 EXIF 信息
- `$(exif.ApertureValue)` - 获取当前上传图片所拍照时的光圈信息
- `$(exif.ApertureValue.val)` - 获取当前上传图片拍照时的具体光圈值

exif 接口返回的 JSON 数据可参考：<http://qiniuphotos.dn.qbox.me/gogopher.jpg?exif>
