<p align="center">
    <h1 align="center">Yii2-xin 开发版本</h1>
</p>

1. >简述

##### 最近学习```Yii2```略有所得,于是想自己搭一个后台管理系统,以后会源源不断的完善.

2. >安装```yii2```环境
```
lnmp 1.4(mysql5.6+php5.6+centos7.2+nginx1.12.1)
composer 1.5.2
git 2.11.0
```
3. >安装```yii2```
#####  直接用```git```拉取本项目,本项目与原生```yii2```相比,仅仅只是多了两个组件:```mdmsoft/yii2-admin```与```dmstr/yii2-adminlte-asset```,前一个是```RBAC```权限接口的组件,后一个是一套漂亮的后台模板,我已经将这两个组件的信息放入了```composer.json```文件中.
##### 接下来用```composer```下载额外组件.由于我自己安装过,所以已经产生了```composer.lock```文件,如果网速好的话可以使用```composer update```命令,但如果网速不好的话,还是使用```composer install```命令吧,不管是```composer update```还是```composer install```,还有一些注意点:
- ```composer```是一个管理依赖关系的工具,并不是包管理器,所以它需要先下载管理```bower```和```npm```,然后用它们去下载前端资源等一系列组件.于是需要先下载```fxp/composer-asset-plugin```去管理```bower```和```npm```.
```
composer global require "fxp/composer-asset-plugin:^1.3.1"
```
- 需要```php opensll```模块支持,```php5.6```是自动支持的,但是```php7```需要编译此模块.
- 如果是用```composer install```命令的话,还需要打开```php```中的```proc_open```,```proc_get_status```函数,这两个函数默认是被禁止使用的.
##### ```composer```成功拉取组件之后,需要初始化```yii```
- 输入```php init```命令生成入口文件等.
- 修改```common/config/main-local.php```中的数据库配置信息.
4. >运行```yii2-xin```
- 配置好```yii2-xin``` ```nginx```运行配置.
```
server {
    charset utf-8;
    client_max_body_size 128M;

    listen 8082; ## listen for ipv4
    #listen [::]:80 default_server ipv6only=on; ## listen for ipv6

    server_name mysite.local;//可以放真实域名上去的
    root        /app/yii2/yii2-xin/backend/web;//项目路径
    index       index.php;//入口文件

    access_log  /home/yii2-xin/nginx/access.log;//web日志
    error_log   /home/yii2-xin/nginx/error.log;//web日志

    location / {
        # Redirect everything that isn't a real file to index.php
        try_files $uri $uri/ /index.php$is_args$args;
    }

    # uncomment to avoid processing of calls to non-existing static files by Yii
    #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
    #    try_files $uri =404;
    #}
    #error_page 404 /404.html;

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9000;//tcp 端口通信
        #fastcgi_pass unix:/var/run/php5-fpm.sock;
        try_files $uri =404;
    }

   location ~* /\. {
        deny all;
    }
}
```
- 由于```lnmp```中```php-fpm```的进程有两种方式,而```nginx```配置中采用的是```tcp```端口通信,所以我们需要修改```php-fpm.conf```的侦听配置.
```
#listen = /tmp/php-fpm.sock
listen = 127.0.0.1:9000//侦听9000端口,只改此项
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = dynamic
pm.max_children = 20
pm.start_servers = 10
pm.min_spare_servers = 10
pm.max_spare_servers = 20
request_terminate_timeout = 100
request_slowlog_timeout = 0
slowlog = var/log/slow.log
```
- ```lnmp restart```
- 查看开放端口,有```9000```端口即配置成功,可以按照地址运行.
```
netstat -ntlp
```
##### 注意:如果还是访问不了,用的服务器是阿里云```ECS```,请检查安全组规则,是否开放了对应端口.
5. >配置```adminlte```后台模板
- 想要有```adminlte```风格的模板很好办,将```vendor/dmstr/yii2-adminlte-asset/example-views/yii2-app/layouts```下的所有文件全都复制到```yii2-xin/backend/layouts```目录下,如果想要有一个漂亮的登录页面,也可以把同目录的```site```目录也复制一下.
- 如果没有配置用户的话,请注释```siteController```下行为的```access```属性.
```
    public function behaviors()
    {
        return [
            'access' => [//注释掉access属性
                'class' => AccessControl::className(),
                'rules' => [
                    [
                        'actions' => ['login', 'error'],
                        'allow' => true,
                    ],
                    [
                        'actions' => ['logout', 'index'],
                        'allow' => true,
                        'roles' => ['@'],
                    ],
                ],
            ],
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'logout' => ['post'],
                ],
            ],
        ];
    }
```
- 运行

---
![Fw9AO.png](https://s1.ax2x.com/2017/11/22/Fw9AO.png)
---
- 当然这只是静态模板,还没有具体与业务结合起来



