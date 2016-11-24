# nginx-uwsgi-django-initctl
使用nginx和uwsgi启动django

#1yum安装nginx(也可以源码包安装)
```
yum install nginx -y
```

##配置nginx
```
cd /etc/nginx/conf.d
vim cmdb.conf
```
文件内容如下:
```
# cmdb nginx conf

# configuration of the server
server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name 192.168.56.101; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django static

    location /static {
        alias /data/www/cmdb/cmdb/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  0.0.0.0:8080;
        include     /data/www/cmdb/uwsgi_params; # the uwsgi_params file you installed
    }
}
```
**将nginx目录下的uwsgi_params拷贝到django的相应目录中(创建一个软链接也可)**

重启nginx，如果没有报错，说明nginx没有问题

#2安装配置uwsgi
##安装uwsgi
```
pip install uwsgi
```
##配置uwsgi参数
进入到django目录中,新建uwsgi.ini,内容如下:
```
[uwsgi]
socket = 0.0.0.0:8080
processes = 8
chdir = /data/www/cmdb/
pythonpath = /usr/bin/python
env = DJANGO_SETTINGS_MODULE=cmdb.settings
module = django.core.wsgi:get_wsgi_application()
daemonize = /var/log/cmdb.log
pidfile = /tmp/cmdb.pid
```
启动uwsgi
```
uwsgi --wsgi-file uwsgi.ini
```

这样子就能启动uwsgi了。但是如果要关闭或者重启uwsgi的话，目前只能通过脚本去killi掉相应的进程然后重新运行上面的命令。

#3uwsgi的启动，停止，重启方法

创建文件**/etc/init/uwsgi.conf**，内容如下:
```
description "uWSGI Emperor"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
exec /usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals/ --logto /var/log/cmdb.log
```

创建/etc/uwsgi/vassals/目录，将之前的uwsgi.ini文件cp到下面

运行命令```initctl stop uwsgi```

**usage:**
```
initctl start | stop | reload | restart uwsgi
```
