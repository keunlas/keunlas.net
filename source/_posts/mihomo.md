---
title: Mihomo 纯内核运行
date: 2025-04-17 22:01:16
tags: 
  - mihomo
  - archlinux
categories: 
  - 折腾
comments: true
abbrlink: a2809040
---

因为在 Archlinux 上使用 Clash Verge Rev 经常会出现一些奇怪的问题，比如说 service 无法运行，界面卡顿加载不出来，甚至会直接卡死。所以很早之前我就有了直接跑纯内核的念头。

故现在我终于开始尝试直接跑纯 Mihomo 内核，目前是已经完成了所有的配置，普通模式和 Tun 模式都可以正常使用。现将过程记录如下。

由于我是在 Archlinux 上进行的安装，我可以确保在 Archlinux 上以下操作全部经过检验。

# 安装 mihomo 核心及其依赖

实际上包管理器会自动处理依赖，所以不用担心。

```bash
yay -S mihomo
```

# 安装 metacubexd 界面

为了能够在 web 端方便的临时调整配置，以及可视化连接及流量，我推荐安装 metacubexd 这个界面。

如果你配置了 archlinuxcn 源，可以直接 pacman 安装预编译版本。

```bash
sudo pacman -S metacubexd-bin 
```

也可以在 AUR 中安装，metacubexd-bin 是预编译版本，metacubexd 是源码需要本机上编译。我比较推荐预编译版本。

```bash
yay -S metacubexd-bin
```

# 启动守护进程

AUR 中提供的 mihomo 软件包里面自带了 systemd 守护进程 mihomo.service，我们需要手动 enable 让 mihomo 内核可以开机自启。

```bash
sudo systemctl enable mihomo.service
```

然后启动 mihomo 内核进行下一步工作。

```bash
sudo systemctl start mihomo.service
```

# 配置文件

通过查看守护进程文件 /etc/systemd/system/multi-user.target.wants/mihomo.service 可以看到我的 mihomo 的工作目录为 /etc/mihomo, mihomo 内核的配置文件在 /etc/mihomo/config.yaml 中，我们需要修改这个文件来配置我们的 mihomo 内核。

config.yaml 这个文件就是最主要的配置文件，只不过我们需要修改一些内容。来适合我的使用习惯。

首先把 config.yaml 中的所有内容清空，然后把机场给出的配置文件整个的复制过来。

接下来就是修改修改了。

## 修改端口

因为 Clash Verge Rev 的默认 mixed-port 是 7897, 我的很多软件的代理都设置为了这个端口，所以我也依旧修改 mixed-port 为 7897.

并且我不喜欢开启局域网访问，所以我将 allow-lan 修改为 false.

现在，我的 config.yaml 大概是这个样子。

```yaml
mixed-port: 7897
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

proxies:
  # ...
proxy-groups:
  # ...
rules:
  # ...
```

我们需要重启 mihomo 内核，让配置生效。

```bash
sudo systemctl restart mihomo.service
```

这个时候，mihomo 就已经可以正常的使用了。

但是 Tun 模式目前还用不了，接下来我们继续设置。


## 配置 metacubexd 界面

```yaml
# 前端控制器界面 (archlinuxcn 源中的 metacubexd)
external-ui: /usr/share/metacubexd
```

只需要在 config.yaml 中加上这一行就可以，archlinuxcn 源中的 metacubexd-bin 的目录是在 /usr/share/metacubexd 中。AUR 中的我没有尝试过，大家下载下来之后可以使用 `pacman -Ql metacubexd-bin` 查看一下具体的路径。

然后我们需要重启 mihomo 内核，让配置生效。

```bash
sudo systemctl restart mihomo.service
```

然然后我们就可以在浏览器中通过配置文件中 `external-controller: 127.0.0.1:9090` 的地址来访问 metacubexd 界面了。

记得加上 `/ui` 这个后缀，否则进不去 metacubexd 的界面。

```
http://127.0.0.1:9090/ui
```

那么现在我们的配置就都完成的差不多了， config.yaml 大概是这个样子。

```yaml
mixed-port: 7897
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

# 前端控制器界面 (archlinuxcn 源中的 metacubexd)
external-ui: /usr/share/metacubexd

proxies:
  # ...
proxy-groups:
  # ...
rules:
  # ...
```

# 高级配置

## DNS 配置

根据我的测试，不配置 dns 就无法使用 Tun 模式，所以我们这里直接给出我已经配置好的配置，大家可以直接复制粘贴到 config.yaml 中。

具体的选项起到什么作用其实还是很直接的，有想自己自定义的可以查看官方 Wiki 上的配置选项。

```yaml
# DNS
dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip # redir-host
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - geosite:fakeip-filter
  default-nameserver:
    - system
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query
```

## Tun 模式配置

Tun 模式的具体配置我没有在 config.yaml 中进行配置，而是直接使用的默认配置，在 web 界面上直接开启 Tun 模式即可使用。

## 其他配置

我还从 Wiki 上找了几个比较有用的配置

```yaml
# web 界面的登录密码
# (因为我这里监听的 127.0.0.1 只有本机能够访问, 暂不设置)
# secret: ""

#【Mihomo专属】TCP连接并发，如果域名解析结果对应多个IP，
# 并发所有IP，选择握手最快的IP进行连接
tcp-concurrent: true

# 配置缓存 (代理中选择的节点和 fake-ip 缓存)
profile:
  store-selected: true
  store-fake-ip: true

#【Mihomo专属】使用geoip.dat数据库( 默认：false 使用 mmdb 数据库)
geodata-mode: true
geo-auto-update: true # 自动更新
geo-update-interval: 24 # 更新间隔 24 小时
geox-url: # 下载geoip.dat数据库的地址
  geoip: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geoip-all.dat"
  geosite: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat"
  mmdb: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-all.mmdb"
  asn: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-ASN-all.mmdb"
```


# 最终配置

最后的配置大概长这个样子。

```yaml
mixed-port: 7897
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

# 前端控制器界面 (archlinuxcn 源中的 metacubexd)
external-ui: /usr/share/metacubexd

# web 界面的登录密码
# (因为我这里监听的 127.0.0.1 只有本机能够访问, 暂不设置)
# secret: ""

#【Mihomo专属】TCP连接并发，如果域名解析结果对应多个IP，
# 并发所有IP，选择握手最快的IP进行连接
tcp-concurrent: true

# 配置缓存 (代理中选择的节点和 fake-ip 缓存)
profile:
  store-selected: true
  store-fake-ip: true

#【Mihomo专属】使用geoip.dat数据库( 默认：false 使用 mmdb 数据库)
geodata-mode: true
geo-auto-update: true # 自动更新
geo-update-interval: 24 # 更新间隔 24 小时
geox-url: # 下载geoip.dat数据库的地址
  geoip: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geoip-all.dat"
  geosite: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat"
  mmdb: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-all.mmdb"
  asn: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-ASN-all.mmdb"


# DNS
dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip # redir-host
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - geosite:fakeip-filter
  default-nameserver:
    - system
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query
  # nameserver-policy:
  #   "geosite:cn,private":
  #     - https://doh.pub/dns-query
  #     - https://dns.alidns.com/dns-query
  #   "geosite:ads":
  #     - rcode://success
  #   "geosite:!cn,!private,!ads":
  #     - https://dns.google/dns-query
  #     - https://cloudflare-dns.com/dns-query


proxies:
  # ...
proxy-groups:
  # ...
rules:
  # ...
```

在 dns 配置中，可以配置 nameserver-policy 来实现不同的域名使用不同的 DNS 服务器。可以非常有效的解决 dns 泄漏问题，但是不知道为什么，我用起来网址解析的非常慢，所以我没有使用这个配置，而是注释掉了。

现在我的配置已经可以正常使用了，无论是走端口代理，还是直接 Tun 模式接管一切都没有任何问题了。


# 手写脚本实现一键更新订阅

首先把需要附加的配置写在 append.yaml 中。

```yaml
# append.yaml

mixed-port: 7897
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

# 前端控制器界面 (archlinuxcn 源中的 metacubexd)
external-ui: /usr/share/metacubexd

# web 界面的登录密码
# (因为我这里监听的 127.0.0.1 只有本机能够访问, 暂不设置)
# secret: ""

#【Mihomo专属】TCP连接并发，如果域名解析结果对应多个IP，
# 并发所有IP，选择握手最快的IP进行连接
tcp-concurrent: true

# 配置缓存 (代理中选择的节点和 fake-ip 缓存)
profile:
  store-selected: true
  store-fake-ip: true

#【Mihomo专属】使用geoip.dat数据库( 默认：false 使用 mmdb 数据库)
geodata-mode: true
geo-auto-update: true # 自动更新
geo-update-interval: 24 # 更新间隔 24 小时
geox-url: # 下载geoip.dat数据库的地址
  geoip: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geoip-all.dat"
  geosite: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat"
  mmdb: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-all.mmdb"
  asn: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-ASN-all.mmdb"


# DNS
dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip # redir-host
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - geosite:fakeip-filter
  default-nameserver:
    - system
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query
  # nameserver-policy:
  #   "geosite:cn,private":
  #     - https://doh.pub/dns-query
  #     - https://dns.alidns.com/dns-query
  #   "geosite:ads":
  #     - rcode://success
  #   "geosite:!cn,!private,!ads":
  #     - https://dns.google/dns-query
  #     - https://cloudflare-dns.com/dns-query
```

因为我的订阅中已经有了一些配置，这些配置会和 append.yaml 中的配置冲突，所以通过 sed 进行删除处理。

```bash
#!/usr/bin/bash

# 获取机场配置，去除前6行，保存到 tmp.yaml
curl -L '<你的订阅地址>' | sed '1,6d' > tmp.yaml

# 附加 append.yaml 到 tmp.yaml
cat append.yaml >> tmp.yaml

# 原来配置的备份到 config.yaml.bak
cp config.yaml config.yaml.bak

# 用 tmp.yaml 替换 config.yaml
cat tmp.yaml > config.yaml

```

之后使用 `sudo systemctl restart mihomo` 重启一下 mihomo 服务，或者在 web 界面中重新读取一下配置即可完成更新。


# 其他

1. 第一次启动的时候 mihomo 会下载 mmdb，可能会因为网络问题下载失败，从而没法成功运行。可以先设置好 geox-url 的 cdn 地址，然后重启即可下载成功。

```yaml
geox-url: # 下载geoip.dat数据库的地址
  geoip: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geoip-all.dat"
  geosite: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat"
  mmdb: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-all.mmdb"
  asn: "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country-ASN-all.mmdb"
```

2. Tun 模式下 ssh 之类的软件都无法使用，因为连接的 ip 都会变成 198.18.0.1 这个地址。经过评论区中大佬的提醒，我找到了一个解决方法。在 dns 配置的 `fake-ip-filter:` 这一项中，把想要 ssh 连接的域名和 ip 填上去就行。否则无论是通过域名进行连接还是直接通过 ip 连接，都会连接不上。

3. 经评论区中大佬提醒，纯内核也支持配置订阅连接。经过一番折腾过后，我发现虽然支持订阅连接，但是 proxy-providers 似乎只导入其中的 proxies，但是对我来说 proxy-groups 和 rules 也是非常需要的，故暂不使用此功能。
