# Launchpad搭建


## 环境
- Ubuntu 20.04服务器版（注意：16.04之前不支持了，22.04也不支持，截止2023-07-04支持的最新操作系统是20.04）
- python3.5 +
- LXD 容器

## 配置LXD 容器环境

1. 安装 snap `sudo apt install snap`
2. 安装 lxd  `sudo snap install lxd`
3. lxd init `lxd init` 先不使用集群模式
4. 修改 lxd 镜像源 `lxc remote add tuna-images https://mirrors.tuna.tsinghua.edu.cn/lxc-images/ --protocol=simplestreams --public`
5. 执行如下脚本
```shell
#! /bin/sh

id=400000  # some large uid outside of typical range, and outside of already mapped ranges in /etc/sub{u,g}id
uid=$(id -u)
gid=$(id -g)
user=$(id -un)
group=$(id -gn)

# give lxc permission to map your user/group id through
sudo usermod --add-subuids ${uid}-${uid} --add-subgids ${gid}-${gid} root

# create a profile to control this
lxc profile create $user >/dev/null 2>&1

# configure profile
cat << EOF | lxc profile edit $user
name: $user
description: allow home dir mounting for $user
config:
  raw.idmap: |
    uid $uid $id
    gid $gid $id
  user.user-data: |
    #cloud-config
    runcmd:
      - "groupadd $group --gid $id"
      - "useradd $user --uid $id --gid $group --groups adm,sudo --shell /bin/bash"
      - "echo '$user ALL=(ALL) NOPASSWD:ALL' >/etc/sudoers.d/90-cloud-init-users"
      - "chmod 0440 /etc/sudoers.d/90-cloud-init-users"
devices:
  home:
    type: disk
    source: $HOME
    path: $HOME
EOF
```
6. 创建一个容器 `lxc launch ubuntu:20.04 lpdev -p default -p $USER`
7. 使用 `lxc info lpdev` 查看容器IP，我的是`10.106.102.170`
8. 执行 `ssh-keygen`
9. 把 `~/.ssh/id_rsa.pub` 中的内容复制到 `~/.ssh/authorized_keys` 中
10. 执行：`ssh -A $USER@10.106.102.170(这里换成容器IP)`进入容器中
11. `mkdir ~/launchpad && cd ~/launchpad`
12. 修改配置文件`~/.ssh/config`
```
Host bazaar.launchpad.net
        User LPUSERNAME(用户名)
Host git.launchpad.net
        User LPUSERNAME(用户名)
```
13. 执行 `curl https://git.launchpad.net/launchpad/plain/utilities/rocketfuel-setup >rocketfuel-setup`
14. `chmod a+x rocketfuel-setup`
15. `./rocketfuel-setup`


## 参考

[https://launchpad.readthedocs.io/en/latest/how-to/running.html](https://launchpad.readthedocs.io/en/latest/how-to/running.html)

