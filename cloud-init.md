# cloud-init

教程：https://mac-blog.org.ua/esxi-cloud-init-ubuntu

支持文档：https://github.com/vmware-archive/cloud-init-vmware-guestinfo

## 下载cloudinit封装好的镜像

- Ubuntu：https://cloud-images.ubuntu.com/

  eg. `jammy-server-cloudimg-amd64.vmdk`

  不要使用 `jammy-server-cloudimg-amd64.ova`，直接使用vmdk现有磁盘，然后克隆一份虚拟机模板就行了

## 编写userdata

文档：https://canonical-cloud-init.readthedocs-hosted.com/en/latest/reference/modules.html

> 注意
> - 第一行必须是 `#cloud-config` 
> - 文件大小有500kb限制，太大了需要gzip压缩


```yaml
#cloud-config

# hostname
hostname: ubuntutest
# 将主机名写入hosts
manage_etc_hosts: true

# ntp
ntp:
  enabled: true
  ntp_client: chrony
  servers:
  - time.windows.com

# timezone
timezone: Asia/Shanghai

# root ssh login
disable_root: false

# reset password
users:
  - name: root
    lock_passwd: false
    plain_text_passwd: password

# change repository
bootcmd:
  - sudo cp /etc/apt/sources.list /etc/apt/sources.list.back
  - sudo sed -i 's/archive.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list
  - sudo sed -i 's/security.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list

# apt update && apt upgrade, reboot if needed
package_update: true
package_upgrade: true
package_reboot_if_required: true
```

base64编码结果：
```
I2Nsb3VkLWNvbmZpZwoKIyBob3N0bmFtZQpob3N0bmFtZTogdWJ1bnR1dGVzdAojIOWwhuS4u+acuuWQjeWGmeWFpWhvc3RzCm1hbmFnZV9ldGNfaG9zdHM6IHRydWUKCiMgbnRwCm50cDoKICBlbmFibGVkOiB0cnVlCiAgbnRwX2NsaWVudDogY2hyb255CiAgc2VydmVyczoKICAtIHRpbWUud2luZG93cy5jb20KCiMgdGltZXpvbmUKdGltZXpvbmU6IEFzaWEvU2hhbmdoYWkKCiMgcm9vdCBzc2ggbG9naW4KZGlzYWJsZV9yb290OiBmYWxzZQoKIyByZXNldCBwYXNzd29yZAp1c2VyczoKICAtIG5hbWU6IHJvb3QKICAgIGxvY2tfcGFzc3dkOiBmYWxzZQogICAgcGxhaW5fdGV4dF9wYXNzd2Q6IHBhc3N3b3JkCgojIGNoYW5nZSByZXBvc2l0b3J5CmJvb3RjbWQ6CiAgLSBzdWRvIGNwIC9ldGMvYXB0L3NvdXJjZXMubGlzdCAvZXRjL2FwdC9zb3VyY2VzLmxpc3QuYmFjawogIC0gc3VkbyBzZWQgLWkgJ3MvYXJjaGl2ZS51YnVudHUuY29tL21pcnJvcnMubmp1LmVkdS5jbi9nJyAvZXRjL2FwdC9zb3VyY2VzLmxpc3QKICAtIHN1ZG8gc2VkIC1pICdzL3NlY3VyaXR5LnVidW50dS5jb20vbWlycm9ycy5uanUuZWR1LmNuL2cnIC9ldGMvYXB0L3NvdXJjZXMubGlzdAoKIyBhcHQgdXBkYXRlICYmIGFwdCB1cGdyYWRlLCByZWJvb3QgaWYgbmVlZGVkCnBhY2thZ2VfdXBkYXRlOiB0cnVlCnBhY2thZ2VfdXBncmFkZTogdHJ1ZQpwYWNrYWdlX3JlYm9vdF9pZl9yZXF1aXJlZDogdHJ1ZQ==
```

## 编写metadata

> metadata首行不需要 `#cloud-config`

```yaml
network:
  version: 2
  ethernets:
    nics:
      match:
        name: e*
      dhcp4: true
      dhcp6: true
```

base64:
```
bmV0d29yazoKICB2ZXJzaW9uOiAyCiAgZXRoZXJuZXRzOgogICAgbmljczoKICAgICAgbWF0Y2g6CiAgICAgICAgbmFtZTogZSoKICAgICAgZGhjcDQ6IHRydWUKICAgICAgZGhjcDY6IHRydWU=
```

## 设置userdata和metadata

在esxi页面或者vSphere Client页面，`编辑虚拟机设置->虚拟机选项->高级->配置参数->编辑配置`

增加下面四行

| 名称 | 值 |
|------|----|
| guestinfo.userdata.encoding | base64 |
| guestinfo.userdata | userdata base64后的内容 |
| guestinfo.metadata.encoding | base64 |
| guestinfo.metadata | metadata base64后的内容 |

## 常见问题

https://canonical-cloud-init.readthedocs-hosted.com/en/latest/reference/faq.html
