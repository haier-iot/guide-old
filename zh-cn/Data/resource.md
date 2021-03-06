# 资源服务 #

## 简介  
>  为Haier U+云平台提供统一的资源上传下载服务。


## 文件上传（2B）  
**接口公共部分**  

- **接入地址：**  `https://resource.haier.net`

- **接口参数** 
应用与uws交互中，应用需要在每个请求Header中传入一些固定的参数；uws的每个响应中也会包含固定的响应码，具体如下：

**输入参数**

|参数名称|类型|位置|必填|说明|
|:------:|:-----:|:-----:|:------:|:------:|
|systemId|String|Header|是|应用ID，40位以内字符,Haier U+ 云平台全局唯一。|
|sign|String|Header|是|对请求进行签名运算产生的签名,签名算法见[附件签名算法示例](#jump2)|
|timestamp|long|Header|是|Unix时间戳，精确到毫秒。|
|Content-Type|String|Header|是|application/json;charset=UTF-8|

​**输出参数**

|参数名称|类型|位置|必填|说明|
|:------:|:-----:|:-----:|:------:|:------:|
|retCode|String|Body|是|返回码（其中00000代表请求成功,其它代表错误，错误码及描述见[附录错误码表](#jump1)）|
|retInfo|String|Body|是|用于调试的返回信息，不支持国际化，也不能直接显示在UI上|

### 文件上传接口  
**使用说明**  
> 上传文件，成功时返回文件的url，失败时返回错误码，错误码参照[附录错误码表](#jump1)

**接口描述**  

- **接入地 址：**  `/rsservice/v1/upload`

- **HTTP Method：** POST

- **输入参数**

|参数名称|类型|位置|必填|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|:------:|
|file|file|Body|是|源文件|目前文件最大限制为10MB|
|fileInfo|String|Body|是|fileInfo组成的json字符串|&emsp;|

**输入参数对象说明**

|**名称**|文件信息对象|&emsp;|&emsp;|fileInfo|
|:------:|:----------:|:-----:|:--------:|:--------:|
|**字段名**|**类型**|**必填**|**说明**|**备注**|
|fileHash|String|是|文件的hash值|原始文件的hash，定长32位具体算法见[附录文件hash值算法](#jump5)|
|fileCreator|String|否|文件创建者|长度限制为：32位，不可使用特殊字符|

- **输出参数**

|参数名称|类型|位置|说明|备注|
|:-------:|:----------:|:-----:|:---------:|:-------:|
|fileHash|String|Body|上传至服务端源文件的hash|下载端可以使用该hash对比，验证文件的完整性|
|fileSize|long|Body|文件的大小|单位：byte(字节)|
|alg|String|Body|服务端计算文件hash的算法|可取值：md5|
|httpUrl|String|Body|文件使用http协议的url|url最大长度:256位|
|httpsUrl|String|Body|文件使用https协议的url|url最大长度:256位|
|retCode|String|Body|请求响应返回码|&emsp;|
|retInfo|String|Body|请求响应返回信息|&emsp;|

**示例**

- **请求样例** 
```
Header:
POST /rsservice/v1/upload HTTP/1.1
systemId: SV-JSZCCSZY966-0000
sign: d21e7dd39a52c979505e8613f683608a517d4869240757a0413c6f6415535420
timestamp: 1626158509
language: zh-cn
Content-Type: multipart/form-data; boundary=
 
Body:
Content-Disposition: form-data; name="fileInfo"
{"expiryTime":77,"fileHash":"202cb962ac59075b964b07152d234b70","fileCreator":"test123","uploadClientId":"test1234"}
Content-Disposition: form-data; name="file"; filename="test.txt"
<test.txt>
```  
>备注：以上body部分只是简单示例，上传文件具体可以参照示例代码


- **请求应答**
```json
{
    "payload":{
        "alg":"md5",
        "fileHash":"202cb962ac59075b964b07152d234b70",
        "fileSize":3,
        "httpUrl":"http://resource.haier.net/rsservice/v1/download/SV-JSZCCSZY966-0000/test1234/resources/9e256de38af34ebc8009438d71cbc8e9",
        "httpsUrl":"https://resource.haier.net/rsservice/v1/download/SV-JSZCCSZY966-0000/test1234/resources/9e256de38af34ebc8009438d71cbc8e9"
    },
    "retCode":"00000",
    "retInfo":"成功"
}
```

### 申请密钥接口

**使用说明**  
> 下载文件需要密钥，因此下载文件之前需要获得密钥。

**接口描述**

- **接入地 址：**  `/rsservice/v1/applySecretKey `

- **HTTP Method：** POST

- **输入参数**

|参数名称|类型|位置|必填|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|:------:|
|fileUrl|String|Body|是|文件的url|&emsp;|
|expiryTime|int|Body|是|秘钥的有效期(单位为分钟)，最大值为：60*24即24小时|&emsp;|

- **输出参数**

|参数名称|类型|位置|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|
|secretKey|String|Body|秘钥（定长8字节）|&emsp;|
|fileHash|String|Body|原始文件的hash（定长32字节）|&emsp;|
|fileSize|long|Body|文件的大小，单位为：byte（字节）|&emsp;|
|alg|String|Body|服务端计算文件hash的算法|&emsp;|
|httpUrl|String|Body|文件使用http协议的url|&emsp;|
|httpsUrl|String|Body|文件使用https协议的url|&emsp;|
|retCode|String|Body|请求响应返回码|&emsp;|
|retInfo|String|Body|请求响应返回信息|&emsp;|

**示例**

- **请求样例**  
```
Header：
systemId: SV-JSZCCSZY966-0000
systemKey: e05d55d2365299f0ad2a806e02b8df8a
sign: 13021f813303a2ab32e0cdcdaac1e8ba70b4bacdf6576f063052ef350c7ab678
timestamp: 1626159991
language: zh-cn
Content-type: application/json;charset=UTF-8
Body:
{
    "fileUrl":"http://resource.haier.net/rsservice/v1/download/systemId/resoureces/85d1e38c753d4109847b036dbb258ce8",
    "expiryTime":60
}
```

- **请求应答**  
```
{
    "payload":{
        "alg":"md5",
        "expiryTime":1626435710000,
        "fileHash":"202cb962ac59075b964b07152d234b70",
        "fileSize":3,
        "httpUrl":"http://resource.haier.net/rsservice/v1/download/SV-JSZCCSZY966-0000/test1234/resources/9e256de38af34ebc8009438d71cbc8e9",
        "httpsUrl":"https://resource.haier.net/rsservice/v1/download/SV-JSZCCSZY966-0000/test1234/resources/9e256de38af34ebc8009438d71cbc8e9",
        "secretKey":"13a8b371"
    },
    "retCode":"00000",
    "retInfo":"成功"
}
```


## 文件上传（2C）

**接口公共部分**  

- **接入地址：**  `https:// resource.haier.net` 

- **接口参数** 
应用与uws交互中，应用需要在每个请求Header中传入一些固定的参数；uws的每个响应中也会包含固定的响应码，具体如下：

**输入参数**

|参数名称|类型|位置|必填|说明|
|:------:|:-----:|:-----:|:------:|:------|
|appId|String|Header|是|应用ID，40位以内字符,Haier U+ 云平台全局唯一。|
|sign|String|Header|是|对请求进行签名运算产生的签名,签名算法见[附录签名算法示例](#jump2)|
|timestamp|long|Header|是|Unix时间戳，精确到毫秒。|
|Content-Type|String|Header|是|application/json;charset=UTF-8|

​**输出参数**

|参数名称|类型|位置|必填|说明|
|:------:|:-----:|:-----:|:------:|:------|
|retCode|String|Body|是|返回码（其中00000代表请求成功,其它代表错误，错误码及描述见[附录错误码表](#jump1)）|
|retInfo|String|Body|是|用于调试的返回信息，不支持国际化，也不能直接显示在UI上|

### 文件上传接口

**使用说明**

> 上传文件，成功时返回文件的url和密钥，失败时返回错误码，错误码参照[附录错误码表](#jump1)

**接口描述**

- **接入地 址：**  `/rsservice/v1/union/upload`

- **HTTP Method：** POST

- **输入参数**

|参数名称|类型|位置|必填|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|:------:|
|file|file|Body|是|源文件|目前文件最大限制为10MB|
|fileInfo|String|Body|是|fileInfo组成的json字符串|&emsp;|

**输入参数fileInfo说明**

|字段名|类型|必填|说明|备注|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|fileHash|String|是|文件的hash值|原始文件的hash，定长32位具体算法见[附录文件hash值算法](#jump5)|
|fileCreator|String|否|文件创建者|长度限制为：32位，不可使用特殊字符|
|uploadClientId|String|是|DeviceId或者UserId|&emsp;|
|expiryTime|int|是|文件的过期时间和秘钥的过期时间|单位：小时，文件有效期和秘钥有效期都为此时间，文件过期则从服务端删除，无法再下载。最大限制为7天，即168小时|

- **输出参数**

|参数名称|类型|位置|说明|备注|
|:-------:|:----------:|:-----:|:---------:|:---------:|
|fileHash|String|Body|上传至服务端源文件的hash|下载端可以使用该hash对比，验证文件的完整性|
|fileSize|long|Body|文件的大小|单位：byte(字节)|
|secretKey|String|Body|秘钥（定长8字节）|下载方下载资源时需要秘钥方可下载|
|expiryTime|long|Body|过期时间|过期时间戳(毫秒),其值为上传文件时的当前时间加上申请时填写的过期时间|
|alg|String|Body|服务端计算文件hash的算法|可取值：md5|
|httpUrl|String|Body|文件使用http协议的url|url最大长度:256位|
|httpsUrl|String|Body|文件使用https协议的url|url最大长度:256位|
|retCode|String|Body|请求响应返回码|&emsp;|
|retInfo|String|Body|请求响应返回信息|&emsp;|

**示例**

- **请求样例**  
```
POST /rsservice/v1/union/upload HTTP/1.1
Header:
appId: MB-SJSZCCSZY-0000
sign: 472e0cf169eda5a1b5b62ec012a230ad28041c5e2084ebeedbcc6b612c9197f3
timestamp: 1626160888
language: zh-cn
Content-Type: multipart/form-data; boundary=
Body:
Content-Disposition: form-data; name="fileInfo"
{"expiryTime":77,"fileHash":"202cb962ac59075b964b07152d234b70","fileCreator":"test123","uploadClientId":"test1234"}
Content-Disposition: form-data; name="file"; filename="test.txt"
<test.txt>
```
>备注：以上body部分只是简单示例，上传文件具体可以参照示例代码

- **请求应答**  
```
{
    "payload":{
        "alg":"md5",
        "expiryTime":1626438088000,
        "fileHash":"202cb962ac59075b964b07152d234b70",
        "fileSize":3,
        "httpUrl":"http://resource.haier.net/rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4",
        "httpsUrl":"https://resource.haier.net/rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4",
        "secretKey":"c1aaf6cd"
    },
    "retCode":"00000",
    "retInfo":"成功"
}
```

## 文件下载接口

**使用说明**
> 上传文件，成功时返回文件的url和密钥，失败时返回错误码，错误码参照[附录错误码表](#jump1)

**接口描述**

- **接入地址：**  `/rsservice/v1/download/{业务参数} `  
> 备注：使用的地址是上传文件返回的url，例如https://resource.haier.net/rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4

- **HTTP Method：** GET

- **输入参数**

|参数名称|类型|位置|必填|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|:------:|
|secretKey|String|header或者url中|是|文件秘钥|定长8字节|
|Range|String|header|否|分包下载|参见[输入参数备注](#memo1)|
|showInBrowser|Boolean|header或者url中|否|是否在浏览器中进行文件预览|该参数的意义为，文件通过浏览器打开时，是否需要在浏览器预览直接显示出来，例如图片，文本，默认值为（false）不显示，弹框下载|&emsp;|

<a id="memo1"> </a>
**输入参数备注**

>1. Range:bytes=128-1024 表示下载128-1024段的文件字节（不包括第1024的字节）。  
>2. Range: bytes=0-表示下载全部文件  
>3. Range: bytes=-2048 表示下载0-2048文件字节  
>4. 不填，默认全部下载.  
>5. 如果Range不在文件长度范围内，则下载全部文件  
>6. 如果请求的文件长度大于文件本身的长度，fileContent会按照文件实际长度返回，会出现请求长度和回复长度不一致的情况，如：请求0-1024长度，但文件本身只有512长度，则实际返回的文件长度是512  
>7. 如果Range参数填写的不符合规则，则返回错误码，如：Range: bytes=1.5-10  

- **输出参数**

|参数名称|类型|位置|说明|备注|
|:------:|:-----:|:-----:|:------:|:------:|
|file|InputStream|Body|源文件|&emsp;|
|alg|String|Header|服务端计算文件hash的算法|&emsp;|
|hash|String|Header|文件的hash值|&emsp;|


**示例**

- **请求样例** 

```
GET /rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4 HTTP/1.1
secretKey: b57b3c7b
User-Agent: PostmanRuntime/7.28.1
Accept: */*
Cache-Control: no-cache
Postman-Token: 92ee9eaa-97e1-419b-aa68-a49172b74693
Host: resource.haier.net
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

- **成功的请求应答**

```
HTTP/1.1 200 OK
Date: Tue, 13 Jul 2021 07:29:41 GMT
Content-Type: application/octet-stream
Content-Length: 3
Connection: keep-alive
Server: Tengine
Accept-Ranges: bytes
Content-Range: bytes 0-3/3
alg: md5
Hash: 202cb962ac59075b964b07152d234b70
Content-Disposition: attachment;filename=test.txt
X-Via: 1.1 PSbjwjBGP2lt148:1 (Cdn Cache Server V2.0), 1.1 fyd92:8 (Cdn Cache Server V2.0)
X-Ws-Request-Id: 60ed40e5_fyd91_3804-53774
 
123
```

- **失败的请求应答**

```
HTTP /1.1 511 
Header:
Content-Type: application/json;charset=UTF-8
Content-Length: 64
Body:{
    "retCode":"B00001",
    "retInfo":"秘钥不存在,请重新申请"
}
```

**备注说明**  

>1. Content-Disposition: attachment;filename=test.txt
此header表示文件直接在浏览器上访问时弹出文件下载对话框，默认的保存文件名为test.txt  
>2. 如果需要在浏览器中进行预览文件，则需要在请求url中加入secretKey和showInBrowser参数。
例如：
https://resource.haier.net/rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4?secretKey=36fb1891&showInBrowser=true  
>3. 当下载文件时，如果服务端错误，HttpCode则会返回511，并且响应体body会返回标准的uws结构体
如果服务端正确，HttpCode则会返回200，响应体返回文件流
>4.示例仅是简单的展示，具体文件下载的实现可以参照示例代码


## 附录

<a id="jump1"> </a>
### 公共错误码

|错误码|描述|
|:----:|:----:|
|B00001|秘钥不存在,请重新申请|
|B00002|文件不存在|
|B00003|文件hash对比错误，文件不完整|
|B00004|参数不符合规则要求|
|B00005|秘钥多于10个,无法继续申请|
|B00006|秘钥与文件不匹配|
|B00007|文件url错误|
|B00008|文件名长度超过512位，不符合规则要求|
|B00009|文件创建者长度超过32位，不符合规则要求|
|B00010|range参数不符合规则要求|
|B00011|秘钥有效期不符合规则|
|B00012|文件大小超过限制|
|B00013|系统错误|
|B00014|文件下载失败，请重试|

<a id="jump2"> </a>
### 签名算法示例

```java
String getSign(String systemId, String systemKey, String timestamp, String body,String url){
    URL urlObj = new URL(url);
    url=urlObj.getPath();
    systemKey = systemKey.trim();
    systemKey = systemKey.replaceAll("\"", "");
    if (body != null) {
        body = body.trim();
    }
    if (!body.equals("")) {
        body = body.replaceAll("", "");
        body = body.replaceAll("\t", "");
        body = body.replaceAll("\r", "");
        body = body.replaceAll("\n", "");
    }
    StringBuffer sb = new StringBuffer();
    sb.append(url).append(body).append(systemId).append(systemKey).append(timestamp);
    log.info("signStr:"+sb);
    MessageDigest md = null;
    byte[] bytes = null;
    try {
        md = MessageDigest.getInstance("SHA-256");
        bytes = md.digest(sb.toString().getBytes("utf-8"));
    } catch (Exception e) {
        e.printStackTrace();
    }

    return BinaryToHexString(bytes);
}

String BinaryToHexString(byte[] bytes) {
    StringBuilder hex = new StringBuilder();
    String hexStr = "0123456789abcdef";
    for (int i = 0; i < bytes.length; i++) {
        hex.append(String.valueOf(hexStr.charAt((bytes[i] & 0xF0) >> 4)));
        hex.append(String.valueOf(hexStr.charAt(bytes[i] & 0x0F)));
    }
    return hex.toString();
}
```

<a id="jump5"> </a>
### 文件hash值算法

```java
//取文件的md5-hash值
import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;
import java.util.Base64;

/**
 * @author: leonardo·li
 * @create: 2020-04-01 13:53
 **/
public class MD5Util{
    private static final char[] hexDigits = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
            'a', 'b', 'c', 'd', 'e', 'f'};

    public static void main(String[] args) throws Exception{
        String filePath = "F:\\img\\20200311.jpeg";
        String md5Hashcode2 = MD5Util2.getFileMD5(new File(filePath));
        System.out.println("MD5Util2计算文件md5值为：" + md5Hashcode2);
        System.out.println("MD5Util2计算文件md5值的长度为：" + md5Hashcode2.length());
    }
    /**
     * Get MD5 of a file (lower case)
     * @return empty string if I/O error when get MD5
     */
    public static String getFileMD5( File file) {
        FileInputStream in = null;
        try {
            in = new FileInputStream(file);
            FileChannel ch = in.getChannel();
            return MD5(ch.map(FileChannel.MapMode.READ_ONLY, 0, file.length()));
        } catch (FileNotFoundException e) {
            return "";
        } catch (IOException e) {
            return "";
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    // 关闭流产生的错误一般都可以忽略
                }
            }
        }

    }
    /**
     * 计算MD5校验
     * @param buffer
     * @return 空串，如果无法获得 MessageDigest实例
     */

    private static String MD5(ByteBuffer buffer) {
        String s = "";
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            md.update(buffer);
            byte tmp[] = md.digest(); // MD5 的计算结果是一个 128 位的长整数，
            // 用字节表示就是 16 个字节
            char str[] = new char[16 * 2]; // 每个字节用 16 进制表示的话，使用两个字符，
            // 所以表示成 16 进制需要 32 个字符
            int k = 0; // 表示转换结果中对应的字符位置
            for (int i = 0; i < 16; i++) { // 从第一个字节开始，对 MD5 的每一个字节
                // 转换成 16 进制字符的转换
                byte byte0 = tmp[i]; // 取第 i 个字节
                str[k++] = hexDigits[byte0 >>> 4 & 0xf]; // 取字节中高 4 位的数字转换, >>>,
                // 逻辑右移，将符号位一起右移
                str[k++] = hexDigits[byte0 & 0xf]; // 取字节中低 4 位的数字转换
            }
            s = new String(str); // 换后的结果转换为字符串

        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return s;
    }
}
```

<a id="jump3"> </a>
### 文件上传示例

```java
public static void sendFile(String url, Map<String, Object> headers, String body, File file) throws Exception {
    HttpClient httpClient = HttpClientBuilder.create().build();
    HttpPost httppost = new HttpPost(url);
    MultipartEntityBuilder builder = MultipartEntityBuilder.create();
    builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
    //第一个参数为 相当于 Form表单提交的file框的name值 第二个参数就是我们要发送文件对象了
    //第三个参数是文件名
    InputStream inputStream = new FileInputStream(file);
    builder.addBinaryBody("file",IOUtils.toByteArray(inputStream),ContentType.create("multipart/form-data"), file.getName());
    // 构建的json串
    builder.addTextBody("fileInfo", body, ContentType.create("multipart/form-data", Consts.UTF_8));
    HttpEntity entity = builder.build();
    if (null != headers && headers.size() > 0) {
        for (Map.Entry<String, Object> entry : headers.entrySet()) {
            httppost.addHeader(entry.getKey(), entry.getValue().toString());
        }
    }
    httppost.setEntity(entity);
    try {
        HttpResponse response = httpClient.execute(httppost);
        if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
            String result = EntityUtils.toString(response.getEntity());
            System.out.println(result);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        if (inputStream!=null){
            inputStream.close();
        }
    }
}
public static void main(String[] args) throws Exception {
    String url = "https://resource.haier.net/rsservice/v1/upload";
    String path = "F:\\img\\20200311.jpeg";
    File uploadFile = new File(path);
    Map<String, Object> header = new HashMap<>();
    Map<String, Object> param = new HashMap<>();
    param.put("fileCreator", "test123");
    param.put("fileHash", GetFileMD5.getFileMD5(path));
    long times = System.currentTimeMillis();
    String sign = SignUtil.getSign("SV-JSZCCSZY966-0000", "e05d55d2365299f0ad2a806e02b8df8a", String.valueOf(times),"", url);//上传文件时获得签名时
    header.put("sign", sign);
    header.put("systemId", "SV-JSZCCSZY966-0000");
    header.put("timestamp", times);
    sendFile(url, header, JSON.toJSONString(param), uploadFile);
}
```

<a id="jump4"> </a>
### 文件下载示例

```java
public void getDownloadTest() {
    int times = 13283 / 1024;
    try {
        for (int i = 0; i <= times ; i++) {
            int start = i*1024;
            int end = (i+1)*1024;
            Map<String, Object> params = new HashMap<>();
            Map<String, Object> header = new HashMap<>();
            String url = "https://resource.haier.net/rsservice/v1/download/MB-SJSZCCSZY-0000/test1234/resources/aaca5d030148413c99763b86762167a4";
            header.put("secretKey", "a6bf288b");
            header.put("Range", "bytes="+start+"-"+end);
            //header.put("Range", "bytes=0-");
            header.put("Content-Type", "application/octet-stream");
            HttpClient httpClient = HttpClientBuilder.create().build();
            URIBuilder uriBuilder = new URIBuilder(url);
            if (null != params && !params.isEmpty()) {
                for (Map.Entry<String, Object> entry : params.entrySet()) {
                    uriBuilder.addParameter(entry.getKey(), (String) entry.getValue());
                }
            }
            URI uri = uriBuilder.build();
            HttpGet httpGet = new HttpGet(uri);
            if (header != null) {
                for (Map.Entry<String, Object> entry : header.entrySet()) {
                    httpGet.setHeader(entry.getKey(), entry.getValue().toString());
                }
            }
            HttpResponse response = httpClient.execute(httpGet);
            if (response.getStatusLine().getStatusCode() == 200) {
                HttpEntity entity = response.getEntity();
                InputStream stream = entity.getContent();
                File file=new File("F://img//test.jpeg");
                OutputStream out=new FileOutputStream(file,true);
                out.write(IOUtils.toByteArray(stream));
                out.flush();
                out.close();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
