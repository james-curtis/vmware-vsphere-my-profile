# 虚拟机自定义规范

自定义脚本

已经在Ubuntu 22.04 LTS版本测试通过

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

fi
```
