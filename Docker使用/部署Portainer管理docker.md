> 本文作者：丁辉
>
> 通过Portainer管理docker

[官方文档](https://docs.portainer.io/)

## (https)协议部署

创建docker存储卷

```
docker volume create portainer_data
```

启动

```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

## (http)协议部署

数据目录挂在至本地

启动

```
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /etc/portainer/data:/data portainer/portainer-ce:latest
```

## docker-compose部署

```
version: "3"
services:
  portainer:
    image: portainer/portainer:latest
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /etc/portainer/data:/data
      - /var/run/docker.sock:/var/run/docker.sock
```

启动或关闭

```
docker-compose up -d
docker-compose down
```

## 远程连接docker

远程连接默认端口是2375

如无法连接则需要配置开启docker2375端口

```
# 1. 编辑docker.service
vim /usr/lib/systemd/system/docker.service

# 找到 ExecStart字段修改如下
ExecStart=/usr/bin/dockerd-current -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock 

# 2. 重启docker重新读取配置文件，重新启动docker服务
systemctl daemon-reload
systemctl restart docker

# 3. 开放防火墙端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent

# 4.刷新防火墙
firewall-cmd --reload

# 5.再次配置连接远程docker就可以了
```

