# Docker 下载

这里只讲解 `Linux` `engine` 版本，`Windows` 和 `Desktop` 版本很简单，官网直接下载安装就好。`Docker` 从 20 版本开始好像已经集成了 `Docker Compose` ，所以不用再单独下载 `Docker Compose` 工具

---

## `Ubuntu` 系

### 官方仓库

使用官方仓库可能因为网络原因，会出现超时连接等问题，自行添加代理。

1. **更新系统安装依赖**

   ```bash
   sudo apt update
   sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
   ```

2. **添加 Docker 官方 GPG 密钥**

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

3. **添加 Docker 官方仓库**

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. **更新安装 Docker**

   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

5. **验证安装**

   ```bash
   # 出现类似下方就是成功
   docker --version
   Docker version 27.3.1, build ce12230
   docker compose version
   Docker Compose version v2.29.7
   ```

---

### 阿里云仓库

如果不会解决网络问题，当然阿里云镜像服务是个更好的选择。

6. **更新系统和安装依赖**

   ```bash
   sudo apt update
   sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
   ```

7. **添加阿里云 Docker GPG 密钥**

   ```bash
   curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-aliyun-keyring.gpg
   ```

8. **添加阿里云 Docker 仓库**

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-aliyun-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

9. **更新并安装 Docker**

   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

10. **验证安装**

```bash
docker --version
```

---

## `Centos` 系

`CentOS` 系，特别是 `CentOS 8` 及后续版本以及基于 `CentOS` 的发行版（如 `AlmaLinux`、`Rocky Linux`）中，`dnf` 已经逐渐取代了 `yum` 作为默认的包管理工具，所以这里不再讲解 `yum` 包管理，只讲解 `dnf` 包管理。

1. **卸载旧版本**

```bash
# 卸载系统中可能已安装的旧版 Docker：
sudo dnf remove -y docker \
                 docker-client \
                 docker-client-latest \
                 docker-common \
                 docker-latest \
                 docker-latest-logrotate \
                 docker-logrotate \
                 docker-engine
```

2. **安装依赖包**

```bash
sudo dnf install -y dnf-plugins-core
```

1. **添加仓库**：官方或者阿里云仓库

```bash
# 官方
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 阿里云
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

2. **安装 Docker**

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io

# 需要安装特定版本，可以列出可用版本：
dnf list docker-ce --showduplicates | sort -r
sudo dnf install -y docker-ce-<version> docker-ce-cli-<version> containerd.io
```

1. **验证安装**

```bash
# 出现类似下方就是成功
docker --version
Docker version 27.3.1, build ce12230
docker compose version
Docker Compose version v2.29.7
```

---

## snap 下载

`snap` 作为跨系统包管理工具，可以在几乎任何 Linux 发行版上安装和使用。并且下载软件的版本也比较新，但是缺点就是资源占用可能会高一些，其它系统的 `snap` 下载这里不再讲解，在 `ubuntu` 系统直接运行 `snap install docker` 就可以自动下载 `snapd` 工具并下载最新版本的 `docker`，非常适合懒人式下载。

# Docker 配置

刚才下载的 `docker-ce`， `docker-ce-cli`和 `containerd.io` 这里也讲解一下吧

这三个软件包是 Docker 运行环境的核心组成部分，它们分别负责不同的功能，但相互协作来提供完整的容器管理功能。以下是它们的详细介绍：

**`docker-ce` (Docker Community Edition)**：`Docker` 的核心包，包含了 `Docker` 的主要功能，如容器创建、管理和网络配置等。提供了所有用户端操作所需的功能，比如通过 `Docker CLI`（命令行接口）发起容器操作，或者通过 `API` 与 `Docker` 守护进程交互。

- Docker 守护进程（`dockerd`）：负责管理容器的生命周期，处理容器的运行、停止、删除等操作。
- 基本的容器运行功能，主要是与 `containerd` 和 `runc` 配合工作。

**`docker-ce-cli` (Docker Community Edition CLI)**：`Docker` 的命令行工具，用于向 Docker 守护进程（`dockerd`）发送指令。提供用户使用的所有命令，比如 `docker run`、`docker ps`、`docker build` 等。如果需要通过命令行管理 Docker，则必须安装该组件。如果只通过 `Docker` 的 `API` 或第三方工具（如 `Portainer`）管理 `Docker`，则可以不安装。

- **管理容器**：创建、启动、停止、删除容器。
- **管理镜像**：下载、构建、删除容器镜像。
- **查询状态**：检查容器的运行状态、日志等。

**`containerd.io`**：一个独立的容器运行时，它负责管理容器的生命周期，包括下载镜像、运行容器以及与底层系统资源的交互。是 `Docker` 使用的核心运行时之一，具体负责底层容器的创建和管理。

- `docker-ce` 使用 `containerd` 来处理容器运行的具体逻辑，是 Docker 守护进程（`dockerd`）运行容器的基础。
- 下载和管理容器镜像，管理容器的生命周期（创建、启动、停止等），与底层的 `runc` 配合，实现容器的实际运行。

## 配置开机自启动

这里直接使用 `systemctl` 系统管理工具，比较通用

```bash
# 开启 docker
sudo systemctl start docker
# 开机自启动
sudo systemctl enable docker
```

## 配置镜像源

众所周知，懂得都懂，当然阿里云自己也提供了自己的 `docker` 容器镜像服务，但是很扯淡的是，你下个 `nginx` 可能都是两年前的版本，并不是太推荐重度开发使用，而自己搭建私人镜像源的话成本开销太大，如果是使用 `cloudflare` 做中转的话也是很不稳定。

### 白嫖方法

这里介绍一个大佬一直在更新的 `docker` 镜像服务可用网站：https://www.wangdu.site/course/2109.html

编辑或创建 `/etc/docker/daemon.json` 文件，输入自己使用的镜像网站（阿里云或者其他）添加以下内容：

```json
{
  "registry-mirrors": ["https://xxx"]
}
```

保存后，重启 `Docker` 和配置文件：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

验证是否生效：

```bash
docker info | grep -A 5 "Registry Mirrors:"

# 正常会出现类似下面输出
 Registry Mirrors:
  https:://xxx
 Live Restore Enabled: false
```

---

### 添加代理

直接添加直连代理即可，这里除了全局或者终端设置也可以为 `Docker` 单独配置。

```bash
# docker 配置文件位置可能不一样，默认为：
vim /usr/lib/systemd/system/docker.service

# 也可能的位置
vim /etc/systemd/system/multi-user.target.wants/docker.service
```

添加后的内容如下（建议不需要 pull 的时候加上注释，不然可能在老的 Linux 内核系统可能会有问题）：

```bash
[Unit]
...

[Service]
# JIMLEE
# 其它都不要修改，在这里添加Environment就好
Environment="HTTP_PROXY=http://IP:PORT/"
Environment="HTTPS_PROXY=http://IP:PORT/"
# 添加docker 代理示例
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
Type=notify
...


[Install]
...
```
