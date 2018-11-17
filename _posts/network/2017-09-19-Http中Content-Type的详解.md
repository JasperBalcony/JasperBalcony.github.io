---
layout: post
title: "Http中Content-Type的详解"
description: Http中Content-Type的详解
category: 网络
---

# application/x-www-form-urlencoded

数据被编码为名称/值对。这是标准的编码格式

数据包
```
POST http://test.com/u1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
cache-control: no-cache
Postman-Token: 331a3936-f70c-4442-ad41-5f41b5667de3
User-Agent: PostmanRuntime/7.4.0
Accept: */*
Host: test.com
cookie: lang=en_US; _qcc=1520495289853
accept-encoding: gzip, deflate
content-length: 11
Connection: keep-alive

k1=v1&k2=v2
```

java中获取此请求参数方式：
```java
 HttpServletRequest request= (HttpServletRequest) req;
 String param = request.getParameter("param");
```

# multipart/form-data

数据被编码为一条消息，页上的每个控件对应消息中的一个部分

数据包
```
POST http://host.com/upload HTTP/1.1
Host: host.com
Connection: keep-alive
Content-Length: 200438
Cache-Control: max-age=0
Origin: http://host.com
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryACGGvNAyOkpm86Oq
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: lang=zh_CN; _uid=118158840; _keyLogin=31b6c1a416bf3751c28fb0446ed213; _rmtk=fbf543df080aa31d51a5c3d027ebe4; JSESSIONID=m1161jtytftqen11pzr5andisfb7j.m116

------WebKitFormBoundaryACGGvNAyOkpm86Oq
Content-Disposition: form-data; name="appId"

default
------WebKitFormBoundaryACGGvNAyOkpm86Oq
Content-Disposition: form-data; name="captchaType"

1
------WebKitFormBoundaryACGGvNAyOkpm86Oq
Content-Disposition: form-data; name="imgFile"; filename="v22.jpg"
Content-Type: image/jpeg

�����JFIF��H�H�����C�
#*%,+)%((.4B8.1?2((:N:?DGJKJ-7QWQHVBIJG���C
```

```java
    @RequestMapping(value = "/upload")
    public Object upload(@RequestParam(value = "imgFile") MultipartFile iconFile,
                         @RequestParam String appId,
                         @RequestParam int captchaType) {
    
  }
```

数据包
```
POST http://test.com/u1 HTTP/1.1
Content-Type: multipart/form-data; boundary=--------------------------010925396901756808874064
cache-control: no-cache
Postman-Token: 3b612f33-f5b0-4238-80a1-75701152650d
User-Agent: PostmanRuntime/7.4.0
Accept: */*
Host: test.com
cookie: lang=en_US; _qcc=1520495289853
accept-encoding: gzip, deflate
content-length: 262
Connection: keep-alive

----------------------------010925396901756808874064
Content-Disposition: form-data; name="k1"

v1
----------------------------010925396901756808874064
Content-Disposition: form-data; name="k2"

v2
----------------------------010925396901756808874064--
```

java中获取此请求参数方式：
```java
private CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();

public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest) request;

    String contentType = req.getContentType();//获取请求的content-type
    if (contentType.contains("multipart/form-data")) {//文件上传请求 *特殊请求
        /*
　　　　　　　CommonsMultipartResolver 是spring框架中自带的类，使用multipartResolver.resolveMultipart(final HttpServletRequest request)方法可以将request转化为MultipartHttpServletRequest
　　　　　　　使用MultipartHttpServletRequest对象可以使用getParameter(key)获取对应属性的值
　　　　　　*/
        MultipartHttpServletRequest multiReq = multipartResolver.resolveMultipart(req);
        request = multiReq;//将转化后的reuqest赋值到过滤链中的参数 *重要
    }
    chain.doFilter(request, response);
}
```

# text/plain

数据以纯文本形式(text/json/xml/html)进行编码，其中不含任何控件或格式字符。
postman软件里标的是RAW

数据包
```
POST http://test.com/u1 HTTP/1.1
cache-control: no-cache
Postman-Token: 42eb8d57-db98-4938-b96f-2d62770379c7
Content-Type: text/plain
User-Agent: PostmanRuntime/7.4.0
Accept: */*
Host: test.com
cookie: lang=en_US; _qcc=1520495289853
accept-encoding: gzip, deflate
content-length: 195
Connection: keep-alive

this is data content
```

java中获取此请求参数方式：
```java
BufferedReader reader = request.getReader();
char[] buf = new char[512];
int len;
StringBuffer contentBuffer = new StringBuffer();
while ((len = reader.read(buf)) != -1) {
    contentBuffer.append(buf, 0, len);
}
String content = contentBuffer.toString();
```

# links

[web过滤器中获取请求的参数](https://www.cnblogs.com/springlight/p/6208908.html)