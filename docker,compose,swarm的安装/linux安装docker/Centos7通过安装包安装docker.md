## Centos7安装docker

[Centos7的阿里云docker安装包下载地址https://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7/Packages/](https://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7/Packages/)  
[docker-compose的官方下载地址https://github.com/docker/compose/releases/](https://github.com/docker/compose/releases/)  
[docker-compose的阿里云下载地址https://mirrors.aliyun.com/docker-toolbox/linux/compose/](https://mirrors.aliyun.com/docker-toolbox/linux/compose/)  
[docker-machine的官方下载地址https://github.com/docker/machine/releases/](https://github.com/docker/machine/releases/)  
[docker-machine的阿里云下载地址https://mirrors.aliyun.com/docker-toolbox/linux/machine/](https://mirrors.aliyun.com/docker-toolbox/linux/machine/)  
[香港内核仓库http://hkg.mirror.rackspace.com/elrepo/kernel/el7/x86_64/RPMS/](http://hkg.mirror.rackspace.com/elrepo/kernel/el7/x86_64/RPMS/)  
[清华大学内核仓库https://mirrors.tuna.tsinghua.edu.cn/elrepo/kernel/el7/x86_64/RPMS/](https://mirrors.tuna.tsinghua.edu.cn/elrepo/kernel/el7/x86_64/RPMS/)  
[中国科技大学内核仓库http://mirrors.ustc.edu.cn/elrepo/kernel/el7/x86_64/RPMS/](http://mirrors.ustc.edu.cn/elrepo/kernel/el7/x86_64/RPMS/)  
[内核仓库http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/](http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/)  
[内核仓库http://elrepo.org/linux/kernel/el7/x86_64/RPMS/](http://elrepo.org/linux/kernel/el7/x86_64/RPMS/)  
[内核仓库http://ftp.colocall.net/pub/elrepo/archive/kernel/el6/x86_64/RPMS/](http://ftp.colocall.net/pub/elrepo/archive/kernel/el6/x86_64/RPMS/)  

> 1.卸载自带的docker服务
>
> 2.切换成阿里的yum源
>
> 3.升级内核，并把该内核启动顺序设置为第一
>
> 4.安装docker，并设置阿里docker加速器
>
> 5.安装docker-compose
>
> 6.安装docker-machine

> #### 1.卸载自带的docker服务

```
yum remove docker docker-common docker-selinux docker-engine
```

> #### 2.更换yum源

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
echo "export LC_ALL=en_US.UTF-8" >> /etc/profile
source /etc/profile
yum clean all
yum makecache
```

> #### 3.升级内核，并把该内核启动顺序设置为第一(有个坑,香港的内核仓库只保留最新的两个版本，先要去到网站查看最先的安装包)

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install -y http://hkg.mirror.rackspace.com/elrepo/kernel/el7/x86_64/RPMS/kernel-lt-4.4.169-1.el7.elrepo.x86_64.rpm    //香港镜像仓库
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
grub2-set-default 0
```

> #### 4.安装18.03版本docker，并设置阿里docker加速器
```
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-engine \
           docker-ce
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
EOF
systemctl enable docker #设置docker服务开机自启动
sudo systemctl restart docker

# 生产环境一定要加graph选项，指定docker镜像和日志的目录为大空间的目录，否则死的时候武眼睇
# echo $PASSWD | sudo tee /etc/docker/daemon.json <<-'EOF'
# {
#    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
#    "graph": "/opt/data/docker"
# }
# EOF
```

##### 安装18.09版本docker
```
# yun安装软件的顺序不能乱
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-engine \
           docker-ce
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.0-3.el7.x86_64.rpm
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-18.09.0-3.el7.x86_64.rpm 
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.09.0-3.el7.x86_64.rpm 
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://1a5q7qx0.mirror.aliyuncs.com"]
}
EOF
systemctl enable docker #设置docker服务开机自启动
sudo systemctl restart docker

# 生产环境一定要加graph选项，指定docker镜像和日志的目录为大空间的目录，否则死的时候武眼睇
# echo $PASSWD | sudo tee /etc/docker/daemon.json <<-'EOF'
# {
#    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
#    "graph": "/opt/data/docker"
# }
# EOF
```

> #### 5.安装docker-compose
```
curl -L https://mirrors.aliyun.com/docker-toolbox/linux/compose/1.21.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

> #### 6.安装docker-machine
```
curl -L https://mirrors.aliyun.com/docker-toolbox/linux/machine/0.15.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-machine
```