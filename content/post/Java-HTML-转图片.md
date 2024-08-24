+++
title = "Java将HTML转为图片"
date = 2021-03-25
tags = ["Java"]
categories = ["Java"]
+++


**记录一个使用Java转图片的第三方工具**

```
<!-- https://mvnrepository.com/artifact/net.sf.cssbox/cssbox -->
<dependency>
  <groupId>net.sf.cssbox</groupId>
  <artifactId>cssbox</artifactId>
  <version>5.0.0</version>
</dependency>
```

使用方法很简单，伪代码如下:

```
 ImageRenderer render = new ImageRenderer();
 ByteArrayOutputStream os = new ByteArrayOutputStream();
 //创建一个临时的HTML文件
 Path tempFile = Files.createTempFile(IdWorker.getMillisecond(), ".html");
 //将HTML字节流写入到临时文件中
 Files.write(tempFile,createHtml());
 render.renderURL(tempFile.toUri().toString(), os);
 String fileName = "test.png";
 //将图片上传到OSS
 AliyunOSSUtil.upload(AliyunOSSUtil.toOSSFilePath(fileName, sysUser.tenantId()), new  ByteArrayInputStream(os.toByteArray()));
 //删除临时文件
 Files.deleteIfExists(tempFile);
```

我的方案是使用`freemarker`模板引擎先将数据渲染到成html文件，然后通过html文件生成一个图片，将图片上传到OSS中，最后删除临时生成的html文件。
