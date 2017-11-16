---
layout: post
title:  "Caddy | 甩开 Nghttpx + Squid"
date:   2017-10-20 10:35:06
categories:
---

### caddy
```
http://***.com https://***.com { 
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
执行`./caddy -conf Caddyfile`
