---
title: 图像处理接口 | 七牛云存储
---

# 图像处理接口

## 目录

- [获取图片基本信息](#fo-imageInfo)
- [获取图片EXIF信息](#fo-imageExif)
- [生成指定规格的缩略图](#fo-imageView)
- [高级图像处理接口（缩略、裁剪、旋转、转化）](#fo-imageMogr)
- [高级图像处理接口（缩略、裁剪、旋转、转化）并持久化存储处理结果](#fo-imageMogrAs)


## 说明

要使用图像处理服务，只需在图片的下载URL上对接本套协议规定的图像处理接口即可进行流式预览或转换。

关于获取图像的下载链接，可以参阅云存储接口：[获取指定资源内容（含下载链接）](/v3/api/io/#rs-Get)

## 协议

<a name="fo-imageInfo"></a>

### 1. 获取图片基本信息

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

<a name="fo-imageExif"></a>

### 2. 获取图片EXIF信息

**请求**

    GET <ImageDownloadURL>?exif

**响应**

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        // ...EXIF Data...
    }


<a name="fo-imageView"></a>

### 3. 生成指定规格的缩略图

**请求**

    GET <ImageDownloadURL>?imageView/<mode>/w/<Width>/h/<Height>/q/<Quality>/format/<Format>/sharpen/<Sharpen>/watermark/<HasWatermark>

**响应**

    200 OK
    <ImageBinaryData>

**请求参数详解**

`<mode>`
: 图像缩略处理的模式，分为如下几种：

- `<mode> = 1`，表示限定目标缩略图的宽度和高度，放大并从缩略图中央处裁剪为指定 `<Width>x<Height>` 大小的图片。
- `<mode> = 2`，指定 `<Width>` 和 `<Height>`，表示限定目标缩略图的长和宽，将缩略图的大小限定在指定的宽高矩形内。
- `<mode> = 2`，指定 `<Width>` 但不指定 `<Height>`，表示限定目标缩略图的宽度，高度等比缩略自适应。
- `<mode> = 2`，指定 `<Height>` 但不指定 `<Width>`，表示限定目标缩略图的高度，宽度等比缩略自适应。

`<Width>`
: 指定目标缩略图的宽度，单位：像素（px）

`<Height>`
: 指定目标缩略图的高度，单位：像素（px）

`<Quality>`
: 指定目标缩略图的图像质量，取值范围 1-100

`<Format>`
: 指定目标缩略图的输出格式，取值范围：jpg, gif, png, tif 等图片格式

`<Sharpen>`
: 指定目标缩略图的锐化指数，值为正整数，此数值越大，锐化度越高，图像细节损失越大

`<HasWatermark>`
: 是否打水印，`<HasWatermark>` 可选值为 0 或者 1。为 0 或不传入 `/watermark/<HasWatermark>` 时表示不打水印；值为 1 时 即 `/watermark/1` 时表示取相应的水印模板进行打水印处理。水印模板设置会在后面介绍。

**示例**

示例1：针对原图进行缩略，并从缩略图的中央部位裁剪为 200x200 的缩略图：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200

[![200x200](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200)

示例2：针对原图进行缩略，并从缩略图的中央部位裁剪为 200x200，图像质量为 1，gif 格式的缩略图：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/1/format/gif

[![200x200，图像质量:1，格式:gif](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/1/format/gif)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/1/format/gif)

示例3：针对原图进行缩略，并从缩略图的中央部位裁剪为 200x200，图像质量为 85，格式为 png，锐度指数为 10 的缩略图：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/85/format/png/sharpen/10

[![200x200，图像质量:85，格式:png，锐度:10](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/85/format/png/sharpen/10)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/1/w/200/h/200/q/85/format/png/sharpen/10)

示例4：针对原图进行缩略，并限定目标缩略图的长边为 200 px，短边自适应，缩略图宽和高都不会超出 200px：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200/h/200

[![限定长边为 200px，短边自适应，宽和高都不会超出 200px](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200/h/200)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200/h/200)

示例5：针对原图进行缩略，并限定目标缩略图的宽度为 200px，高度等比缩略自适应：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200

[![限定宽度为 200px, 高度等比缩略自适应](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/w/200)

示例6：针对原图进行缩略，并限定目标缩略图的高度为 200px，宽度等比缩略自适应：

    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200

[![限定高度为 200px, 宽度等比缩略自适应](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200)](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView/2/h/200)


<a name="fo-imageMogr"></a>

### 4. 高级图像处理接口（缩略、裁剪、旋转、转化）

除了对图像进行缩略有单独的处理接口，七牛云存储还提供了比较高级的图像处理接口，包含缩略、裁剪、旋转等一系列的功能，该接口规格如下：

**请求**

    GET <ImageDownloadURL>?imageMogr

        /thumbnail/<ImageSizeGeometry>
        /gravity/<GravityType> =NorthWest, North, NorthEast, West, Center, East, SouthWest, South, SouthEast
        /crop/<ImageSizeAndOffsetGeometry>
        /quality/<ImageQuality>
        /rotate/<RotateDegree>
        /format/<DestinationImageFormat> =jpg, gif, png, tif, etc.
        /auto-orient

注意，以上规格实际上为一行字符串，为了排版展示所以多行隔开陈述。

**响应**

    200 OK
    <ImageBinaryData>

示例：

    GET <ImageDownloadURL>?imageMogr/auto-orient/thumbnail/!50x50r/gravity/center/crop/!50x50/quality/80

如此可对 \<ImageDownloadURL\> 文件进行缩略，根据原图EXIF信息进行自动旋转调正，并裁剪为 50x50 大小且以原图片格式压缩品质为80进行输出。

您可能留意到部分参数以 ! 开头，这是参数被转义的标识。为了方便阅读，我们采用了特殊的转义方法。以下是转义符号列表：

    p => % (percent)
    r => ^ (reverse)
    a => + (add)

也就是 !50x50r 其实代表 50x50^ 这样一个字符串。!50x50 代表 50x50 这样一个字符串（实际上这个字符串不需要转义）。`<ImageSizeAndOffsetGeometry>` 中的 OffsetGeometry 部分可以省略，缺省为 +0+0。也就是 /crop/50x50 等价于 /crop/!50x50a0a0，执行 -crop 50x50+0+0 语义。

如下是 `/thumbnail/<ImageSizeGeometry>` 和 `/crop/<ImageSizeAndOffsetGeometry>` 参数规格详解。

指定图片缩略或裁剪后的尺寸：

  size | 规格说明 | 样例
-------| ------- | ------------------------------------------------------
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

<table>
  <thead>
    <tr>
      <th width="20%">{size}{offset}</th>
      <th width="80%">指定偏移量 (缺省是 +0+0)，{size} 代表上述表格中的任意规格</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>{size}{+-}x{+-}y</td>
      <td>指定子图片相对于源图片的坐标，x代表横轴，y代表纵轴，单位像素。偏移量会受 gravity 参数的影响，但不受 {size} 操作符比如 % 的影响。</td>
    </tr>
  </tbody>
</table>

x 为正数时为从源图区域左上角的横坐标，为负数时，左上角坐标为0，然后从截出的子图片右边减去x象素宽度。
y 为正数时为从源图区域左上角的纵坐标，为负数时，左上角坐标为0，然后从截出的子图片上边减去y象素高度。

例如，

`/crop/!300×400a10a10` 表示从源图坐标为 x:10 y:10 截取 300×400 的子图片。
`/crop/!300×400-10a10` 表示从源图坐标为 x:0  y:10 截取 290×400 的子图片。

这个源图可以是 `/thumbnail/<ImageSizeGeometry>` 参数处理过后的图片，意味着 `thumbnail` 和 `crop` 之间的操作可以链式处理。


<a name="fo-imageMogrAs"></a>

### 5. 高级图像处理接口（缩略、裁剪、旋转、转化）并持久化存储处理结果

也可以将高级图像处理接口处理过的图片进行云端持久化，即将一个存储在七牛云存储的图片进行缩略、裁剪、旋转和格式转化处理后的缩略图作为一个新文件持久化存储到七牛云存储服务器上，这样就可以供后续直接使用而不用每次都传入参数进行图像处理。

图像处理持久化的接口规格如下：

**请求**

    POST <ImageDownloadURL>?imageMogr
        /thumbnail/<ImageSizeGeometry>
        /gravity/<GravityType> =NorthWest, North, NorthEast, West, Center, East, SouthWest, South, SouthEast
        /crop/<ImageSizeAndOffsetGeometry>
        /quality/<ImageQuality>
        /rotate/<RotateDegree>
        /format/<DestinationImageFormat> =jpg, gif, png, tif, etc.
        /auto-orient
        /save-as/<EncodedEntryURI>

注意，以上规格实际上为一行字符串，为了排版展示所以多行隔开陈述。

**参数**

和之前的流式图像处理接口 `imageMogr` 相比，持久化的图像处理接口多了一个 `save-as` 参数，指定了处理过后的缩略图存放的目标路径。

\<EncodedEntryURI\>
: 指定目标缩略图存放的地址。格式为 [URLSafeBase64Encode](/v3/api/words/#URLSafeBase64Encode)([EntryURI](/v3/api/words/#EntryURI))。

该 POST 请求需要进行签名认证才能调用，参考 [认证授权](/v3/api/auth/#app-auth)。

**响应**

    200 OK
    {"hash" => "FrOXNat8VhBVmcMF3uGrILpTu8Cs"}
