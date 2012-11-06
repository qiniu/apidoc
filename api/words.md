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

### VarExpression

指定上传操作成功后回调执行指定的 API，这里的 API 用 `$(VarExpression)` 表示。参考 [生成上传授权凭证 uploadToken 之 escape 参数详解](#escape-expression) 。

VarExpression 可以是如下形式：

- `imageInfo` - [获取上传图片的基本信息](/v3/api/foimg/#fo-imageInfo)
- `imageInfo.width` - [获取上传图片的宽度(px)](/v3/api/foimg/#fo-imageInfo)
- `imageInfo.height` - [获取上传图片的高度(px)](/v3/api/foimg/#fo-imageInfo)
- `exif` - [获取图片EXIF信息](/v3/api/foimg/#fo-imageExif)
