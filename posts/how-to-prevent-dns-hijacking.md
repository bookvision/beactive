传统的 DNS 协议是明文传输，所以在糟糕的网络环境中，DNS 劫持（也称作“DNS 投毒”和“DNS 污染”）是经常遇到的问题。构建一个可靠安全的 DNS 服务是稳定网络环境的基础。本文介绍在 Linux 上自建加密 DNS 服务器来避免遭遇 DNS 劫持的方法。

加密 DNS 协议有很多种，例如：DoH（DNS over HTTPS）、DoT（DNS over TLS）、DNSCrypt 等。这些协议虽然各有优劣，但是都可满足基础的防劫持的要求。支持加密 DNS 的软件也很丰富。本文选择 dnscrypt-proxy，因为其配置简单，开箱即用。

目前主流的 Linux 发行版都可顺利安装和运行 dnscrypt-proxy，本文简要描述基础的安装过程。详细的安装和配置手册请参考：[这里](https://github.com/DNSCrypt/dnscrypt-proxy/wiki)

dnscrypt-proxy 的可执行程序在[这里](https://github.com/DNSCrypt/dnscrypt-proxy/releases)下载。下载完成后解压缩至任意目录，简单起见，可以解压至 Home 目录下。解压后将得到 dnscrypt-proxy 的程序目录，包含 dnscrypt-proxy 的可执行文件和配置例程。

打开终端，进入 dnscrypt-proxy 程序目录。将 example-dnscrypt-proxy.toml 文件复制一份，改名为 dnscrypt-proxy.toml。

```
$ cp example-dnscrypt-proxy.toml dnscrypt-proxy.toml
```

在终端中直接启动 dnscrypt-proxy 程序，dnscrypt-proxy 程序将自动读取当前目录下的 dnscrypt-proxy.toml 配置文件。

```
$ sudo ./dnscrypt-proxy
```

不出意外的话，在终端中会看到 dnscrypt-proxy 连接上游服务器的日志。当看到以下输出时，表明 DNS 服务已经可用了。

```
[2025-10-22 14:32:17] [NOTICE] -   475ms dnscry.pt-johor-ipv4
[2025-10-22 14:32:17] [NOTICE] Server with the lowest initial latency: dnscry.pt-hanoi-ipv4 (rtt: 83ms)
[2025-10-22 14:32:17] [NOTICE] dnscrypt-proxy is ready - live servers: 201
```

接下来可以测试一下。再打开一个终端，使用 dig 命令做一次 DNS 解析。

```

$ dig google.com @127.0.0.1

; <<>> DiG 9.20.11-4-Debian <<>> google.com @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20658
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             2394    IN      A       142.250.73.78

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Wed Oct 22 15:24:51 CST 2025
;; MSG SIZE  rcvd: 55
```

需要注意的是：有些发行版中，systemd 内置了 DNS 解析服务，并且绑定在本机的 53 端口上，所以需要关闭它。总之默认配置需要监听 53 端口，所以需要确保其没有被其他程序占用。

由于 DoH 基于 Https 协议，所以连接 DoH 服务器钱也需要对其地址进行 DNS 解析。dnscrypt-proxy 通过配置文件中 bootstrap_resolvers 配置项设置其初始化时使用的 DNS 服务器。须确保能够连接到这些服务器。另外，DNSCrypt 在 github 上维护了一份公共 DoH 服务器的列表，dnscrypt-proxy 在启动时，会下载这份列表。如果无法下载，需要手动将 DoH 服务器列表写入 public-resolvers.md 文件中，配置文件中有详细说明。
