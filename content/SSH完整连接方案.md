# SSH 完整连接解决方案

## 概述
本文档提供 Windows 和 Linux 环境下 SSH 连接的完整解决方案，包括密钥生成、配置、权限设置和常见问题解决。

## 一、生成 SSH 密钥对

### Linux / macOS / WSL
```bash
# 生成 RSA 密钥对（推荐 4096 位）
ssh-keygen -t rsa -b 4096 -f ~/.ssh/visa_ubuntu -C "your_email@example.com"

# 或生成 Ed25519 密钥对（更安全，更快）
ssh-keygen -t ed25519 -f ~/.ssh/visa_ubuntu -C "your_email@example.com"
```

### Windows (PowerShell)
```powershell
# 使用 OpenSSH（Windows 10/11 内置）
ssh-keygen -t rsa -b 4096 -f $env:USERPROFILE\.ssh\visa_ubuntu -C "your_email@example.com"

# 或使用 Git Bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/visa_ubuntu -C "your_email@example.com"
```

## 二、复制公钥到服务器

### 方法 1：使用 ssh-copy-id（推荐）

#### Linux / macOS / WSL
```bash
ssh-copy-id -i ~/.ssh/visa_ubuntu ubuntu@49.51.196.48
```

#### Windows
Windows 没有内置 ssh-copy-id，需要手动操作或使用 Git Bash。

### 方法 2：手动复制

#### 获取公钥内容
```bash
# Linux/macOS/WSL
cat ~/.ssh/visa_ubuntu.pub

# Windows PowerShell
Get-Content $env:USERPROFILE\.ssh\visa_ubuntu.pub
```

#### 在服务器上添加公钥
```bash
# 登录到服务器
ssh ubuntu@49.51.196.48

# 创建 .ssh 目录（如果不存在）
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 添加公钥到 authorized_keys
echo "你的公钥内容" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## 三、设置正确的权限

### Linux / macOS / WSL
```bash
# 设置私钥权限
chmod 600 ~/.ssh/visa_ubuntu

# 设置 .ssh 目录权限
chmod 700 ~/.ssh

# 设置 config 文件权限（如果有）
chmod 600 ~/.ssh/config
```

### Windows
Windows 权限设置更复杂，使用 PowerShell（管理员权限）：

```powershell
# 移除继承权限
icacls $env:USERPROFILE\.ssh\visa_ubuntu /inheritance:r

# 只给当前用户完全控制权限
icacls $env:USERPROFILE\.ssh\visa_ubuntu /grant:r "$env:USERNAME:F"

# 移除其他用户权限
icacls $env:USERPROFILE\.ssh\visa_ubuntu /remove "NT AUTHORITY\Authenticated Users"
icacls $env:USERPROFILE\.ssh\visa_ubuntu /remove "BUILTIN\Users"
```

## 四、配置 SSH Config 文件

### 配置文件位置
- Linux/macOS/WSL: `~/.ssh/config`
- Windows: `%USERPROFILE%\.ssh\config`

### 配置内容
```
Host ubuntu-server 49.51.196.48
    HostName 49.51.196.48
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/visa_ubuntu
    # Windows 用户使用以下路径格式
    # IdentityFile C:\Users\YourUsername\.ssh\visa_ubuntu
    
    # 可选配置
    ServerAliveInterval 60      # 每60秒发送心跳包
    ServerAliveCountMax 3       # 3次心跳无响应则断开
    TCPKeepAlive yes           # 保持连接活跃
    Compression yes            # 启用压缩
    ForwardAgent yes           # 转发 SSH 代理（谨慎使用）
```

## 五、测试连接

### 使用别名连接
```bash
ssh ubuntu-server
```

### 使用 IP 直接连接
```bash
ssh 49.51.196.48
```

### 调试连接问题
```bash
# 查看详细连接过程
ssh -vvv ubuntu-server

# 指定密钥文件测试
ssh -i ~/.ssh/visa_ubuntu ubuntu@49.51.196.48
```

## 六、Windows 特定配置

### 1. 启用 OpenSSH 客户端（Windows 10/11）
```powershell
# 检查是否已安装
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Client*'

# 安装 OpenSSH 客户端
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

### 2. 使用 Windows Terminal 或 PowerShell
```powershell
# 创建 SSH 配置目录
New-Item -ItemType Directory -Force -Path $env:USERPROFILE\.ssh

# 编辑配置文件
notepad $env:USERPROFILE\.ssh\config
```

### 3. 使用 PuTTY（替代方案）
1. 使用 PuTTYgen 转换密钥格式（.ppk）
2. 在 PuTTY 中配置会话
3. 指定私钥文件路径

## 七、常见问题及解决方案

### 1. Permission denied (publickey)
```bash
# 检查密钥权限
ls -la ~/.ssh/visa_ubuntu

# 修复权限
chmod 600 ~/.ssh/visa_ubuntu
```

### 2. Host key verification failed
```bash
# 清除已知主机记录
ssh-keygen -R 49.51.196.48

# 重新连接并接受新的主机密钥
ssh ubuntu@49.51.196.48
```

### 3. Connection refused
- 检查服务器 SSH 服务是否运行
- 检查防火墙设置
- 确认 SSH 端口（默认 22）

### 4. Windows 上的权限问题
- 使用 Git Bash 而不是 CMD/PowerShell
- 或正确设置 Windows 文件权限（见上文）

### 5. SSH 代理转发
```bash
# 启动 ssh-agent
eval $(ssh-agent)

# 添加密钥到代理
ssh-add ~/.ssh/visa_ubuntu

# 使用代理转发连接
ssh -A ubuntu-server
```

## 八、安全建议

1. **使用强密码保护私钥**
   ```bash
   ssh-keygen -p -f ~/.ssh/visa_ubuntu
   ```

2. **定期更换密钥对**
   - 建议每 6-12 个月更换一次

3. **限制密钥使用范围**
   - 在 `authorized_keys` 中添加限制：
   ```
   from="特定IP地址" ssh-rsa AAAAB3...
   ```

4. **禁用密码登录**（在服务器上）
   ```bash
   sudo vim /etc/ssh/sshd_config
   # 设置 PasswordAuthentication no
   sudo systemctl restart sshd
   ```

5. **使用跳板机**
   ```
   Host jump-server
       HostName jump.example.com
       User jumpuser
   
   Host target-server
       HostName 49.51.196.48
       User ubuntu
       ProxyJump jump-server
   ```

## 九、高级技巧

### 1. 多密钥管理
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github

Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/gitlab
```

### 2. SSH 隧道
```bash
# 本地端口转发
ssh -L 8080:localhost:80 ubuntu-server

# 远程端口转发
ssh -R 9000:localhost:3000 ubuntu-server

# 动态端口转发（SOCKS 代理）
ssh -D 1080 ubuntu-server
```

### 3. 批量操作脚本
```bash
#!/bin/bash
# 批量执行命令
for host in server1 server2 server3; do
    echo "=== $host ==="
    ssh $host "uname -a; uptime"
done
```

---
*更新时间：2025-06-22*