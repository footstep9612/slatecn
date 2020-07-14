---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python
  - php
  - java
  - csharp

toc_footers:
  - <a href='https://wiki.aichotels.net.cn/en/'> English </a>
  - <a href='https://wiki.aichotels.net.cn/cn/'> 中文 </a>
  - <a href='https://www.aichotels.com'>Click here to  visit AIC hotels website</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - content_api
  - price_api # update might necessary for some of the chapter in price_api
  - fqa
  - appendix

search: true

code_clipboard: true
---

# 历史版本

| Author     | Version | Date        | Description     |
| ---------- | ------- | ----------- | --------------- |
| Kyle Shang | 1.0.0   | 08 Aug 2017 | 初版 |
| Kyle Shang | 1.0.1   | 02 Oct 2017 | 分离静态信息和动态价���、预定接口 |
| Kyle Shang | 1.1.0   | 20 Nov 2017 | 基于酒店 id 搜索增加早餐信息废弃 search id 的使用增加预定前校验接口 |
| Kyle Shang | 1.1.1   | 02 Jan 2018 | 增加Book Read 功能以及特殊请求 |
| Kyle Shang | 1.2.0   | 11 Jan 2018 | Internal algrithm update |
| Kyle Shang | 1.2.1   | 23 Feb 2018 | 增加人数信息 |
| Kyle Shang | 2.0.0   | 11 Jun 2018 | 增加静态数据接口 |
| Kyle Shang | 2.0.1   | 03 Dec 2018 | 更改目标Endpoint地址 |
| Kyle Shang | 2.1.1   | 01 Feb 2019 | 增加缓存API，变价API |
| Kyle Shang | 3.0.0   | 15 Feb 2019 | 更改目标Endpoint地址 |
| Kyle Shang | 3.0.1   | 21 Feb 2019 | 增加多酒店接口 |
| Justin Zhang | 3.1.0   | 07 Sep 2019 | 更改至线上版本，增加Python脚本。增加Postman环境。优化阅读体验 |
| Johnny Lee | 3.1.1 | 20 Dec 2019 | 更新字段层级展示方式，字段说明，对缺失字段补充说明 |
| Johnny Lee | 3.2.0 | 23 Mar 2020 | Room Availability、PreBook接口增加“国籍”、“儿童年龄”可选参数 |


# 简介

 **AIC Open API** 是基于RESTful标准涉及的接口，用来进行和分销商之间的商务流程技术对接

接口中会提供两种不同的环境：  **Test Environment** 以及**Live Environment**. 每个环境中又有两种类型的数据接口：

- **Content API :** 用来获取酒店，城市等静态信息
- **Rate API :** 用来获取动态价格信息以及下单，取消的业务操作。

**Test**:

| Type   | Endpoint |
| ------ | -------- |
| Content |  https://api-uat.aichotels.net.cn/content/public      |
| Rate | https://api-uat.aichotels.net.cn/rate/public      |

**Live**:

| Type   | Endpoint |
| ------ | -------- |
| Content |  https://api.aichotels.net.cn/content/public      |
| Rate | https://api.aichotels.net.cn/rate/public      |


# 鉴权

> To authorize, use this code:

```python
"""
Below Python script used native Python Module:
hmac, base64, and datetime.
Below Script are only for demostration purpose
Might not suitable for Production Enviroment
"""
import hmac
import base64
from hashlib import sha1
import datetime

class AicHeader():
    def __init__(self, APIClientKey, ClientSecret, bin_Method, bin_reqURL):
        """
        :param APIClientKey: Key provided by Amerilink
        :param ClientSecret: Secret provided by Amerilink
        :param bin_Method:  POST
        :param bin_reqURL:  /rate/public/ping
        """
        self.APIClientKey = APIClientKey
        self.ClientSecret = ClientSecret
        self.bin_Method = bin_Method
        self.bin_reqURL = bin_reqURL
 
    def create_headers(self):
        """
        :return: UTC Time formatting
                Signature Calculation
                Encyrpt use hmac-sha1 then encrept the digested value use base64 and then                 trasfer to string.
        """
      utc_time = datetime.datetime.utcnow()
        header_date = utc_time.strftime("%a,%d %b %Y %H:%M:%S UTC").encode("utf-8")
        strsign = (self.bin_Method + b" " + self.bin_reqURL + b"\n" + header_date)
        hmacsha1= hmac.new(self.ClientSecret, strsign, sha1).digest()
        signature = str(base64.encodebytes(hmacsha1).decode("utf-8").rstrip("\n"))
        headers = {
            'APIClientKey': self.APIClientKey,
            'Date': header_date,
            'APIClientToken': signature,
            'Content-Type': "application/json",
        }
        return headers


if __name__ == '__main__':
    aic = AicHeader("TestClientKey", b"Secret provided by Amerilink", b"POST",
                    b"/rate/public/ping")
```

```php
function enctyption($method, $reqUrl, $date, $sercret)
{
 //String concatenation
 $stringToSign = $method. " ". $reqUrl. "\n". ($date);
 //use sha1 algorithm to generate hash string
 $signature = hash_hmac('sha1', $stringToSign, $sercret, true);
 // base64 encode
 return base64_encode($signature);
}
```

```java
package xxx;
import org.apache.commons.codec.binary.Base64;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
class Encryption
{
 public static String generateSignature(String method, String reqUrl, String date,
String secret)
 {
 StringBuilder sign = new StringBuilder();
 sign.append(method);
 sign.append(" ");
 sign.append(reqUrl);
 sign.append("\n");
 sign.append(date);
 byte[] sha1 = hmac_sha1(sign.toString(), secret);
 String signature;
 try
 {
 signature = new String(Base64.encodeBase64(sha1), "UTF-8");
 return signature;
 }
 catch (UnsupportedEncodingException e)
 {
 return "";
 }
 }
private static byte[] hmac_sha1(String value, String key)
 {
 try
 {
 byte[] keyBytes = key.getBytes();
 SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA1");
 Mac mac = Mac.getInstance("HmacSHA1");
 mac.init(signingKey);
 return mac.doFinal(value.getBytes());
 }
 catch (Exception e)
 {
 return null;
 }
 }
}
```

```csharp
using System;
using System.Text;
using System.Security.Cryptography;
using System.IO;
class AuthValidatorUtil
{
 public string generateSignature(string method, string reqUrl, string date, string
secret)
 {
 StringBuilder sign = new StringBuilder();
 sign.Append(method);
 sign.Append(" ");
 sign.Append(reqUrl);
 sign.Append("\n");
 sign.Append(date);
 byte[] authorization = getSignature(Encoding.UTF8.GetBytes(sign.ToString()),
Encoding.UTF8.GetBytes(secret));
 return Convert.ToBase64String(authorization);
 }
 public static byte[] getSignature(byte[] data, byte[] key)
 {
 HMACSHA1 hmac = new HMACSHA1(key);
 CryptoStream cs = new CryptoStream(Stream.Null, hmac, CryptoStreamMode.Write);
 cs.Write(data, 0, data.Length);
 cs.Close();
 return hmac.Hash;
 }
}
```

>HTTP Header Sample

```json
{
    "APIClientKey":"TestClientKey",
    "Date":"Thu,11 Jan 2018 03:55:10 UTC",
    "APIClientToken":"uh3ap5Yq1iBId49o/3KdyTdzoDs=",
    "content_type":"application/json"
}
```

所有鉴权信息都需要放在**HTTP Header**中，总共有三个字段需要传输：*APIClientKey*, *Date*以及 *APIClientToken*

因为所有涉及到的请求日志皆为JSON格式，则 `Content-Type `衡为 `application/json`



<aside class="warning">请注意<code>Date</code>中的值为基于<code>UTC</code>时间拼接而成</aside>
| HeaderName   | Description           |
| ------------ | --------------------- |
| APIClientKey | 由美联国际颁发的调用方的唯一标识。 |
|Date| 基于 UTC 时区的日期时间。格式需要遵循 "D,d M Y H:i:s T"。例子: Tue, 02 Dec 2017 16:45:12 UTC |
|APIClientToken| 美联国际会分配密钥给 API 对接方。对接方基于以下规则生成认证token：（更多代码示例参考附录） |

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>


# Ping

`POST https://{endpoint}/rate/public/ping`

简单易懂的鉴权验证方法。

## Request

> Request Message

```json
{
"message" : "Hello World"
}
```
| Parameter | Count | Type   | Description                   |
| --------- | ----- | ------ | ----------------------------- |
| message   | 1     | String | 任意请求信息 i.e. Hello World |

## Response

> Response Message

```json
{
 "result": {
 "return_status": {
        "success": "true",
        "exception": ""
 }
 },
 "message": "Hello World"
}
```

| Parameter | Count | Type   | Description |
| --------- | ----- | ------ | ----------- |
| result    | 1     | Object |             |
| result.return_status | 1| Object|             |
| result.success | 1| Boolean| 请求状态 |
| result.exception | 1| Boolean|请求失败时的失败描述|
| result.message | 1| Boolean| 请求中的测试信息 |

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

