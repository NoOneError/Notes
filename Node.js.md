# Node.js

[toc]

##  安装配置

###  环境

- 环境： CentOS Linux
- 版本

```bash
cat /etc/centos-release  # 查看centos系统版本
12
```

- 安装node.js依赖

```ba
yum -y install gcc gcc-c++ kernel-devel
```

###  安装node.js

- 下载源码，你需要在[网址](https://nodejs.org/zh-cn/download/)下载最新的Nodejs版本

```bash
cd /usr/local/src/
wget https://nodejs.org/dist/v14.17.5/node-v14.17.5-linux-x64.tar.xz
```

- 解压源码

```bash
tar -xvf node-v14.17.5-linux-x64.tar.xz
```

-  配置NODE_HOME，进入profile编辑环境变量

```
vim /etc/profile
```

- 设置 nodejs 环境变量，在 ***export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL*** 一行的上面添加如下内容:

```
#set for nodejs
export NODE_HOME=/usr/local/node/
export PATH=$NODE_HOME/bin:$PATH
```

- :wq保存并退出，编译/etc/profile 使配置生效

```
source /etc/profile
```

- 验证是否安装配置成功

```
node -v
```

##  [linux后台运行nodejs项目](https://www.cnblogs.com/SimonHu1993/p/11646709.html)

1.安装pm2，这里默认你已经安装了node.js和npm

```
npm install pm2 -g
```

![img](../../Pictures/Saved%20Pictures/1162521-20191010105742110-1144179429.png)

 2.创建软连接

 1）全局path路径

```
 echo $PATH
```

![img](../../Pictures/Saved%20Pictures/1162521-20191010105909779-749918278.png)

 2）pm2安装路径

安装pm2时，可看到pm2安装路径

3）建立软连接

```
ln` `-s ``/usr/sbin/nodejs/bin/pm2` `/usr/local/bin/` `# 前一个为pm2安装目录，后一个选择path内任意一个：分割的地址
```

3.确认是否安装成功 

```
pm2 list
```

![img](../../Pictures/Saved%20Pictures/1162521-20191010110359178-256352176.png)

 这里我们启动一个node项目

```
pm2 start app.js
```

![img](../../Pictures/Saved%20Pictures/1162521-20191010110457761-2082169703.png)

```
常用命令
$ pm2 start app.js ``# 启动app.js应用程序``
$ pm2 start app.js –name=”api” ``# 启动应用程序并命名为 “api”``
$ pm2 start app.js –``watch` `# 当文件变化时自动重启应用``
$ pm2 start script.sh ``# 启动 bash 脚本` `
$ pm2 list ``# 列表 PM2 启动的所有的应用程序` `
$ pm2 monit ``# 显示每个应用程序的CPU和内存占用情况` `
$ pm2 show [app-name] ``# 显示应用程序的所有信息` `
$ pm2 logs ``# 显示所有应用程序的日志` `
$ pm2 logs [app-name] ``# 显示指定应用程序的日志``
$ pm2 stop all ``# 停止所有的应用程序` `$ pm2 stop 0 ``# 停止 id为 0的指定应用程序` `
$ pm2 restart all ``# 重启所有应用` `$ pm2 reload all ``# 重启 cluster mode下的所有应用
$ pm2 gracefulReload all ``# Graceful reload all apps in cluster mode` `
$ pm2 delete all ``# 关闭并删除所有应用` `
$ pm2 delete 0 ``# 删除指定应用 id 0` `
$ pm2 scale api 10 ``# 把名字叫api的应用扩展到10个实例` `
$ pm2 reset [app-name] ``# 重置重启数量` `
$ pm2 startup ``# 创建开机自启动命令` `
$ pm2 save ``# 保存当前应用列表` `
$ pm2 resurrect ``# 重新加载保存的应用列表` `
$ pm2 update ``# Save processes, kill PM2 and restore processes` `
$ pm2 generate ``# Generate a sample json configuration file
```

##  放行端口

首先确保防火墙是开着的

```
查看防火墙状态

systemctl status firewalld
```

```
开启防火墙

systemctl start firewalld
```

```shell
防火墙放行端口
1.添加端口
 6666 代表端口号
firewall-cmd --zone=public --add-port=6666/tcp --permanent
#说明:
#–zone #作用域
#–add-port=80/tcp #添加端口，格式为：端口/通讯协议
#–permanent 永久生效，没有此参数重启后失效

#多个端口:
firewall-cmd --zone=public --add-port=1-6/tcp --permanent
2.刷新生效
firewall-cmd --reload
3.查看防火墙放行列表
firewall-cmd --list-all


```

```
多说一点

firewall-cmd --state                           #查看防火墙状态，是否是running

firewall-cmd --reload                          #重新载入配置，比如添加规则之后，需要执行此命令

firewall-cmd --get-zones                       #列出支持的zone

firewall-cmd --get-services                    #列出支持的服务，在列表中的服务是放行的

firewall-cmd --query-service ftp               #查看ftp服务是否支持，返回yes或者no

firewall-cmd --add-service=ftp                 #临时开放ftp服务

firewall-cmd --add-service=ftp --permanent     #永久开放ftp服务

firewall-cmd --remove-service=ftp --permanent  #永久移除ftp服务

firewall-cmd --add-port=80/tcp --permanent     #永久添加80端口 

firewall-cmd --remove-port=80/tcp --permanent  #永久移除80端口

firewall-cmd --list-ports                      #查看已经开放的端口

iptables -L -n                                 #查看规则，这个命令是和iptables的相同的

man firewall-cmd                               #查看帮助
```

