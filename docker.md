# docker

## linux安装docker

1、移除docker

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



2、配置yum源

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```



3、安装docker

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io


#以下是在安装k8s的时候使用
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```



4、启动

```bash
systemctl enable docker --now
```



5、配置加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 下载镜像

```bash
#从docker 公网仓库下载nginx
docker pull nginx:laster

#从docker 搭建的私有仓库下载nginx
docker pull 192.168.1.66:5000/nginx:laster

## 下载来的镜像都在本地
docker images  #查看所有镜像

redis = redis:latest

docker rmi 镜像名:版本号/镜像id
```



## 启动容器

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

【docker run  设置项   镜像名  】 镜像启动运行的命令（镜像里面默认有的，一般不会写）

# -d：后台运行
# --restart=always: 开机自启
docker run --name=mynginx   -d  --restart=always -p  88:80   nginx


# 查看正在运行的容器
docker ps
# 查看所有
docker ps -a
# 删除停止的容器
docker rm  容器id/名字
docker rm -f mynginx   #强制删除正在运行中的

#停止容器
docker stop 容器id/名字
#再次启动
docker start 容器id/名字

#应用开机自启
docker update 容器id/名字 --restart=always
```



## 修改容器内容

1、进入容器内部修改

```bash
# 进入容器内部的系统，修改容器内容
docker exec -it 容器名/容器id  /bin/bash
```

2、挂载目录/文件 到外部修改

```bash
docker run --name=mynginx   \
-d  --restart=always \
-p  88:80 -v /data/html:/usr/share/nginx/html:ro  \
nginx

# 修改页面只需要去 主机的 /data/html
```



## 提交改变

```bash
#docker commit :从容器创建一个新的镜像。
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

docker commit -a "leifengyang"  -m "首页变化" 341d81f7504f guignginx:v1.0

```



## [镜像传输](https://blog.csdn.net/qq_44273583/article/details/114387368)

```bash
# 将镜像保存成压缩包
docker save 0fdf2b4c26d3 > hangge_server.tar
docker save -o images.tar postgres:9.6 mongo:3.4

# 别的机器加载这个镜像
docker load < hangge_server.tar

#或者
docker export f299f501774c > hangger_server.tar
docker import - new_hangger_server < hangger_server.tar

二者有差别，详细看标题的网址
```



## 上传私有仓库

1.、登录远程仓库

如果你的镜像仓库不需要登录就可以直接push的话，可以忽略这一步。

```bash
# docker login --username=管理仓库的用户名 仓库地址
docker login --username=xua0407 192.168.1.50:7780
```

2、 标记要上传的镜像为本地镜像

```bash
# docker tag 镜像ID 远程仓库地址/目标镜像名:[标签名]
docker tag 镜像ID 192.168.1.50:7780/junpeng/my_docker:1.0
```

3、将镜像上传到远程仓库

```bash
# docker push 远程仓库地址/目标镜像名:[标签名]
docker push 192.168.1.50:7780/junpeng/my_docker:1.0
```

4、推出登录

```bash
docker logout
```



## 补充

```bash
docker logs 容器名/id  标签name

docker exec -it 容器id /bin/bash


# docker 经常修改nginx配置文件
docker run -d -p 80:80 \
-v /data/html:/usr/share/nginx/html:ro \
-v /data/conf/nginx.conf:/etc/nginx/nginx.conf \
--name mynginx-02 \
nginx


#把容器指定位置的东西复制出来 
docker cp 5eff66eec7e1:/etc/nginx/nginx.conf  /data/conf/nginx.conf
#把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf  5eff66eec7e1:/etc/nginx/nginx.conf
```



## 打包镜像

1.常用指令讲解

```crystal
FROM [镜像:版本]：指定所依赖的基础镜像

MAINTAIMER 	指定维护者的信息

RUN <命令行命令>：等同于在终端执行的shell命令

RUN ["可执行文件", "参数1", "参数2"]：等同于在终端shell中执行 ./可执行文件

COPY <源文件> <目标文件>：将Dockerfile同目录下的文件拷贝到容器里面

ADD <源文件> <目标文件>：类似于COPY，区别在于如果文件是*.tar、*.gzip、*.bzip2等文件，会自动解压缩

WORKDIR		指定工作目录

VOLUME		设置挂载卷

EXPORT		指定对外的端口信息

ENV			设置环境变量，这个是全局的

ENTRYPOINT	容器启动后执行的命令

CMD			指定容器启动后执行的命令
```

2、编写dockerfile

```bash
[root@docker docker]# vim Dockerfile 
FROM frolvlad/alpine-glibc:alpine-3.14_glibc-2.33
MAINTAINER dxy
ADD ./jdk-8u301-linux-x64.tar.gz /opt/
ENV PATH=/opt/jdk1.8.0_301/bin:$PATH
CMD ["/bin/sh"]
# 查看目录下有的文件
[root@docker docker]# ls
Dockerfile  jdk-8u301-linux-x64.tar.gz

```



3、打包镜像

```bash
docker build -t java/jdk:8_301 .
# 参数
-t：指定镜像的名字和版本
-f：指定dockerfile文件
.：是使用当前目录的dockerfile

```

