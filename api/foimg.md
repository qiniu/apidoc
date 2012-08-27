---
title: 图像处理接口 | 七牛云存储
---

# 图像处理接口

## 目录

- [获取图片信息](#fo-imageInfo)
- [获取图片EXIF信息](#fo-imageExif)
- [缩略图预览](#fo-imagePreview)
- [自定义缩略图规格](#fo-imagePreviewEx)
- [自定义图像处理接口别名](#fo-image-alias)
- [高级图像处理接口（缩略、裁剪、旋转、转化）](#fo-imageMogr)
- [高级图像处理接口（缩略、裁剪、旋转、转化）并持久化存储处理结果](#fo-imageMogrAs)


## 说明

要使用图像处理服务，只需在图片的下载URL上对接本套协议规定的图像处理接口即可进行流式预览或转换。

关于获取图像的下载链接，可以参阅云存储接口：[获取指定资源内容（含下载链接）](/v3/api/io/#rs-Get)

## 协议

<a name="fo-imageInfo"></a>

### 1. 获取图片信息

**请求**

    GET <ImageDownloadURL>/imageInfo

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


<a name="fo-imagePreview"></a>

### 3. 缩略图预览

**请求**

    GET <ImageDownloadURL>/imagePreview/<DestImageType>

注意，针对使用 [publish 接口](/v3/api/io/#rs-Publish)创建的公开外链图片，获取缩略图的规格如下：

    GET <ImageDownloadURL>?imagePreview/<DestImageType>

只需将 `<ImageDownloadURL>` 之后的斜杠“/”改为问号“?”即可，问号传参是兼容模式，两者均可用。

**响应**

    200 OK
    <ImageBinaryData>

这个 imagePreview 操作，实际上是为生成图片预览图之用。传入的 \<DestImageType\> 用以指定预览图的规格，枚举值。可以是：

0
: 用以 Web 的图片大图预览。具体规格为：image(w800h600q85)，格式可能是 jpeg, png, gif，视传入的原始格式而定，尽量保留原始格式。

1
: 用以 Web 的图片小图预览。具体规格为：image(w128h128q80)，格式可能是 jpeg, png, gif，视传入的原始格式而定，尽量保留原始格式。

10
: 用以 iPhone 的图片小图预览。具体规格为：image(w150h150q80)，但裁剪长边。

21
: 方块图(w640h640q92)。

22
: 限宽640，高自适应。q92

23
: 限宽320，高自适应。q92

24
: 限高640，宽度自适应。q92

25
: 方块图(w150h150q95)。

50
: 宽60px，高40px。q92

<a name="fo-imagePreviewEx"></a>

### 4. 自定义缩略图规格

自定义规格的缩略图同 imagePreview 接口，请求规格不变：

    GET <ImageDownloadURL>/imagePreviewEx/<DestImageType>

注意，针对使用 [publish 接口](/v3/api/io/#rs-Publish)创建的公开外链图片，获取缩略图的规格如下：

    GET <ImageDownloadURL>?imagePreviewEx/<DestImageType>

只需将 `<ImageDownloadURL>` 之后的斜杠“/”改为问号“?”即可，问号传参是兼容模式，两者均可用。

其中 \<DestImageType\> 为缩略图规格的枚举值，该值可以自定义，自定义的值可以是字符串（比如small）也可以是数字（比如1）。

枚举值 \<DestImageType\> 和具体的缩略图规格之间存在与之一一对应的关系。简单地说，枚举值是特定缩略图规格的简易符号表达。

可以通过七牛云存储的扩展工具 [qboxrsctl](https://github.com/qiniu/devtools) 来修改或新增 \<DestImageType\> 枚举值，然后在 <ImageDownloadURL>/imagePreviewEx/ 后面跟上相应的 \<DestImageType\> 即可，如此达到自定义缩略图规格的目的。

自定义 \<DestImageType\> 方法如下：

    qboxrsctl login <User> <Passwd>
    qboxrsctl setstyle <ImageStyleName> <Style>

其中 login 指令用来登录，登录成功后即可使用 setstyle 指令进行自定义缩略图规格设置。

qboxrsctl setstyle 指令有两个参数，第一个参数是枚举值 \<DestImageType\> 的名称，第二个参数 \<Style\> 为具体的规格表达式。

\<Style\> 可以是如下4种表达形式：

    square:<Size>;q:<Quality>

    <Width>x<Height>;q:<Quality>

    <Width>x;q:<Quality>

    x<Height>;q:<Quality>

模式1表示对图片缩略后裁剪为指定 \<Size\> 像素的方形图，质量为具体的 \<Quality\> 值。

模式2表示对图片进行指定宽度\<Width\>和高度\<Height\>的缩略，并按质量为具体的 \<Quality\> 值进行输出。

模式3表示对图片以指定宽度\<Width\>为基础，高度等比缩放进行缩略裁剪，并按质量为具体的 \<Quality\> 值进行输出。

模式4表示对图片以指定高度\<Height\>为基础，宽度等比缩放进行缩略裁剪，并按质量为具体的 \<Quality\> 值进行输出。


**注意**：使用 qboxrsctl setstyle 指令进行设置后，需要等待5分钟后新的自定义设置才能全局同步生效。


**示例**

模式1，输出一个尺寸为 128x128 px，质量为 85 的方形缩略图：

    qboxrsctl setstyle small "square:128;q:85"

    GET <ImageDownloadURL>?imagePreviewEx/small

模式2，输出一个宽度为400px，高度为300px，质量为85的缩略图：

    qboxrsctl setstyle medium "400x300;q:85"

    GET <ImageDownloadURL>?imagePreviewEx/medium

模式3，输出一个限宽 800px，高度等比缩放，质量为85的缩略图：

    qboxrsctl setstyle xlarge "800x;q:85"

    GET <ImageDownloadURL>?imagePreviewEx/xlarge

模式4，输出一个限高 600px，宽度等比缩放，质量为85的缩略图：

    qboxrsctl setstyle ylarge "x600;q:85"

    GET <ImageDownloadURL>?imagePreviewEx/ylarge


<a name="fo-image-alias"></a>

### 5. 自定义图像处理接口别名

觉得 `<ImageDownloadURL>/imagePreviewEx/<DestImageType>` 这种形式的图像处理接口太长或者不够语义化？您也可以自定义更加友好的接口别名，定义别名需要借助辅助工具 [qboxrsctl](https://github.com/qiniu/devtools) 。

例如，以下两种方式同为输出一个限高600px，宽度等比缩放，图像质量为85的缩略图。

假设 `<ImageDownloadURL>` 为 `http://iovip.qbox.me/test_img_bucket/img_key`

方式一：

    qboxrsctl setstyle ylarge "x600;q:85"
    GET http://iovip.qbox.me/test_img_bucket/img_key?imagePreviewEx/ylarge

方式二：

    // 给具体的图像处理规格定义别名
    qboxrsctl style <BucketName> <AliasName> <StyleSpec>

    // 指定<ImageDownloadURL>与<AliasName>之间的分割符
    qboxrsctl separator <BucketName> <Sep>

    // 最终，可以通过如下约定的规格进行访问
    GET <ImageDownloadURL><Sep><AliasName>

示例：

    // 自定义缩略图规格
    qboxrsctl setstyle ylarge "x600;q:85"

    // 给该自定义缩略图规格定义一个友好别名叫 ylarge.jpg ，并作用于 test_img_bucket
    qboxrsctl style test_img_bucket ylarge.jpg imagePreviewEx/ylarge

    // 指定 <ImageDownloadURL> 与规格别名 ylarge.jpg 之间的分隔符为中划线“-”
    qboxrsctl separator test_img_bucket -

    // 那么，此高度为600px质量为85的缩略图能以如下方式进行等价访问
    GET http://iovip.qbox.me/test_img_bucket/img_key-ylarge.jpg

`qboxrsctl style` 与 `qboxrsctl separator` 指令不仅仅用于 `imagePreviewEx` 接口，同样适用于给图像或其他文件的处理接口定义别名。


<a name="fo-imageMogr"></a>

### 6. 高级图像处理接口（缩略、裁剪、旋转、转化）

除了对图像进行缩略有单独的处理接口，七牛云存储还提供了比较高级的图像处理接口，包含缩略、裁剪、旋转等一系列的功能，该接口规格如下：

**请求**

    GET <ImageDownloadURL>/imageMogr

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

    GET <ImageDownloadURL>/imageMogr/auto-orient/thumbnail/!50x50r/gravity/center/crop/!50x50/quality/80

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
  scale% | 基于原图大小，按照指定的百分比进行缩放。 | [50%](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/!50p)
  scale-x%xscale-y% | 以百分比的形式指定缩略图的宽或高，另一边自适应等比缩放，只能使用一个 % 限定。 | [50%x](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/!50p) [x50%](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/!x50p)
  width | 指定缩略图宽度，高度等比自适应。 | [200](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/200)
  xheight | 指定缩略图高度，宽度等比自适应。 | [x100](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/x100)
  widthxheight | 以指定的小边为基准进行缩略。若宽度大于高度，以给定的高度为基准，宽度自适应等比缩放；若宽度小于高度，以给定的宽度为基准，高度自适应等比缩放。 | [100x200](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/100x200) [200x100](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/200x100)
  widthxheight^ | 以指定的大边为基准进行缩略。若宽度大于高度，以给定的宽度为基准，高度自适应等比缩放；若宽度小于高度，以给定的高度为基准，宽度自适应等比缩放。 | [100x200^](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/100x200^) [200x100^](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/200x100^)
  widthxheight! | 缩略图按照给定的宽和高强行缩略，忽略原图宽和高的比例，可能会变形。 | [100x200!](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/100x200!) [200x100!](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/200x100!)
  widthxheight> | 当原图尺寸超出给定的宽度或高度时，按照给定 widthxheight 规格进行缩略。 | [100x200>](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/100x200%3E) [1000x2000>](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/1000x2000%3E)
  widthxheight< | 当原图尺寸低于给定的宽度和高度时，按照给定 widthxheight 规格进行拉伸。 | [100x200<](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/200x100<) [1000x2000<](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/2000x1000<)
  area@ | 缩略图按原始图片高宽比例等比缩放，但缩放后的宽乘高的总分辨率不超过给定的总像素。 | [20000@](http://qiniuphotos.dn.qbox.me/gogopher.jpg?imageMogr/thumbnail/20000@)

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

### 7. 高级图像处理接口（缩略、裁剪、旋转、转化）并持久化存储处理结果

也可以将高级图像处理接口处理过的图片进行云端持久化，即将一个存储在七牛云存储的图片进行缩略、裁剪、旋转和格式转化处理后的缩略图作为一个新文件持久化存储到七牛云存储服务器上，这样就可以供后续直接使用而不用每次都传入参数进行图像处理。

图像处理持久化的接口规格如下：

**请求**

    POST <ImageDownloadURL>/imageMogr
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
