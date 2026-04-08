# Linux下的Geant4安装

## 1 WSL2和Ubuntu-22.04
- 控制面板 -> 程序和功能 -> 启用或关闭Windows功能，勾选“适用于Linux的Windows子系统”以及“虚拟机平台”，重启

- 管理员身份运行powershell
```bash
wsl --set-default-version 2

wsl --update

wsl --shutdown
```
```
#https://cloud-images.ubuntu.com/wsl/jammy/current/ 下载ubuntu-jammy-wsl-amd64-ubuntu22.04lts.rootfs.tar.gz

mkdir D:\WSL\Ubuntu2204

#执行导入操作（请根据你的实际路径和文件名修改）

wsl --import Ubuntu-22.04 D:\WSL\Ubuntu2204 D:\ubuntu-jammy-wsl-amd64-ubuntu22.04lts.rootfs.tar.gz --version 2

#导入后，系统默认以 root 登录，你需要创建一个普通用户并设置为默认用户
useradd -m -s /bin/bash fanghaodu
passwd fanghaodu  # 设置密码
usermod -aG sudo fanghaodu  # 赋予sudo权限

#(exit)，然后在 PowerShell 中设置该用户为默认登录用户
wsl --manage Ubuntu-22.04 --set-default-user fanghaodu
```

```


- VScode下载WSL插件，`ctrl shift P`选择“连接WSL”

## 2 Geant4
### Preview
- gcc安装
```bash

sudo apt update
sudo apt upgrade
sudo apt-get install build-essential -y
```


