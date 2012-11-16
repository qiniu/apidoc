# 七牛云存储常见问题

## 一、概念篇

1. [七牛云存储是什么？有什么用处？](#qiniu-cloud-storage)
2. [七牛云存储能为我解决什么问题？](#problems-qiniu-cloud-storage-solved)
3. [七牛云存储和其他云存储相比有什么优势？](#superiority)
4. [七牛云存储如何保证数据安全性？](#security-guarantee)

## 二、计费篇

1. [七牛云存储如何收费？价格怎样？](#price)
2. [是否提供试用，如何申请试用，试用周期多长？](#try)
3. [如何查询账单？](#bill)
4. [如何获取发票？](#invoice)

## 三、使用篇

### [文件上传](#upload-file)

1. [七牛云存储可以存储哪些数据？](#data-stores-in-qiniu)
2. [是否限制单个文件大小？](#file-size)
3. [如何上传文件到七牛云存储？](#how-to-upload)

### [AccessKey/SecretKey](#key)

1. [什么是密钥（AccessKey/SecretKey）？密钥的作用是什么？](#accesskey-secretkey)
3. [如何获取和保护密钥？](#accesskey-secretkey-security)

### [Bucket](#bucket)

1. [什么是空间（Bucket）？有何作用？](#bucket-usefulness)
2. [如何创建空间？](#create-bucket)
3. [我可以创建多少个空间？](#bucket-number-limitation)
4. [每个空间提供多大配额？](#bucket-capacity-limitation)
5. [我应该如何使用空间？](#use-bucket)
6. [如何删除空间？](#delete-bucket)

### [文件外链](#file-link)

1. [如何为空间绑定域名，为文件提供URL链接访问？](#publish-with-custom-domain)
2. [如何自定义处理 404 Not Found ？](#404-not-found)
3. [如何解除域名绑定，取消文件URL链接访问？](#unpublish)

### [防盗链](#antileech)

1. [什么是 “防盗链” ？](#what-is-antileech)
2. [什么是 “refer 防盗链” ？](#what-is-refer-antileech)
3. [如何设置 “防盗链”　？](#how-to-antileech)
4. [什么是白名单 / 黑名单？](#white-list-black-list)
5. [如何设置白名单或黑名单？](#white-list-black-list-setting)
6. [如何取消 “防盗链” ？](#cancel-antileech)

### [镜像存储](#mirror-storage)

1. [什么是镜像存储？有什么作用？](#what-is-mirror-storage)
2. [如何使用镜像存储？](#how-to-mirror-storage)
3. [如何取消镜像存储使用？](#cancel-mirror-storage)

### [发布静态网站](#publish-static-website)

1. [什么是静态网站？](#static-website)
2. [如何用七牛云存储发布和更新静态网站？](#use-qiniu-cloud-storage-publish-static-website)
3. [七牛云存储针对静态资源资源访问是否提供 GZip 压缩输出加速访问？](#gzip-your-website-static-resources)
4. [如何为静态网站自定义 404 Not Found 页面？](#static-website-404-notfound)
5. [如何取消静态网站访问？](#cancel-static-website)

### [图像处理](#image-process)

1. [七牛云存储是否提供自定义缩略图处理？如何提供使用？](#custom-thumbnail)
2. [如何自定义缩略图规格？](#thumbinal-style)
3. [如何自定义友好URL风格？](#friendly-style)
4. [如何给图片打水印？](#watermark)
5. [如何设置原图保护？](#origin-image-protection)

### [视频/音频处理](#video-audio)

1. [七牛云存储是否支持为上传的视频/音频源文件提供压缩转码处理？](#video-audio-compress-transcoding)
2. [如何使用七牛云存储的视频/音频处理服务？](#qiniu-cloud-storage-video-audio-service)

## 四、工具篇

1. [如何使用命令行工具同步文件到七牛云存储？](#sync-file-to-qiniu-cloud-storage)
2. [有无提供命令行工具辅助开发调试？](#command-line-debug-helper-tools)

## 五、开发篇

1. [七牛云存储服务集群如何架构？](#qiniu-architecture)
2. [Web 网页如何向七牛云存储直传文件？](#upload-file-to-qiniu-from-web)
3. [iOS / Android 移动端如何向七牛云存储直传文件？](#upload-file-to-qiniu-from-iOS-Android)
4. [PC / 服务端程序如何向七牛云存储直传文件？](#upload-file-to-qiniu-from-PC-Server)
