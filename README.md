# RBAC for Django and DRF

基于 Django (和 Django Rest Framework, JWT) 构建 RBAC.

在利用第三方插件或者 Oauth 2 的服务之前, 先通过 django 实现一次 RBAC 的逻辑.
## Original
借鉴 
[rest_xops](https://github.com/xufqing/rest_xops)
项目, 从中提取出 RBAC 功能, 此项目到 2022 年 4 月, 已经年久失修, django 2, vue 2.5, 大量依赖已经 deprecated, 但是仔细调教, 仍然能跑起来, 接下来将逐步将此项目进行升级.

以下各种描述都是截止于 [rest_xops](https://github.com/xufqing/rest_xops) 之前的进展.

## Progress

- [x] 修复 AUTH_USER_MODEL, 使得在 admin 页面可以管理自定义的 rbac 的 user 表. (always use a custom user model for all new Django projects.)
- [ ] 升级 django 到 v4, 包括 djangorestframework-simplejwt 的替换.
- [ ] 集成 django-organizations，以便管理采用 saas 服务的企业/机构.




<br>
<br>
<br>
<br>
<br>

---


## Ex-project 进展程度(已停更)

管理员账号:admin 密码admin@1234

![1.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/1.png)

![2.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/2.png)

![3.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/3.png)

![4.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/4.png)

**持续构建样例**

![11.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/11.png)
![22.png](https://raw.githubusercontent.com/xufqing/rest_xops/master/demo/22.png)

***特色功能开发进度***

- 已完成
- [X] 基于RBAC的用户权限管理
- [X] 资产扫描功能（目前只能扫描linux服务器）
- [X] 持续构建（websocket实时显示过程）
- [X] 项目应用控制（启动停止）
- [X] 应用日志的实时查看（websocket）
- [X] 对象级权限控制（密码管理功能，只能查看编辑自己的密码）
- [X] 应用日志下载
- 待开发
- [ ] 工单系统
- [ ] 作业平台（ansible api）
- [ ] 文件管理
- [ ] WEBSSH（堡垒机）
- [ ] SQL作业平台
- ...........

*** fork 过来的项目是采用以下技术，这些都需要之后升级 ***

前端：

vue 2.5.17

elementUI 2.6.0

websocket

后端：

django 2.1.7

django rest framework 3.9.1

django rest framework-jwt 1.11.0

django channels 2.1.7

celery 4.2.1

软件运行环境

Python 3.6.8

Redis 5.0

MySql 5.6.42

操作系统：

CentOS 7

***系统更新方式***
```
cd 你的项目地址
sh upgrade.sh
更新完毕后，修改管理员密码
python manage.py changepassword admin

前端更新build后放到你的nginx静态目录
```

***基础环境安装***


1、安装npm运行环境(略)

    安装前端项目依赖包
    
    cd xops_qd/
    npm install

2、修改你的配置文件

    xops_qd/config

    dev.env.js #开发环境

    prod.env.js #生成环境

3、安装mysql（略）

4、安装redis（略）

5、创建python虚拟环境, pipenv

- 安装  
yum install git gcc make patch gdbm-devel openssl-devel sqlite-devel readline-devel zlib-devel bzip2-devel libffi-devel -y  

- 多版本管理工具
1. 安装pipenv 
```
pip3 install pipenv
```
2、安装项目运行模块
```
# pipenv --python 3.9
pipenv shell
pipenv install -r requirements.txt
```
3、修改配置
```
vi rest_xops/settings.py 
# 修改数据库
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'rest_xops',
        'HOST': '127.0.0.1',
        'USER': 'root',
        'PASSWORD': '123456',
        'PORT': '3306',
        'OPTIONS': { 'init_command': 'SET storage_engine=INNODB;' }
    }
}
# 修改redis
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("127.0.0.1", 6379)],
        },
    },
}

#修改redis
vi rest_xops/celery.py

BROKER_URL = 'redis://localhost:6379/1' # Broker配置，使用Redis作为消息中间件

CELERY_RESULT_BACKEND = 'redis://localhost:6379/1' # Backend设置，使用redis作为后端结果存储


```


4、登陆MYSQL，创建数据库

```
CREATE DATABASE rest_xops DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

5、执行创建表信息

python manage.py makemigrations rbac

python manage.py makemigrations cmdb

python manage.py makemigrations deployment

python manage.py migrate

**导入初始化数据**

python manage.py loaddata init_data/*.json

如果遇到mysql模块的问题

ImportError: libmysqlclient.so.18: cannot open shared object file: No such file or directory

则：

ln -s /usr/local/mysql/lib/libmysqlclient.so.18 /usr/lib64/libmysqlclient.so.18



6、修改管理员密码（必须操作）
    
    python manage.py changepassword admin
    

7、安装扫描工具

    yum -y install nmap

8、安装进程管理

    cd ~ && yum -y install epel-release && yum -y install supervisor


9、开机启动

    systemctl enable supervisord
    systemctl start supervisord
10、配置supervisor来管理celery

- celery_worker
```
mkdir -p /var/log/rest_xops

vim /etc/supervisord.d/celery_worker.ini

[program:celery-worker]
command=/home/xufeng/.pyenv/versions/rest_xops/bin/celery worker -A rest_xops -l INFO
directory=/home/xufeng/rest_xops
environment=PATH="/home/xufeng/.pyenv/versions/rest_xops/bin/"
stdout_logfile=/var/log/rest_xops/worker_celery.log
stderr_logfile=/var/log/rest_xops/worker_celery.log
autostart=true
autorestart=true
priority=901

#flower是celery的监控，请注意修改redis连接
[program:celery-flower]
command=/home/xufeng/.pyenv/versions/rest_xops/bin/celery flower --broker=redis://localhost:6379/1
directory=/home/xufeng/rest_xops
environment=PATH="/home/xufeng/.pyenv/versions/rest_xops/"
stdout_logfile=/var/log/rest_xops/flower_celery.log
stderr_logfile=/var/log/rest_xops/flower_celery.log
autostart=true
autorestart=true
priority=900


执行
supervisorctl start all

```


***开发环境的运行***

运行前端
```
cd ./xops_qd
npm run dev
#接下来启动后端即可进行调试
```
运行后端

    python manage.py runserver 0.0.0.0:8000
    
浏览器登陆

    http://localhost:8013/   前端
    http://你的后端地址:8000/docs/  API文档地址
    

***生产环境的运行***

**1、安装nginx（略）**

    以下所有/home/xufeng/rest_xops 都改为你的项目地址

    /home/xufeng/.pyenv/versions/rest_xops 是你的python虚拟环境的版本

**2、设置supervisord**
- xops_gunicorn
```
vim /etc/supervisord.d/xops_gunicorn.ini

[program:xops]
command=/home/xufeng/.pyenv/versions/rest_xops/bin/gunicorn rest_xops.wsgi:application -c /home/xufeng/rest_xops/rest_xops/gunicorn.py
directory=/home/xufeng/rest_xops
stdout_logfile=/var/log/rest_xops/xops.log
stderr_logfile=/var/log/rest_xops/xops.log
stdout_logfile_maxbytes = 20MB
autostart=true
autorestart=false
```
- xops_asgi
```
vim /etc/supervisord.d/xops_asgi.ini 

[fcgi-program:asgi]
socket=tcp://127.0.0.1:8001
directory=/home/xufeng/rest_xops
command=/home/xufeng/.pyenv/versions/rest_xops/bin/daphne -u /run/daphne%(process_num)d.sock --fd 0 --access-log - --proxy-headers rest_xops.asgi:application
numprocs=4
process_name=asgi%(process_num)d
autostart=true
autorestart=true
stdout_logfile=/var/log/rest_xops/asgi.log
stderr_logfile=/var/log/rest_xops/asgi.log
stdout_logfile_maxbytes = 20MB
```

**3、重启supervisord**

    supervisorctl reload

    systemctl restart supervisord

**4、查看运行状态**

    supervisorctl status

以下输出为成功

```
[root@xuxu xufeng]# supervisorctl status
asgi:asgi0                       RUNNING   pid 15331, uptime 1:42:43
asgi:asgi1                       RUNNING   pid 15329, uptime 1:42:44
asgi:asgi2                       RUNNING   pid 15343, uptime 1:42:42
asgi:asgi3                       RUNNING   pid 15335, uptime 1:42:43
celery-flower                    RUNNING   pid 15320, uptime 1:42:45
celery-worker                    RUNNING   pid 15321, uptime 1:42:45
xops                             RUNNING   pid 15328, uptime 1:42:44
[root@xuxu xufeng]# supervisorctl status
```
**5、生产静态页面**
```
cd rest_xops/xops_qd
npm run build
将xops_qd/dist目录下生成的文件复制到你的服务器nginx web root目录
```
**6、配置NGINX

```
upstream xops {
   server 127.0.0.1:8000;
}
upstream channels-backend {
   server 127.0.0.1:8001;
}

server {
    listen      80;
    server_name demo.xufqing.cn;#你的访问地址,和前端生产的地址一致
    location / {
        try_files $uri $uri/ /index.html;
        root /home/xufeng/rest_xops/xops_web; # 这是前端静态文件
    }
    location ^~/api  {
        proxy_pass http://xops;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location ^~/auth  {
        proxy_pass http://xops;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location ^~/media  {
        alias /home/xufeng/rest_xops/media;
    }
    location ^~/websocket/console {
        try_files $uri @proxy_to_app;
    }
    #location /flower {
    #    proxy_pass http://127.0.0.1:5555;
    #}
    location @proxy_to_app {
        proxy_pass http://channels-backend;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
    }
}
```

