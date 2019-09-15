---
layout: post
title:  "V2ray | Caddy"
date:   2017-11-16 10:35:06
categories:
---

```
bash <(curl -L -s https://install.direct/go.sh)
```

### 服务端

`/etc/v2ray/config.json`
```
{
  "inbounds": [{
    "port": 22393,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "60644d74-2710-4c96-a341-03e4b5475bb7",
          "level": 1,
          "alterId": 64
        }
      ]
    },
"streamSettings": {
      "network": "ws",
        "wsSettings": {
        "path": "/html"
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
### Caddyfile

```
curl https://getcaddy.com | bash -s personal
```

```
***.com { 
        
        proxy / http://127.0.0.1:8080 {
                header_upstream Host {host}
                header_upstream X-Real-IP {remote}
                header_upstream X-Forwarded-For {remote}
                header_upstream X-Forwarded-Proto {scheme}
        }
        proxy /path localhost:81 {
        websocket
        header_upstream -Origin
        }          
}
```
执行`./caddy -conf Caddyfile`


### 客户端

```
{
  "log": {
    "access": "",
    "loglevel": ""
  },
  "inbound": {
    "port": 1080,
    "listen": "127.0.0.1",
    "protocol": "http",
    "settings": {
      "auth": "noauth",
      "udp": false
    }
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "",
          "port": 443,
          "users": [
            {
              "id": "60644d74-2710-4c96-a341-03e4b5475bb7",
              "alterId": 64
            }
          ]
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "security": "tls",
      "tlsSettings": {
        "serverName": ""
      },
      "wsSettings": {
        "path": "/html"
      }
    }
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "direct"
        },
        {
          "type": "chinasites",
          "outboundTag": "direct"
        },
        {
          "type": "chinaip",
          "outboundTag": "direct"
        }
      ]
    }
  }
}


```