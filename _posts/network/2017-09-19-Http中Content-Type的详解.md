---
layout: post
title: "Http中Content-Type的详解"
description: Http中Content-Type的详解
category: 网络
---

# Content-Type 类型

## application/x-www-form-urlencoded

数据被编码为名称/值对。这是标准的编码格式,浏览器的原生 form 表单，如果不设置 enctype 属性，
那么最终就会以 `application/x-www-form-urlencoded` 方式提交数据。

首先，Content-Type 被指定为 `application/x-www-form-urlencoded`；其次，提交的数据按照 `key1=val1&key2=val2` 的方式进行编码，
key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。例如 PHP 中，`$_POST[‘title’]` 可以获取到 title 的值，`$_POST[‘sub’]` 可以得到 sub 数组。
很多时候，我们用 Ajax 提交数据时，也是使用这种方式。例如 JQuery 和 QWrap 的 Ajax，
Content-Type 默认值都是「`application/x-www-form-urlencoded;charset=utf-8`」。

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

## multipart/form-data

数据被编码为一条消息，页上的每个控件对应消息中的一个部分

这个例子稍微复杂点。首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。
然后 Content-Type 里指明了数据是以 `mutipart/form-data` 来编码，本次请求的 boundary 是什么内容。
消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 –boundary 开始，
紧接着内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，
还要包含文件名和文件类型信息。消息主体最后以 –boundary– 标示结束看。

这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

上面提到的这两种 POST 数据的方式，都是浏览器原生支持的，而且现阶段原生 form 表单也只支持这两种方式。
但是随着越来越多的 Web 站点，尤其是 WebApp，全部使用 Ajax 进行数据交互之后，我们完全可以定义新的数据提交方式，给开发带来更多便利。

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

## text/plain

数据以纯文本形式(text/json/xml/javascript/html)进行编码，其中不含任何控件或格式字符。
postman软件里标的是RAW

![image](https://jasperbalcony.github.io/images/network/raw.jpg)

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
    public static String getBodyString(ServletRequest request) {
        StringBuilder sb = new StringBuilder();
        InputStream inputStream = null;
        BufferedReader reader = null;
        try {
            inputStream = request.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("UTF-8")));
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (IOException e) {
            logger.error("getBodyString error", e.getMessage());
        } finally {
            IOUtils.close(inputStream);
            IOUtils.close(reader);
        }
        return sb.toString();
    }
```

通过这种简单的解析方式，将http请求中的body解析成字符串的形式；然后再对字符串的格式进行业务解析；
这种处理方式的优点是：能解析 出content-Type 为 multipart/form-data、(text/json/xml/javascript/html) 

但是这种方式有一个**很大的缺点**：那就是当从请求中获取流以后，流被filter中的这个
inputStreamToString(InputStream in) 这个方法处理后就被“消耗”了，
这会导致，chain.doFilter(request, res)这个链在传递 request对象的时候，里面的请求流为空，导致责任链模式下，
其他下游的链无法获取请求的body,从而导致程序无法正常运行，这也使得我们的这个filter虽然可以获取请求信息，
但是它会导致整个应用程序不可用，那么它也就失去了意义；


解决思路如下：将取出来的字符串，再次转换成流，然后把它放入到新request对象中，
在chain.doFiler方法中 传递新的request对象；要实现这种思路，需要自定义一个类

涉及的三个类的代码如下：

- 自定义的HttpServletRequestWrapper

```java
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.Charset;
import java.util.Enumeration;


public class BodyReaderHttpServletRequestWrapper extends HttpServletRequestWrapper {

    private final byte[] body;

    public BodyReaderHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
        body = HttpHelper.getBodyString(request).getBytes(Charset.forName("UTF-8"));
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {

        final ByteArrayInputStream bais = new ByteArrayInputStream(body);

        return new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return bais.read();
            }
        };
    }

    @Override
    public String getHeader(String name) {
        return super.getHeader(name);
    }

    @Override
    public Enumeration<String> getHeaderNames() {
        return super.getHeaderNames();
    }

    @Override
    public Enumeration<String> getHeaders(String name) {
        return super.getHeaders(name);
    }

}
```

- 辅助类HttpHelper

```java
import com.alibaba.fastjson.util.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.ServletRequest;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.Charset;


public class HttpHelper {

    private static final Logger logger = LoggerFactory.getLogger(HttpHelper.class);

    /**
     * 获取请求Body
     *
     * @param request
     * @return
     */
    public static String getBodyString(ServletRequest request) {
        StringBuilder sb = new StringBuilder();
        InputStream inputStream = null;
        BufferedReader reader = null;
        try {
            inputStream = request.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("UTF-8")));
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (IOException e) {
            logger.warn("getBodyString error", e.getMessage());
        } finally {
            IOUtils.close(inputStream);
            IOUtils.close(reader);
        }
        return sb.toString();
    }
}
```

- 过滤器

```java

public class RequestFilter implements Filter {

    @Override
    public void init(FilterConfig config) throws ServletException {
    }


   @Override
   public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
       HttpServletRequest request = (HttpServletRequest) req;
       //获取标准化param
       Map<String, String> paramMap = getRequestParams(request);
       String reqMethod = request.getMethod();
       if ("POST".equals(reqMethod)) {
           //POST 获取body
           ServletRequest requestWrapper = new BodyReaderHttpServletRequestWrapper(request);
           String body = HttpHelper.getBodyString(requestWrapper);
           chain.doFilter(requestWrapper, resp);
       } else {
           chain.doFilter(req, resp);
       }
   }

    @Override
    public void destroy() {
    }

    private Map<String, String> getRequestParams(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap();
        Enumeration<String> params = request.getParameterNames();
        while (params.hasMoreElements()) {
            String paramName = params.nextElement();
            String paramValue = request.getParameter(paramName);
            paramMap.put(paramName, paramValue);
        }
        return paramMap;
    }
}
```


# 参考

[解决在Filter中读取Request中的流后，后续controller或restful接口中无法获取流的问题](https://blog.csdn.net/pyxly1314/article/details/51802652)