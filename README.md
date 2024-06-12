# GO Simple Tunnel - 威力加强版

## 加强特性

- QUIC：默认改用 BBR，加入 [Hysteria](https://hysteria.network/) 同款 [Brutal](https://hysteria.network/zh/docs/misc/Hysteria-Brutal/) 固定速率流控
- REALITY：支持 [Xray](https://xtls.github.io) 同款 [REALITY](https://github.com/XTLS/REALITY) 安全传输层，可作为服务器及客户端，支持多路复用
- Forward：流量转发支持 Proxy Protocol 传递客户端IP（仅支持TCP）

## 配置示例

### QUIC + Brutal

最简配置命令行（服务端）

`gost -L http+quic://:6666?recvMbps=100`

最简配置命令行（客户端）

`gost -L http://:10880 -F http+quic://myserver.ltd:6666?sendMbps=100`

详细配置（服务端）

```yaml
services:
- name: service-0
  addr: :6666
  handler:
    type: http
  listener:
    type: quic
    metadata:
      recvMbps: 100
      maxStreams: 1024
      initStreamReceiveWindow: 8388608
      maxStreamReceiveWindow: 8388608
      initConnReceiveWindow: 20971520
      maxConnReceiveWindow: 20971520
      DisablePathMTUDiscovery: false
```

详细配置（客户端）

```yaml
services:
- name: service-0
  addr: ":10880"
  handler:
    type: http
    chain: chain-0
  listener:
    type: tcp
chains:
- name: chain-0
  hops:
  - name: hop-0
    nodes:
    - name: node-0
      addr: myserver.ltd:6666
      connector:
        type: http
      dialer:
        type: quic
        metadata:
          sendMbps: 100
          maxStreams: 1024
          initStreamReceiveWindow: 8388608
          maxStreamReceiveWindow: 8388608
          initConnReceiveWindow: 20971520
          maxConnReceiveWindow: 20971520
          DisablePathMTUDiscovery: false
```

> [!NOTE]  
> 具体参数请参考 [Hysteria 文档](https://v2.hysteria.network/zh/docs/advanced/Full-Server-Config/#quic) QUIC 配置部分，参数含义一致

> [!WARNING]  
> 使用 Brutal 固定速率发送时务必提前确认服务器参数，错误的配置会导致网络拥堵甚至崩溃，也可能会因为暴力发包而被主机商停用或运营商限制网络

### REALITY

最简配置命令行（服务端）

`gost -L="http+tlr://:8888?dest=ocsp.apple.com:443&privateKey=OAKBrpGrtUUGzdYbtqNSMrBeAbLZfmd6w3tdesPCPUo&serverNames=ocsp.apple.com&shortIds=cafe123456780000"`

> [!TIP]
> 拨号器名称 可使用 `tlr` 或 `reality`，如多路复用可使用 `mtlr` 或 `mreality`
> TLR 为 *Transport Layer REALITY*（传输层现实协议） 的缩写

最简配置命令行（客户端）

`gost -L http://:10880 -F="http+tlr://myserver.tld:8888?serverName=ocsp.apple.com&publicKey=XslBGcN9ChALOZRiDWmh6CfgwvJ8n9cx65egDAnNUzQ&shortId=cafe123456780000"`

详细配置（服务端）

```yaml
services:
- name: service-0
  addr: :8888
  handler:
    type: http
  listener:
    type: reality
    reality:
      show: false
      xver: 0
      dest: "ocsp.apple.com:443"
      privateKey: "OAKBrpGrtUUGzdYbtqNSMrBeAbLZfmd6w3tdesPCPUo"
      maxTimeDiff: 30m
      serverNames:
        - "ocsp.apple.com"
      shortIds:
        - "cafe123456780000"
```

详细配置（客户端）

```yaml
services:
- name: service-0
  addr: ":10880"
  handler:
    type: http
    chain: chain-0
  listener:
    type: tcp
chains:
- name: chain-0
  hops:
  - name: hop-0
    nodes:
    - name: node-0
      addr: myserver.tld:8888
      connector:
        type: http
      dialer:
        type: reality
        reality:
          show: false
          serverName: ocsp.apple.com
          publicKey: XslBGcN9ChALOZRiDWmh6CfgwvJ8n9cx65egDAnNUzQ
          shortId: cafe123456780000
```

> [!NOTE]  
> 具体参数请参考 [Xray 文档](https://xtls.github.io/config/transport.html#realityobject) RealityObject 配置部分，参数含义一致

### Forward 发送 Proxy Protocol

仅支持在配置文件中使用，仅支持 TCP 协议

```yaml
services:
- name: service-0
  addr: :8080
  handler:
    type: tcp
  listener:
    type: tcp
  forwarder:
    nodes:
    - name: target-0
      addr: 192.168.1.1:80
  metadata:
    sendProxy: 2 # 0: 不发送，1: 发送 Proxy Protocol v1，2: 发送 Proxy Protocol v2
```

兼容所有代理链，会在收到转发数据时附加 Proxy Protocol 头部，转发给目标节点

### 相关项目

- [Hysteria](https://v2.hysteria.network/) - 强大、快速、抗审查的代理工具
- [quic-go](https://github.com/mzz2017/quic-go) - 修改版 quic-go，支持 Hysteria 的 Brutal 流控 及 BBR
- [Xray](https://xtls.github.io/) - 网络代理工具合集
- [REALITY](https://github.com/XTLS/REALITY) - 一个用于隐匿特征的安全传输层

------

以下是原介绍

### GO语言实现的安全隧道

[![zh](https://img.shields.io/badge/Chinese%20README-green)](README.md) [![en](https://img.shields.io/badge/English%20README-gray)](README_en.md)

## 功能特性

- [x] [多端口监听](https://gost.run/getting-started/quick-start/)
- [x] [多级转发链](https://gost.run/concepts/chain/)
- [x] [多协议支持](https://gost.run/tutorials/protocols/overview/)
- [x] [TCP/UDP端口转发](https://gost.run/tutorials/port-forwarding/)
- [x] [反向代理](https://gost.run/tutorials/reverse-proxy/)和[隧道](https://gost.run/tutorials/reverse-proxy-tunnel/)
- [x] [TCP/UDP透明代理](https://gost.run/tutorials/redirect/)
- [x] DNS[解析](https://gost.run/concepts/resolver/)和[代理](https://gost.run/tutorials/dns/)
- [x] [TUN/TAP设备](https://gost.run/tutorials/tuntap/)
- [x] [负载均衡](https://gost.run/concepts/selector/)
- [x] [路由控制](https://gost.run/concepts/bypass/)
- [x] [准入控制](https://gost.run/concepts/admission/)
- [x] [限速限流](https://gost.run/concepts/limiter/)
- [x] [插件系统](https://gost.run/concepts/plugin/)
- [x] [Prometheus监控指标](https://gost.run/tutorials/metrics/)
- [x] [动态配置](https://gost.run/tutorials/api/config/)
- [x] [Web API](https://gost.run/tutorials/api/overview/)
- [ ] GUI/WebUI

## 概览

![Overview](https://gost.run/images/overview.png)

GOST作为隧道有三种主要使用方式。

### 正向代理

作为代理服务访问网络，可以组合使用多种协议组成转发链进行转发。

![Proxy](https://gost.run/images/proxy.png)

### 端口转发

将一个服务的端口映射到另外一个服务的端口，同样可以组合使用多种协议组成转发链进行转发。

![Forward](https://gost.run/images/forward.png)

### 反向代理

利用隧道和内网穿透将内网服务暴露到公网访问。

![Reverse Proxy](https://gost.run/images/reverse-proxy.png)

## 下载安装

### 二进制文件

[https://github.com/go-gost/gost/releases](https://github.com/go-gost/gost/releases)

### 安装脚本

```bash
# 安装最新版本 [https://github.com/go-gost/gost/releases](https://github.com/go-gost/gost/releases)
bash <(curl -fsSL https://github.com/go-gost/gost/raw/master/install.sh) --install
```
```bash
# 选择要安装的版本
bash <(curl -fsSL https://github.com/go-gost/gost/raw/master/install.sh)
```

### 源码编译

```
git clone https://github.com/go-gost/gost.git
cd gost/cmd/gost
go build
```

### Docker

```
docker run --rm gogost/gost -V
```

## 工具

### GUI

[go-gost/gostctl](https://github.com/go-gost/gostctl)

### WebUI

[cnwhy/gost-ui](https://github.com/cnwhy/gost-ui)

### Shadowsocks Android插件

[xausky/ShadowsocksGostPlugin](https://github.com/xausky/ShadowsocksGostPlugin)

## 帮助与支持

Wiki站点：[https://gost.run](https://gost.run)

YouTube: [https://www.youtube.com/@gost-tunnel](https://www.youtube.com/@gost-tunnel)

Telegram：[https://t.me/gogost](https://t.me/gogost)

Google讨论组：[https://groups.google.com/d/forum/go-gost](https://groups.google.com/d/forum/go-gost)

旧版入口：[v2.gost.run](https://v2.gost.run)
