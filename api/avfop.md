---
title: 音频 / 视频处理接口 | 七牛云存储
---

# 音频 / 视频处理接口


## 目录

- [音频转换](#audio-convert)
- [视频转换](#video-convert)
- [视频帧缩略图](#video-thumbnail)


## 说明

要使用七牛云存储提供的在线音/视频处理服务，只需在音/视频的下载URL上对接本套协议规定的音视频处理接口即可进行音视频处理。

关于获取音/视频文件的下载链接，可以参考：[七牛云存储接口之下载文件](/v3/api/io/#download) 。


## 协议

<a name="audio-convert"></a>

### 1. 音频转换

**请求**

    [GET] <AudioDownloadURL>?avthumb/<Format> \
                             /ab/<BitRate> \
                             /aq/<AudioQuality> \
                             /ar/<SamplingRate>

注意：反斜杠（\）因排版换行需要，实际情况下请忽略。

**响应**

    HTTP/1.1 200 OK
    Body: <AudioBinaryData>

<a name="audio-convert-args"></a>

**请求参数详解**

`<Format>`
: 要转换输出的目标音频格式，比如 mp3，aac，m4a 等。 详见：[支持的音频格式](http://ffmpeg.org/general.html#Audio-Codecs)

`<BitRate>`
: 静态码率（CBR），单位：比特每秒（bit/s），常用码率：320k, 256k, 192k, 128k, 64k 等。

`<AudioQuality>`
: 动态码率（VBR），取值范围为 0 ~ 9，值越小，码率越高，不得和上面的静态码率`<BitRate>`(CBR) 参数合用。

`<SamplingRate>`
: 音频采样频率，单位：赫兹（Hz），常用采样频率：44100，22050，12050，8000等。

**示例**

示例1：将 wav 音频格式转换为 mp3 格式：
    
    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3

示例2：将 wav 音频格式转换为 mp3 格式，并指定比特率为 192k：
    
    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3/ab/192k

示例3：将 wav 音频格式转换为 mp3 格式，并指定 VBR 参数为3，采样频率为44100：
    
    [GET] http://apitest.qiniudn.com/sample.wav?avthumb/mp3/ar/44100/aq/3

<a name="video-convert"></a>

### 2. 视频转换

**请求**

    [GET] <VideoDownloadURL>?avthumb/<Format> \
                             /r/<FrameRate> \
                             /vb/<VideoBitRate> \
                             /vcodec/<VideoCodec> \
                             /acodec/<AudioCodec> \
                             /ab/<BitRate> \
                             /aq/<AudioQuality> \
                             /ar/<SamplingRate>

注意：反斜杠（\）因排版换行需要，实际情况下请忽略。

**响应**

    HTTP/1.1 200 OK
    Body: <VideoBinaryData>
    
**请求参数详解**

`<Format>`
: 要转换输出的目标视频格式，比如 avi，flv，mp4 等。

`<FrameRate>`
: 视频帧率，每秒显示的帧数，单位：赫兹（Hz），常用帧率：24，25，30 等，一般用默认值。

`<VideoBitRate>`
: 视频比特率，单位：比特每秒（bit/s），常用视频比特率：128k 1.25m 5m 等。

`<VideoCodec>`
: 视频编码方案，支持方案：libx264，libvpx，libtheora，libxvid。

`<AudioCodec>`
: 音频编码方案，支持方案：libmp3lame，libfaac，libvorbis。

其余参数参见：[音频转换参数详解](#audio-convert-args) 。

**示例**

注意：反斜杠（\）因排版换行需要，实际情况下请忽略。

示例1：将 mp4 视频格式转换为 flv 格式，帧率为 24，使用 x264 进行视频编码：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/flv \
                                                     /r/24 \
                                                     /vcodec/libx264

示例2：将 mp4 视频格式转换为 avi 格式，使用 mp3 进行音频编码，且音频比特率为64k：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/avi \
                                                     /ab/64k \
                                                     /acodec/libmp3lame

示例3：将 mp4 视频格式转换为 flv 格式，帧率 30，视频比特率 256k，使用 x264 进行视频编码，音频采样频率 22050，音频比特率 64k，使用 mp3 进行音频编码：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?avthumb/flv \
                                                     /r/30 \
                                                     /vb/256k \
                                                     /vcodec/libx264 \
                                                     /ar/22050 \
                                                     /ab/64k \
                                                     /acodec/libmp3lame

**支持的格式**

支持转换的视频格式详见：<http://ffmpeg.org/general.html#File-Formats>

支持的视频 Codec 有：libx264，libvpx，libtheora，libxvid 。

支持的音频 Codec 有：libmp3lame，libfaac，libvorbis 。


<a name="video-thumbnail"></a>

### 3. 视频帧缩略图

**请求**

    GET <VideoDownloadURL>?vframe/<Format> \
                           /offset/<Second> \
                           /w/<Width> \
                           /h/<Height>

注意：反斜杠（\）因排版换行需要，实际情况下请忽略。

**响应**

    HTTP/1.1 200 OK
    Body: <ImageBinaryData>
    
**请求参数详解**

`<Format>`
: 要输出的目标缩略图格式，支持 jpg，png 。

`<Second>`
: 取视频的第几秒。

`<Width>`
: 缩略图宽度，范围为 1 ~ 1920。

`<Height>`
: 缩略图高度，范围为 1 ~ 1080。

**示例**

示例：取视频第 7 秒的截图，图片格式为 jpg，宽度为 480px，高度为 360px：

    [GET] http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg \
                                                     /offset/7 \
                                                     /w/480 \
                                                     /h/360

上述示例效果如下：

[![Go——基于连接与组合的语言](http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360)](http://open.qiniudn.com/thinking-in-go.mp4?vframe/jpg/offset/7/w/480/h/360)