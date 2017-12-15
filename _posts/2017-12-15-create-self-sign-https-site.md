建立自签名证书 Https 站点
==========================

# 简介

建立自签名证书 Https 站点可以分为三步:

1. 生成自签名 RootCA 证书
2. 为web站点生成证书
3. 将证书配置到Web服务器

# Create RootCA Certificate

create_root_ca_crt.sh

```
#!/bin/sh


subject="/C=CN/CN=example.net self-signed CA"
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -subj "$subject" -out rootCA.crt

```

# Create Server Certificate

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

证书生成完毕后，可以使用如下命令查看内容：

```
$ openssl x509 -text -noout -in example.com.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 15976749170229864979 (0xddb8d0b8cd116a13)
    Signature Algorithm: sha256WithRSAEncryption
    ... ...
```

# 配置Web服务器

以nginx 为例，首先将上一步生成的服务器私钥与证书拷贝至 nginx 配置目录下（该示例中为 conf/ssl/），然后在nginx.conf中配置https站点：

```
   # https
    server {
        listen 443 ssl;
        ssl_certificate     ssl/battlecare.net.crt;
        ssl_certificate_key ssl/battlecare.net.key;

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

# end
