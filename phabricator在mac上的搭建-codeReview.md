#facebook pharbricator mac 安装教程
## 介绍
    * pharbricator是一套开源代码审核平台
    * 官网：https://www.phacility.com/phabricator/
    * 安装说明：phabricator主要是由php写的，而且是以website方式运行的，所以mac上要先安装好
    php + nginx(或apache) + mysql(很多配置会保存在数据库里)

## 安装教程

### 前提：
    * 在mac下安装，可以使用Homebrew进行安装软件，会使得安装软件变得更加方便。
    * 每次使用都必须要运行：mysql, nginx, php-fpm 这三个环境    

### 从Github上clone项目
    1. 在本机新建一个存放项目组件的目录，并进入，
    2. 使用以下命令下载项目
    
       $ git clone https://github.com/facebook/libphutil.git
       $ git clone https://github.com/facebook/arcanist.git
       $ git clone https://github.com/facebook/phabricator.git
    
    PS: 这次使用的目录是： /Users/markmylove-mac/phabricator

### 安装数据库：MySql
    到官网下载安装包进行安装。
    下载地址： https://dev.mysql.com/downloads/mysql/
    配置连接到MySql的信息： 
        $ ./bin/config set mysql.host 127.0.0.1     // 配置MySql host  
        $ ./bin/config set mysql.port 3306          // 配置MySql 端口号  
        $ ./bin/config set mysql.user root          // 配置MySql 用户名  
        $ ./bin/config set mysql.pass admin         // 配置MySql 密码  
        $ ./bin/storage upgrade                     // 存储配置更新

### 安装服务器：nginx
    这次教程使用nginx作为容器，也可以使用其他，如apache
    1. 执行一下命令进行安装： 
        $ brew install nginx
    2. 运行nginx
        $ sudo nginx
        能在浏览器访问localhost:8080，说明安装成功
    3. 配置nginx
        到/usr/local/etc/nginx/nginx.conf （配置文件路径），写入一下代码： 
            `
            server {
                listen 8088;
                server_name phabricator.example.com;
                root        /path/to/phabricator/webroot;
                  location / {
                    index index.php;
                    rewrite ^/(.*)$ /index.php?__path__=/$1 last;
                  }

                  location /index.php {
                    fastcgi_pass   localhost:9000;
                    fastcgi_index   index.php;

                    #required if PHP was built with --enable-force-cgi-redirect
                    fastcgi_param  REDIRECT_STATUS    200;

                    #variables to make the $_SERVER populate in PHP
                    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
                    fastcgi_param  QUERY_STRING       $query_string;
                    fastcgi_param  REQUEST_METHOD     $request_method;
                    fastcgi_param  CONTENT_TYPE       $content_type;
                    fastcgi_param  CONTENT_LENGTH     $content_length;

                    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;

                    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
                    fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

                    fastcgi_param  REMOTE_ADDR        $remote_addr;
                  }
            }
            `

        PS: 
          * listen : 可以注释注释掉，设置端口号。
          * server_name : 用于访问的服务名称，可以用ip和域名，不能用localHost
          * 其他设置看链接 : https://secure.phabricator.com/book/phabricator/article/configuration_guide/ 
    
    4: nginx常用命令: 
        * $ sudo nginx  # 打开
        * $ nginx -s reload|reopen|stop|quit  # 重新加载配置|重启|停止|退出 
        * $ nginx -t  # 测试配置是否有语法错误

### 安装php-fpm
        mac系统自带phtp5， 而php-fpm是集成在php内，所以不用重新安装。但是直接运行php-fpm，会直接报错，
    所以需要更改php-fpm.conf文件。
        报错如下: 
            `
            ERROR: failed to open configuration file '/private/etc/php-fpm.conf': No such file or directory (2)
            ERROR: failed to load configuration file '/private/etc/php-fpm.conf'
            ERROR: FPM initialization failed
            `
     步骤：
        1. 备份php-fpm.conf：
            $ sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
        2. 对php-fpm.conf进行修改
            * 打开php-fpm.conf， 路径：/private/etc/php-fpm.conf
            * 去掉pid的注释，并修改为: pid=/var/run/php-fpm.pid
            * 去掉error_log的注释，并修改为: error_log=/var/log/php-fpm.log
        3. 重新启动php-fpm:
            $ sudo php-fpm

    PS: php-fpm 常用命令:
        * $ sudo php-fpm    # 打开php-fpm

### 进入phabricator
    在浏览器直接输入nginx配置好的域名，直接打开。前面配置没有出错的话，可以进入配置管理员帐号的界面。

### 配置使用
    参考官网进行具体的配置
