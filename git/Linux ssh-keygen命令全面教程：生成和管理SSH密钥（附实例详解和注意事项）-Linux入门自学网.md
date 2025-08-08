---
title: "Linux ssh-keygen命令全面教程：生成和管理SSH密钥（附实例详解和注意事项）-Linux入门自学网"
source: "https://bashcommandnotfound.cn/article/linux-ssh-keygen-command"
author:
  - "[[command not found]]"
published:
created: 2025-04-17
description: "在Linux系统中，ssh-keygen 是一款非常重要的安全工具，用于生成、管理和转换认证密钥，如SSH密钥。这些密钥用于SSH协议，以提供比传统密码认证更安全的登录方式。本文通过详细的实例和说明，帮助你掌握如何使用 ssh-keygen 命令。 Linux ssh-keygen命令介绍 ssh-"
tags:
  - "clippings"
---
在Linux系统中， `ssh-keygen` 是一款非常重要的安全工具，用于生成、管理和转换认证密钥，如SSH密钥。这些密钥用于SSH协议，以提供比传统密码认证更安全的登录方式。本文通过详细的实例和说明，帮助你掌握如何使用 `ssh-keygen` 命令。

## 0.1 Linux ssh-keygen命令介绍

`ssh-keygen` （SSH key generator）是一个用于创建SSH密钥的程序，它可以生成公钥和私钥对，用于SSH认证。这个工具可以帮助用户在远程登录时不再依赖密码，而是使用密钥对来进行身份验证，大大提高了安全性。

## 0.2 Linux ssh-keygen命令适用的Linux版本

`ssh-keygen` 命令在现代Linux系统中普遍存在，如Debian、Ubuntu、Alpine、Arch Linux、Kali Linux、RedHat/CentOS、Fedora、Raspbian等。如果系统中未安装SSH相关包，可以使用以下命令安装：

```bash
# 基于apt的发行版（如Debian、Ubuntu、Raspbian、Kali Linux等）
sudo apt-get update && sudo apt-get install openssh-client

# 基于yum的发行版（如RedHat，CentOS 7等）
sudo yum update && sudo yum install openssh-clients

# 基于dnf的发行版（如Fedora，CentOS 8等）
sudo dnf update && sudo dnf install openssh-clients

# 基于apk的发行版（如Alpine Linux）
sudo apk add --update openssh-client

# 基于pacman的发行版（如Arch Linux）
sudo pacman -Syu && sudo pacman -S openssh

# 基于zypper的发行版（如openSUSE）
sudo zypper ref && sudo zypper in openssh

# 基于pkg的FreeBSD发行版
sudo pkg update && sudo pkg install openssh

# 基于Homebrew的OS X/macOS发行版
brew update && brew install openssh
```

## 0.3 Linux ssh-keygen命令的基本语法

语法格式：

```bash
ssh-keygen [选项]
```

## 0.4 Linux ssh-keygen命令的常用选项或参数说明

| 选项 | 说明 |
| --- | --- |
| `-b` | 指定密钥的位大小，例如2048或4096位 |
| `-t` | 指定密钥的类型，常见的有rsa、dsa、ecdsa或ed25519 |
| `-C` | 为密钥添加一个注释信息 |
| `-f` | 指定生成密钥文件的文件名 |
| `-N` | 提供一个新密钥的密码短语 |
| `-p` | 更改私钥的密码短语 |
| `-q` | 安静模式，不输出任何信息 |
| `-m` | 指定私钥导出格式 |

## 0.5 Linux ssh-keygen命令实例详解

### 0.5.1 实例1：生成一对RSA密钥

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t rsa -b 2048 -C "your_email@example.com"
```

这个命令会生成一对2048位的RSA密钥，并且会提示你输入文件保存位置和密钥密码。 `-C` 用于添加一个注释，通常是你的邮箱地址。

### 0.5.2 实例2：更改密钥的密码

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -p -f ~/.ssh/id_rsa
```

使用 `-p` 选项并指定私钥文件 `-f` ，可以更改现有私钥的密码。系统会提示你输入旧密码，然后输入新密码。

### 0.5.3 实例3：生成一对不带密码短语的密钥

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t rsa -b 2048 -C "your_email@example.com" -N ""
```

在这个实例中， `-N` 选项后跟一个空字符串，表示生成的密钥对将不会有密码短语。这意味着你不需要在使用密钥时输入密码，但这降低了安全性。

### 0.5.4 实例4：生成ed25519类型的密钥

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t ed25519 -C "your_email@example.com"
```

`ed25519` 是一种使用Elliptic Curve Cryptography的公钥类型，它提供更好的安全性和更快的速度。这个命令将生成一个ed25519密钥对。

### 0.5.5 实例5：将公钥添加到远程服务器

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote-host
```

虽然这不是 `ssh-keygen` 的直接用法，但 `ssh-copy-id` 命令通常与生成的密钥一起使用，将公钥复制到远程服务器的 `authorized_keys` 文件中，以便于无密码登录。

### 0.5.6 实例6：检查SSH密钥的指纹

为了验证密钥的真实性或进行故障排除，你可能需要查看SSH密钥的指纹。使用以下命令：

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -l -f ~/.ssh/id_rsa.pub
```

这个命令会显示公钥文件 `~/.ssh/id_rsa.pub` 的指纹和随机艺术图像。

### 0.5.7 实例7：创建一个SSH密钥，并在生成时保存到指定文件

如果你想要将生成的密钥保存到特定的文件中，可以使用 `-f` 选项：

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/my_custom_key
```

这个命令会创建一个新的4096位RSA密钥对，并将其保存在 `~/.ssh/my_custom_key` （私钥）和 `~/.ssh/my_custom_key.pub` （公钥）。

### 0.5.8 实例8：生成具有特定密码短语的密钥

为了在命令行中直接设置一个密钥的密码短语，可以使用 `-N` 选项：

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t rsa -b 2048 -C "your_email@example.com" -N "mysecretpassphrase"
```

这将创建一个RSA密钥，并将 "mysecretpassphrase" 设置为密钥的密码短语。但请注意，直接在命令行中包含密码短语可能不安全，因为它可能会留在shell的历史记录中。

### 0.5.9 实例9：创建一个只用于认证的SSH密钥

如果你需要创建一个密钥只用于SSH认证，并且确保它不被用于其他目的，可以使用 `-O` 选项：

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t rsa -b 2048 -C "your_email@example.com" -O use-only-for-authentication=yes
```

### 0.5.10 实例10：在生成密钥时指定密钥的有效期

SSH密钥可以具有有效期，这意味着密钥在特定的时间后将不再有效。这可以通过 `-V` 选项来设置：

```bash
[linux@bashcommandnotfound.cn ~]$ ssh-keygen -t ed25519 -C "temporary_key" -V +52w
```

这个命令生成一个ed25519密钥对，其有效期为从当前时间开始的52周。

## 0.6 注意事项

- **安全性** ：密钥对的安全性非常高，但如果你的私钥落入他人之手，他们就可以作为你的身份登录到任何使用该密钥的系统。因此，请确保保护好你的私钥，不要泄露，并且为其设置一个强密码短语。
- **权限** ：确保 `~/.ssh` 目录和里面的私钥文件的权限设置正确。通常，目录的权限应该是 `700` （即只有所有者可以读写执行），私钥文件的权限应该是 `600` （即只有所有者可以读写）。
- **密钥类型和长度** ：尽管RSA密钥是最广泛支持的，但建议使用ed25519密钥类型，因为它更短、更安全。如果你选择RSA密钥，请至少使用2048位长度，更安全的选择是4096位。
- **备份** ：定期备份你的SSH密钥，尤其是在生成密钥后和更改密钥密码短语后。
- **定期更换** ：定期更换SSH密钥对可以提高安全性。如果你的SSH密钥可能已经被泄露，立即生成新的密钥对并更新所有使用旧密钥的服务器。