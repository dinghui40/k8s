> 本文作者：丁辉

[官网文档](https://docs.gitea.io/zh-cn/)

创建docker-compose文件

```
vi docker-compose
```

```
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    networks:
      - gitea
    volumes:
      - /etc/gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
```

启动或关闭

```
docker-compose up -d
docker-compose down
```

