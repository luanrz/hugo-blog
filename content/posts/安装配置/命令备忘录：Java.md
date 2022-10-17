---
title: 命令备忘录：Java
date: 2021-04-02 09:50:12
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

## Java序列化

`new ObjectOutputStream(new FileOutputStream("resp")).writeObject(resp);`

## 配置VM参数"javaagent"以支持切面编程

```java
- javaagent:/home/luanrzh/.m2/repository/org/aspectj/aspectjweaver/1.9.4/aspectjweaver-1.9.4.jar
```

## 启用远程调试

```java
java -jar \
-Dserver.port=8080 \
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8081 \
parent-0.0.1-SNAPSHOT.jar
```

## jasypt加密解密

```java
java -cp jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input=123456 password=1234-1234-1234-1234 algorithm=PBEWithMD5AndDES

java -cp jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input="jvp0W0qUkv0WmG/JzwyUTA==" password=1234-1234-1234-1234 algorithm=PBEWithMD5AndDES
```
