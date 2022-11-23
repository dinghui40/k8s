> 本文作者：丁辉

[官网](https://docs.drone.io/)

https://0-8-0.docs.drone.io/zh/getting-started/

## docker部署

> 基于：Github,https
>
> DRONE_RPC_SECRET="openssl rand -hex 16"

```
docker run \
  -v /var/lib/drone:/data \
  -v /data/certs/server.crt:/etc/certs/drone.company.com/server.crt \
  -v /data/certs/server.key:/etc/certs/drone.company.com/server.key \
  --env=DRONE_GITHUB_CLIENT_ID= \
  --env=DRONE_GITHUB_CLIENT_SECRET= \
  --env=DRONE_RPC_SECRET= \
  --env=DRONE_SERVER_HOST=dingh.co \
  --env=DRONE_SERVER_PROTO=https \
  --env=DRONE_TLS_AUTOCERT=true \
  --env=DRONE_TLS_CERT=/etc/certs/drone.company.com/server.crt \
  --env=DRONE_TLS_KEY=/etc/certs/drone.company.com/server.key \
  --env=DRONE_USER_CREATE=username:dinghui40,admin:true \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```

## docker-compose部署

> 基于：Gitea,http
>
> DRONE_RPC_SECRET=Gitea令牌

```
version: '3'
services:
  drone-server:
    restart: always
    image: drone/drone:2
    ports:
      - "9999:80"
    volumes:
      - /etc/drone/drone:/var/lib/drone/
      - /etc/drone/data:/data/
    environment:
      - DRONE_GITEA_SERVER=http://dingh.co:3000
      - DRONE_GITEA_CLIENT_ID=
      - DRONE_GITEA_CLIENT_SECRET=
      - DRONE_SERVER_HOST=dingh.co:9999
      - DRONE_SERVER_PROTO=http
      - DRONE_RPC_SECRET=
      - DRONE_GIT_ALWAYS_AUTH=true
      - DRONE_GIT_USERNAME=offends
      - DRONE_GIT_PASSWORD=offends@dh4
      - DRONE_USER_CREATE=username:offends,admin:true
  drone-runner-docker:
    restart: always
    image: drone/drone-runner-docker:1
    ports:
      - "10000:3000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=
      - DRONE_RUNNER_NAME=drone-runner-docker
      - DRONE_RUNNER_CAPACITY=2
```

