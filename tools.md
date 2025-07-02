# Conda

## 更新 conda 版本失效

```bash
conda update conda
```

更新 conda 应该在 base 环境下，并且很可能更新完后 conda 版本仍然不变，原因是 conda 版本更新需要依赖其他的包，其他的包没更新，那么 conda，也不会更新，正确做法应该是：

```bash
conda update --all
```

# WSL

如果又想用 windows，又想用 linux，建议使用 Windows11 中的 WSL2

## 通过 ssh 远程连接 WSL

### 在 WSL 安装 ssh 服务

```bash
sudo pacman -S openssh
```

### systemctl 设置自动启动

```bash
sudo systemctl enable sshd
```

还有这些命令启动，查看状态，关闭 ssh

```bash
sudo systemctl start sshd
sudo systemctl status sshd
sudo systemctl stop sshd
```

### 设置 wsl 中的 ssh 端口号为 2222

ssh 端口号一般为 22，为了不和 windows 的 ssh 端口号冲突，所以 wsl 中 ssh 端口号设置为 2222

编辑`/etc/ssh/sshd_config`文件，找到以下行

```bash
Port 2222
#AddressFamily any
ListenAddress 0.0.0.0
#ListenAddress ::
```

设置完可能需要重启一下 wsl

### 在 windows 添加 portproxy 规则

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=[your wsl address] connectport=2222
```

You can list all your portproxy rules like this if you're concerned:

```powershell
netsh interface portproxy show v4tov4
```

You can remove them all if you want with

```powershell
netsh int portproxy reset all
```

### 其他电脑配置 ssh

```
Host wyh.4090.WslArchlinux
  HostName 10.191.53.56
  User yuhang
  Port 2222
```

注意，这里的 ip 地址应该是安装 wsl 的 windows 的 ip 地址

### 添加端口转发

在 windows 的 powershell 中输入以下命令，添加端口转发规则

```powershell
netsh advfirewall firewall add rule name=”Open Port 2222 for WSL2” dir=in action=allow protocol=TCP localport=2222
```

## WSL 如何翻墙

WSL 翻墙一般使用 windows 的翻墙，

> 更多参考:[在 WSL2 中使用 Clash for Windows 代理连接\_ > ](https://eastmonster.github.io/2022/10/05/clash-config-in-wsl/)

### 方法一

使用 WSL2 的最新特性，在 `C:\Users\<UserName>\.wslconfig` 文件中 (如果不存在就手动创建一个) 加入以下内容:

```
[wsl2]
networkingMode=mirrored
autoProxy=true
```

配置文件更改后需要重启 wsl
这个方法可以让 WSL2 使用 Windows 的代理，还会导致 WSL 的 ip 与 Windows 一样，这样用 ssh 远程 wsl 会连接不了，但是不想用 ssh 连接 wsl，而是只是想在本机 Windows 上用 wsl，那这个方法最快最简单

### 方法二

翻墙软件要允许局域网连接 allow alan，同时记住端口号，clash for windows 是 7890，

![](./images/clash%20for%20windows.png)

使用 WSL2 的最新特性，在 `C:\Users\<UserName>\.wslconfig` 文件中 (如果不存在就手动创建一个) 加入以下内容:

```
[wsl2]
dnsTunneling=false
autoProxy=false
```

配置文件更改后需要重启 wsl

这个不会强制让 WSL 使用 Windows 的代理，WSL 会是一台独立的电脑，ip 与 windows 不一样，这样就还能保证 ssh 远程连接 wsl

编辑 wsl 中的`./bashrc`文件，如果使用 zsh 终端，应该编辑`./zshrc`，加入

```bash
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export http_proxy="http://$host_ip:[端口]"
export https_proxy="http://$host_ip:[端口]"
```

保存完后，在终端输入，如果使用 zsh，应该改成`source ./zshrc`

```
source ./bashrc
```

#### 最关键一步

还有关键的一步，windows 的防火墙要允许翻墙软件通讯，clash for winodws 和 clash-win64 都设置

![](./images/firewall.png)

再终端输入如下命令，进行验证，没有 wget 可以先安装

或者在 powershell 中输入

```bash
New-NetFirewallRule -DisplayName "Allow Clash 7897 from WSL" -Direction Inbound -LocalPort 7897 -Protocol TCP -Action Allow
```

这条命令明确告诉防火墙：“我允许来自所有来源（包括 WSL、局域网等）的请求访问本机的 7897 端口”；

### 验证

```bash
wget www.google.com
```

返回这个表示可以

![](./images/wget%20test.png)
