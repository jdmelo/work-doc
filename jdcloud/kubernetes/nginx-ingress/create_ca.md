# 1. 创建CA证书和私钥

自建CA, 用来为后续用户申请创建数字签名证书.

## 1.1 安装cfssl工具集

```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

## 1.2 创建配置文件

CA 配置文件用于配置根证书的使用场景 (profile) 和具体参数 (usage: 过期时间、服务端认证、客户端认证、加密等)，后续在签名其它证书时需要指定特定场景。
```
mkdir /root/cert
cd /root/cert
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "ingress": {
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

## 1.3 创建证书签名请求文件

```
cd /root/cert
cat > ca-csr.json <<EOF
{
  "CN": "ingress",
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
      "OU": "4Paradigm"
    }
  ],
  "ca": {
    "expiry": "876000h"
 }
}
EOF
```

## 1.4 生成CA证书和私钥

```
cd /root/cert
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls ca*.pem
```

CA制作完成

# 2. 创建SSL证书和私钥

SSL证书帮助您实现网站的HTTPS化，为网站提供安全、有效的访问。

## 2.1 创建证书签名请求

```
cd /root/cert
cat > web-server-csr.json <<EOF
{
  "CN": "ingress",
  "hosts": [
    "web-server-test.jdcloud.com"
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
      "OU": "4Paradigm"
    }
  ]
}
EOF
```

## 2.2 申请SSL证书

```
cd /root/cert
cfssl gencert -ca=/root/cert/ca.pem \
  -ca-key=/root/cert/ca-key.pem \
  -config=/root/cert/ca-config.json \
  -profile=ingress web-server-csr.json | cfssljson -bare web-server
ls web-server*.pem
```