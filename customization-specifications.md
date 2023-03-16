# 虚拟机自定义规范

自定义脚本

已经在Ubuntu 22.04 LTS版本测试通过

> 先决条件
> 1. 确保Ubuntu默认shell已修改为bash，而不是默认的dash
> 
> https://blog.csdn.net/Aoutlaw/article/details/127406626
> 
> 2. 确保vmware tools开启了自定义脚本
> 
> https://docs.vmware.com/cn/VMware-vSphere/8.0/vsphere-vm-administration/GUID-EB5F090E-723C-4470-B640-50B35D1EC016.html

```
#!/bin/sh

if [ x$1 == x"precustomization" ]; then
echo Do Precustomization tasks >> /tmp/precustomization

elif [ x$1 == x"postcustomization" ]; then
echo Do Postcustomization tasks >> /tmp/postcustomization

# 换源
sed -i.bak 's/archive.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list

# 设置root密码
echo "root:1" | chpasswd

# 放行root密码ssh登录
sed -i.bak 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd

# 修改时区
timedatectl set-timezone Asia/Shanghai

# 更新
apt update -y

# 时间
apt install chrony -y
sed -i.bak "s/pool [a-zA-Z0-9\.]*\s*iburst maxsources [0-9]*/server time.windows.com iburst/g" /etc/chrony/chrony.conf
systemctl restart chronyd

# 禁用ipv6
echo "
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
sysctl -p

fi
```
