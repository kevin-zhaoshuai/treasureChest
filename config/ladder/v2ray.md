# Setup a V2ray + tls + Nginx + ws from Scratch
## Prequsition
### VPS and a domain
VPS, Buy one.
Domain, I use Godaddy to offer a domain which is $0.99 per year, while it also offer DNS support for this domain.
You do not need any more steps with this.

### HTTPS/TLS cert
Use Let's Encrypt to get free cert for you domain
Stop Nginx first, as the certbot operation need to use http to verify the certification for the domain. So
stop the process which is occupying the port 80. If you are purchase vpn from cloud vendors, please also make sure
that the 80 port is permitted access by the security group.

Run command to get cert:
```
certbot certonly --standalone -d tlanyan.me -d www.tlanyan.me
```
replace the tlanyan.me to your domain.
After that you will get the cert file at /etc/letsencrypt/live

Refer: [Get free cert](https://tlanyan.me/use-lets-encrypt-certificate/)

### V2ray setup
Very easy.
```
bash <(curl -L -s https://install.direct/go.sh)
```

vim /etc/v2ray/config.json, add StreamSettings like this:
```
{
  "inbounds": [{
    "port": 20892,           //此处为安装时生成的端口，可修改随意，但是保证和下面提到的端口号相同
    "listen":"127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "xxxxxxxxx", //此处为安装时生成的id
          "level": 1,
          "alterId": 64      //此处为安装时生成的alterId
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/SoftDown"   //此处为路径，需要和下面NGINX上面的路径配置一样
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```
```
systemctl enable v2ray
systemctl start v2ray
```

### Configure Nginx

vim /etc/nginx/conf.d/default.conf
```
server {
    listen       443 ssl;
    server_name  bozaibozai.ml;                  #修改为自己的域名
    ssl_certificate /etc/letsencrypt/live/*****/fullchain.pem;       #替换为自己的fullchain地址
    ssl_certificate_key /etc/letsencrypt/live/*****/privkey.pem;   #替换为自己的key地址
 
    location /SoftDown {                         #修改为你自己的路径，需要和V2RAY里面的路径一样
        proxy_redirect off;
        proxy_pass http://127.0.0.1:20892;       #修改为你自己的v2ray服务器端口，就是这里需要和上面V2RAY配置文件里面的端口号相同。
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 60s;
        proxy_read_timeout 86400s;
        proxy_send_timeout 60s;
    }
 
}
```
systemctl restart nginx

### Configure the v2ray Client
Refer to https://tlanyan.me/v2ray-traffic-mask/

