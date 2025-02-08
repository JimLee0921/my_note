好的，以下是 **FRP 配置与使用** 笔记的大纲，供你参考：

## 引言

### 什么是内网穿透

内网穿透指的是通过特定技术手段，将内网（例如：家庭、公司或其他私有网络）中的服务暴露到公网，使外部设备可以访问这些服务。由于内网机器通常没有公网 IP，且可能被 NAT（网络地址转换）或防火墙限制，内网穿透可以通过在公网服务器上建立代理或隧道，绕过这些限制，实现外部网络对内网服务的访问。

### FRP 简介

FRP（Fast Reverse Proxy）是一款高性能的内网穿透工具，它通过在公网和内网之间建立反向代理，能够使内网服务暴露给外部网络。FRP 提供了非常灵活和高效的穿透能力，适用于多种协议，包括 TCP、UDP、HTTP 和 HTTPS。其核心优势包括：

- **高效稳定**：FRP 采用高效的协议与通信方式，适合长期稳定运行。
- **灵活性**：支持多种协议和传输方式，可以满足多种场景需求。
- **简易配置**：FRP 配置简单，易于上手，不需要复杂的设置。
- **开源免费**：FRP 是开源软件，完全免费，适合开发者使用。
  FRP 支持端口映射、负载均衡、访问控制等多种功能，适合需要将内网服务暴露到外网的开发者和运维人员。
  > 本人之前用过比如花生壳和 `ngork` 等内网穿透工具，但是简言之，花生壳 1M 的水流，🐕 都不用，ngork 在国内也不好用，所以最后还是选择了自建 FRP 作为连接个人服务器的工具(但是需要有一个公网服务器作为中转，所以还是需要一些成本在)

### 适用场景概述

内网穿透技术，如 FRP，适用于多种场景：

1. **远程访问内网服务**：例如，开发人员可以通过 FRP 从外部网络访问公司内网的数据库、Web 服务、远程桌面等。
2. **穿透 NAT**：许多家庭或公司网络使用 NAT 来共享公网 IP，这导致内网设备无法直接访问外部网络。FRP 通过公网服务器建立连接，解决了这种问题。
3. **防火墙穿透**：在一些严格的网络环境中，防火墙可能会阻止内网服务直接暴露到外部。FRP 可以通过创建隧道，在不直接暴露内网 IP 的情况下，让外部网络访问内网服务。
4. **远程管理与监控**：FRP 可以用来远程管理和监控家中的服务器或设备，例如家庭服务器、监控摄像头等，无需担心公网 IP 变动或防火墙阻拦。
   通过使用 FRP，用户可以轻松实现将内网服务暴露到公网，从而在各种网络环境下访问内网资源，提升远程操作与管理的灵活性与安全性。

## FRP 安装

> 这里操作系统是以 Ubuntu 24 系统为例，FRP 版本是最新的 0.61.1 ，需要注意 frp 从`v0.52.0`开始将使用  `toml`  作为配置文件类型 低于`v0.52.0`使用  `ini`  但是使用配置上基本没啥区别。

上面废话较多，直接开始吧

### 准备工作

首先 FRP 分为两个部分  **服务端** 和 **客户端**。

- **服务端（frps）**：其实就是拥有公网 IP 的服务器，这里使用的是阿里云的轻量服务器。
- **客户端（frpc）**：部署在需要穿透的客户端 例如：**NAS**、**软路由**  或 **个人服务器** 等，这里用的就直接是个人 Ubuntu 系统。

### FRP 下载

下载上服务器和客户端都是一样的，都直接从 GitHub 的[FRP GitHub Releases](https://github.com/fatedier/frp/releases) 选择合适的版本进行下载（这里演示使用的是 0.61.1 版本）。

#### 下载压缩包

**下载适合当前操作系统的版本**：

- **Linux**：通常是 `.tar.gz` 格式的压缩包。
- **Windows**：`.zip` 格式的压缩包。
- **macOS**：`.tar.gz` 格式的压缩包。
  如果在 Linux 系统也可以直接使用 wget 等工具进行拉取

```bash
wget https://github.com/fatedier/frp/releases/download/v0.61.1/frp_0.61.1_linux_amd64.tar.gz
```

> 因为 GFW 等如果遇到网络问题，自行使用代理进行拉取，也可以在 Windows 上下载完成后使用 scp 命令拷贝到服务器。

#### 解压压缩包

在服务器拉去压缩包后运行下面命令来进行解压，建议解压后根据个人喜好指定路径并重命名文件夹。

```bash
# 解压后放在指定目录 目录下文件夹名修改为 frp
tar -zxvf frp_0.61.1_linux_amd64.tar.gz -C /path/to/ frp
```

- **frpc**：这是 FRP 客户端的可执行文件，运行在内网机器上，负责将内网服务通过 FRP 穿透暴露到公网。通过运行 `frpc` 来启动客户端。
- **frps**：这是 FRP 服务端的可执行文件，部署在公网服务器上，负责接收来自客户端的请求并转发数据。通过运行 `frps` 来启动服务端。
- **frpc.toml**：这是 FRP 客户端的配置文件，用 TOML 格式编写，指定客户端如何连接到服务端并映射内网服务到公网。配置包括服务端的地址、端口和映射的协议（如 HTTP、TCP）等。
- **frps.toml**：这是 FRP 服务端的配置文件，同样采用 TOML 格式，指定服务端的监听端口、认证信息等。它设置了服务端的配置，告诉服务端如何处理来自客户端的请求。

#### 主要配置文件

上述步骤完成后进入解压后的文件夹 `cd /path/to/frp` ，然后查看当前文件夹下 FRP 的主要文件，应该如下所示

```bash
# ls
frpc  frpc.toml  frps  frps.toml  LICENSE

```

**安装服务端（frps）**

- 下载与安装。
- 配置 `frps.ini` 文件：主要设置服务端监听端口、权限控制、认证密钥等。
- **安装客户端（frpc）**
  - 下载与安装。
  - 配置 `frpc.ini` 文件：配置要穿透的端口、目标服务器地址、认证密钥等。

## FRP 配置

> 这里使用 客户端 TCP 的 SSH 连接端口进行演示

关于配置，服务端需要配置 frps.ini 或者 frps.toml，而客户端需要配置 frpc.ini 或 frpc.timl（根据版本不同，配置文件不同，但是配置项基本都是一样的），这里主要使用常用的一些配置项，能够做到大多场景下 FRP 正常工作即可。具体配置见官网[FRP 文档](https://gofrp.org/zh-cn/docs/)。

### 服务端配置

#### 修改配置文件

直接使用记事本编辑工具 vim，vi 或者 nano 修改 frps.toml，`vim frps.toml` 即可。

```toml
# 修改文件内容
# frps.toml - 服务端配置

# FRP 服务器监听的地址 0.0.0.0代表监听所有网络接口,127.0.0.1代表只能本机
bindAddr = "0.0.0.0"
# frps 服务端运行端口
bindPort = 7000
# 身份验证方式(令牌)
auth.method = "token"
# 令牌密钥，客户端与服务端需要保持一致，可以直接使用uuid等
auth.token = "your_token"

# 仪表盘监听的IP设为127.0.0.1只能本机访问(仪表盘只要查看运行相关情况)
webServer.addr = "0.0.0.0"
# 仪表盘端口 默认 7500
webServer.port = 7500
# 仪表盘登录用户名和密码
webServer.user = "admin"
webServer.password = "admin"

# 日志存储位置 console 或 文件路径
log.to = "/var/log/frps.log"
# 日志级别 trace, debug, info, warn, error
log.level = "info"
# 日志保留天数 默认 3 天
log.maxDays = 3
# 是否禁用日志颜色
log.disablePrintColor = false

# 心跳超时时间 默认 90 秒如果设为负数 心跳检测会被禁用
transport.heartbeatTimeout = 90

# 每个代理最大连接池数量 默认是 5
transport.maxPoolCount = 5
```

上面写了这么多，其实最简单的就是只传入个`bindPort = 7000`就足够了。

> ⚠ 必须把服务端的 7000 端口放开，并且如果 Dashboard 面板也需要把 7500 放开。

#### 服务端运行

这里可以直接运行，但是更建议的是使用 systemd 注册 frps 服务使得在后台运行。
直接运行的话在 FRP 所在文件夹内运行下面命令即可，如果配置档使用了日志记录等那么运行结果不会直接输出到控制台，如果没有使用面板，那么 7500 端口当然也不会进行监听/

```bash
./frps -c frps.toml

# 出现类似下面内容就是运行成功
2025-02-08 10:30:39.088 [I] [frps/root.go:105] frps uses config file: /home/frps/frps.toml
2025-02-08 10:30:39.234 [I] [server/service.go:237] frps tcp listen on 0.0.0.0:7000
2025-02-08 10:30:39.234 [I] [frps/root.go:114] frps started successfully
2025-02-08 10:30:39.234 [I] [server/service.go:351] dashboard listen on 0.0.0.0:7500
```

但是更方便的还是使用**systemd**在后台运行，在 `/etc/systemd/system/` 下新建 `frps.service`

```bash
vim /etc/systemd/system/frps.service
```

写入下面内容

```toml
# 写入下面内容
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为的frps的安装路径
ExecStart = /path/to/frps -c /path/to/frps.toml

[Install]
WantedBy = multi-user.target
```

然后就可以使用 systemctl 来管理 frps.service 服务，重新加载 `systemd` 后启动 `frps` 服务即可，然后观察日志和查看运行状态看是否正常启动。

- **重新加载 `systemd`**：sudo systemctl daemon-reload
- **启动 `frps`**：sudo systemctl start frps
- **设置开机自启**：sudo systemctl enable frps
- **查看运行状态**：sudo systemctl status frps
- **停止 `frps`**：sudo systemctl stop frps
- **重启 `frps`**：sudo systemctl restart frps
- **禁用开机自启**：sudo systemctl disable frps

### 客户端配置

服务端运行之后就可以进行客户端的配置了

#### 修改配置文件

客户端是需要把自己需要被连接的端口绑定到服务端的指定端口上，所以配置档跟服务端当然基本不一样，修改客户端配置文件

```bash
vim frpc.toml
```

输入以下内容，这里很重要的是 localPort 就是我们希望远程连接进客户端的端口，而 remotePort 是服务器需要开启的转发到客户端连接端口的。

```toml
# frpc.toml - 客户端配置

# 指定 frps 服务器的地址和端口（与你的 frps.toml 保持一致）
serverAddr = "xxx.xxx.xx.xx"
serverPort = 7000

# 认证方式（必须和 frps 保持一致）
auth.method = "token"
auth.token = "your_token"

# 日志配置
log.to = "/var/log/frpc.log"
log.level = "info"
log.maxDays = 3
log.disablePrintColor = false

# 配置具体的代理
[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22  # 本地 SSH 端口
remotePort = 1234  # 在 frps 服务器上暴露的端口
```

> 修改完成后就可以启动客户端，但是需要切记把服务端绑定的转发给客户端的 22 端口的 1234 端口也需要放开

#### 客户端运行

这里也可以直接运行，但是更建议的是使用 systemd 注册 frps 服务使得在后台运行。直接运行在终端输入以下命令即可。

```bash
./frpc -c frpc.toml
```

但是当然还是注册为服务更好

```bash
vim /etc/systemd/system/frpc.service
```

输入以下内容

```toml
[Unit]
Description=FRP Client Service
After=network.target

[Service]
# 启动frpc的命令，需修改为的frpc的安装路径
ExecStart=/path/to/frpc -c /path/to/frpc.toml
Restart=always
User=nobody
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

然后 `systemctrl`命令跟上面一样进行启动 `frpc.service` 服务即可

> 注意客户端我们注册的服务是 frpc.service，而服务端我们注册的是 frps.service，不要搞错了

## 连接测试

在所有都配置好了以后，并且确保相关端口也都暴露了出去，注意这里的端口是在客户端配置的 1234 那个端口，也就是在连接服务端的 1234 端口时会自动转发给我们自己服务器的 22 端口然后就可以可以从其他设备上测试 SSH 连接：

```bash
ssh -p frps服务器转发给内网端口 用户名@frps服务器IP
```

## FRP 补充

FRP（Fast Reverse Proxy）实现内网穿透的核心思想是通过搭建一个代理服务器（frps），使得内网机器（frpc）能够将内网服务暴露到公网。其工作流程大致如下：

### 客户端与服务端通信流程

- **启动服务端（frps）**：首先，在公网服务器上启动 FRP 服务端（frps）。它监听一个端口，等待来自客户端（frpc）的连接。
- **启动客户端（frpc）**：在内网机器上启动 FRP 客户端（frpc）。客户端通过配置文件（如 `frpc.toml`）指定服务端的地址和端口，连接到服务端。
- **建立隧道连接**：客户端（frpc）连接到服务端（frps）后，建立一个持久的 TCP 连接（通常通过 `bind_port` 配置的端口）。这时，客户端和服务端之间会保持一个稳定的隧道连接。
- **注册与代理映射**：客户端向服务端注册需要代理的内网服务，例如 HTTP、SSH、TCP 等。服务端将客户端注册的服务映射到公网。

### 数据转发机制

- **客户端发送请求**：当有外部用户访问服务端时，数据会通过服务端的公网 IP 地址和端口传输到服务端。
- **转发到客户端**：FRP 服务端会将接收到的请求通过已经建立的隧道转发到客户端（frpc）。客户端根据请求的数据，决定将数据转发到相应的内网服务（例如，Web 服务器、SSH 服务等）。
- **返回数据**：客户端处理完请求后，结果会通过隧道返回给服务端，服务端再转发给外部请求者，完成内网穿透的过程。

### 协议支持

FRP 支持多种协议，可以根据不同的需求选择合适的协议来进行内网穿透。常见的协议包括：

- **TCP**：支持 TCP 协议的内网穿透，适用于各种基于 TCP 的服务，比如数据库、SSH、FTP 等。客户端可以将内网的 TCP 服务暴露到公网，服务端通过代理将请求转发到内网机器。
- **UDP**：支持 UDP 协议的穿透，适用于实时通信、视频流、在线游戏等低延迟、高吞吐量的场景。UDP 数据包不会进行连接管理，因此 FRP 使用 UDP 协议时，客户端和服务端之间的连接相对轻量。
- **HTTP/HTTPS**：FRP 还支持 HTTP 和 HTTPS 协议的内网穿透。这对于 Web 服务的穿透非常有用，例如将内网的 HTTP 服务暴露给外部用户访问。HTTPS 支持加密通信，确保数据传输的安全性。
- **自定义协议**：FRP 也支持通过 `type` 配置项自定义协议，实现各种类型的数据传输。

> 通过上述原理，FRP 能够将内网服务暴露给公网用户，适用于多种场景，如远程访问内网 Web 服务、SSH 服务、数据库等。其实归根到底其实还是端口转发，比如 MySQL 就用转发到 3306，web 就转发到 8080

### FRP 高级功能

以下是 **FRP 高级功能** 的简单讲解

> 个人使用基本不用这么复杂，就个端口映射就差不多了，本身就是为了免费加稳定加配置简单，所以这里只是简单提一下，如果未来真的可能用得到再补充把

#### 1. 多端口映射

FRP 支持多个端口映射功能，使得一个客户端可以同时将多个不同的内网服务暴露到公网。通过在配置文件中添加多个服务条目，用户可以映射多个内网端口到不同的公网端口。例如，可以同时映射 HTTP 服务和 SSH 服务：

```toml
[web]
type = "http"
local_ip = "127.0.0.1"
local_port = 80
custom_domains = "www.example.com"

[ssh]
type = "tcp"
local_ip = "127.0.0.1"
local_port = 22
remote_port = 6000
```

#### 2. 域名与负载均衡

FRP 支持通过自定义域名进行内网服务映射。当有多个客户端或多个映射需要分配到同一个域名时，FRP 可以配置负载均衡，自动将流量分配到不同的服务器或端口。比如，可以根据请求的路径或请求源 IP 地址来决定流量的转发。

#### 3. TCP 和 UDP 协议的支持

除了常见的 HTTP/HTTPS 协议，FRP 还支持 TCP 和 UDP 协议的内网穿透：

- **TCP**：适用于常规的网络服务，如数据库、SSH 连接等。
- **UDP**：适用于需要低延迟、高性能的实时应用，如视频流、在线游戏等。

#### 4. 内网穿透的身份验证

FRP 支持身份验证机制（token），确保只有合法的客户端能够连接到服务端。可以在服务端和客户端配置相同的 `token`，通过身份验证防止未经授权的客户端接入。

#### 5. 支持代理与访问控制

FRP 可以配置访问控制列表（ACL），指定哪些 IP 地址或网络可以访问某个内网服务。这增强了内网服务的安全性，只允许指定的客户端访问。

#### 6. 加密与压缩

为了提高传输效率和安全性，FRP 支持对数据进行加密和压缩。启用加密可以确保数据在传输过程中的保密性，而压缩可以减少带宽占用，提高传输速度。

#### 7. 动态端口映射

FRP 还支持动态端口映射，可以在客户端请求时动态地分配公网端口。这对于需要多个临时服务的场景非常有用，例如临时暴露某个测试服务时，自动分配一个公网端口。

#### 8. API 接口

FRP 提供了 HTTP API 接口，允许通过编程方式控制和管理 FRP 服务端。用户可以通过 API 动态添加、删除服务映射，查看连接状态等。
