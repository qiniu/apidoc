---
title: 应用接入与认证授权
---

# 应用接入与认证授权

## 目录

1. [应用接入](#app-access)
2. [认证授权](#app-auth)
    - [请求签名](#req-signature)
        - [流程](#workflow)
        - [示例](#examples)
            - [PHP 数字签名示例程序](#php-example)
            - [Ruby 数字签名示例程序](#ruby-example)
    - [请求认证](#req-auth)

<a name="app-access"></a>

## 应用接入

要对接七牛云存储服务，您需要七牛云存储服务端颁发给您的 `ACCESS_KEY` 和 `SECRET_KEY`。`ACCESS_KEY` 用于标识客户方的身份，在网络请求中会以某种形式进行传输。`SECRET_KEY` 作为私钥形式存放于客户方本地并不在网络中传递，`SECRET_KEY` 的作用是对于客户方发起的具体请求进行数字签名，用以保证该请求是来自指定的客户方并且请求本身是合法有效的。使用 `ACCESS_KEY` 进行身份识别，加上 `SECRET_KEY` 进行数字签名，即可完成应用接入与认证授权。

您可以通过如下步骤获得 `ACCESS_KEY` 和 `SECRET_KEY`：

1. [开通七牛开发者帐号](https://portal.qiniu.com/)
2. [登录七牛开发者自助平台，查看 ACCESS_KEY 和 SECRET_KEY](https://portal.qiniu.com/setting/key)

在获取到 `ACCESS_KEY` 和 `SECRET_KEY` 之后，您可以参照接下来要介绍的 [认证授权](#auth) 内容进行签名运算。

<a name="app-auth"></a>

## 认证授权

<a name="req-signature"></a>

### 请求签名

<a name="workflow"></a>

#### 流程

认证授权主要是对客户方的请求进行数字签名，签名运算分为如下几个步骤：

1. 分解请求的URL，从中分离得出 `path` 和 `query_string` 的内容；

2. 用字符串拼接的方式组装 `path` 和 `query_string`，两者之间以 `?` 连接；

3. 在以上连接后的字符串末尾添加一个换行符 `\n`；

4. 如果存在 form 表单形式的参数，将这些参数序列化为 `query_string` 形式的字符串；

5. 在原来已拼接的字符串基础上再添加表单参数序列化后的字符串；

6. 用 hmac/sha1 进行签名，其中参数一 `SECRET_KEY` 为私钥，参数二是已拼接的字符串为要签名的数据本身；

7. 对签名后得到的 `digest` 进行URL安全形式的base64编码；

8. 用 `access_key` 明文与编码后的 `digest` 进行拼接，中间使用冒号 `:` 连接，得到一个总的 `access_token`；

<a name="examples"></a>

#### 示例

<a name="php-example"></a>

##### PHP 数字签名示例程序

以 PHP 脚本为例，如下代码描述了如何使用 `ACCESS_KEY` 和 `SECRET_KEY` 对一个请求进行数字签名：

    <?php

    /**
     * urlsafe_base64_encode
     *
     * @desc URL安全形式的base64编码
     * @param string $str
     * @return string
     */
    function urlsafe_base64_encode($str){
        $find = array("+","/");
        $replace = array("-", "_");
        return str_replace($find, $replace, base64_encode($str));
    }

    /**
     * generate_access_token
     *
     * @desc 签名运算
     * @param string $access_key
     * @param string $secret_key
     * @param string $url
     * @param array  $params
     * @return string
     */
    function generate_access_token($access_key, $secret_key, $url, $params){
        $parsed_url = parse_url($url);
        $path = $parsed_url['path'];
        $access = $path;
        if (isset($parsed_url['query'])) {
            $access .= "?" . $parsed_url['query'];
        }
        $access .= "\n";
        if($params){
            if (is_array($params)){
                $params = http_build_query($params);
            }
            $access .= $params;
        }
        $digest = hash_hmac('sha1', $access, $secret_key, true);
        return $access_key.':'.urlsafe_base64_encode($digest);
    }

    /**
     * 测试
     */
    $access_key = '<APPLY_YOUR_ACCESS_KEY_HERE>';
    $secret_key = '<APPLY_YOUR_SECRET_KEY_HERE>';
    $url = 'http://iovip.qbox.me/put-auth/';
    $params = array('a' => 'test');
    $access_token = generate_access_token($access_key, $secret_key, $url, $params);
    var_dump($access_token);


<a name="ruby-example"></a>

##### Ruby 数字签名示例程序

以 Ruby 脚本为例，如下代码描述了如何使用 `ACCESS_KEY` 和 `SECRET_KEY` 对一个请求进行数字签名（依赖 gem `ruby-hmac`）：

    require 'rubygems'
    require 'hmac-sha1'
    require 'base64'
    require 'uri'
    require 'cgi'

    def urlsafe_base64_encode content
        Base64.encode64(content).strip.gsub('+', '-').gsub('/','_').gsub(/\r?\n/, '')
    end

    def generate_access_token(access_key, secret_key, url, params)
        uri = URI.parse(url)
        access = uri.path
        query_string = uri.query
        access += '?' + query_string if !query_string.nil? && !query_string.empty?
        access += "\n";
        if params.is_a?(Hash)
            total_param = params.map { |key, value| %Q(#{CGI.escape(key.to_s)}=#{CGI.escape(value.to_s).gsub('+', '%20')}) }
            access += total_param.join("&")
        end
        hmac = HMAC::SHA1.new(secret_key)
        hmac.update(access)
        encoded_digest = urlsafe_base64_encode(hmac.digest)
        %Q(#{access_key}:#{encoded_digest})
    end

    access_key = '<APPLY_YOUR_ACCESS_KEY_HERE>'
    secret_key = '<APPLY_YOUR_SECRET_KEY_HERE>'
    url = 'http://iovip.qbox.me/put-auth/'
    params = {'a' => 'test'}
    access_token = generate_access_token(access_key, secret_key, url, params)
    puts access_token


<a name="req-auth"></a>

### 请求认证

将以上示例代码中签名运算的最终结果（`access_token`）附加到所在请求的 HTTP Headers 中即可，如下示例子，在 HTTP Headers 中新增一个名为 Authorization 的字段，并将 `QBox access_token` 作为该字段的值：

    Authorization: QBox 3fPHl_SLkPXdioqI_A8_NGngPWVJhlDk2ktRjogH:6q3ojKAnANibjfQzUOxlYFXvGRk=

如此，请求七牛其他资源服务比如云存储或图像处理服务就无需显示地在URL中传递 `access_token` 了。
