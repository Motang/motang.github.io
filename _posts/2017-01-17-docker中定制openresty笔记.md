---
layout: post
title: docker中定制openresty笔记
category: docker
tags: [docker openresty nginx]
comments: true
share: true
---
## 目录 ##

* Table of Contents
{:toc}

## 介绍 ##
前后端微服务架构，便于前端人员和后端人员做开发调试，可以在本地PC安装openresty，配置好静态文件和动态API接口相关配置，达到前后端开发调试无缝对接。

## 步骤 ##

> 编写Dockerfile，内容如下，目的拉取docker的openresty的镜像，创建新的openresty工作目录，并启动

    FROM openresty/openresty:latest

	MAINTAINER Morly Tang <tangjimo@foonsu.com>

	# 1) Run OpenResty

	RUN  mkdir /usr/local/openresty/workspace

	ENTRYPOINT ["/usr/local/openresty/bin/openresty", "-p", "/usr/local/openresty/workspace", "-g", "daemon off;"]

> 在本机新编辑~/openresty/test/conf/nginx.conf文件，内容如下

```lua
worker_processes  1;        #nginx worker 数量
error_log logs/error.log error;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {
    ## 设置纯Lua的路径库，（';;' is the default path）
    lua_package_path '$prefix/lua/?.lua;;';
    #lua_package_path '$prefix/lua/?.lua;/blah/?.lua;;';

    ## 热加载开关
    lua_code_cache off;
    ## 使用Lua shared dict，这个cache是nginx所有worker之间共享的，内部使用的LRU算法（最近最少使用）来判断缓存是否在内存占满时被清除
    lua_shared_dict my_cache 128m;

    upstream backend-web {
        server www.1dadan.com;
        #ip_hash;
    }

    server {
        #监听端口，若你的80端口已经被占用，则需要修改
        listen 80;

        location / {                                                          
            root   html;                                                      
            index  index.html index.htm;                                      
        }

        #后端的API接口配置
        location /api {
            proxy_pass  http://backend-web;
            proxy_read_timeout 150;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        #前端配置，文件位置，相对目录下的kdzs目录下
        location ^~ /kdzs/ {
            root kdzs;
            index  index.html index.htm;
        }
    }
}
```

> 进入Dockerfile文件所在的目录，执行下面的命令

    docker build -t morly/myopenresty:1.0 -f Dockerfile .

> 执行运行下面

    docker run --name openresty-test -v /home/morly/openresty/test:/usr/local/openresty/workspace -p 80:80 -d morly/myopenresty:1.0

> 测试, http://localhost/kdzs/index.html  http://localhost/api/captcha/captchaImage
