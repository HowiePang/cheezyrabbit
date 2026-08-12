# Xray


**基于 vless + reality + gost 科学上网方案**

## 1、服务端部署

xray官网：https://github.com/XTLS/Xray-core?tab=readme-ov-file

xray详细文档：https://xtls.github.io/

第三方已打包：https://fr1.teddyvps.com/shadowsocks/rhel/el9/x86_64/

服务器配置如下：

```go
cat /etc/xray/config.json
------------------------------
{
  "log": {
    "level": "warning"
  },
  "inbounds": [
    {
      "port": 8443,          //客户端连接端口
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "acab0c96-d304-4b2e-838xxxxxxxxxxxxx1",    // uuid，客户端要保持一致
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none",
        "fallbacks": []
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.kafkatool.com:443",        // 要偷取的域名
          "serverNames": ["www.kafkatool.com"],   // 要偷取的域名
          "privateKey": "aKlY6HItDUh-dZRxxxxxxxxxxxxxxxxxxxxxxxxx",    // 私钥
		      "minClientVer": "1.0.0",
          "shortIds": ["b662c7c3"],      //短码，16进制
          "clientFingerprint": "chrome",  //指纹特征
          "show": false
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}

---------------------
xray x25519    //生成公钥和私钥
xray uuid      //生成uuid
```

### <font color="red">附</font>：prometheus指标暴露配置

```go
{
  "log": {
    "level": "warning"
  },
  "stats": {},
  "api": {
        "tag": "api",
        "services": [
            "StatsService"
        ]
  },
  "policy": {
        "levels": {
            "0": {
                "statsUserUplink": true,
                "statsUserDownlink": true
            }
        },
        "system": {
            "statsInboundUplink": true,
            "statsInboundDownlink": true,
            "statsOutboundUplink": true,
            "statsOutboundDownlink": true
        }
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "xxxxxxxxxxxxxx",
            "flow": "xtls-rprx-vision",
            "email": "howie@hk.local",
            "level": 0
          }
        ],
        "decryption": "none",
        "fallbacks": []
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.avaya.com:443",
          "serverNames": ["www.avaya.com"],
          "privateKey": "xxxxxxxxxxxxxxxxxxx",
          "minClientVer": "1.0.0",
          "shortIds": ["b662c5c7"],
          "clientFingerprint": "chrome",
          "show": false
        }
      }
    },
    // 本地API监听，不会占用外网端口、不影响上网
    {
      "tag": "api",
      "listen": "127.0.0.1",
      "port": 54321,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    },
    // API专用出站规则
    {
      "tag": "api",
      "protocol": "freedom"
    }
  ],
  // 新增路由，仅对内网api流量生效，外网代理毫无影响
  "routing": {
    "rules": [
      {
        "type": "field",
        "inboundTag": ["api"],
        "outboundTag": "api"
      }
    ]
  }
}
```

## 2、客户端配置

Clash Mi 客户端下载地址（全平台开源）：https://clashmi.app/download

Clash Verge Re 桌面全平台下载：https://www.clashverge.dev/install.html

OneXray: https://github.com/OneXray/OneXray

Stash (付费方案)：app store下载即可

客户端配置例子如下 (UI 界面导入)：

```go
# Profile Template for Clash Verge
proxies:
  - name: "HK-REALITY-demo"
    type: vless
    server: xxx.xxx.xxx.xxx
    port: 8443
    uuid: "acab0c96-d304-4b2e-838xxxxxxxxxxxxx1"
    network: tcp
    tls: true
    servername: "www.kafkatool.com"
    flow: xtls-rprx-vision
    reality-opts:
      public-key: "SzECFLQ_bN9r5uZhxEHjuxxxxxxxxxxxxxxx"
      short-id: "b662c7c3"
    client-fingerprint: chrome

proxy-groups:
  - name: "AUTO"
    type: select
    proxies:
      - "HK-REALITY-demo"

rules:
  - MATCH, AUTO
```

### <font color="red">附</font>：xray客户端配置（Linux无 UI 环境）

桌面版本参考上面即可，此处主要介绍在Linux环境下配置客户端，笔者的环境为国内服务器，Rocky Liunx9.8系统。

客户端配置 (同时代理 http 和 socks 端口)：

```go
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "listen": "本机内网ip",
            "port": 1080,
            "protocol": "socks",
            "settings": {
                "udp": true
            }
        },
        {
            "listen": "本机内网ip",
            "port": 1081,
            "protocol": "http"
        }
    ],
    "outbounds": [
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "服务端ip",
                        "port": 443,
                        "users": [
                            {
                                "id": "c40bcxxxxxxx",
                                "encryption": "none",
                                "flow": "xtls-rprx-vision"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false,
                    "fingerprint": "chrome",
                    "serverName": "www.avaya.com",
                    "publicKey": "qIzG5JoBqnxxxxxxxxxxxxxxxxxx",
                    "shortId": "b662c5c7",
                    "spiderX": ""
                }
            },
            "tag": "proxy"
        },
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ]
}
```

可以通过 Docker 代理拉取镜像和 “curl -I www.google.com” 命令检验即可，相关可见链接（linux客户端附加部分）：https://wiki.uhowie.com/tech/servers_deploy/bypass2/shadowsocks

## 3、中转方案

（1）采用 Gost 中转到另一个 vless+reality 服务

Gost下载地址：https://github.com/go-gost/gost/releases

​              https://github.com/go-gost/gost/releases/download/v3.2.6/gost_3.2.6_linux_amd64.tar.gz

Gost的服务配置如下：

```go
// cat /etc/systemd/system/gost.service
[Unit]
Description=gost proxy chain
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/gost -C /etc/gost/config.yaml
Restart=always
User=root
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```go
// cat /etc/gost/config.yaml
services:
- name: la01
  addr: :8080
  handler:
    type: tcp
  listener:
    type: tcp
  forwarder:
    nodes:
    - name: la01
      addr: xxx.xxx.xxx.xxx:443
log:
  level: warn
```

（2）采用 Gost 中转到另一个 socks 服务

服务端配置：

```go
cat /etc/xray/config-socks.json
------------------------
{
  "log": {
    "level": "warning"
  },
  "inbounds": [
    {
      "port": 8081,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "c40c1e95-2c21-xxxxxxxxxxxxxxxxxxxx",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none",
        "fallbacks": []
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.kafkatool.com:443",
          "serverNames": ["www.kafkatool.com"],
          "privateKey": "IFjsZykhRo25xxxxxxxxxxxxxxxxxxxx",
		      "minClientVer": "1.0.0",
          "shortIds": ["a462baaa"],
          "clientFingerprint": "chrome",
          "show": false
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "xxx.xxx.xxx.xxx",
            "port": xxxxx,
            "users": [
              {
                "user": "xxxxxxx",
                "pass": "xxxxxxx"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

