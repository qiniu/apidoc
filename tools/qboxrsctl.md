---
title: qboxrsctl 命令行辅助工具使用文档 | 七牛云存储
---

# qboxrsctl 命令行辅助工具使用文档

## 简介

qboxrsctl 是根据七牛云存储API实现的一个简易命令行辅助工具。

qboxrsctl 命令行辅助工具下载地址：[https://github.com/qiniu/devtools/tags](https://github.com/qiniu/devtools/tags)

## 用法

- [登录](#qboxrsctl-login)
- [上传单个文件](#qboxrsctl-put)
- [同步一个目录](#qboxrsctl-sync)
- [发布静态网站](#qboxrsctl-publish)
- [自定义404页面](#qboxrsctl-page404)
- [重新发布一个静态网站](#qboxrsctl-republish)
- [取消发布一个静态网站](#qboxrsctl-unpublish)
- [清空并删除整个资源表](#qboxrsctl-drop)
- [自定义缩略图规格](#qboxrsctl-setstyle)

qboxrsctl 用法可以在命令行直接输入 qboxrsctl 不带参数来获得。

<a name="qboxrsctl-login"></a>

### 登录

    qboxrsctl login <User> <Passwd>

用你的企业帐号登录七牛云存储，登录成功后，才能进行如下命令行操作。

<a name="qboxrsctl-put"></a>

### 上传单个文件

    qboxrsctl put <Table> <Key> <LocalFile>

将本地文件 \<LocalFile\> 上传到表 \<Table\> 中，键名为 \<Key\>

<a name="qboxrsctl-sync"></a>

### 同步一个目录

    qboxrsctl sync <Table> <LocalDir>

将你本地目录 \<LocalDir\> 中的文件同步到表 \<Table\> 中。这样，你本地的 \<LocalDir\>/foo/bar/file 文件会被上传到七牛云存储表 \<Table\> 中，键值为 foo/bar/file。

<a name="qboxrsctl-publish"></a>

### 发布静态网站

    qboxrsctl publish <Domain> <Table>

将你在七牛云存储中的表 \<Table\> 发布到 \<Domain\>，\<Domain\> 需要在 DNS 管理里边 CNAME 到 iovip.qbox.me 。

这样，用户就可以通过 http://\<Domain\>/\<Key\> 来访问表 \<Table\> 中的文件。键值为 foo/bar/file 的文件对应访问 URL 为 http://\<Domain\>/foo/bar/file。
另外，\<Domain\> 可以是一个真实的域名，比如 www.example.com，也可以是七牛云存储的二级路径，比如 iovip.qbox.me/example

例如：执行 qboxrsctl publish cdn.example.com \<Table\> 后，那么键名为 foo/bar/file 的文件可以通过 http://cdn.example.com/foo/bar/file 访问。

**特别注意**

由于众所周知的原因，使用 `qboxrsctl publish` 操作的 `<Domain>` 若是您自己（或您公司）的域名，必须是已经备案过的。您需要将该域名备案信息发送到我们的邮箱 [support@qiniutek.com](mailto:support@qiniutek.com)，我们的工作人员会完成后续处理。

<a name="qboxrsctl-page404"></a>

### 自定义404页面

    qboxrsctl put <Table> errno-404 /path/to/404.html

当您发布静态网站后，若公开的外链资源找不到，即可使用您上传的“自定义404文件”进行替代，自定义404输出需将\<Key\>的值设置为固定的：errno-404

<a name="qboxrsctl-republish"></a>

### 重新发布一个静态网站

    qboxrsctl sync <Table2> <LocalDir>
    qboxrsctl publish <Domain> <Table2>
    qboxrsctl drop -f <Table>

也就是说，我们推荐想将网站同步到新的表 \<Table2\>，然后将域名关联到新的表 \<Table2\> 。

确认一切正常后，可以在一段时间后删除原来的 \<Table\> 。

如果在这段时间内发现有任何问题，可以随时 qboxrsctl publish \<Domain\> \<Table\> 来切换到原来的网站。

<a name="qboxrsctl-unpublish"></a>

### 取消发布一个静态网站

    qboxrsctl unpublish <Domain>

将已经发布过资源的域名取消外链访问。

<a name="qboxrsctl-drop"></a>

### 清空并删除整个资源表

    qboxrsctl drop -f <Table>

将指定的资源表 \<Table\> 清空并删除掉。

<a name="qboxrsctl-setstyle"></a>

### 自定义缩略图规格

参考图像处理接口之[自定义缩略图规格](/v3/api/foimg/#fo-imagePreviewEx)

