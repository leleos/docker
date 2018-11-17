这是Docker的部署环境

1.安装docker
① 卸载现有的docker，这里只说明centos下的。其他方案请见：https://docs.docker.com/install/linux/docker-ce/centos

$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

② 安装相关包
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2


③ 设置源
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo


④ 安装最新版本的docker
sudo yum install docker-ce

⑤ 启动
sudo systemctl start docker

⑥ 验证是否成功

可以尝试运行
sudo docker run hello-world

也可以看看版本
sudo docker version
输出如下信息：
Client:
 Version:           18.09.0
 API version:       1.39
 Go version:        go1.10.4
 Git commit:        4d60db4
 Built:             Wed Nov  7 00:48:22 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.0
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       4d60db4
  Built:            Wed Nov  7 00:19:08 2018
  OS/Arch:          linux/amd64
  Experimental:     false

安装成功



2.安装docker-compose

① 下载
curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose


② 设置权限
chmod +x /usr/local/bin/docker-compose

③ 查看版本

docker-compose version

docker-compose version 1.22.0, build f46880fe
docker-py version: 3.4.1
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.1.0f  25 May 2017



3.下载相关的images

在相关内容描述
