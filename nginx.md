# [Nginx单节点搭建](https://lionli.blog.csdn.net/article/details/92586378)

获取nginx镜像

```bash
docker pull nginx
```

使用nginx镜像来创建nginx容器实例-----**获取配置文件**

```bash
docker run --name nginx-test -p 80:80 -d nginx
```

**将nginx关键目录映射到本机**

```ba
mkdir -p /root/nginx/www /root/nginx/logs /root/nginx/conf


docker cp 481e121fb29f:/etc/nginx/nginx.conf /root/nginx/conf
```

创建新nginx容器nginx-web,并将**www,logs,conf**目录映射到本地

```ba
docker run -d -p 80:80 --name nginx -v /root/nginx/www:/usr/share/nginx/html -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/logs:/var/log/nginx nginx
```

**设置反向代理**

进入到/root/conf/nginx.conf 添加如下即可

```ba
    
   	upstream nacos_server {
  		 server 192.168.1.160:8848;
  		 server 192.168.1.161:8848;
  		 server 192.168.1.162:8848;
	} 
    server{
       listen 80; //访问端口
       charset utf-8;
       server_name 192.168.112.135; //本机ip
 
       location / {
          proxy_pass http://nacos_server;//
          proxy_redirect default;
       }
    }
```

# nginx集群



