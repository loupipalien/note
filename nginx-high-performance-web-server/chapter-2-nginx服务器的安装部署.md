### nginx 服务器的安装部署

#### 如何获取 Nginx 服务器安装文件
TODO
#### 安装 Nginx 服务器和基本配置
TODO

#### Nginx 服务的启停控制
Nginx 服务在运行中, 会保持一个主进程和一个或多个 worker process 工作进程
TODO

#### Nginx 服务器基础配置指令
默认的 Nginx 服务器配置文件存放在 conf 目录中, 主配置文件名为 nginx.conf; 本节将介绍此文件的内容和基本配置方法, 以下是默认的 nginx.conf 文件中的完整内容 (部分省略)
```
worker_processes 1;  # 全局生效

events {
    worker_connections 1024;   # 在 events 部分中生效
}

http {  # 以下指令在 http 部分生效
    include mime.types;  
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    server {  # 以下指令在 http 的 server 部分中生效
        listen 80;
        server_name localhost;
        location / {  # 以下指令在 http/server 的 location 中生效
            root html;
            index index.html index.htm;
        }
        error_page 500 502 503 504 /50x.html
        location = /50x.html {
            root html;
        }
    }
}
```
