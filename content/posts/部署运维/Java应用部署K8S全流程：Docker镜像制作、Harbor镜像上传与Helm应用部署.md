---
title: "Java应用部署K8S全流程：Docker镜像制作、Harbor镜像存储与Helm应用部署"
date: 2022-12-11T11:01:19+08:00
categories:
- 部署运维
tags:
- K8S
- Docker
- Java
- Golang
---

## 一、前言

Java应用部署到K8S（Kubernetes），一般需要镜像制作、镜像存储与应用部署这三个步骤。

- 镜像制作：根据Java源代码生成符合OCI（开放容器标准）的镜像。可以使用`Docker`完成镜像制作。
- 镜像存储：将镜像上传到镜像仓库以便后续使用。这里的镜像仓库一般是指`Harbor`。
- 应用部署：根据镜像仓库中的镜像及其对应配置，将应用部署到K8S集群。K8S原生支持手动编写配置文件实现应用部署，`Helm`简化了应用部署的过程。

为了更好地理解云环境部署，可以类比传统的虚机部署。在云环境部署中：

- 镜像制作就相当于：maven打包生成war包（实际上war包是镜像的一部分）。
- 镜像存储就相当于：将war包上传到Maven仓库。
- 应用部署就相当于：将Maven仓库中的war包安装到JBoss容器。

在此正式介绍上述三个过程之前，还需要准备一些环境。除了Java基础环境之外，还需要准备一个[Docker](https://docs.docker.com/get-docker/)环境与Kubernete集群（可以使用[minikube](https://minikube.sigs.k8s.io/docs/start/)）。

> 国内网络环境启动minikube时，可以使用：`minikube start --image-mirror-country='cn' --kubernetes-version=v1.23.13`指令，其中，`cn`表示使用国内镜像，`v1.23.13`指定K8S版本小于`v1.24`，具体原因参见[Kubernetes 1.24 中的移除和弃用](https://kubernetes.io/zh-cn/blog/2022/04/07/upcoming-changes-in-kubernetes-1-24/)。

## 二、Docker镜像制作

Docker制作的镜像符合OCI标准，可以直接在K8S中运行。本地制作的镜像一般需要上传到[Harbor仓库](#三harbor镜像存储)。

### （一）准备一个Java项目

使用spring-web搭建一个简单的web项目，pom文件如下：

```xml
<project>
    ...
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

启动类如下：

```java
@SpringBootApplication
@RestController
public class Application {
    @RequestMapping("/index.html")
    public String home() throws InterruptedException {
        return "Hello World";
    }
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### （二）编写Dockerfile脚本

在项目根目录下新建文件`Dockerfile`，内容如下：

```Dockerfile
FROM maven:3.8.4-openjdk-8-slim AS base
WORKDIR /app
COPY pom.xml ./
RUN --mount=type=cache,target=/root/.m2 mvn dependency:go-offline
COPY src ./src

FROM base AS build
RUN --mount=type=cache,target=/root/.m2 mvn package

FROM openjdk:8-jre-slim-buster AS production
EXPOSE 8080
COPY --from=build /app/target/*.jar /app/app.jar
CMD java -jar /app/app.jar

```

其中，`--mount`语法需要启用Docker的[BuildKit](https://docs.docker.com/build/buildkit/)功能，`mvn dependency:go-offline`表示缓存maven包以供后续重复使用。

### （三）生成Docker镜像

在项目根目录进入终端，执行`docker build`命令：

`docker build -t app:0.0.1 .`

查看镜像是否成功生成：

`docker images`

如果有看到`REPOSITORY`为`app`且`TAG`为`0.0.1`的记录，证明镜像制作完成。



## 三、Harbor镜像存储

Harbor是用来存储符合OCI标准镜像的一种仓库实现，Docker客户端、K8S集群都可以从Harbor仓库拉取指定镜像。在[Helm应用部署](#四helm应用部署)中的镜像仓库配置中，同样可以使用Harbor仓库。


### （一）安装Harbor

Harbor的具体安装过程可以跟着[官方安装教程](https://goharbor.io/docs/2.6.0/install-config/)走，安装过程不复杂，简而言之就是：下载安装包、修改配置文件、执行安装脚本三步走。

这里有一点需要特别说明，当前Harbor支持HTTP与HTTPS两种访问方式，正常来说，本地测试使用HTTP就可以了，但Helm限制了Harbor的访问协议必须是HTTPS，因此还需要为HTTPS配置一些额外的证书。

在Harbor启用SSL后，客户端需要配置ca.crt证书，否则在拉取镜像时会报x509验证错误。这里的客户端可以是Docker，也可以是K8S，下面分别介绍这两种客户端的配置方法。

（1） Docker客户端配置SSL

将ca.crt放入指定目录，Linux系统位于`/etc/docker/certs.d/`，windows系统位于`%HOMEPATH%/.docker/certs.d/`，有关Docker客户端SSL验证详见[官方文档](https://docs.docker.com/engine/security/certificates/)。

> 这里的ca.crt可以手动生成，也可以使用下面介绍的minikube自带的ca.crt。

> Windows系统也可以可以双击ca.crt文件以导入证书。

（2） K8S客户端配置SSL（以minikube为例）

minikube的`~/.minikube`目录下已经有了ca.crt文件，直接使用这个文件即可，可以跳过[Harbor配置HTTPS的官方文档](https://goharbor.io/docs/2.6.0/install-config/configure-https/)中的[Generate a Server Certificate](https://goharbor.io/docs/2.6.0/install-config/configure-https/#generate-a-server-certificate)这一步。

至此，根据上面的流程已经配置好了可以通过HTTPS访问的Harbor服务：`https://harbor.local.com`，这个域名是自定义的，可以在Harbor配置安装的时候手动指定。同时，需要在hosts文件中配置对应的dns映射。

### （二）验证Harbor

验证Harbor连通性最简单的访问方式就是在浏览器中访问对应的Harbor服务，域名为`https://harbor.local.com`，默认用户名密码为`admin`与`Harbor12345`。

同时，也可以使用`docker login`验证Harbor连通性，在终端输入`docker login harbor.local.com`，接着输入对应的用户名密码，提示`Login Succeeded`，即证明Harbor正常连通。

### （三）上传镜像到Harbor

上传之前，先要确保已经通过`docker login`完成了Harbor仓库的鉴权。

然后，执行`docker push`命令将之前制作的镜像推送到Harbor：

```shell
docker push app:0.0.1
```

执行完毕后，会发现报了以下错误：

```log
The push refers to repository [docker.io/library/app]
An image does not exist locally with the tag: app
```

如果没有指定`app`的前缀，那么`docker push`命令会默认使用dockerhub的自带仓库，这与预期不符，因此需要手动给该镜像加上一个仓库路径前缀。如下所示：

```shell
docker tag app:0.0.1 harbor.local.com/luanrz/app:0.0.1
```

此命令会生成一条新的image记录，它的IMAGE ID与原来的镜像一致，现在，重新推送：

```shell
docker push harbor.local.com/luanrz/app:0.0.1
```

又会报一条错：

```log
unauthorized: project luanrz not found: project luanrz not found
```

可以看到是`luanrz`这个项目没有创建，登录到Harbor页面，点击“新建项目”，创建对应的项目。

再次执行`docker push`进行推送，如果没有问题将显示推送成功的结果。至此，镜像成功上传到Harbor仓库。


## 四、Helm应用部署

Helm是K8s中的包管理器，类似于Linux中pacman、apt、yum等包管理器的职责。Helm简化了安装应用到K8S的过程。

### （一）安装Helm

可以参照[官方教程](https://helm.sh/zh/docs/intro/install/)完成Helm的安装，这里不再赘述。安装完之后，在控制台输入`helm version`正常输出结果即证明安装成功。

### （二）制作Chart

首先，创建一个Helm Chart：

```shell
helm create app
```
进入app后，修改`values.yaml`文件：

```yaml
...
image:
  repository:  harbor.local.com/luanrz/app
  pullPolicy: Always
  tag: "latest"
...
service:
  type: ClusterIP
  port: 8080
...

```
其中，image.repository与image.tag唯一定位了一个Harbor仓库中的镜像地址，service.port则表明了对外暴露的服务端口。

### （三）安装应用

在app目录下执行：

```shell
helm install app .
```

如果执行无误，将会打印出当前Release的状态，通过`helm list`可查看所有的Release。

正常情况下，helm会将values文件与templates文件组合成应用部署的相关指令，发给K8S集群，后续就可以使用kubelet控制集群中的应用了。

### （四）验证应用

执行`kubectl get`系列命令查看deployments、pods与services的状态：

```shell
kubectl get deployments
kubectl get pods
kubectl get services
```

正常情况下上述资源都会有一条对应的app记录，如果pod状态不是Running，可以通过`kubectl describe`查看事件（Events）排查原因：

```shell
kubectl describe pods <POD_NAME>
```

比如，在Harbor镜像拉取失败时，就可能出现类似于以下的各种原因：

- Harbor只配置了HTTP访问模式，此时会报错提示不支持http访问方式或重定向至https失败（解决方案：将Harbor的访问模式配置为HTTPS）
- Harbor配置了HTTPS访问模式，此时可能也会报错提示x509验证失败（解决方案：详见[安装Harbor中的SSL相关说明](#一安装harbor)）
- 其它等等

排查原因解决问题后，如果一切无误，应用也就部署成功了。但外面可能还没办法访问，可以通过ingress或者port-forward进一步暴露服务端口，下面以port-forward为例：

```shell
kubectl port-forward services/<pod-name> 8080:8080
```

现在，可以在外面访问容器应用了：

```shell
curl http://localhost:8080/index.html
```

控制台正常输入`Hello World`，应用部署到此结束。


## 五、总结

Java应用部署到K8S按照顺序执行以下步骤：

- 镜像制作：使用`docker build`指令生成Docker镜像。
- 镜像存储：使用`docker login`指令和`docker push`指令登录到并推送Harbor仓库，需要针对针对Docker客户端和K8S集群调整Harbor的SSL配置。
- 应用部署：使用`helm create`创建一个Chart，使用`helm install`安装一个Release，使用Kubelet系列指令管理部署完成的应用。

至此，一个Java应用，从源代码，到镜像制作、镜像存储，最后到应用部署，完成了它上云全部步骤，后面的维护和管理，将是更大的挑战。