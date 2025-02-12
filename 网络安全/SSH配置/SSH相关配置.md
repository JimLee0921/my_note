## SSH 相关命令

### ssh 命令

`ssh` 是用于 **远程登录服务器**、**执行命令**、**文件传输** 和 **端口转发** 的安全协议。它默认使用 **端口 22**，支持 **公钥认证** 和 **密码认证**。

---

#### 1. SSH 基本语法

```sh
ssh [选项] 用户名@远程主机
```

- **用户名**：要连接的远程用户（如 `root`、`ubuntu`）。
- **远程主机**：可以是 **IP 地址** 或 **域名**（如 `192.168.1.100` 或 `example.com`）。

---

#### 2. SSH 常用参数

| 参数 | 作用                                                  |
| ---- | ----------------------------------------------------- |
| `-p` | 指定端口（默认是 22）                                 |
| `-i` | 指定 SSH 私钥文件（如 `.pem`、`.ppk`）                |
| `-L` | 本地端口转发（**本地端口 → 远程主机**）               |
| `-R` | 远程端口转发（**远程主机 → 本地端口**）               |
| `-D` | 动态端口转发（**SOCKS5 代理**，用于科学上网）         |
| `-N` | 仅建立连接，不执行命令（常用于端口转发）              |
| `-T` | 禁用伪终端（减少带宽消耗）                            |
| `-C` | 开启数据压缩，提高慢速网络的传输效率                  |
| `-o` | 指定 SSH 配置选项（如 `-o StrictHostKeyChecking=no`） |
| `-v` | 详细模式，调试连接问题（`-vv` 更详细）                |

---

#### 3. SSH 使用示例

（1）基本连接

```sh
ssh root@192.168.1.100
```

- 连接 **192.168.1.100** 服务器，使用 **root 用户**。

```sh
ssh -p 2222 user@example.com
```

- 连接 **example.com**，使用 **非默认端口 `2222`**。

（2）使用私钥连接

```sh
ssh -i ~/.ssh/id_rsa user@192.168.1.100
```

- 使用 **私钥 `id_rsa`** 进行无密码连接（适用于 AWS、GitHub、VPS）。

（3）远程执行命令

```sh
ssh user@192.168.1.100 "ls -la /var/log"
```

- 登录后 **执行 `ls -la /var/log`**，然后 **退出**。

（4）本地端口转发（内网穿透）

```sh
ssh -L 8080:localhost:3306 user@remote_host
```

- 访问 **本地 `8080` 端口**，相当于访问 **远程 `3306` 端口**（用于连接远程 MySQL）。

（5）远程端口转发

```sh
ssh -R 9000:localhost:80 user@remote_host
```

- 让 **远程服务器 `9000` 端口** 访问 **本地 `80` 端口**（反向代理）。

（6）动态端口转发（SOCKS5 代理）

```sh
ssh -D 1080 -N user@remote_host
```

- 创建 SOCKS5 代理，可以用于 科学上网：**浏览器代理** 设置为 `127.0.0.1:1080` 或使用 **curl** 或 **Python** 等工具进行使用。

（7）跳板机（ProxyJump）

```sh
ssh -J user@jump_host user@target_host
```

- 通过 **跳板机**（`jump_host`）连接 **目标服务器**（`target_host`）。

（8）防止 SSH 断开

```sh
ssh -o ServerAliveInterval=60 -o ServerAliveCountMax=3 user@192.168.1.100
```

- **每 60 秒发送一个心跳**，避免 SSH 断开。

---

#### 4. SSH 配置文件（简化连接）

vscode 中的 Remote SSH 跟这个同原理，配置文件路径：

```sh
vim ~/.ssh/config
```

写入以下内容：

```ini
Host myserver
    HostName 192.168.1.100
    User root
    Port 2222
    IdentityFile ~/.ssh/id_rsa
```

以后直接用：

```sh
ssh myserver
```

代替

```sh
ssh -p 2222 -i ~/.ssh/id_rsa root@192.168.1.100
```

---

### ssh-keygen 命令

`ssh-keygen` 主要用于创建 SSH 密钥（公钥 + 私钥），然后可以用于免密码登录远程服务器。

（1）生成 SSH 密钥对

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- `-t rsa`：指定密钥类型（RSA）。
- `-b 4096`：设置密钥长度（默认 2048，4096 更安全）。
- `-C "your_email@example.com"`：添加注释，通常是邮箱地址。

（2）密钥生成过程

```sh
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):  # 按回车使用默认路径
Enter passphrase (empty for no passphrase):  # 可选，设置密码短语
Enter same passphrase again:
```

- **默认路径**：`~/.ssh/id_rsa`（私钥） 和 `~/.ssh/id_rsa.pub`（公钥）。
- **如果设置了密码短语**，每次使用私钥时都需要输入密码，提高安全性。

（3）查看生成的密钥

```sh
ls -l ~/.ssh/
cat ~/.ssh/id_rsa.pub  # 查看公钥
```

#### 常用参数

**指定密钥类型（`-t`）**

```bash
ssh-keygen -t rsa      # 生成 RSA 密钥（默认 3072 位）
ssh-keygen -t rsa -b 4096  # 生成 4096 位的 RSA 密钥
ssh-keygen -t ecdsa    # 生成 ECDSA 密钥（默认 256 位）
ssh-keygen -t ed25519  # 生成 Ed25519 密钥（推荐，安全性高）
```

常见的密钥类型：

- `rsa`：最常见，但密钥长度要足够大（推荐 4096 位）。
- `ecdsa`：基于椭圆曲线，比 RSA 更短但同样安全（推荐 `256` 或 `384`）。
- `ed25519`：更快、更安全（推荐）。
- `dsa`：已废弃，不推荐使用。

---

**指定密钥位数（`-b`）**

```bash
ssh-keygen -t rsa -b 4096  # 指定 RSA 密钥长度为 4096
ssh-keygen -t ecdsa -b 521  # 指定 ECDSA 密钥长度为 521
```

- **RSA**：建议 `2048` 及以上，推荐 `4096`。
- **ECDSA**：支持 `256`、`384` 和 `521`。
- **Ed25519**：固定长度，不需要 `-b` 选项。

---

**指定输出文件（`-f`）**

```bash
ssh-keygen -t rsa -f ~/.ssh/my_key
```

- 生成的密钥文件：
  - 私钥：`~/.ssh/my_key`
  - 公钥：`~/.ssh/my_key.pub`

---

**添加密钥注释（`-C`）**

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

- 这通常用于标识密钥，例如 GitHub 认证时，可以添加邮箱作为注释。

---

**设置密码保护（`-N`）**

```bash
ssh-keygen -t rsa -N "mypassword"
```

- 如果不想输入密码，可使用 `-N ""`（不推荐，降低安全性）。

---

**指定哈希算法（`-E`）**

```bash
ssh-keygen -l -E sha256 -f ~/.ssh/id_rsa.pub
```

- `sha256` 是现代推荐的哈希算法。

---

#### 管理密钥

**查看密钥指纹（`-l`）**

```bash
ssh-keygen -l -f ~/.ssh/id_rsa.pub
```

- 显示密钥的指纹（默认 `SHA256` 格式）。

---

**变更密钥密码（`-p`）**

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

- 允许更改私钥的密码。

---

**转换密钥格式（`-m`）**

```bash
ssh-keygen -m PEM -f ~/.ssh/id_rsa
```

- 将私钥转换为 `PEM` 格式，适用于 OpenSSL。

---

**删除指定密钥（`-R`）**

```bash
`ssh-keygen -R example.com / IP_ADDRESS`
```

- 这个命令会从 `~/.ssh/known_hosts` 文件中删除 `example.com` 或指定`IP_ADDRESS` 相关的所有条目。

---

**查找主机密钥（`-F`）**

```bash
ssh-keygen -F hostname
```

在  `known_hosts`  文件中查找指定主机的公钥记录。

---

#### 高级用法

##### 生成符合 FIDO U2F 标准的密钥

使用 `-t ecdsa-sk` 或 -t ed25519-sk`参数，如果有安全密钥（如 YubiKey），可以生成硬件绑定的密钥：

```bash
ssh-keygen -t ed25519-sk -f ~/.ssh/id_ed25519_sk
```

- `ecdsa-sk`：基于 ECDSA 的 FIDO2 认证密钥。
- `ed25519-sk`：基于 Ed25519 的 FIDO2 认证密钥（推荐）。

---

##### 生成多主机密钥

使用 `-h` 参数

```bash
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -h
```

- 适用于 SSH 服务器的 `HostKey` 生成。

---

##### 从私钥提取公钥

使用 `-y` 参数

```bash
ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.pub
```

- 当丢失 `.pub` 文件时，可以从私钥重新生成公钥。

---

### ssh-copy-id 命令

`ssh-copy-id` 用于 **自动将 SSH 公钥添加到远程服务器** 的 `~/.ssh/authorized_keys` 文件中，实现免密码登录。

（1）自动复制公钥

```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote_host
```

- `-i ~/.ssh/id_rsa.pub`：指定要复制的公钥文件（默认 `id_rsa.pub`）。
- `user@remote_host`：远程服务器的 **用户名 和 IP/域名**。

**⚠️ 注意：**

- 第一次执行时需要输入远程服务器的密码。
- 之后即可使用 SSH 免密登录该服务器。

（2）手动复制公钥

如果 `ssh-copy-id` 命令不可用，也可以手动复制：

```sh
cat ~/.ssh/id_rsa.pub | ssh user@remote_host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

- 这条命令将本地 `id_rsa.pub` 追加到远程服务器的 `~/.ssh/authorized_keys` 文件中，并设置合适的权限。

### ssh-agent 和 ssh-add

如果 **私钥设置了密码短语**，每次连接都要输入很麻烦，可以用 `ssh-agent` 缓存私钥：

```sh
eval "$(ssh-agent -s)"  # 启动 ssh-agent
ssh-add ~/.ssh/id_rsa   # 添加私钥（输入一次密码）
ssh user@remote_host    # 之后可免密登录
```

## SSH 免密登录

现在终端输入 `ssh -V` 查看 SSH 服务是否可用，如果 SSH 存在，会返回类似：

```bash
OpenSSH_8.4p1, OpenSSL 1.1.1g  21 Apr 2020
```

如果命令不可用，则需要安装 OpenSSH。

在大多数 Linux 发行版（如 Ubuntu、Debian、CentOS）中，`ssh`、`ssh-keygen` 和 `ssh-copy-id` 等工具通常都包含在 `OpenSSH` 客户端或服务器包中。如果命令不可用，可以根据系统情况安装相关软件包。

**2. 安装 OpenSSH 客户端和服务器**

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install -y openssh-client openssh-server

# 安装完成后，确保 SSH 服务器正在运行
sudo systemctl enable ssh
sudo systemctl start ssh
```

- `openssh-client`：提供 `ssh`、`ssh-keygen`、`scp` 等命令。
- `openssh-server`：提供 `sshd`，允许服务器接受 SSH 连接。

```bash
# CentOS / Rocky Linux / AlmaLinux
sudo yum/dnf install -y openssh-clients openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
```

```bash
# Arch Linux
sudo pacman -S openssh
sudo systemctl enable sshd
sudo systemctl start sshd
```

macOS 自带 `OpenSSH` 客户端，不需要额外安装。如果 `ssh-copy-id` 不可用，可使用：

```bash
brew install ssh-copy-id
```

Windows 10+ 自带 SSH 客户端，但如果 `ssh-keygen` 不可用，可以安装 `Git for Windows` 或 `OpenSSH`

> 在 Ubuntu 中，OpenSSH 服务器的服务名称是 **`ssh`** 而不是 **`sshd`**（`sshd` 是 OpenSSH 服务器的守护进程名称，但在 `systemctl` 中，它的服务名是 `ssh`）。

### 生成 SSH 密钥对

首先需要在本地机器上生成一个 SSH 密钥对（包括私钥和公钥）这里如果 Windows 直接使用 git bash 终端。在终端中运行以下命令：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**命令讲解**

- `-t rsa` 表示使用 RSA 算法。
- `-b 4096` 表示密钥的位数为 4096 位，提供较强的安全性。
- `-C` 用来添加备注，一般是邮箱。

按提示操作，设置文件保存路径（默认 Linux 是 `~/.ssh/id_rsa`，Windows 是 `C:\Users\your_username\.ssh\id_rsa`），并设置一个密钥密码（也可以不设置）。

### 复制公钥到服务器

生成密钥对后，需要将公钥复制到目标服务器上。可以直接在服务器使用 `ssh-copy-id` 命令：

```bash
ssh-copy-id user@hostname
```

当然更建议的还是手动复制公钥。如果是 Linux 服务器直接使用 cat 命令复制即可

```bash
cat ~/.ssh/id_rsa.pub
```

Windows 直接使用 notepad++ 等记事本工具打开复制即可

然后，登录到服务器，编辑 `~/.ssh/authorized_keys` 文件，将公钥内容复制到该文件中。如果 `~/.ssh` 目录不存在需要手动创建：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys
```

把公钥粘贴到 `authorized_keys` 文件里，保存并退出编辑器。然后，设置文件权限：

```bash
chmod 600 ~/.ssh/authorized_keys
```

`chmod 700 ~/.ssh` 修改权限不能少，也可以直接权限全开 777，看自己对于安全程度的理解了。

### 配置 SSH 服务

确保服务器的 SSH 服务允许密钥认证。编辑 `/etc/ssh/sshd_config` 文件，确认以下配置项：

```bash
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

重启 SSH 服务：

```bash
sudo systemctl restart sshd
```

### SSH 免密登录测试

```bash
ssh user@hostname
```

如果一切配置正确，应该不会再要求输入密码，而是直接登录。

### 注意事项

- 私钥（`id_rsa`）一定保管好。
- 如果在多个设备之间使用 SSH 密钥对，可以将私钥复制到其他设备的 `~/.ssh/` 目录中就可以了。
- 有些服务器可能会启用防火墙或 `SELinux`，具体问题具体分析。
