---
title: Kubernetes入门
date: 2022-07-01 11:30:48
categories:
- 部署运维
tags:
- K8S
- Docker
- Golang
---

kubernetes（k8s）是容器编排的利器。

本文将介绍kubernetes本地环境的安装与基础使用，包括：创建集群、部署应用、故障排除、暴露服务、缩放应用、更新应用等内容，这里面大部分内容都来自于[Kubernetes官方教程](https://kubernetes.io/zh-cn/docs/tutorials/)。

<!--more-->

kubernetes环境至少需要以下前置条件：

1. Linux系统（本文以ArchLinux为例，包管理器为pacman，apt与yum等同理。也可以使用Mac、Window）
2. 虚拟化环境（本文以docker为例，也可以使用其它虚拟化平台）

## 一、创建集群

本地测试环境可以使用minikube创建**单节点**集群，生产环境可以使用docker创建**多节点**集群，下面将分别介绍这两种方式。

<!-- tab 使用minikube创建单节点集群 -->

minikube是一个入门级的单节点kubernetes集群，麻雀虽小，五脏俱全。

1. **启动docker**

    ```shell
    sudo systemctl start docker
    ```

    > 在minikube中docker不是必选项，也可以使用virtualbox等驱动

2. **安装minikube**

    ```shell
    sudo pacman -S minikube
    ```

3. **启动minikube**

    ```shell
    minikube start --image-mirror-country='cn'
    ```

    > minikube指定`--image-mirror-country='cn'`参数将从阿里云下载依赖，感谢[alibaba的贡献](https://github.com/kubernetes/minikube/issues/12535)

4. **查看minikube版本**

    ```shell
    minikube version
    ```

<!-- endtab -->

<!-- tab 使用docker创建多节点集群 -->

// TODO

<!-- endtab -->



## 二、部署应用

- **查看kubernetes版本**

    ```shell
    kubectl version 
    ```

- **查看集群节点**

    ```shell
    kubectl get nodes
    ```

- **创建deployment**

    ```shell
    kubectl create deployment kubernetes-bootcamp --image=gcr.io/2/kubernetes-bootcamp:v1
    ```

    > 创建deployment时会自动创建一个包含对应容器的pod，一个deployment也可以有多个pod（后面[缩放应用](#五缩放应用)章节将会介绍这种情况）
    
    > 同时，pod是kubernetes管理的最小单元。一个工作节点（node）可以包含多个pod，一个pod可以包含多个容器

    > gcr.io被墙了，启动应该会报错，但问题不大，先假装它启动成功了


- **查看deployment状态**

    ```shell
    kubectl get deployments
    ```

    > 此时，deployment中的应用只允许集群内部的其它pod或者service访问，为了在外部能够访问，可以选择[暴露应用](#四暴露应用)，也可以使用`proxy`

- **启用proxy**

    ```shell
    kubectl proxy
    ```

    > 执行此命令需要另开一个终端来常驻proxy服务

- **通过porxy访问集群内部服务**

    ```shell
    curl http://localhost:8001/version
    ```

    ```shell
    export POD_NAME=$(kubectl get pods | grep kubernetes-bootcamp | awk '{print $1}') # 获取pod名
    export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')  # 获取pod名的官方写法，与上面等价，两者取其一即可
    curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/ 
    ```


## 三、故障排除

- **检查应用配置**

    ```shell
    kubectl get nodes
    kubectl describe pods
    ```
    
- **使用proxy访问集群内部服务**

    参见上一小节

- **查看容器日志**

    ```shell
    kubectl logs $POD_NAME # POD_NAME在前面定义了
    ```

    > 容器启动了才会有日志    

- **在容器中执行命令**

    ```shell
    kubectl exec $POD_NAME -- env  # POD_NAME在前面定义了
    kubectl exec -ti $POD_NAME -- bash
    ```

## 四、暴露应用

- **创建service**

    ```shell
    kubectl get pods # 确认应用正在运行
    kubectl get services # 查看service状态
    kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 # 创建service
    kubectl get services # 再次查看service状态
    kubectl describe services/kubernetes-bootcamp # 查看刚刚新建的service的描述信息
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') # 获取暴露出来的节点端口
    curl $(minikube ip):$NODE_PORT # 在外面通过service暴露的端口访问应用，如果创建集群时用的不是minikube，则$(minikube ip)需要换成对应的节点ip
    ```

    > 使用`expose`指令来创建service

- **使用标签**

    ```shell
    kubectl describe deployment # 查看deployment自动创建的Labels
    kubectl get pods -l app=kubernetes-bootcamp # 通过Label来查看pod
    kubectl get services -l app=kubernetes-bootcamp  # 通过Label来查看service
    export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') # 获取pod名
    kubectl label pods $POD_NAME version=v1 # 为pod增加Label
    kubectl describe pods $POD_NAME # 查看pod中的Labels
    kubectl get pods -l version=v1 # 再次通过Label来查看pod，不过这次条件变成了version=v1
    ```

    > 使用`label`指令来增加标签，使用`-l`参数来使用标签

- **删除service**

    ```shell
    kubectl delete service -l app=kubernetes-bootcamp # 通过Lable删除service
    kubectl get services # 查看service是否被删除
    curl $(minikube ip):$NODE_PORT # 验证外部能否访问应用（不能）
    kubectl exec -ti $POD_NAME -- curl localhost:8080 # 验证内部能否访问应用（能）
    ```

    > 使用`delete`指令来删除service，deployment等同理

## 五、缩放应用

- **扩展deployment中的pod**

    ```shell
    kubectl get deployments # 查看deployment状态
    kubectl get rs # 查看deployment所创建的pod副本集合
    kubectl scale deployments/kubernetes-bootcamp --replicas=4 # 将pod副本个数扩展到4
    kubectl get deployments # 再次查看deployment状态
    kubectl get pods -o wide # 查看pod状态
    kubectl describe deployments/kubernetes-bootcamp # 在deployment描述信息中查看Event与Replicas
    ```

    > 使用`scale`指令来缩放应用副本

- **负载均衡**

    ```shell
    kubectl describe services/kubernetes-bootcamp # 查看service暴露的ip和端口
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') # 获取暴露出来的节点端口（和前面一样）
    curl $(minikube ip):$NODE_PORT # 通过暴露的ip和端口访问应用
    ```

    > 无须额外配置，使用`scale`指令扩展的应用自动支持负载均衡

- **缩小deployment中的pod**

    ```shell
    kubectl scale deployments/kubernetes-bootcamp --replicas=2 # 将pod副本个数扩展到2（减少2个）
    kubectl get deployments # 查看deployment状态
    kubectl get pods -o wide # 查看pod状态
    ```
    
    > 扩展或缩小deployment中的本质是：将pod数调整到指定数目

## 六、更新应用

- **更新应用到指定版本**

    ```shell
    kubectl get deployments # 查看deployment状态
    kubectl get pods -o wide # 查看pod状态
    kubectl describe pods # 查看pod描述信息
    kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2 # 将镜像版本设置为v2，并开始滚动更新
    ```

    > 使用`set image`指令来更新应用到指定版本

- **验证更新**

    ```shell
    kubectl describe services/kubernetes-bootcamp # 查看service暴露的ip和端口（和前面一样）
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') # 获取暴露出来的节点端口（和前面一样）
    curl $(minikube ip):$NODE_PORT # 通过暴露的ip和端口访问应用
    kubectl rollout status deployments/kubernetes-bootcamp # 查看更新状态
    kubectl describe pods # 查看pod中的镜像是否是最新版本
    ```

    > 使用`rollout status`指令查看更新状态

- **回滚应用到上一版本**

    ```shell
    kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10 # 将镜像版本设置为v10，并开始滚动更新
    kubectl get deployments # 查看deployment状态
    kubectl get pods # 查看pod状态
    kubectl describe pods # 查看pod描述信息，在Events中找到ImagePullBackOff的原因：v10版本不存在
    kubectl rollout undo deployments/kubernetes-bootcamp # 回滚到上一个稳定版本
    kubectl get pods # 再次查看pod状态
    kubectl describe pods # 再次查看pod描述信息
    ```
    
    > 使用`rollout undo`指令来回滚应用到上一版本
