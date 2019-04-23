# CentOS7 + PHP7 + Apache + Nginx 环境配置与多站点部署

## 声明 <!-- omit in toc -->

以下[参考]链接，如有侵权，请联系删除，在此先感谢在网络上无私奉献的人们~

## 目录 <!-- omit in toc -->

- [CentOS7 + PHP7 + Apache + Nginx 环境与多站点部署](#centos7--php7--apache--nginx-环境与多站点部署)
  - [安装 PHP 7.2](#安装-php-72)
    - [1. 安装 epel](#1-安装-epel)
    - [2. 安装 php yum 仓库](#2-安装-php-yum-仓库)
    - [3. 安装 php 7.2 及扩展](#3-安装-php-72-及扩展)
    - [4. 修改 `php.ini`](#4-修改-phpini)
    - [5. 修改 `www.conf`](#5-修改-wwwconf)
    - [6. 开启服务](#6-开启服务)
  - [安装 Nginx 1.14](#安装-nginx-114)
    - [1. 下载安装包并解压](#1-下载安装包并解压)
    - [2. 安装 zlib pcre openssl 库](#2-安装-zlib-pcre-openssl-库)
    - [3. 配置、编译并安装](#3-配置编译并安装)
    - [4. 启动](#4-启动)
    - [5. 开放端口](#5-开放端口)
    - [6. 创建启动脚本 `nginx.service`](#6-创建启动脚本-nginxservice)
    - [7. 配置 php 支持](#7-配置-php-支持)
  - [安装 Apache 2.4](#安装-apache-24)
    - [1. 安装并启动 Apache](#1-安装并启动-apache)
    - [2. 创建 `index.php` 页面并测试](#2-创建-indexphp-页面并测试)
  - [多站点部署](#多站点部署)
    - [Nginx 多站点部署](#nginx-多站点部署)
      - [1. 修改 php-fpm 配置](#1-修改-php-fpm-配置)
      - [2. 创建多站点配置文件目录与文件](#2-创建多站点配置文件目录与文件)
      - [3. 修改 `nginx.conf`](#3-修改-nginxconf)
      - [4. 为项目目录赋予权限](#4-为项目目录赋予权限)
      - [5. 修改 SELinux 参数](#5-修改-selinux-参数)
      - [6. 重启 nginx 服务](#6-重启-nginx-服务)
    - [Apache 多站点部署](#apache-多站点部署)
      - [1. 配置 php-fpm](#1-配置-php-fpm)
      - [2. 创建多站点配置](#2-创建多站点配置)
      - [3. 修改 `httpd.conf`](#3-修改-httpdconf)
      - [4. 赋予权限](#4-赋予权限)
      - [5. 修改 SELinux](#5-修改-selinux)
      - [6. 重启 apache 服务](#6-重启-apache-服务)
    - [Nginx 反向代理 Apache 项目（Nginx 与 Apache 共存）](#nginx-反向代理-apache-项目nginx-与-apache-共存)
      - [1. 统一工作进程用户](#1-统一工作进程用户)
      - [2. 情况一：项目部署在 Nginx 下](#2-情况一项目部署在-nginx-下)
      - [3. 情况二：项目部署在 Apache 下](#3-情况二项目部署在-apache-下)
  - [注意事项](#注意事项)
    - [1. `nginx.pid` 丢失导致 nginx 启动失败](#1-nginxpid-丢失导致-nginx-启动失败)

## 安装 PHP 7.2

### 1. 安装 epel

```sh
sudo yum install epel-release
```

### 2. 安装 php yum 仓库

```sh
sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

### 3. 安装 php 7.2 及扩展

```sh
sudo yum install php72w
sudo yum install php72w-* --skip-broken
```

### 4. 修改 `php.ini`

```sh
sudo gedit /etc/php.ini
```

修改以下对应内容：

```ini
display_errors = off  # 不显示错误信息(不输出到页面或屏幕上)
log_errors = On  # 记录错误信息(保存到日志文件中)
error_log = "/var/log/php/error_log"  # 填写日志路径
error_reporting = E_ALL&~E_NOTICE
```

### 5. 修改 `www.conf`

```sh
sudo gedit /etc/php-fpm.d/www.conf
```

修改以下对应内容：

```ini
catch_workers_output = yes  # 取消注释
```

### 6. 开启服务

```sh
systemctl start php-fpm
systemctl enable php-fpm  # 开机启动
```

---

## 安装 Nginx 1.14

### 1. 下载安装包并解压

```sh
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar zxvf nginx-1.14.2.tar.gz
cd nginx-1.14.2/
```

### 2. 安装 zlib pcre openssl 库

```sh
sudo yum -y install zlib zlib-devel
sudo yum -y install pcre pcre-devel
sudo yum -y install openssl openssl-devel
```

### 3. 配置、编译并安装

```sh
sudo ./configure --with-http_ssl_module
su    # 用root进行make，sudo仍会出现权限问题
make && make install
```

### 4. 启动

注意：因为是自行编译安装，不能用 systemctl 直接启动服务

```sh
# 如果之前已安装 Apache，先停止 Apache 服务
# systemctl stop httpd
sudo /usr/local/nginx/sbin/nginx
```

浏览器访问 `localhost` 即可看到 nginx 欢迎页面

### 5. 开放端口

```sh
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=9000/tcp --permanent
firewall-cmd --reload
```

### 6. 创建启动脚本 `nginx.service`

> 参考 [centos 7 nginx启动脚本- sunmmi - 博客园](https://www.cnblogs.com/sunmmi/articles/8439166.html)

```sh
sudo gedit /usr/lib/systemd/system/nginx.service
```

插入以下内容并保存

```ini
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

开启服务

```sh
systemctl start nginx
systemctl enable nginx  # 开机启动
```

### 7. 配置 php 支持

- 修改 `nginx.conf`

  ```sh
  sudo gedit /usr/local/nginx/conf/nginx.conf
  ```

  ```nginx
  location / {
      root   html;

      # 添加 index.php
      index  index.html index.htm index.php;
  }
  ...
  # 去掉这部分注释
  location ~ \.php$ {
      root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;

      # 将 /scripts 改成 $document_root
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;

      include        fastcgi_params;
  }
  ```

- 创建 `info.php` 页面并测试

  在目录 `/usr/local/nginx/html/` 下建立一个 PHP 文件 `info.php`，加入以下代码：

  ```php
  <?php phpinfo(); ?>
  ```

- 重启 nginx 服务

  ```sh
  systemctl restart nginx
  ```

- 浏览器浏览：`localhost/info.php`

---

## 安装 Apache 2.4

### 1. 安装并启动 Apache

```sh
sudo yum install httpd httpd-devel

# 如果之前已安装 Nginx，先停止 Nginx 服务
# systemctl stop nginx
systemctl start httpd
systemctl enable httpd

# 开启防火墙必要端口的访问，用于远程访问
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```

### 2. 创建 `index.php` 页面并测试

这里是复制前面所创的页面，新建请参考本文 [7. 配置 php 支持](#7-配置-php-支持)

```sh
sudo cp /usr/local/nginx/html/info.php /var/www/html/index.php
```

浏览器访问 `localhost` 即可看到 php 信息

---

## 多站点部署

### Nginx 多站点部署

> 参考 [Nginx 的多站点配置 - ouyangzhan的专栏 - CSDN博客](https://blog.csdn.net/ouyangzhan/article/details/6015243)

注意：**nginx 的工作进程用户**要和 **php-fpm 的工作进程用户**保持一致，同时该用户**有权限访问项目目录**

#### 1. 修改 php-fpm 配置

- 修改 `/etc/php-fpm.d/www.conf`

  ```ini
  # 设置为启动用户
  user = current_user
  group = current_user
  ```

- 重启服务

  ```sh
  systemctl restart php-fpm
  ```

#### 2. 创建多站点配置文件目录与文件

```sh
# 新建站点配置存放目录
sudo mkdir /usr/local/nginx/conf.vhost.d

# 新建项目配置文件（如：a.conf）
sudo gedit /usr/local/nginx/conf.vhost.d/a.conf
```

在配置文件 `a.conf` 中插入项目配置并保存（多站点方式：**监听不同端口**或**使用不同域名**）

```nginx
server {
    # 监听 80 端口
    listen       80;
    # 域名，如果是任意域名，需要在 /etc/hosts 中映射该域名到 127.0.0.1
    server_name  test.a.com;
    # 指向项目根目录
    root         /path/to/project/a/dir;
    index        index.html index.htm index.php;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO  $fastcgi_path_info;
        fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
        include        fastcgi_params;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

- 映射域名
  - 编辑 `/etc/hosts`

  ```sh
  sudo gedit /etc/hosts
  ```

  - 插入以下内容并保存

  ```ini
  127.0.0.1    test.a.com
  ```

#### 3. 修改 `nginx.conf`

```nginx
# 取消 user 的注释，设置为启动用户
user current_user;
...
http {
    ...

    # 加载所有站点的配置文件
    include /usr/local/nginx/conf.vhost.d/*.conf
}
```

#### 4. 为项目目录赋予权限

```sh
sudo chmod 777 -R /path/to/project/a/dir
```

#### 5. 修改 SELinux 参数

```sh
sudo setsebool -P httpd_can_network_connect_db 1
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_read_user_content 1
restorecon -R -v /var/www
```

注意：彻底关闭 SELinux 会有安全隐患，建议根据错误提示进行操作

#### 6. 重启 nginx 服务

```sh
sudo systemctl restart nginx
```

### Apache 多站点部署

> 参考 [（Ubuntu/Centos）apache多站点配置 - LSGOZJ的博客 - CSDN博客](https://blog.csdn.net/baidu_30000217/article/details/53782479)

注意：**apatch 的工作进程用户**要和 **php-fpm 的工作进程用户**保持一致，同时该用户**有权限访问项目目录**

#### 1. 配置 php-fpm

> 参考本文 [1. 修改 php-fpm 配置](#1-修改-php-fpm-配置)

#### 2. 创建多站点配置

> 参考本文 [2. 创建多站点配置文件目录与文件](#2-创建多站点配置文件目录与文件)

```sh
# 新建站点配置存放目录
sudo mkdir /etc/httpd/conf.vhost.d

# 新建项目配置文件（如：b.conf）
sudo gedit /etc/httpd/conf.vhost.d/b.conf
```

在配置文件 `b.conf` 中插入项目配置并保存（多站点方式：**监听不同端口**或**使用不同域名**）

```apache
# 监听 80 端口
<VirtualHost *:80>
    # 域名，如果是任意域名，需要在 /etc/hosts 中映射该域名到 127.0.0.1
    ServerName test.a.mccn.com
    DocumentRoot /path/to/project/b/dir

    <Directory />
          Options +Includes +FollowSymLinks -Indexes
          AllowOverride none
          Require all granted
          Order Deny,Allow
          Allow from All
    </Directory>
</VirtualHost>
```

#### 3. 修改 `httpd.conf`

```sh
sudo gedit /etc/httpd/conf/httpd.conf
```

```apache
...
# 设置为启动用户
User current_user
Group current_user
...
# 在文档尾部插入
# Load config files in the "/etc/httpd/conf.vhost.d" directory, if any.
IncludeOptional conf.vhost.d/*.conf
```

#### 4. 赋予权限

> 参考本文 [4. 为项目目录赋予权限](#4-为项目目录赋予权限)

#### 5. 修改 SELinux

> 参考本文 [5. 修改 SELinux 参数](#5-修改-selinux-参数)

#### 6. 重启 apache 服务

```sh
sudo systemctl restart httpd
```

### Nginx 反向代理 Apache 项目（Nginx 与 Apache 共存）

> 参考 [Linux服务器下Nginx与Apache共存 - ITYangs的博客 - CSDN博客](https://blog.csdn.net/ITYang_/article/details/53907937)

注意：**同一个端口不能同时有两个程序监听**。将 nginx 作为代理服务器和 web 服务器使用，nginx 监听 80 端口，Apache 监听除 80 以外的端口，这里使用 8080 端口。**因固定端口，多项目用域名区分**。

#### 1. 统一工作进程用户

参考前文修改 `www.conf` `nginx.conf` `httpd.conf` 中的 user 和 group，并重启相应服务。

#### 2. 情况一：项目部署在 Nginx 下

此情况下，Nginx 充当 Web 服务器，参考本文 [Nginx 多站点部署](#nginx-多站点部署) 配置不同域名即可。

#### 3. 情况二：项目部署在 Apache 下

此情况下，Nginx 充当代理服务器，转发到本服务器 8080 端口。

- 修改 `httpd.conf`，监听 8080 端口，并同时启用 Apache 和 Nginx 服务

  ```sh
  sudo gedit /etc/httpd/conf/httpd.conf
  ```

  ```apache
  ...
  # 将 80 改成 8080
  Listen 8080
  ...
  ```

- 参考本文 [Nginx 多站点部署](#nginx-多站点部署)，在 Nginx 下新建项目配置文件，如：`c.apache.conf`

  ```sh
  # 新建项目配置文件
  sudo gedit /usr/local/nginx/conf.vhost.d/c.apache.conf
  ```

  在配置文件 `c.apache.conf` 中插入项目配置并保存（多站点方式：**使用不同域名**）

  ```nginx
  server {
      # 监听 80 端口
      listen       80;
      # 域名，需要在 /etc/hosts 中映射该域名到 127.0.0.1
      server_name  test.c.apache.com;
      # 指向项目根目录
      root         /path/to/project/c/dir;
      index        index.html index.htm index.php;

      # 此处 Nginx 做静态资源服务器，动态 php 请求转发到 Apache。如果 Nginx 只是单纯的做代理服务器，可将 "location ~ \.php$" 换成 "location /"
      location ~ \.php$ {
          proxy_pass              http://127.0.0.1:8080;
            proxy_redirect          off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```

- 参考本文 [Apache 多站点部署](#apache-多站点部署)，在 Apache 下新建对应项目配置文件，如：`c.conf`

  ```sh
  # 新建项目配置文件
  sudo gedit /etc/httpd/conf.vhost.d/c.conf
  ```

  ```apache
  # 监听 8080 端口
  <VirtualHost *:8080>
      # 与刚才设置的域名保持一致
      ServerName test.c.apache.com
      DocumentRoot /path/to/project/c/dir

      <Directory />
            Options +Includes +FollowSymLinks -Indexes
            AllowOverride none
            Require all granted
            Order Deny,Allow
            Allow from All
      </Directory>
  </VirtualHost>
  ```

- 映射相应域名后，重启 Apache 和 Nginx 服务，即可用浏览器访问该域名。

---

## 注意事项

### 1. `nginx.pid` 丢失导致 nginx 启动失败

> 参考
> [CentOS 7 解决丢失nginx.pid - 个人文章- SegmentFault 思否](https://segmentfault.com/a/1190000018281637)
> [nginx重启 failed (98: Address already in use) - zqinghai的专栏 - CSDN博客](https://blog.csdn.net/zqinghai/article/details/73484394)

重新生成该文件即可

```sh
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

如果出现以下错误

```console
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] still could not bind()
```

查找占用 80 端口的进程

```sh
sudo netstat -tunlp | grep 80
```

输出

```console
tcp    0    0 0.0.0.0:80    0.0.0.0:*    LISTEN    20870/nginx: master
```

杀死占用进程（这里以 pid=20870 为例）

```sh
sudo kill 20870
```

再次生成 pid 文件，并重启服务

```shell
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
systemctl stop nginx
systemctl start nginx
```
