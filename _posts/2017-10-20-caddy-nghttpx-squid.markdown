---
layout: post
title:  "Caddy | nghttpx+squid"
date:   2017-10-20 10:35:06
categories:
---

### caddy
```
http://***.com https://***.com {  # 同时启用 http 和 https 不会自动转跳
        gzip
        forwardproxy {
        basicauth caddyuser1 0NtCL2JPJBgPPMmlPcJ
        basicauth caddyuser2 key
        }
        proxy / http://127.0.0.1:8080 {
                header_upstream Host {host}
                header_upstream X-Real-IP {remote}
                header_upstream X-Forwarded-For {remote}
                header_upstream X-Forwarded-Proto {scheme}
        }
}
```
执行`./caddy -conf caddyfile`
