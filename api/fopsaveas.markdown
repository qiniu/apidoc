---
layout: default
title: "云处理结果持久化"
---

七牛云存储的云处理API满足如下规格:

    [GET] url?<fop1>|<fop2>|<fop3>|<fopN>

云处理操作，是七牛服务器通过相应的处理程序计算而得，当遇到耗时相对较长的云处理操作时(比如：视频转码),那么这个云处理请求可能得不到及时的响应，
针对这种耗时较长的云处理操作，我们提供了一个云处理操作`saveas`,来把传入的云处理的结果保存到用户指定的空间内，从而达到云处理结果的持久化保存。
当保存成功后，下一次可直接通过指定的key来访问该资源。

`saveas` 接口的规格如下

    [GET] url?<fop1>|<fop2>|<fopN>|saveas/<encodedEntryURI>/sign/<sign>


**参数**

名称            | 类型   | 必须 | 说明
----------------|--------|------|------------------------------------------------------------------------------
encodedEntryURI | string | 是   | 保存资源的空间和key，`encodedEntryURI = urlsafe_base64_encode("bucket:key")`
sign            | string | 是   | AccessKey:urlsafe_base64_encode(hmac_sha1(SecretKey, "Domain/Key?Fop/Params|saveas/encodedEntryURI"))

## 样例

- 原资源是一个视频文件
  - http://woyao.qiniudn.com/thinking-in-go.mp4
- 将 mp4 视频格式转换为 flv 格式，帧率为 24，使用 x264 进行视频编码 
  - http://woyao.qiniudn.com/thinking-in-go.mp4?avthumb/flv/r/24/vcodec/libx264
- 对上述云处理结果进行持久化保存
  - 保存资源为："woyao:thinking-in-go.flv",那么encodedEntryURI结果为："d295YW86dGhpbmtpbmctaW4tZ28uZmx2"
  - 需要签名的内容是"woyao.qiniudn.com/thinking-in-go.mp4?avthumb/flv/r/28/vcodec/libx264"，urlsafe_base64_encode(hmac_sha1("woyao.qiniudn.com/thinking-in-go.mp4?avthumb/flv/r/28/vcodec/libx264"),结果为："jx5twELFWIwgzID4fXIRC80owsk="
  - 完整的sign为:"Bmja3JzCXdQbvLwIwvFGa9WWJYhRT37WqsRA3dCo:jx5twELFWIwgzID4fXIRC80owsk="
  - 完整的请求URL:"http://woyao.qiniudn.com/thinking-in-go.mp4?avthumb/flv/r/24/vcodec/libx264|saveas/d295YW86dGhpbmtpbmctaW4tZ28uZmx2/sign/Bmja3JzCXdQbvLwIwvFGa9WWJYhRT37WqsRA3dCo:jx5twELFWIwgzID4fXIRC80owsk=
"
- 保存的转码后的资源可通过如下访问
  - http://woyao.qiniudn.com/thinking-in-go.flv


生成saveas请求的完整go代码如下：

func makeSaveasUrl(URL, accessKey string, secretKey []byte, saveBucket, saveKey string) string {

	encodedEntryURI := base64.URLEncoding.EncodeToString([]byte(saveBucket+":"+saveKey))

	URL += "|saveas/" + encodedEntryURI

	h := hmac.New(sha1.New, secretKey)
	//签名内容不包括 scheme
	u, _ := url.Parse(URL)
	io.WriteString(h, u.Host + u.RequestURI())
	d := h.Sum(nil)
	sign := accessKey + ":" + base64.URLEncoding.EncodeToString(d)

	return URL + "/sigin/" + sign
}

## 备注

- `urlsafe_base64_encode()` 函数按照标准的 [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt) 实现，开发者可以参考 [github.com/qiniu](https://github.com/qiniu) 上各SDK的样例代码。
- `AccessKey:EncodedSign` 这里的冒号是字符串，仅作为连接分隔符使用，最终连接组合的 downloadToken 也是一个字符串（String）。
- 当要持久化保存的fop耗时较长时候，saveas请求会返回CDN超时，但是只要保证发送的saveas请求合法，七牛服务器还是会对请求做正确处理。
