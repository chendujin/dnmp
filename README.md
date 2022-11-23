> 使用前最好提前阅读一遍[目录](#目录)，以便快速上手，遇到问题也能及时排除。

## 1.目录结构

```
/
├── data                        数据库数据目录
│   ├── esdata                  ElasticSearch 数据目录
│   ├── mongo                   MongoDB 数据目录
│   ├── mysql8                  MySQL8 数据目录
│   └── mysql5                  MySQL5 数据目录
├── services                    服务构建文件和配置文件目录
│   ├── elasticsearch           ElasticSearch 配置文件目录
│   ├── mysql8                  MySQL8 配置文件目录
│   ├── mysql5                  MySQL5 配置文件目录
│   ├── nginx                   Nginx 配置文件目录
│   ├── php73                   PHP5.6 - PHP7.4 配置目录
│   ├── php54                   PHP5.4 配置目录
│   └── redis                   Redis 配置目录
├── logs                        日志目录
├── docker-compose.yml          Docker 服务配置示例文件
├── env.smaple                  环境配置示例文件
└── www                         PHP 代码目录
```

## 2.快速使用
1. 本地安装
    - `git`
    - `Docker`(系统需为Linux，Windows 10 Build 15063+，或MacOS 10.12+，且必须要`64`位）
    - `docker-compose 1.7.0+`
2. 拷贝并命名配置文件（Windows系统请用`copy`命令），启动：
    ```
    $ cd docker-lnmp-server                             # 进入项目目录
    $ cp env.sample .env                                # 复制环境变量文件
    $ cp docker-compose.sample.yml docker-compose.yml   # 复制 docker-compose 配置文件。
    $ docker-compose up nginx                            # 启动，默认启动个服务:
    ```
3. 在浏览器中访问：`http://localhost`或`https://localhost`(自签名HTTPS演示)就能看到效果，PHP代码在文件`./www/localhost/index.php`。

## 3.PHP和扩展
### 3.1 快速安装php扩展
1.进入容器:

```sh
docker exec -it php73 /bin/bash

install-php-extensions apcu 
```
2.支持快速安装扩展列表

扩展示例[https://github.com/mlocati/docker-php-extension-installer](https://github.com/mlocati/docker-php-extension-installer)
参考示例文件

**容器内使用composer命令**

还有另外一种方式，就是进入容器，再执行`composer`命令，以PHP7容器为例：
```bash
docker exec -it php73 /bin/bash
cd /www/localhost
composer update
```
    
## 4.管理命令
### 4.1 服务器启动和构建命令
如需管理服务，请在命令后面加上服务器名称，例如：
```bash
$ docker-compose up                         # 创建并且启动所有容器
$ docker-compose up -d                      # 创建并且后台运行方式启动所有容器
$ docker-compose up nginx php mysql         # 创建并且启动nginx、php、mysql的多个容器
$ docker-compose up -d nginx php  mysql     # 创建并且已后台运行的方式启动nginx、php、mysql容器


$ docker-compose start php                  # 启动服务
$ docker-compose stop php                   # 停止服务
$ docker-compose restart php                # 重启服务
$ docker-compose build php                  # 构建或者重新构建服务

$ docker-compose rm php                     # 删除并且停止php容器
$ docker-compose down                       # 停止并删除容器，网络，图像和挂载卷
```

### 4.2 添加快捷命令
在开发的时候，我们可能经常使用`docker exec -it`进入到容器中，把常用的做成命令别名是个省事的方法。

首先，在主机中查看可用的容器：
```bash
$ docker ps           # 查看所有运行中的容器
$ docker ps -a        # 所有容器
```
输出的`NAMES`那一列就是容器的名称，如果使用默认配置，那么名称就是`nginx`、`php`、`php56`、`mysql`等。

然后，打开`~/.bashrc`或者`~/.zshrc`文件，加上：
```bash
alias dnginx='docker exec -it nginx /bin/sh'
alias dphp='docker exec -it php /bin/sh'
alias dphp56='docker exec -it php56 /bin/sh'
alias dphp54='docker exec -it php54 /bin/sh'
alias dmysql='docker exec -it mysql /bin/bash'
alias dredis='docker exec -it redis /bin/sh'
```
下次进入容器就非常快捷了，如进入php容器：
```bash
$ dphp
```

### 4.3 查看docker网络
```sh
ifconfig docker0
```
用于填写`extra_hosts`容器访问宿主机的`hosts`地址

## 5.使用Log

Log文件生成的位置依赖于conf下各log配置的值。

### 5.1 Nginx日志
Nginx日志是我们用得最多的日志，所以我们单独放在根目录`log`下。

`log`会目录映射Nginx容器的`/var/log/nginx`目录，所以在Nginx配置文件中，需要输出log的位置，我们需要配置到`/var/log/nginx`目录，如：
```
error_log  /var/log/nginx/nginx.localhost.error.log  warn;
```


### 5.2 PHP-FPM日志
大部分情况下，PHP-FPM的日志都会输出到Nginx的日志中，所以不需要额外配置。

另外，建议直接在PHP中打开错误日志：
```php
error_reporting(E_ALL);
ini_set('error_reporting', 'on');
ini_set('display_errors', 'on');
```

如果确实需要，可按一下步骤开启（在容器中）。

1. 进入容器，创建日志文件并修改权限：
    ```bash
    $ docker exec -it php /bin/sh
    $ mkdir /var/log/php
    $ cd /var/log/php
    $ touch php-fpm.error.log
    $ chmod a+w php-fpm.error.log
    ```
2. 主机上打开并修改PHP-FPM的配置文件`conf/php-fpm.conf`，找到如下一行，删除注释，并改值为：
    ```
    php_admin_value[error_log] = /var/log/php/php-fpm.error.log
    ```
3. 重启PHP-FPM容器。

### 5.3 MySQL日志
因为MySQL容器中的MySQL使用的是`mysql`用户启动，它无法自行在`/var/log`下的增加日志文件。所以，我们把MySQL的日志放在与data一样的目录，即项目的`mysql`目录下，对应容器中的`/var/log/mysql/`目录。
```bash
slow-query-log-file     = /var/log/mysql/mysql.slow.log
log-error               = /var/log/mysql/mysql.error.log
```
以上是mysql.conf中的日志文件的配置。



## 6.数据库管理
本项目默认在`docker-compose.yml`中不开启了用于MySQL在线管理的*phpMyAdmin*，以及用于redis在线管理的*phpRedisAdmin*，可以根据需要修改或删除。

### 6.1 phpMyAdmin
phpMyAdmin容器映射到主机的端口地址是：`8080`，所以主机上访问phpMyAdmin的地址是：
```
http://localhost:8080
```

MySQL连接信息：
- host：(本项目的MySQL容器网络)
- port：`3306`
- username：（手动在phpmyadmin界面输入）
- password：（手动在phpmyadmin界面输入）

### 6.2 phpRedisAdmin
phpRedisAdmin容器映射到主机的端口地址是：`8081`，所以主机上访问phpMyAdmin的地址是：
```
http://localhost:8081
```

Redis连接信息如下：
- host: (本项目的Redis容器网络)
- port: `6379`


## 7.在正式环境中安全使用
要在正式环境中使用，请：
1. 在php.ini中关闭XDebug调试
2. 增强MySQL数据库访问的安全策略
3. 增强redis访问的安全策略


## 8 常见问题
### 8.1 如何在PHP代码中使用curl？
参考这个issue：[https://github.com/yeszao/dnmp/issues/91](https://github.com/yeszao/dnmp/issues/91)

### 8.2 Docker使用cron定时任务 
[Docker使用cron定时任务](https://www.awaimai.com/2615.html)

### 8.3 Docker容器时间
容器时间在.env文件中配置`TZ`变量，所有支持的时区请看[时区列表·维基百科](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)或者[PHP所支持的时区列表·PHP官网](https://www.php.net/manual/zh/timezones.php)。

### 8.4 如何连接MySQL和Redis服务器
这要分两种情况，

第一种情况，在**PHP代码中**。
```php
// 连接MySQL
$dbh = new PDO('mysql:host=mysql;dbname=mysql', 'root', '123456');

// 连接Redis
$redis = new Redis();
$redis->connect('redis', 6379);
```
因为容器与容器是`expose`端口联通的，而且在同一个`networks`下，所以连接的`host`参数直接用容器名称，`port`参数就是容器内部的端口。更多请参考[《docker-compose ports和expose的区别》](https://www.awaimai.com/2138.html)。

第二种情况，**在主机中**通过**命令行**或者**Navicat**等工具连接。主机要连接mysql和redis的话，要求容器必须经过`ports`把端口映射到主机了。以 mysql 为例，`docker-compose.yml`文件中有这样的`ports`配置：`3306:3306`，就是主机的3306和容器的3306端口形成了映射，所以我们可以这样连接：
```bash
$ mysql -h127.0.0.1 -uroot -p123456 -P3306
$ redis-cli -h127.0.0.1
```
这里`host`参数不能用localhost是因为它默认是通过sock文件与mysql通信，而容器与主机文件系统已经隔离，所以需要通过TCP方式连接，所以需要指定IP。

### 8.5 容器内的php如何连接宿主机MySQL
1.宿主机执行`ifconfig docker0`得到`inet`就是要连接的`ip`地址
```sh
$ ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ...
```
2.运行宿主机Mysql命令行
```mysql
 mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
 mysql>flush privileges;
// 其中各字符的含义：
// *.* 对任意数据库任意表有效
// "root" "123456" 是数据库用户名和密码
// '%' 允许访问数据库的IP地址，%意思是任意IP，也可以指定IP
// flush privileges 刷新权限信息
```

3.接着直接php容器使用`172.0.17.1:3306`连接即可

### 8.6 SQLSTATE[HY000] [1130] Host '172.19.0.2' is not allowed to connect to this MySQL server
1. 目前使用mysql-server `8.0.28`以上的版本,php版本需要`7.4.7`以上才能连接
## License
MIT

