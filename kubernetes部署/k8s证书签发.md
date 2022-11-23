> 本文作者：丁辉

# k8s证书签发

## 安装 CFSSL

```
mkdir /root/k8s -p
cd /root/k8s
```

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
```

```
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

## 创建 CA 证书签名请求

```bash
cat > ca-csr.json <<EOF
{
    "CN": "kubernetes",
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "k8s",
        "OU": "System"
      }
    ],
      "ca": {
         "expiry": "87600h"
      }
  }
EOF
```

**生成 CA 证书和私钥**

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

**创建 CA 配置文件**

```bash
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

## 创建 kubernetes 证书

```bash
cat > kubernetes-csr.json <<EOF
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "192.168.1.10",
      "192.168.1.20",
      "192.168.1.30",
      "10.244.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

**生成 kubernetes 证书和私钥**

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
```

## 创建 admin 证书

```bash
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```

**生成 admin 证书和私钥**

```生成 admin 证书和私钥：bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
```

## 创建 kube-proxy 证书

```bash
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

**生成 kube-proxy 客户端证书和私钥**

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

## 创建 kube-controller-manager 证书

```bash
cat > kube-controller-manager-csr.json <<EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
      "127.0.0.1",
      "192.168.1.10",
      "192.168.1.20", 
      "192.168.1.30"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "system:kube-controller-manager",
        "OU": "System"
      }
    ]
}
EOF
```

**生成 kube-scheduler 客户端证书和私钥**

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

### 创建 kube-scheduler 证书

```bash
cat > kube-scheduler-csr.json <<EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
      "127.0.0.1",
      "192.168.1.10",
      "192.168.1.20", 
      "192.168.1.30"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "system:kube-scheduler",
        "OU": "System"
      }
    ]
}
EOF
```

**生成 kube-scheduler 客户端证书和私钥**

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

## 校验证书

**openssl 校验**

```bash
openssl x509  -noout -text -in  kubernetes.pem
```

**cfssl-certinfo 校验**

```bash
cfssl-certinfo -cert kubernetes.pem
```

## 分发证书

```bash
mkdir -p /etc/kubernetes/pki
cp *.pem /etc/kubernetes/pki
```

**使用证书的组件如下：**

- etcd：使用 kubernetes-key.pem、kubernetes.pem
- kube-apiserver：使用 kubernetes-key.pem、kubernetes.pem
- kubelet：使用 ca.pem
- kube-proxy：使用 kube-proxy-key.pem、kube-proxy.pem
- kubectl：使用 dmin-key.pem、admin.pem
- kube-controller-manager：使用 kube-controller-manager-key.pem、kube-controller-manager.pem
- kube-scheduler ：使用 kube-scheduler-key.pem、kube-scheduler.pem

[本文借鉴来源](https://jimmysong.io/kubernetes-handbook/practice/create-tls-and-secret-key.html)

