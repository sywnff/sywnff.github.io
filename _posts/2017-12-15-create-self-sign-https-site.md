建立自签名证书 Https 站点
==========================

# 简介

建立自签名证书 Https 站点可以分为三步:

1. 生成自签名 RootCA 证书
2. 为web站点生成证书
3. 将证书配置到Web服务器

# Create RootCA Certificate
## 脚本
create_root_ca_crt.sh

```
#!/bin/sh


subject="/C=CN/CN=example.com self-signed CA"
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -subj "$subject" -out rootCA.crt

```

## 说明

上面脚本包括两个步骤：
1. openssl genrsa: 生成RSA 私钥，长度为2048bits；
2. openssl req: 生成自签名证书 (rootCA.crt)；

证书本质上是将拥有者身份描述信息（名称、组织、域名等）、身份识别信息（公钥）以及对上述信息的认证数据（签名）等信息保存在一起的文件（下文展示其详细）。顾名思义，自签名证书即是用自己的私钥对证书内容（包括公钥）进行签名。上面脚本在第一步生成私钥的同时也就生成了公钥，如下从RSA 私钥文件获取对应公钥(反之不成立，为什么[看这里](https://stackoverflow.com/questions/5244129/use-rsa-private-key-to-generate-public-key))：

```
$ openssl rsa -in rootCA.key -pubout
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyV62BHg7w2LuhoAu5FVt
ThTVCAvTIR+c8Yp/4CenD9GRmgGW/3wE0poMZrtLwgB9km+xUu4VdHZ7NltiZRmg
xsGLeloJGJH8773Ma5ClVkxxbwa3S5Cc1JqmY8HVip44mkIjkFyJ5eiqGkU7tWM9
M5k4FADpsoX2csrvANmHke9JxFx/3vXMwnvZ4OVN+rNbrfqZmgpMR6U9Cx/vI0OG
a+SbgcXH1RF71uW9HLj0pFIufTF8jCMLwKIRDohumVRXjQrmzVxbD2S+LFKhrFyv
wkizex78kDSYOhBHuP9v0t76JfeiLwlV5FVZF0dMgwTvThs4pCt+tnlel70eLwuD
4QIDAQAB
-----END PUBLIC KEY-----
```

生成了根证书，我们就可以自建CA了，使用根证书对其它服务证书请求进行签名，生成具体业务证书。在客户端将根证书加入到授信白名单中，通过其签名的业务证书都会被信任。对自签名根证书自身的认证与保护已不能依赖PKI 体系自身，这份Wiki有讨论: https://en.wikipedia.org/wiki/Self-signed_certificate

## 证书内容

使用如下命令，可以查看证书内容：

```
$ openssl x509 -text -noout -in rootCA.crt 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 12902947919297576859 (0xb3107ac51518ff9b)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, CN=example.com self-signed CA
        Validity
            Not Before: Dec 15 07:14:03 2017 GMT
            Not After : Dec 13 07:14:03 2027 GMT
        Subject: C=CN, CN=example.com self-signed CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c9:5e:b6:04:78:3b:c3:62:ee:86:80:2e:e4:55:
                    6d:4e:14:d5:08:0b:d3:21:1f:9c:f1:8a:7f:e0:27:
                    a7:0f:d1:91:9a:01:96:ff:7c:04:d2:9a:0c:66:bb:
                    4b:c2:00:7d:92:6f:b1:52:ee:15:74:76:7b:36:5b:
                    62:65:19:a0:c6:c1:8b:7a:5a:09:18:91:fc:ef:bd:
                    cc:6b:90:a5:56:4c:71:6f:06:b7:4b:90:9c:d4:9a:
                    a6:63:c1:d5:8a:9e:38:9a:42:23:90:5c:89:e5:e8:
                    aa:1a:45:3b:b5:63:3d:33:99:38:14:00:e9:b2:85:
                    f6:72:ca:ef:00:d9:87:91:ef:49:c4:5c:7f:de:f5:
                    cc:c2:7b:d9:e0:e5:4d:fa:b3:5b:ad:fa:99:9a:0a:
                    4c:47:a5:3d:0b:1f:ef:23:43:86:6b:e4:9b:81:c5:
                    c7:d5:11:7b:d6:e5:bd:1c:b8:f4:a4:52:2e:7d:31:
                    7c:8c:23:0b:c0:a2:11:0e:88:6e:99:54:57:8d:0a:
                    e6:cd:5c:5b:0f:64:be:2c:52:a1:ac:5c:af:c2:48:
                    b3:7b:1e:fc:90:34:98:3a:10:47:b8:ff:6f:d2:de:
                    fa:25:f7:a2:2f:09:55:e4:55:59:17:47:4c:83:04:
                    ef:4e:1b:38:a4:2b:7e:b6:79:5e:97:bd:1e:2f:0b:
                    83:e1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                3F:1A:19:6F:AF:D1:60:C4:65:23:88:31:B5:43:9E:E4:C2:E7:B9:FF
            X509v3 Authority Key Identifier:
                keyid:3F:1A:19:6F:AF:D1:60:C4:65:23:88:31:B5:43:9E:E4:C2:E7:B9:FF

            X509v3 Basic Constraints:
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         7c:5b:bf:d4:77:29:a9:25:4c:60:bf:6f:62:9b:df:e9:83:50:
         bd:eb:b3:8a:03:02:8a:87:4d:e7:f6:ef:e2:44:94:b4:84:11:
         3e:e3:d4:e0:b8:e8:75:23:86:f6:a1:83:2f:cd:5e:25:03:22:
         f8:11:7a:0d:3a:20:7b:8c:e4:e4:73:9d:a6:0c:db:4b:71:6f:
         5b:b5:68:e4:cd:95:5c:a5:d8:7f:d4:55:6f:6f:7a:2d:3f:b1:
         a7:58:57:c9:13:63:a3:c4:87:fc:bd:c6:41:72:32:1e:c9:d6:
         7a:72:70:cb:48:d7:df:5d:e8:eb:5b:9b:ff:35:fa:ad:71:09:
         d3:b2:5c:34:23:d5:2e:29:ce:3a:3f:1e:fc:c2:8f:9e:25:2f:
         27:01:4a:55:bf:ef:33:0c:43:fc:7d:f9:2d:87:79:a7:4a:10:
         03:30:ea:2d:fd:4f:93:12:2a:a7:e2:e1:5e:56:96:2c:33:6a:
         5e:3c:df:39:b5:61:db:d6:36:66:59:91:61:27:ce:a5:79:f6:
         5a:58:27:f9:88:5e:e7:8d:d7:36:96:1f:65:a2:c7:d7:2a:b3:
         99:d8:53:2a:cc:57:8a:09:85:e1:7c:53:fe:26:69:bf:b4:c2:
         94:8a:bb:3e:ef:26:be:ca:39:df:fa:17:91:ca:27:55:8f:ca:
         86:36:ab:3a
```

证书内容按其功能，可以分为：
1. 身份描述信息：描述业务信息，包括 Issuer, Validity, Subject 等。
2. 身份识别信息：Public-Key，公钥与服务器上保存的私钥共同组成了属主在数字世界的身份标示。
3. 认证信息：Signature, 对于自签名证书，签名私钥对应的公钥包含在证书中，因此只能验证证书数据的完整性（Hash是否匹配），不能在PKI体系内对证书来源是否合法进行认证（授信白名单机制）。对于普通证书（由第三方CA机构进行签名），则通过信任的CA 根证书对其进行认证：能使用对应根证书公钥对签名进行解密并验证Hash。

# Create Server Certificate
## 脚本
create_crt.sh

```
#!/bin/sh

KEY_BITS=2048
CRT_DAYS=3650 # invalidate after 10 years

domain="example.com"
subject="/CN=$domain/O=${domain}/C=CN"

echo "Create private key ..."
openssl genrsa -out $domain.key $KEY_BITS

echo "Create certificate sign request ..."
openssl req -new -key $domain.key -sha256 -subj "$subject" -out $domain.csr

echo "Sign the csr with self key ..."
openssl x509 -req -sha256 -extfile extension.conf -in $domain.csr -CA rootCA.crt -CAkey rootCA.key -out $domain.crt -days $CRT_DAYS

cat $domain.crt rootCA.crt > chain.$domain.crt

```

extension.conf 中配置 x509v3 扩展信息，非必须，其内容如下：

```
$ cat extension.conf
subjectAltName=DNS:www.example.com,DNS:*.example.com
```

## 说明
生成普通证书包括三个步骤：
1. openssl genrsa: 生成业务私钥文件（包含公钥、私钥对）；
2. openssl req: 生成证书签名请求文件（CSR, certificate signing request），该文件内包含证书中除了签名之外的信息（身份描述、公钥等）；
3. openssl x509: 使用CA 私钥对CSR 文件进行签名，并增加扩展信息（如有效期、可用站点域名等），生成业务证书；

同样可用 "openssl x509 -text" 命令查看证书内容。

# 配置Web服务器

在上一步生成业务证书后，就可以配置Https 站点了。以nginx 为例说明，首先将上一步生成的服务器私钥与证书拷贝至 nginx 配置目录下（该示例中为 conf/ssl/），然后在nginx.conf中配置https站点。

```
   # https
    server {
        listen 443 ssl;
        ssl_certificate     ssl/example.com.crt;
        ssl_certificate_key ssl/example.com.key;

        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
        ssl_prefer_server_ciphers on;

        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;

        location / {
            proxy_pass http://127.0.0.1:80;
            include proxy_params;
            proxy_set_header X-Forwarded-Proto https;
        }
    }

```

执行 nginx -s reload 就可以生效https 站点了, 此时访问站点浏览器会告警（提示站点不安全），将第一步生成的rootCA.crt 导入到授信的根证书颁发机构即可（或者将服务器证书导入到授信站点）。

# 小结

本文说明了如何使用openssl 工具自建证书，配置Https 站点的步骤（也存在许多其它生成证书的工具），对证书内容也进行了简单介绍：首先生成一个自签名的CA 根证书，使用根证书对业务证书请求进行签名，可以灵活的为业务生成一系列证书。当然，这些证书要被客户端信任，需要将根证书加入到客户端的授信白名单中。

我们当然也可以直接用自签名证书配置Https站点，但如果我们面对的是一个比较复杂的业务，需要为不同的子系统使用独立的证书，就需要将所有这些证书都加入到客户端的授信白名单中，这显然没有之需要对一个根证书进行授信简便，系统的可靠性与维护的复杂性成反比。
