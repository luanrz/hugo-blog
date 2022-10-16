---
title: Maven依赖机制浅析
date: 2022-04-22 13:49:08
categories:
- 编程语言
- Java
tags:
- Java
- Maven
---

Maven通过pom文件来管理Java包的依赖关系，多个pom文件组合在一起可以理解为一颗抽象的依赖树，父节点或祖父节点的包会传递给子节点或祖孙节点，这被称作**传递依赖**(Transitive-Dependencies)。

传递依赖一般通过两个标签来实现：parent与dependency，其中，parent定义了上级依赖，dependency定义了下级依赖。

随着项目复杂性的提升，依赖树中包发生冲突的概率也会增加，Maven通过**依赖仲裁**(Dependency-Mediation)与**依赖管理**(Dependency-Management)来唯一确定依赖树中包的版本。


<!--more-->


## 一、依赖仲裁

> Maven使用dependencies标签来引入**使用**包。

依赖仲裁，简而言之，就是在依赖树中包的版本发生冲突时，选择哪一个包的过程，Maven采取**就近原则**，即： 

1. **首先选择依赖层级最浅的** 
2. **若依赖层次一致，则选择最先定义的**

如对于以下依赖树：（参见[官网](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies)）

```
  A
  ├─ B
  │  └─ C
  │      └─ D（2.0）
  └─ E
      └─ D（1.0）
```

D包有两个版本，其对应了两条依赖路径：A --> B --> C --> D 与 A --> E --> D，第二条路径最短（依赖层级最浅），符合上述**就近原则**的第一条规则，最终选择D包的1.0版本。

又如，对于以下依赖树：

```
  A
  ├─ B
  │  └─ D（2.0）
  │
  └─ C
  │  └─ D（1.0）
```

D同样有两个版本，他们的依赖层级一样，A --> B --> D路径上的D最先定义，符合上述**就近原则**的第二条规则，最终选择D包的2.0版本。

如果想要某个包的版本优先使用，可以将节点提到上级pom节点（满足依赖层级最浅的规则），也可以将对应节点交换位置（满足最先定义的规则）。

当然，还有另外一种方法，那就是**依赖管理**。

## 二、依赖管理

> Maven使用dependencyManagement标签来**定义**包。

与dependencies不同的地方在于，dependencyManagement中的包只是一个定义，只有在dependencies中引入对应包时，dependencyManagement中的定义才有意义。

凡是在dependencyManagement中定义的包，dependencies中引入对应包时，都可以省略版本号，Maven会自动从dependencyManagement中找到对应版本，这一特性为包的版本统一提供了极大的便利性。

同时，有一点需要非常注意：**在子模块中，dependencies中包的版本无法覆盖dependencyManagement中对应包的版本**，如下所示：

```xml
<project>
...
    <dependencies>
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>4.9.3</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.squareup.okhttp3</groupId>
                <artifactId>okhttp</artifactId>
                <version>3.8.1</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
...
</project>
```

在上述pom中dependencies与dependencyManagement使用了okhttp，但它们的版本不一样，此时，Maven将优先使用在dependencyManagement中定义的3.8.1版本，因为在Maven中，**依赖管理优先于依赖仲裁** ，即使是在父节点（根节点除外）的dependencies引入一个层级更浅的okhttp版本，这里依旧会取dependencyManagement中okhttp的版本，因为不管okhttp位于哪个节点（根节点除外），它都是处于依赖仲裁的范围内，它的优先级永远低于依赖管理。

同时，顶层根节点（如启动类所在的web节点）的优先级最高。

因此，在Maven依赖中存在以下优先级：

**顶层根节点引入的依赖**  >  **依赖管理**  >  **依赖仲裁**

## 三、总结

至此，对于如何唯一确定一个包，并解决包冲突，我们也有了明确的方向：

1. 根据依赖管理原则，在dependencyManagement中寻找包的版本
2. 根据依赖仲裁原则，在dependencies中寻找包的版本
3. 在顶层根节点寻找包的版本
4. 按照上述 顶层根节点 `引入的依赖  >  依赖管理  >  依赖仲裁` 找到合适的包版本

> 参考文档
1. [Maven Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
