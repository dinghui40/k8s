> 本文作者：丁辉
>
> Gogs和Gitea类似直接进入部署环节

```
docker run -p 222:22 -p 3000:3000 --name=gogs \
-e TZ="Asia/Shanghai" \
-v /etc/gogs:/data  \
-d gogs/gogs
```

