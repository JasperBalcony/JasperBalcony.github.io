---
layout: post
title: "关于ESAPI获取资源文件问题"
description: 关于ESAPI获取资源文件问题
category: 个人随笔
---


近期项目中需要使用到组件包ESAPI（ESAPI是owasp提供的一套API级别的web应用解决方案），其官方网站为：https://www.owasp.org/， 
有兴趣的小伙伴可以了解一下。此处不是本文重点，本文重点记录一下使用此组件时遇到的资源加载问题。

引入jar
```cmd
<!-- https://mvnrepository.com/artifact/org.owasp.esapi/esapi -->
<dependency>
    <groupId>org.owasp.esapi</groupId>
    <artifactId>esapi</artifactId>
    <version>2.1.0.1</version>
</dependency>
```


# 问题

在使用此API时，需要引入三个资源文件,分别是`ESAPI.properties` 、`validation.properties`、`antisamy-esapi.xml`

so easy，直接放入项目resources目录中的esapi下，单元测试跑起来

```java
    @Test
    public void esapiTest() {
        String input = "<p>test <b>this</b> <script>alert(document.cookie)</script><i>right</i> now</p>";
        try {
            String safe2 = ESAPI.validator().getValidSafeHTML("get safe html", input, Integer.MAX_VALUE, true);
            logger.info("getValidSafeHTML:{}", safe2);
        } catch (ValidationException e) {
            logger.error("error", e);
        }
    }
```
运行结果
```
System property [org.owasp.esapi.opsteam] is not set
Attempting to load ESAPI.properties via file I/O.
Attempting to load ESAPI.properties as resource file via file I/O.
Not found in 'org.owasp.esapi.resources' directory or file not readable: E:\intelliGit\Tiffany\XSS\ESAPI.properties
Found in SystemResource Directory/resourceDirectory: E:\intelliGit\Tiffany\XSS\target\classes\esapi\ESAPI.properties
Loaded 'ESAPI.properties' properties file
SecurityConfiguration for Validator.ConfigurationFile.MultiValued not found in ESAPI.properties. Using default: false
Attempting to load validation.properties via file I/O.
Attempting to load validation.properties as resource file via file I/O.
Not found in 'org.owasp.esapi.resources' directory or file not readable: E:\intelliGit\Tiffany\XSS\validation.properties
Found in SystemResource Directory/resourceDirectory: E:\intelliGit\Tiffany\XSS\target\classes\esapi\validation.properties
Loaded 'validation.properties' properties file
System property [org.owasp.esapi.devteam] is not set
Attempting to load antisamy-esapi.xml as resource file via file I/O.
Not found in 'org.owasp.esapi.resources' directory or file not readable: E:\intelliGit\Tiffany\XSS\antisamy-esapi.xml
Found in SystemResource Directory/resourceDirectory: E:\intelliGit\Tiffany\XSS\target\classes\esapi\antisamy-esapi.xml
十一月 08, 2018 10:39:59 上午 org.owasp.esapi.reference.JavaLogFactory$JavaLogger log

2018-11-08 10:39:59 [main] INFO com.tiffany.xss.EsapiTest  - getValidSafeHTML:<p>test <b>this</b> <i>right</i> now</p>

```
ok, 大功告成，引入web项目，jetty启动，访问结果

```
Not found in 'org.owasp.esapi.resources' directory or file not readable: 
Not found in SystemResource Directory/resourceDirectory: .esapi\ESAPI.properties
Not found in 'user.home' (C:\Users\wangxinguo) directory: C:\Users\wangxinguo\esapi\ESAPI.properties

Not found in 'org.owasp.esapi.resources' directory or file not readable:
Not found in SystemResource Directory/resourceDirectory: .esapi\validation.properties
Not found in 'user.home' (C:\Users\wangxinguo) directory: C:\Users\wangxinguo\esapi\validation.properties

Not found in 'org.owasp.esapi.resources' directory or file not readable:
Not found in SystemResource Directory/resourceDirectory: .esapi\antisamy-esapi.xml
Not found in 'user.home' (C:\Users\wangxinguo) directory: C:\Users\wangxinguo\esapi\antisamy-esapi.xml


Caused by: org.owasp.esapi.errors.ConfigurationException: Couldn't find antisamy-esapi.xml
	at org.owasp.esapi.reference.validation.HTMLValidationRule.<clinit>(HTMLValidationRule.java:55)
	... 33 more
Caused by: java.io.FileNotFoundException
	at org.owasp.esapi.reference.DefaultSecurityConfiguration.getResourceStream(DefaultSecurityConfiguration.java:532)
	at org.owasp.esapi.reference.validation.HTMLValidationRule.<clinit>(HTMLValidationRule.java:53)
	... 33 more
```
什么鬼？资源文件找不到？？不是已经放在classpath中了么？

- 难道没打包？

  直接查看打包war文件，
`/webapp/WEB-INF/classes/esapi`三个文件的确存在呀，再说单元测试也可以通过呀，这不开玩笑么？

- 难道jetty容器解压有毒？（不合理的怀疑）
  
  直接上服务器jetty的work目录中，`/webapp/WEB-INF/classes/esapi`三个文件也的确存在
     
没办法，再看异常日志
```
Not found in 'org.owasp.esapi.resources' directory or file not readable
```
看到可以通过指定jvm启动环境变量来指定资源文件的路径
```
-Dorg.owasp.esapi.resources={path}
```
重新启动，发现文件解决

但是这个不是我想要的结果，我是想让程序直接加载我classpath路径下的资源文件，
这样不用在容器启动的时候还要强制声明resource path

那就直接看源码呗，直接debug走一波
`org.owasp.esapi.reference.DefaultSecurityConfiguration` 中`getResourceFile`方法

```java

public File getResourceFile(String filename) {
		logSpecial("Attempting to load " + filename + " as resource file via file I/O.");

		if (filename == null) {
			logSpecial("Failed to load properties via FileIO. Filename is null.");
			return null; // not found.
		}

		File f = null;

		// first, allow command line overrides. -Dorg.owasp.esapi.resources
		// directory
		f = new File(customDirectory, filename);
		if (customDirectory != null && f.canRead()) {
			logSpecial("Found in 'org.owasp.esapi.resources' directory: " + f.getAbsolutePath());
			return f;
		} else {
			logSpecial("Not found in 'org.owasp.esapi.resources' directory or file not readable: " + f.getAbsolutePath());
		}

        // if not found, then try the programmatically set resource directory
        // (this defaults to SystemResource directory/resourceFile
        URL fileUrl = ClassLoader.getSystemResource(resourceDirectory + "/" + filename);
        if ( fileUrl == null ) {
            fileUrl = ClassLoader.getSystemResource("esapi/" + filename);
        }

		if (fileUrl != null) {
			String fileLocation = fileUrl.getFile();
			f = new File(fileLocation);
			if (f.exists()) {
				logSpecial("Found in SystemResource Directory/resourceDirectory: " + f.getAbsolutePath());
				return f;
			} else {
				logSpecial("Not found in SystemResource Directory/resourceDirectory (this should never happen): " + f.getAbsolutePath());
			}
		} else {
			logSpecial("Not found in SystemResource Directory/resourceDirectory: " + resourceDirectory + File.separator + filename);
		}

		// If not found, then try immediately under user's home directory first in
		//		userHome + "/.esapi"		and secondly under
		//		userHome + "/esapi"
		// We look in that order because of backward compatibility issues.
		String homeDir = userHome;
		if ( homeDir == null ) {
			homeDir = "";	// Without this,	homeDir + "/.esapi"	would produce
							// the string		"null/.esapi"		which surely is not intended.
		}
		// First look under ".esapi" (for reasons of backward compatibility).
		f = new File(homeDir + "/.esapi", filename);
		if ( f.canRead() ) {
			logSpecial("[Compatibility] Found in 'user.home' directory: " + f.getAbsolutePath());
			return f;
		} else {
			// Didn't find it under old directory ".esapi" so now look under the "esapi" directory.
			f = new File(homeDir + "/esapi", filename);
			if ( f.canRead() ) {
				logSpecial("Found in 'user.home' directory: " + f.getAbsolutePath());
				return f;
			} else {
				logSpecial("Not found in 'user.home' (" + homeDir + ") directory: " + f.getAbsolutePath());
			}
		}

		// return null if not found
		return null;
	}
```
源码流程也很简单

- 首先通过 `-Dorg.owasp.esapi.resources`指定路径获取，如果存在，直接返回

- 上面过程获取不到，通过` ClassLoader.getSystemResource("esapi/" + filename);`来获取，如果存在，直接返回

- classloader也获取不到，查找userHome中是否存在


调试分析

- 单元测试启动

`ClassLoader.getSystemClassLoader()`

 ![image](https://jasperxgwang.github.io/images/essay/esapi-2.jpg)

`Thread.currentThread().getContextClassLoader()`

 ![image](https://jasperxgwang.github.io/images/essay/esapi-5.jpg)
 
 使用的都是jvm的ClassLoader，并且资源文件都可以找到


- jetty容器启动

`ClassLoader.getSystemClassLoader()`

 ![image](https://jasperxgwang.github.io/images/essay/esapi-2.jpg)
 
 ![image](https://jasperxgwang.github.io/images/essay/esapi-1.jpg)

`Thread.currentThread().getContextClassLoader()`

 ![image](https://jasperxgwang.github.io/images/essay/esapi-3.jpg)
 
 ![image](https://jasperxgwang.github.io/images/essay/esapi-4.jpg)

发现当前线程的classloader是通过jetty容器`WebAppClassLoader`获取，而并非jvm的classloader

至于此次问题怎么发现，是源于之前隐约记得在看 [JAVA类加载器](https://jasperxgwang.github.io/2018/06/06/JAVA%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8/)
有提到过双亲模式的破坏，**Tomcat的WebappClassLoader 就会先加载自己的Class，找不到再委托parent**

当时看到这里的时候也是大概看了一下，也没深入去理解，只在脑海中停留了一下，

借此在网上找到有位大神的讲解正符合此情景，于是就仔细去拜读了一下，加深容器类加载器的理解，避免以后再出现此类诡异的问题
[Jetty源码阅读---Jetty的类加载器WebAppClassLoader](https://blog.csdn.net/acm_lkl/article/details/79253048)

另外还有一位小伙伴的分享也是不错的,故此处记录一下[关于getSystemResource, getResource 的总结](https://www.cnblogs.com/drwong/p/5389631.html)







 
















