# [CentOS7 安装远程桌面]

yum 源使用是阿里的：https://developer.aliyun.com/mirror/[
](https://opsx.alibaba.com/mirror?lang=zh-CN)

```
rm -rf /etc/yum.repos.d/*
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

 

## 一、桌面环境

首先安装桌面环境，这里安装 GNOME。

```
yum -y groups install "GNOME Desktop"
```

关于桌面环境相关命令


```
# 从命令行切换到桌面环境
startx

# 获取当前启动模式
systemctl get-default

# 修改启动模式为图形化
systemctl set-default graphical.target

# 修改启动模式为命令行
systemctl set-default multi-user.target
```



默认启动桌面环境后以 root 用户自动登录



```
# 修改配置文件
vi /etc/gdm/custom.conf


# 增加如下配置
[daemon]
AutomaticLoginEnable=True
AutomaticLogin=root
```

 

## 二、远程服务设置


```
# Windows 远程登录需要安装 Xrdp，需要 epel 源
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum install -y xrdp

# Xrdp 会调用 VNC，安装 tigervnc-server
yum install -y tigervnc-server

# 修改 Xrdp 最大连接数
vim /etc/xrdp/xrdp.ini
max_bpp=32

# 启动 Xrdp 并设置开机启动
systemctl start xrdp
systemctl enable xrdp

# 开放 3389 端口，或者关闭防火墙
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
# 关闭防火墙
systemctl stop firewalld
# 禁止防火墙开机启动
systemctl disable firewalld
```