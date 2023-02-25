# 虚拟机自定义规范

自定义脚本

```
#!/bin/sh

#检测网络链接畅通
function network()
{
    #超时时间
    local timeout=1

    #目标网站
    local target=www.baidu.com

    #获取响应状态码
    local ret_code=`curl -I -s --connect-timeout ${timeout} ${target} -w %{http_code} | tail -n1`

    if [ "x$ret_code" = "x200" ]; then
        #网络畅通
        return 1
    else
        #网络不畅通
        return 0
    fi

    return 0
}

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

network
if [ $? -eq 0 ];then
	echo "网络不畅通，请检查网络设置！"
	exit -1
fi

echo "网络畅通，你可以上网冲浪！"

# 更新
apt update -y

# 时间
apt install chrony -y

# 修改时间服务器
sed -i.bak "s/pool [a-zA-Z0-9\.]*\s*iburst maxsources [0-9]*/server time.windows.com iburst/g" /etc/chrony/chrony.conf

fi

```
