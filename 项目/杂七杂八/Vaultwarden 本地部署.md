## 引言

### 什么是 Vaultwarden

Vaultwarden（以前称为 Bitwarden_RS）是一个轻量级、开源的密码管理系统，它是 Bitwarden 的一个自托管版本。Bitwarden 是一个流行的密码管理工具，允许用户安全地存储、生成和管理他们的密码，Vaultwarden 实现了与 Bitwarden 完全相同的功能，但设计上更加轻量，适合自托管使用。

Vaultwarden 完全兼容 Bitwarden 的各种客户端于插件， 支持通过 Web 界面、浏览器扩展、移动应用和桌面客户端进行访问，因此它可以跨设备同步密码，确保用户能够随时随地访问自己储存的信息。

### Vaultwarden 的优势

1. **轻量级与资源高效**  
   Vaultwarden 设计上比官方的 Bitwarden 服务器要轻量得多，它不依赖于复杂的数据库或后台服务。通常，它仅需要一个简单的 SQLite 或 MySQL 数据库，适合在资源受限的环境中运行，尤其适合低配置的个人服务器或树莓派等设备。
2. **开源与免费**  
   Vaultwarden 是开源软件，意味着可以自由地查看、修改和分发其代码。这种开放的开发模式增加了透明度，并且无需支付任何费用。
3. **兼容 Bitwarden 客户端**  
   Vaultwarden 完全兼容 Bitwarden 的官方客户端，包括浏览器扩展、移动应用和桌面客户端。用户可以继续使用他们熟悉的 Bitwarden 客户端进行访问，无需适应新的工具或接口。
4. **自托管与完全控制**  
   使用 Vaultwarden 自托管意味着可以完全掌控自己的数据。与一些基于云的密码管理服务不同，Vaultwarden 允许将所有密码数据存储在自己的服务器上，避免了将敏感数据交给第三方服务的问题，从而提高了安全性和隐私性。
5. **支持多种认证方式**  
   Vaultwarden 支持多种身份验证方式，如单一密码、多因素认证（2FA）、以及自定义认证插件，提供更灵活的安全配置。
6. **易于部署**  
   Vaultwarden 提供了 Docker 镜像和 Docker Compose 配置，极大简化了安装与配置过程。可以在几分钟内通过容器化方式将其部署到自己的服务器上，无需担心复杂的依赖或系统配置。
7. **社区支持与活跃开发**  
   Vaultwarden 拥有活跃的开源社区，开发者和用户不断贡献新的功能和改进。可以轻松找到文档、教程和支持资源，帮助解决部署和使用过程中的问题。

> 当然没用有 1password 那种那么功能强大，但是作为免费可自建并且可以完全兼容 Bitwarden 的产品，已经算很不错了，白嫖党永不付费！（开发所需除外）

## 准备工作

这里是使用 Docker Compose 的方式基于 Ubuntu 24 部署的，其实环境用了 Docker 容器之后环境就没那么必要了，但是需要注意的是必须要有 HTTPS 证书才能使用，官网介绍的也有这个问题，要么就只能本机 localhost 了，但是那样就失去了随时随地访问还能一键填充的意义。
需求如下：

- 安装 Docker 和 Docker Compose（如果没有安装：[Docker 安装配置](http://localhost:4000/post/25807.html)）
- 具有公网 IP 的服务器（就算部署到本地也需要公网 IP 转发）
- 个人域名+HTTPS 证书（尽量两个都有，不然还不如用 `KeePass` 就放本地）
- 个人邮箱（注册使用，如果只是个人用的话，开启注册功能自己乱填也可以）
- 查看文档的脑子 🧠（最为重要，因为深入配置都在文档里）

## 本地部署步骤

接下来，我们将详细讲解如何在本地环境中部署 Vaultwarden。部署过程中，将使用 Docker 和 Docker Compose，确保简化配置和管理。这里就不说废话了~

#### 拉取 Vaultwarden 镜像

运行下面命令来拉取 Vaultwarden 镜像，当然在 Docker Compose 文件中指定镜像后运行也可以自动拉取。

```bash
docker pull vaultwarden/server:latest
```

#### 配置 Docker Compose 文件

使用 Docker Compose 可以简化容器配置和启动流程。创建一个新的目录来存放 Vaultwarden 配置，并在其中创建一个名为 `docker-compose.yml` 的文件，`/data/docker/` 是我从上司那里学来的习惯，可以自行更改。

```bash
mkdir -p /data/docker/vaultwarden
cd /data/docker/vaultwarden
vim docker-compose.yml
```

然后编辑 `docker-compose.yml` 文件，加入以下配置：

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - ADMIN_TOKEN=6932490c-6a9a-40ec-a299-f758a14d76d9
      - SIGNUPS_ALLOWED=false  # 禁止注册
      - SMTP_HOST=smtp.163.com  # SMTP 邮件服务器地址
      - SMTP_PORT=465  # SMTP 服务器端口
      - SMTP_USERNAME=your_email@163.com  # 发件人邮箱
      - SMTP_PASSWORD=your_code  # 授权码，不是邮箱密码
      - SMTP_FROM=force_tls@163.com  # 发送邮件的邮箱
      - SMTP_SECURITY=force_tls  # 使用 TLS/STARTTLS
      - DOMAIN=https://your.domain
    ports:
      - "58080:80"  # HTTP 端口
    volumes:
      - ./vw-data:/data  # 持久化数据存储路径
```

- **`SIGNUPS_ALLOWED`**：其实这里可以开启，然后把 `ADMIN_TOKEN` 取消，就是每个人进入页面后都可以注册，但是这样确实不够安全，所以使用 `ADMIN_TOKEN` 开启后台管理的好处就是使用邮箱来邀请注册。
- **`WEBSOCKET_ENABLED`**：启用 WebSocket 支持，可以提升与客户端的实时同步。
- **`ADMIN_TOKEN`**：设置一个管理员令牌，用于在 Web 界面上进行管理员操作。
- **`volumes`**：用于数据持久化，将 Vaultwarden 数据保存在本地 `data` 文件夹中。
- **`SMTP 配置`**：关于邮箱配置，这里是用的网易邮箱，具体看个人用啥有邮箱了，但是 `SMTP_SECURITY` 在测试的时候发现必须启用 `force_tls` 进行安全验证。
- **`DOMAIN`**：这里如果是启用了 `admin` 后台进行邀请注册的话是必须配置的，不然发送注册邀请域名就成了 `localhost`(默认的)。
  > 具体的配置项见：[Vaultwarden wiki 文档](https://github.com/dani-garcia/vaultwarden/wik)，这里我只用了个 HTTP 端口是因为待会儿用 Nginx 做一层代理转发来使用证书。

#### 启动 Vaultwarden 服务

完成 `docker-compose.yml` 文件配置后，使用以下命令启动 Vaultwarden 服务：

```bash
docker-compose up -d
```

此命令会启动 Vaultwarden 容器并在后台运行。如果一切顺利，你的 Vaultwarden 服务就会在本地的 80 端口启动（如果你配置了 HTTPS，则是 443 端口）。

可以通过以下命令查看容器的运行状态或查看日志来观察是否启动成功：

```bash
# 查看运行状态
docker ps

# 查看日志
docker logs -f -t your_container_name
```

#### 配置 Nginx 和 证书（可选）

说是可选，其实是必选，因为不配置 HTTPS 的话现代有些浏览器的安全 API 是没法用的，所以这里直接把域名和证书都给弄好，Nginx 的安装和证书申请这里就只简单提一下，域名的话弄个二级域名就行了。

首先，安装 Nginx（如果未安装）：

```bash
sudo apt/dnf update
sudo apt/dnf install nginx
```

然后，在 Nginx 配置文件中设置反向代理：

```bash
sudo vim /etc/nginx/sites-available/vaultwarden
```

加入以下配置：

```nginx
server {
	# HTTPS 请求
    listen 443 ssl;
    server_name vaultwarden.your.domain;

    ssl_certificate /your/cert/path/fullchain.pem;
    ssl_certificate_key /your/cert/path/privkey.pem;

    location / {
        proxy_pass http://localhost:58080/;  # 将请求转发到 Vaultwarden 服务
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_ssl_verify off;
    }

    access_log /var/log/nginx/vaultwarden_access.log;
    error_log /var/log/nginx/vaultwarden_error.log;
}

server {
	# HTTP 请求
    listen 80;
    server_name vaultwarden.your.domain;

    return 301 https://$host$request_uri;  # 强制跳转到 HTTPS
}
```

然后创建一个符号链接并重新加载 Nginx 配置：

```bash
sudo ln -s /etc/nginx/sites-available/vaultwarden /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 具体使用步骤

### 浏览器访问

在浏览器中访问 `https://your.domain`（不要使用 HTTP ，会有问题）。将看到 Vaultwarden 的登录页面。如果环境变量没有设置禁止注册就可以看到有注册按钮，如果设置了禁止注册就把设置 ADMIN_TOKEN 来开启后台管理页面。

![登录页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_1.png)

### 注册登录

**如果是没用禁止注册就注册完登录**

![注册页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_2.png)

**如果禁止注册并且设置了 ADMIN_TOKEN 就进入后台邀请注册（邀请注册必须开启邮箱功能和设置域名！！！）**
输入 `https://your.domain/admin` 即可进入

> 注意后台管理页面没有中文，只有英文，英语不太行就还是注册登录吧

![进入后台页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_4.png)

输入 token 后便可进入后台管理页面

![后台管理页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_5.png)

然后点击 user 进入用户管理于邀请注册

![用户管理页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_6.png)

### 进入首页进行使用

上面注册登陆后就进入了首页，这里只简单介绍一下常规登录项目，别的什么和进阶功能自己去官网吧，如果要扯就太多了。

![首页](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_3.png)

![新建登录项目](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_7.png)

### 配合各种客户端

如果只是上面的这些有人会说了，这特么不就是个常规密码存储么，用记事本再用 git 也能做啊，那就大错特错了，作为 Bitwarden 的拓展项目来说，Bitwarden 所有的客户端功能，一个不少，web 端比如 chrome 就在浏览器拓展插件搜索 `Bitwarden 密码管理器`[chrome 插件地址](https://chromewebstore.google.com/detail/bitwarden-password-manage/nngceckbapebfimnlniiiahkandclblb?hl=zh-CN&utm_source=ext_sidebar)，手机 iOS 或者谷歌商店搜索 Bitwarden ，PC 端当然也有，[Bitwarden 官网下载](https://bitwarden.com/download/)。

这里主要就拿 chrome 插件来介绍一下如何集成自建地址使用。下载好并启用插件后，点击拓展程序进行配置

![自托管地址](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_8.png)

然后保存完个人地址后回到首页进行登录刚才注册的账号

![插件页面](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_9.png)

然后就可以看到自己的各种项目信息并且也可以直接进行新增项目等操作

![自托管地址](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_10.png)

然后我们可以找一个自己保存了 url 和账号密码的网址进行测试

![一键填充密码](http://cache.jimlee.top/post_images/vaultwarden_local_deployment_11.png)

> 注意有时候在不同客户端新增密码后可能不会自动刷新，需要重新登陆一下。