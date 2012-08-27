---
title: API | 七牛云存储
---

# API

[**应用接入与认证授权**](/v3/api/auth/)

- [应用接入](/v3/api/auth/#app-access)
- [认证授权](/v3/api/auth/#app-auth)
    - [请求签名](/v3/api/auth/#req-signature)
        - [流程](/v3/api/auth/#workflow)
        - [示例](/v3/api/auth/#examples)
            - [PHP 数字签名示例程序](/v3/api/auth/#php-example)
            - [Ruby 数字签名示例程序](/v3/api/auth/#ruby-example)
    - [请求认证](/v3/api/auth/#req-auth)

[**云存储接口**](/v3/api/io/)

- [客户端直传](/v3/api/io/#rs-PutAuth)
    - [取得上传授权](/v3/api/io/#rs-PutAuth)
    - [上传文件](/v3/api/io/#rs-PutFile)
- [服务端直传](/v3/api/io/#server-upload)
- [获取指定资源信息](/v3/api/io/#rs-Stat)
- [获取指定资源内容（可下载）](/v3/api/io/#rs-Get)
- [发布公开资源](/v3/api/io/#rs-Publish)
- [取消资源发布](/v3/api/io/#rs-Unpublish)
- [删除指定资源](/v3/api/io/#rs-Delete)
- [删除整张资源表](/v3/api/io/#rs-Drop)
- [批量处理接口](/v3/api/io/#rs-Batch)
- [移动或重命名操作接口](/v3/api/io/#rs-Move)

[**图像处理接口**](/v3/api/foimg/)

- [获取图片信息](/v3/api/foimg/#fo-imageInfo)
- [获取图片EXIF信息](/v3/api/foimg/#fo-imageExif)
- [缩略图预览](/v3/api/foimg/#fo-imagePreview)
- [自定义缩略图规格](/v3/api/foimg/#fo-imagePreviewEx)
- [自定义图像处理接口别名](/v3/api/foimg/#fo-image-alias)
- [高级图像处理接口（缩略、裁剪、旋转、转化）](/v3/api/foimg/#fo-imageMogr)
- [高级图像处理接口（缩略、裁剪、旋转、转化）并持久化存储处理结果](/v3/api/foimg/#fo-imageMogrAs)

[**理解常用术语**](/v3/api/words/)

- [Entry](/v3/api/words/#Entry)
- [EntryURI](/v3/api/words/#EntryURI)
- [EncodedEntryURI](/v3/api/words/#EncodedEntryURI)
- [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode)
- [MimeType](/v3/api/words/#MimeType)
- [EncodedMimeType](/v3/api/words/#EncodedMimeType)
- [CustomMeta](/v3/api/words/#CustomMeta)
- [EncodedCustomMeta](/v3/api/words/#EncodedCustomMeta)
- [FileCRC32Checksum](/v3/api/words/#FileCRC32Checksum)

[**错误码参照表**](/v3/api/code/)

- [认证错误码](/v3/api/code#acc)
