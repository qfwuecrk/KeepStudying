# Nginx

## 基本命令
```shell
# 测试配置文件是否正确
nginx -t

# 重启/查看/停止
systemctl restart nginx
systemctl status nginx
systemctl stop nginx

# log
cat /var/log/nginx/error.log
cat /var/log/nginx/access.log

# ---------------------------------

# conf
/etc/nginx/nginx.conf

# 在该文件夹下添加配置文件
/etc/nginx/sites-available/

# 将sites-available文件夹中的配置文件软链到这里生效
/etc/nginx/sites-enabled/
# 配置时补充绝对路径
ln -s /etc/nginx/sites-available/site_nginx.conf /etc/nginx/sites-enabled/ 

# ---------------------------------

# uwsgi
/etc/nginx/uwsgi_params 中包含了一些标准的配置信息
```

## 部署Django

请求逻辑：http请求 -> nginx -> uwsgi -> web应用

1. 安装uwsgi

    **pip安装的可以直接用（要使用venv环境），不用再配置。其他方法安装应该还要配置python。**
    ```shell
    python -m pip install uwsgi -i https://pypi.tuna.tsinghua.edu.cn/simple
    ```

2. 设置当前用户权限
检查用户以及用户组，保证启动uwsgi和创建相关配置时拥有权限。可以将其加入www-data用户组。

3. uwsgi配置文件
```shell
# 创建uwsgi.ini文件

[uwsgi]
uid=1000
gid=33
chdir=/var/www/nikki_web
module=nikki.wsgi:application
master=True
pidfile=/etc/nikki_web/nikki_web.pid
socket=/etc/nikki_web/nikki.sock
vacuum=True
harakiri=20
max-requests=5000
home=/var/www/nikki_web/.venv
daemonize=/var/log/uwsgi/nikki_web.log
```

4. 添加nginx配置文件
```shell
# vim /etc/nginx/sites-available/mysite.conf

# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    server unix:///etc/nikki_web/nikki.sock; # for a file socket
    # server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      10080;
    # the domain name it will serve for
    # server_name 47.97.7.143; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    # location /media  {
    #     alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    # }

    location /static {
        alias /var/www/nikki_web/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

5. 创建软链
```shell
ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/ 
```

6. 检查nginx配置文件
```shell
nginx -t
```

7. 启动uwsgi
```shell
uwsgi --ini uwsgi.ini
```

8. SSL
```sh
# FreeBSD nginx示例
server {
    listen      443 ssl;
    server_name localhost;

    ssl_certificate     cert.pem; # 证书路径
    ssl_certificate_key cert.key; # 私钥路径

    ssl_session_cache   shared:SSL:1m;
    ssl_session_timeout 5m;

    location / {
        root    html;
        index   index.html index.htm;
    }
}
```
