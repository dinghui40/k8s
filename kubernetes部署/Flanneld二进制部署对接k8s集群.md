> 本文作者：丁辉

# 部署 Flannel 集群

准备安装包

[Flannel](https://github.com/flannel-io/flannel/releases)

解压安装包

```
tar -xvf flannel-v*-linux-amd64.tar.gz
mv flanneld mk-docker-opts.sh /usr/local/bin/
```

定义API

```
ETCDCTL_API=3
```

向 ETCD 集群写入网段信息 

```
etcdctl --cacert=/etc/kubernetes/pki/ca.pem --cert=/etc/kubernetes/pki/kubernetes.pem --key=/etc/kubernetes/pki/kubernetes-key.pem --endpoints="https://192.168.1.10:2379,https://192.168.1.20:2379,https://192.168.1.30:2379"  put /coreos.com/network/config  '{ "Network": "10.244.0.0/16", "Backend": {"Type": "vxlan"}}'
```

查看信息

```
etcdctl get /coreos.com/network/config
```

配置 Flannel

```
cat > /etc/kubernetes/flanneld.conf <<EOF
FLANNEL_OPTIONS="--etcd-endpoints=https://192.168.1.10:2379,https://192.168.1.20:2379,https://192.168.1.30:2379 -etcd-cafile=/etc/kubernetes/pki/ca.pem -etcd-certfile=/etc/kubernetes/pki/kubernetes.pem -etcd-keyfile=/etc/kubernetes/pki/kubernetes-key.pem"
EOF
```

创建 Flannel 系统启动文件

```
vi /usr/lib/systemd/system/flanneld.service
```

```
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/etc/kubernetes/flanneld.conf
ExecStart=/usr/local/bin/flanneld --ip-masq $FLANNEL_OPTIONS
ExecStartPost=/usr/local/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
Restart=on-failure

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

配置 Docker 启动指定子网段

```
vi /usr/lib/systemd/system/docker.service
```

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service

[Service]
Type=notify
EnvironmentFile=/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0l
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
```

启动 docker 和 flanneld

```
systemctl daemon-reload && systemctl enable flanneld && systemctl start flanneld && systemctl status flanneld && systemctl stop docker && systemctl start docker
```

查看 Flannel 服务设置 docker0 网桥状态

`ip a` 查看 `docker0`

验证 Flannel 服务

```
cat /run/flannel/subnet.env
```
