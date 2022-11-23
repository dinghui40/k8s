> 本文作者：丁辉

# 本文介绍如何部署 JumpServer

## 环境准备

> docker 部署，k8s版本的话，直接去使用官网的helm部署就好
>
> [官网](https://docs.jumpserver.org/zh/master/install/setup_by_fast/)
>
> [JumpServer安装包](https://github.com/jumpserver/installer/releases)

## 部署 Mysql 数据库

```
docker run -itd --name jump-mysql \
--restart=always -p 3306:3306 \
-v /usr/local/jumpserver/data:/var/lib/mysql \
-v /usr/local/jumpserver/logs:/var/log/mysql \
-v /usr/local/jumpserver/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=jumpserver \
-e MYSQL_DATABASE=jumpserver \
mysql:5.7
```

解压 JumpServer 安装包

```
tar -xf jumpserver-installer-v*.tar.gz
cd jumpserver-installer-v*
```



```
yum -y install wget
```

修改 `config-example.txt` 文件配置参数

启动

```
./jmsctl.sh install
```

