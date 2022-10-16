---
title: Nginx反向代理配置
date: 2018-11-11 14:50:00
categories:
- 安装配置
- 应用软件
tags: 
- Linux
description: 以PC机上的Tomcat环境与树莓派上的LNMP环境整合过程为例, 演示Nginx反向代理简单配置过程
---

# Nginx反向代理配置
> 整合PC机上的Tomcat环境与树莓派上的LNMP环境

## 一、启动Tomcat服务与Apache服务
- 访问http://192.168.43.209:8080 ,测试PC机上的Tomcat环境
- 访问http://192.168.43.105:80 ,测试树莓派上的LNMP环境

## 二、修改nginx.conf
`sudo vim /etc/nginx/nginx.conf `

    -----------------------------------------------------------
    http    {
        upstream tomcat {  
            server 192.168.43.209:8080;  
        } 
        upstream ksweb {
            server 192.168.43.1:8888;
        }
        server {
            listen       80;
            server_name  tomcat;
            location / {
                proxy_pass http://tomcat;  
            }
        }
        server {
            listen       80;
            server_name  pi;
            location / {
                proxy_pass http://pi; 
                index index.php index.html index.htm;	
            }
        }
    }
    -----------------------------------------------------------
    
## 三、修改本地hosts
`sudo vim /etc/hosts `

    -----------------------------------------------------------
    192.168.43.209	tomcat
    192.168.43.105	pi
    -----------------------------------------------------------

## 四、测试
`sudo vim ~/Desktop/test.html`

    -----------------------------------------------------------
    <a href ="http://tomcat">tomcat</a>
    <a href ="http://pi">pi</a>
    -----------------------------------------------------------
    
运行test.html,点击"tomcat"与"pi",将分别tomcat首页与LNMP首页

## 五、总结
关键字:反向代理、负载均衡、分布式
1. 反向代理的主要作用是负载均衡
2. PC机和树莓派可以看作是一个简单的分布式系统
