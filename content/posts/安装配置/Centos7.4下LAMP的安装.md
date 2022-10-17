---
title: Centos7.4下LAMP的安装
date: 2018-03-28 14:17:00
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

Centos7.4下Apache.Mysql,PHP及phpMyAdmin的安装与配置

### Linux+Apache+MySQL+PHP安装总结（centos7.4）

#### 一、linux

linux系统为centos7.4版本。
    
#### 二、Apche
```shell
yum install httpd -y #（安装apache）
systemctl start httpd #（启动apache，也可以是service httpd start）
```

#### 三、Mysql
Centos7.4版本下安装mariadb（MySQL的一个开源分支）。
```shell
yum install mariadb mariadb-server #（安装MySql）
systemctl start mariadb #（启动MySQL）
mysql_secure_installation #（设置MySql，可在此处修改root密码）
```

#### 四、PHP
```shell
yum install php php-mysql（安装php、php-mysql模板）
```

#### 五、phpMyAdmin
```shell
yum install phpMyAdmin（安装数据库web端管理入口phpMyAdmin）
vim /etc/httpd/conf.d/phpMyAdmin.conf（修改本配置文件才能够远程访问）
```

```
<RequireAny>
	#Require ip 127.0.0.1
	#Require ip ::1
	Require all granted
</RequireAny>
```

```shell
vim /etc /phpMyAdmin/config.inc.php（修改本配置文件才能正常登入MySQL）
```

```
$cfg['Servers'][$i]['user']     = 'root';
$cfg['Servers'][$i]['password'] = 'Wise0823';（该密码须与数据库密码一致）
```

#### 六、结束
使用http://106.14.184.75/phpMyAdmin访问数据库管理页。（登录密码不能为‘0’）