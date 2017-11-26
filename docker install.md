### Install Docker

ubuntu 16.04
```bash
# 卸载旧版本：
sudo apt-get remove docker docker-engine docker.io

sudo apt-get update

## ubuntu 14.04 安装内核可选模块
sudo apt-get install \
linux-image-extra-$(uname -r) \
linux-image-extra-virtual
## end


# 安装https
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

# 安装源

# 国内源
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key	add -
sudo add-apt-repository \
"deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
$(lsb_release -cs) \
stable"
# 官方源
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release	-cs) \
stable"

# 安装docker-ce
sudo apt-get update

sudo apt-get install docker-ce

# 还可以脚本自动安装
curl -fsSL get.docker.com -o get-docker.sh 
sudo sh get-docker.sh --mirror Aliyun

# 启动docker
sudo systemctl enable docker 
sudo systemctl start docker

## ubuntu 14.04 
sudo service docker start
## end

# 增加docker 用户组 
# 用户组在安装完docker自动增加的，如果没有可手动增加
sudo groupadd docker
# 将当前用户增加到docker组
sudo usermod -aG docker $USER

# 设置镜像加速器
# 在阿里注册申请加速器
# https://1f3mevbc.mirror.aliyuncs.com

# 编辑  /etc/systemd/system/multi-user.target.wants/docker.service 文件
# 找到  ExecStart= 追加  --registry-mirror=https://1f3mevbc.mirror.aliyuncs.com
sudo systemctl daemon-reload
sudo systemctl restart docker

## ubuntu 14.04 
## 编辑/etc/default/docker文件, DOCKER_OPTS追加  --registry-mirror=https://1f3mevbc.mirror.aliyuncs.com
sudo service docker restart
## end

```

### 容器命令行工具
```bash
sudo apt-get install util-linux
sudo docker ps
# 进入容器
tpid=`sudo docker inspect -f {{.State.Pid}} 404d5b9dc923`
sudo nsenter --target $tpid --mount --uts --ipc --net --pid
```

### 开启 docker 远程TCP访问
```bash
# 
# 编辑 /etc/systemd/system/multi-user.target.wants/docker.service 文件
# 找到 ExecStart= 追加 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 私有镜像仓库

```bash
# 不能访问私有http仓库
# 解决方法：
# 1. 编辑 /etc/docker/daemon.json 文件
# {"insecure-registries" : ["10.8.30.163:5005"]}

# 2. 编辑  /etc/systemd/system/multi-user.target.wants/docker.service 文件
# 找到  ExecStart= 追加 --insecure-registry 10.8.30.163:5005

```
### jenkins

```bash
docker pull jenkins
sudo mkdir /docker/jenkins
sudo chown 1000:1000 /var/jenkins

sudo docker run \
-d \
-p 8088:8080 -p 50005:50000 \
-v /docker/jenkins/:/var/jenkins_home \
-v /versions:/versions \
-v ~/.ssh:/var/jenkins_home/.ssh \
-v /etc/localtime:/etc/localtime:ro \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
-e JAVA_OPTS=-Duser.timezone=Asia/Shanghai \
--restart=always \
--name=myjenkins 10.8.30.163:5005/jenkins:10.28

# 共享主机 /var/run/docker.sock 后，jenkins 执行docker 会提示没有权限
# 进入Jenkins 容器 
chown 1000:1000 /var/run/docker.sock

# 创建 /etc/init.d/changedpermission.sh 脚本
chown 1000:1000 /var/run/docker.sock
# 把上面命令写入，保存。
ln -s /etc/init.d/changedpermission.sh /etc/rc*.d/changedpermission

```

### 私有镜像仓库
```bash
docker run -d -p 5005:5000 -v /my_registry:/var/lib/registry --restart=always --name registry-srv registry:2

docker run -it -p 8088:8080 --restart=always --name registry-web --link registry-srv -e REGISTRY_URL=http://10.8.30.163:5005/v2 -e REGISTRY_NAME=10.8.30.163:5005 hyper/docker-registry-web

```
