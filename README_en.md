# GO Simple Tunnel - Over Powered Edition

## Enhanced Features

- QUIC: Default to BBR, with [Brutal](https://hysteria.network/zh/docs/misc/Hysteria-Brutal/) fixed-rate congestion control by [Hysteria](https://hysteria.network/)
- REALITY: Support Transport Layer [REALITY](https://github.com/XTLS/REALITY) by [Xray](https://xtls.github.io), can be used as server and client, supports multiplexing
- Forward: Traffic forwarding supports Proxy Protocol to pass client IP (TCP only)

## Configuration Example

### QUIC + Brutal

CLI (Server)

`gost -L http+quic://:6666?recvMbps=100`

CLI (Client)

`gost -L http://:10880 -F http+quic://myserver.ltd:6666?sendMbps=100`

YAML (Server)

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

YAML (Client)

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
> Please refer to [Hysteria Documentation](https://v2.hysteria.network/docs/advanced/Full-Server-Config/#quic-parameters) QUIC Configuration section for specific parameters with the same meaning.

> [!WARNING] 
> It is important to check the server parameters in advance when using Brutal congestion, as misconfiguration can lead to network crashes, as well as deactivation by the hosting provider or network restriction by the operator due to brute force packet sending.

### REALITY

CLI (Server)

`gost -L="http+tlr://:8888?dest=ocsp.apple.com:443&privateKey=OAKBrpGrtUUGzdYbtqNSMrBeAbLZfmd6w3tdesPCPUo&serverNames=ocsp.apple.com&shortIds=cafe123456780000"`

> [!TIP]
> Dialer name can be `tlr` or `reality`, and `mtlr` or `mreality` for multiplexing
> TLR stands for *Transport Layer REALITY*

CLI (Client)

`gost -L http://:10880 -F="http+tlr://myserver.tld:8888?serverName=ocsp.apple.com&publicKey=XslBGcN9ChALOZRiDWmh6CfgwvJ8n9cx65egDAnNUzQ&shortId=cafe123456780000"`

YAML (Server)

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

YAML (Client)

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
> Please refer to [Xray Documentation](https://xtls.github.io/config/transport.html#realityobject) RealityObject Configuration section for specific parameters, which have the same meaning.

### Forward with Proxy Protocol

Supported only in configuration files, TCP protocol only

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
    sendProxy: 2 # 0: disabled, 1: Proxy Protocol v1, 2: Proxy Protocol v2
```

Compatible with all proxy chains, the Proxy Protocol header is appended to the forwarded data when it is received and forwarded to the target node.


## Related Projects

- [Hysteria](https://v2.hysteria.network/) - A powerful, lightning fast and censorship resistant proxy.
- [quic-go](https://github.com/mzz2017/quic-go) - Modified version of quic-go to support Hysteria's Brutal and BBR congestion.
- [Xray](https://xtls.github.io/) - a set of network proxy tools
- [REALITY](https://github.com/XTLS/REALITY) - A secure transport layer for network traffic concealment

------

Here is the original README

# GO Simple Tunnel

### A simple security tunnel written in golang

[![en](https://img.shields.io/badge/English%20README-green)](README_en.md) [![zh](https://img.shields.io/badge/Chinese%20README-gray)](README.md)

## Features

- [x] [Listening on multiple ports](https://gost.run/en/getting-started/quick-start/)
- [x] [Multi-level forwarding chain](https://gost.run/en/concepts/chain/)
- [x] Rich protocol
- [x] [TCP/UDP port forwarding](https://gost.run/en/tutorials/port-forwarding/)
- [x] [Reverse Proxy](https://gost.run/en/tutorials/reverse-proxy/) and [Tunnel](https://gost.run/en/tutorials/reverse-proxy-tunnel/)
- [x] [TCP/UDP transparent proxy](https://gost.run/en/tutorials/redirect/)
- [x] DNS [resolver](https://gost.run/en/concepts/resolver/) and [proxy](https://gost.run/en/tutorials/dns/)
- [x] [TUN/TAP device](https://gost.run/en/tutorials/tuntap/)
- [x] [Load balancing](https://gost.run/en/concepts/selector/)
- [x] [Routing control](https://gost.run/en/concepts/bypass/)
- [x] [Admission control](https://gost.run/en/concepts/limiter/)
- [x] [Bandwidth/Rate Limiter](https://gost.run/en/concepts/limiter/)
- [x] [Plugin System](https://gost.run/en/concepts/plugin/)
- [x] [Prometheus metrics](https://gost.run/en/tutorials/metrics/)
- [x] [Dynamic configuration](https://gost.run/en/tutorials/api/config/)
- [x] [Web API](https://gost.run/en/tutorials/api/overview/)
- [ ] GUI/WebUI

## Overview

![Overview](https://gost.run/images/overview.png)

There are three main ways to use GOST as a tunnel.

### Proxy

As a proxy service to access the network, multiple protocols can be used in combination to form a forwarding chain for traffic forwarding.

![Proxy](https://gost.run/images/proxy.png)

### Port Forwarding

Mapping the port of one service to the port of another service, you can also use a combination of multiple protocols to form a forwarding chain for traffic forwarding.

![Forward](https://gost.run/images/forward.png)

### Reverse Proxy

Use tunnel and intranet penetration to expose local services behind NAT or firewall to public network for access.

![Reverse Proxy](https://gost.run/images/reverse-proxy.png)

## Installation

### Binary files

[https://github.com/go-gost/gost/releases](https://github.com/go-gost/gost/releases)

### install script

```bash
# install latest from [https://github.com/go-gost/gost/releases](https://github.com/go-gost/gost/releases)
bash <(curl -fsSL https://github.com/go-gost/gost/raw/master/install.sh) --install
```
```bash
# select version for install 
bash <(curl -fsSL https://github.com/go-gost/gost/raw/master/install.sh)
```

### From source

```
git clone https://github.com/go-gost/gost.git
cd gost/cmd/gost
go build
```

### Docker

```
docker run --rm gogost/gost -V
```

## Tools

### GUI

[go-gost/gostctl](https://github.com/go-gost/gostctl)

### WebUI

[cnwhy/gost-ui](https://github.com/cnwhy/gost-ui)

### Shadowsocks Android

[xausky/ShadowsocksGostPlugin](https://github.com/xausky/ShadowsocksGostPlugin)

## Support

Wiki: [https://gost.run](https://gost.run/en/)

YouTube: [https://www.youtube.com/@gost-tunnel](https://www.youtube.com/@gost-tunnel)

Telegram: [https://t.me/gogost](https://t.me/gogost)

Google group: [https://groups.google.com/d/forum/go-gost](https://groups.google.com/d/forum/go-gost)

Legacy version: [v2.gost.run](https://v2.gost.run/en/)
