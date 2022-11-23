> 本文作者：丁辉
本文记录 docker 搭建 code-server 过程
````
docker run -d -u root   -p 8080:8080 \
--name code-server -v /data/vscode/config.yaml:/root/.config/code-server/config.yaml \
-v /data/vscode-data:/home/code \
codercom/code-server
````
[官网文档地址](https://github.com/coder/code-server)
