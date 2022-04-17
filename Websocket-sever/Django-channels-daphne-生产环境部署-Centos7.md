# Django-channels-daphne-生产环境部署-Centos7

## Djano 环境安装

**一、更新系统软件包**
yum update -y

**二、安装软件管理包和可能使用的依赖**

```c
yum -y groupinstall "Development tools"
yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel psmisc libffi-devel
```

**三、下载Pyhton3到/usr/local 目录**

```c
cd /usr/local
```

```c
wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tgz
```

解压

```c
tar -zxvf Python-3.6.6.tgz
```

进入 Python-3.6.6路径

```c
cd Python-3.6.6
```

编译安装到指定路径

```c
./configure --prefix=/usr/local/python3
```

注意：/usr/local/python3 路径可以自己指定，自己记着就行，下边要用到。

安装python3

```c
make
make install
```

安装完成之后 建立软链接 添加变量 方便在终端中直接使用python3

```c
ln -s /usr/local/python3/bin/python3.6 /usr/bin/python3
```

Python3安装完成之后pip3也一块安装完成，不需要再单独安装
同样给pip3建立软链接

```c
ln -s /usr/local/python3/bin/pip3.6 /usr/bin/pip3
```

**四、查看Python3和pip3安装情况**

![](http://tuchuang.deloped.com/PicGopip_install.jpg)

**五、安装virtualenv ，建议大家都安装一个virtualenv，方便不同版本项目管理。**

```
pip3 install virtualenv
```

建立软链接

```
ln -s /usr/local/python3/bin/virtualenv /usr/bin/virtualenv
```

安装成功在根目录下建立两个文件夹，主要用于存放env和网站文件的。(个人习惯，其它人可根据自己的实际情况处理)

```
mkdir -p /data/env
mkdir -p /data/wwwroot
```

**六、切换到/data/env/下，创建指定版本的虚拟环境。**

```
virtualenv --python=/usr/bin/python3 pyweb
```

然后进入/data/env/pyweb/bin
启动虚拟环境：

```C
source activate
```

![img]()

![timg.jpg](https://www.django.cn/media/upimg/timg_20180709220840_146.jpg)

留意我标记的位置，出现(pyweb)，说明是成功进入虚拟环境。

**七、虚拟环境里用pip3安django和uwsgi**

```C
pip3 install django （如果用于生产的话，则需要指定安装和你项目相同的版本）
pip3 install uwsgi
```

**留意：****uwsgi要安装两次**，先在系统里安装一次，然后进入对应的虚拟环境安装一次。

给uwsgi建立软链接，方便使用

```C
ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi
```

**留意：** 对于本项目，项目文件存放在/data/wwwroot/project下，至此Django环境部署完毕

## 安装Redis 并配置开机启动

**一、安装gcc依赖**

由于 redis 是用 C 语言开发，安装之前必先确认是否安装 gcc 环境（gcc -v），如果没有安装，执行以下命令进行安装

```c
 yum install -y gcc
```

**二、下载并解压安装包**

```
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar -zxvf redis-5.0.3.tar.gz
```

**三、cd切换到redis解压目录下，执行编译**

```
cd redis-5.0.3
make
```

**四、安装并指定安装目录**

```make install PREFIX=/usr/local/redis```

**五、启动服务**

***5.1前台启动***

``````C
cd /usr/local/redis/bin/
./redis-server
``````

***5.2后台启动***

从 redis 的源码目录中复制 redis.conf 到 redis 的安装目录

```cp /usr/local/redis-5.0.3/redis.conf /usr/local/redis/bin/```

修改 redis.conf 文件，把 daemonize no 改为 daemonize yes

``` vi redis.conF```

**六、设置开机启动**

***添加开机启动服务***

``` vi /etc/systemd/system/redis.service``

***复制粘贴以下内容：***

```c
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

*** 注意：ExecStart配置成自己的路径  ***

***设置开机启动***

```
[root@localhost bin]# systemctl daemon-reload
[root@localhost bin]# systemctl start redis.service
[root@localhost bin]# systemctl enable redis.service
```

***创建 redis 命令软链接***

```
[root@localhost ~]# ln -s /usr/local/redis/bin/redis-cli /usr/bin/redis
```

***测试 redis**

![](http://tuchuang.deloped.com/PicGoredis.png)

**附：服务操作命令**

```
systemctl start redis.service   #启动redis服务
systemctl stop redis.service   #停止redis服务
systemctl restart redis.service   #重新启动服务
systemctl status redis.service   #查看服务当前状态
systemctl enable redis.service   #设置开机自启动
systemctl disable redis.service   #停止开机自启动
```



## 安装daphne

**安装Daphne**

```pip install daphne```

**在setting.py同级的目录下新建一个asgi.py文件，内容如下**

```python
"""
ASGI entrypoint. Configures Django and then runs the application
defined in the ASGI_APPLICATION setting.
"""
import os
import django
from channels.routing import get_default_application
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
django.setup()
application = get_default_application()
```

**使用Daphne来启动项目**

```c
daphne -b 0.0.0.0 -p 8001 myproject.asgi:application
```

> 这种方式只是在后台启动了你的项目，我们为了保证项目的稳定性，保证项目在意外死亡以后可以迅速的重启，我们来使用supervisor来管理我们的daphne服务。

**什么是supervisor**

> Supervisor是用Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具，不支持Windows系统。
> 作用：它可以很方便的监听、启动、停止、重启一个或多个进程。
> 用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉起
> 很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。
> 说白了，它真正有用的功能是俩个将非daemon(守护进程)程序变成deamon方式运行对程序进行监控，当程序退出时，可以自动拉起程序。
> 但是它无法控制本身就是daemon的服务。

## 安装supervisor

```c
#yum安装：
yum install python-setuptools
easy_install supervisor
或者
yum install -y epel-release
yum install -y supervisor  
#手动安装：
wget https://pypi.python.org/packages/source/s/supervisor/supervisor-3.1.3.tar.gz
tar zxf supervisor-3.1.3.tar.gz
cd supervisor
python setup.py install
#pip安装：
pip install supervisor
```

**生成配置文件**

```
echo_supervisord_conf > /etc/supervisord.conf
```

**使用supervisor管理daphne进程**

**编辑/etc/supervisord.conf加入配置,supervisord.conf，内容如下**

```
[program:daphne]
directory=/opt/app/devops  #项目目录
command=daphne -b 127.0.0.1 -p 8001 --proxy-headers devops.asgi:application #启动命令
autostart=true
autorestart=true
stdout_logfile=/tmp/websocket.log  #日志
redirect_stderr=true
```

**启动supervisor**

```
supervisord -c /etc/supervisord.conf
```

## nginx安装和配置

**安装nginx和配置nginx.conf文件**

**进入home目录，执行下面命令**

```c
cd /home/
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -zxvf nginx-1.13.7.tar.gz
cd nginx-1.13.7
./configure
make
make install
# nginx一般默认安装好的路径为/usr/local/nginx
# 在/usr/local/nginx/conf/中先备份一下nginx.conf文件，以防意外。
cp nginx.conf nginx.conf.bak
```

***然后打开nginx.conf，把原来的内容删除，直接加入以下内容：***

```c
events {
    worker_connections  1024;
}
http{
upstream channels-backend {
    server 127.0.0.1:8001;
}
server {
    location / {
        try_files $uri @proxy_to_app;
    }
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
}
```

*** 进入/usr/local/nginx/sbin/目录***

**执行./nginx -t命令先检查配置文件是否有错，没有错就执行以下命令：**

```
./nginx
```

## 总结

> 本教程主要针对本人使用，中间省去了一些步骤，本人认为该部分最难得为.conf文件的配置和Daphne配置，故花费了大量篇幅在这两块地方，其他简略部分并不难。
>
> 后续应该还会补充上HTTPS的配置教程，编写ing！
>
> 对于毕设项目而言，不用懂得服务器配置的原理，只需按照教程配置成功即可！

