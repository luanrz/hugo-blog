---
title: "Sleuth在K8S中的连接重置问题"
date: 2022-12-13T14:22:49+08:00
categories:
- 部署运维
tags:
- K8S
- Golang
---

## 一、前言

> Sleuth是Java中的一种链路追踪组件

最近在使用K8S部署Java应用时，发现引入Sleuth组件后，日志中将会出现大量“Connection reset by peer”（连接重置）错误。这个错误不影响正常业务流程，但会极大污染日志内容。下面记录一下此问题的排查过程，以便后续参考。

（如果不想看过程的话，可以直接跳转到[结论](#四结论)部分）


## 二、问题复现

首先，K8S集群中要有一个可以正常运行的Java应用，K8S健康检查的服务路径是`/doc.html`（该服务由应用的Swagger组件提供）。

随后，引入Sleuth组件。在pom文件中加入下述依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

Sleuth可以与Zipkin组合使用，它们的默认都是开启状态，为了隔离影响，先把Zipkin关掉，修改application.yml文件如下：

```yml
spring:
  sleuth:
    enabled: true
  zipkin:
    enabled: false
```

最后，重新打镜像部署，启动容器之后，查看日志可以发现以下报错：

```log
TraceFilter.java:171 - Uncaught exception thrown
org.apache.catalina.connector.ClientAbortException: java.io.IOException: Connection reset by peer
        at org.apache.catalina.connector.OutputBuffer.realWriteBytes(OutputBuffer.java:356)
        at org.apache.catalina.connector.OutputBuffer.flushByteBuffer(OutputBuffer.java:815)
        at org.apache.catalina.connector.OutputBuffer.append(OutputBuffer.java:720)
        at org.apache.catalina.connector.OutputBuffer.writeBytes(OutputBuffer.java:391)
        at org.apache.catalina.connector.OutputBuffer.write(OutputBuffer.java:369)
        at org.apache.catalina.connector.CoyoteOutputStream.write(CoyoteOutputStream.java:96)
        at org.springframework.cloud.sleuth.instrument.web.TraceServletOutputStream.write(TraceServletOutputStream.java:120)
        ...
Caused by: java.io.IOException: Connection reset by peer
        at sun.nio.ch.FileDispatcherImpl.write0(Native Method)
        at sun.nio.ch.SocketDispatcher.write(SocketDispatcher.java:47)
        at sun.nio.ch.IOUtil.writeFromNativeBuffer(IOUtil.java:93)
        ...
```

## 三、排查过程

### （一）打印更多日志

通过日志的第一行可以看到这个报错可能与`TraceFilter`类有关（该类位于Sleuth包中），通过阅读源码发现有部分日志是DEBUG级别。因此，为了在日志中得到更多信息，需要修改日志级别，在java启动命令后加上如下参数：

```shell
--logging.level.root=DEBUG \
--logging.level.org.springframework.web.servlet.DispatcherServlet=DEBUG  \
--logging.level.org.springframework.cloud.sleuth=DEBUG
```

> 该参数可以在Helm Chart中指定，也可以在rancher中动态修改。

### （二）健康检查探针的问题

启用DEBUG日志之后，发现K8S健康检查探针在不断地调用`/doc.html`服务，且异常出现的频率非常稳定，怀疑是探针请求`/doc.html`服务发生的错误。修改探针重试间隔时间：

```yml
livenessProbe:
  httpGet:
    path: /doc.html
    port: 8080
  periodSeconds: 20 # 修改存活探针重发间隔时间
readinessProbe:
  httpGet:
    path: /doc.html
    port: 8080
  periodSeconds: 30 # 修改就绪探针重发间隔时间
```

发现日志每隔20秒或30秒都会出现一次异常异常，调用频次与异常出现的频次基本吻合，初步断定该问题出现在健康检查探针调用应用的`doc.html`服务过程中。

随后，在日志中又发现了探针调用`doc.html`的请求内容：

```log
## DirectJDKLog.java:179 - Received [GET /doc.html HTTP/1.1
Host: 102.0.1.1:8080
User-Agent: kube-probe/1.17
Accept-Encoding: gzip
Connection: close

]
```

鉴于出现“Connection reset by peer”（连接重置）可能的原因是：客户端与服务端在发送数据过程中，客户端断开了连接。因此，初步推测，**健康检查探针的`Connection: close`请求头可能是出现问题的原因之一**。

### （二）应用服务doc.html的问题

为了验证是不是`doc.html`这个服务本身的问题，尝试直接在外部直接调用`doc.html`服务，发现没有复现此问题。

随后，尝试修改健康检查的服务路径（`httpGet.path`）为自定义的`inde.html`，该文件中只有`Hello World`这几个字符串。发现此问题没有复现了！

想到`index.html`与`doc.html`唯一的不同就只有`index.html`的内容太小，因此再次向`index.html`中写入10000个`Hello World`，问题再次复现了！

因此，初步推测，**健康检查探针调用服务时的响应体过大可能是出现问题的原因之一**。

### （三）Sleuth的问题

为了验证Sleuth本身的问题，在本地创建以下demo程序：

pom.xml：

```xml
<project>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
    </dependencies>
</project>
```

启动类：
```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/index.html")
    public String home() throws InterruptedException {
        return getResult(); // 在此处打断点
    }

    private String getResult() {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            result.append("Hello World");
        }
        return result.toString();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

启动应用，并在响应返回之前打断点，在浏览器中访问`localhost:8080/index.html`，并马上点击停止按钮，以此来模拟客户端与服务端断开连接的情况。发现本地复现了同样的问题：

```log
o.s.c.sleuth.instrument.web.TraceFilter  : Uncaught exception thrown
org.apache.catalina.connector.ClientAbortException: java.io.IOException: 你的主机中的软件中止了一个已建立的连接。
	at org.apache.catalina.connector.OutputBuffer.doFlush(OutputBuffer.java:322) ~[tomcat-embed-core-8.5.40.jar:8.5.40]
	at org.apache.catalina.connector.OutputBuffer.flush(OutputBuffer.java:285) ~[tomcat-embed-core-8.5.40.jar:8.5.40]
	at org.apache.catalina.connector.CoyoteOutputStream.flush(CoyoteOutputStream.java:118) ~[tomcat-embed-core-8.5.40.jar:8.5.40]
	at org.springframework.cloud.sleuth.instrument.web.TraceServletOutputStream.flush(TraceServletOutputStream.java:128) ~[spring-cloud-sleuth-core-1.3.6.RELEASE.jar:1.3.6.RELEASE]
	...
Caused by: java.io.IOException: 你的主机中的软件中止了一个已建立的连接。
	at sun.nio.ch.SocketDispatcher.write0(Native Method) ~[na:1.8.0_331]
	at sun.nio.ch.SocketDispatcher.write(SocketDispatcher.java:51) ~[na:1.8.0_331]
	at sun.nio.ch.IOUtil.writeFromNativeBuffer(IOUtil.java:93) ~[na:1.8.0_331]
    ...
```


根据日志内容与Sleuth源码，在经历响应返回前的过滤（`Filter`）阶段，`TraceFilter`抛出异常，而异常出现的原因就是：客户端断开了连接。


## 四、结论

综上，出现“Connection reset by peer”（连接重置）错误的主要原因是：在客户端与服务端进行数据交互的过程中，客户端断开了连接，而数据没有传输完成。

体现在当前应用场景就是：K8S健康检查时未完全接受到响应体之前终止了连接（类似于网页上点击停止按钮），而此时响应体数据正在在持续传输，因此出现了连接重置异常，最后该异常又被Sleuth组件的`TraceFilter`过滤器捕获到。

因此，出现该问题的两个诱因是：

1. K8S健康检查时未完全接受到响应体之前终止了连接
2. Sleuth组件将此问题表现出来了（就算不加Sleuth组件，可能也有此问题，只是没有显现出来）

其中，K8S健康检查中止服务的可能原因有两个：健康检查探针包含了`Connection: close`请求头、健康检查探针获取的响应体过大（这两个原因存疑，需要后续继续阅读kube-probe源码获取更多信息）。

综上，为了解决这个问题，有两种简单方式：

1. 替换掉Sleuth组件，眼不见心不烦。
2. 将原始的`doc.html`服务路径修改为`index.html`，且新的服务只返回少量的内容（一个`Hello World`足矣）。
