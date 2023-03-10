# 虚拟机自定义规范

自定义脚本

已经在Ubuntu 22.04 LTS版本测试通过

> 对于校园网，需要在模板虚拟机上传登录脚本到 `/home/ubuntu/main.py` 和 `/home/ubuntu/login.sh`

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

function campus()
{
    if [ "`curl -s test.ipw.cn | grep 1.1.1.1`"x != ""x ]; then
        # 需要校园网验证
        return 1
    fi

    return 0
}

function addSupervisorConf(){
    echo "
[program:campus]
command=sh /home/ubuntu/login.sh
autorestart=false
user=root
" > /etc/supervisor/conf.d/campus.conf

}

function addCron(){
    echo "*/5 * * * *    root    sh /home/ubuntu/login.sh  >> /var/log/login.log 2>&1" >> /etc/crontab
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
	echo "network off"
	exit -1
fi
echo "network ok"

# 判断校园网
campus
if [ $? -eq 1 ];then
    # 开始校园网验证
    if [ "`python3 /home/ubuntu/main.py`" != "success"x ];then
        if [ "`python3 /home/ubuntu/main.py`" != "success"x ];then
            if [ "`python3 /home/ubuntu/main.py`" != "success"x ];then
                echo "login fail"
            fi
        fi
    fi
    echo "login ok"
fi

# 更新
apt update -y

# 时间
apt install chrony -y
sed -i.bak "s/pool [a-zA-Z0-9\.]*\s*iburst maxsources [0-9]*/server time.windows.com iburst/g" /etc/chrony/chrony.conf
systemctl restart chronyd

# 定时登录校园网
addCron
sed -i.bak "s#\#cron.*#cron.*     /var/log/cron.log#" /etc/rsyslog.d/50-default.conf
service rsyslog restart
service cron restart

fi
```
