## Nginx

[下载](http://nginx.org/en/download.html)

> 在线下载: wget -c http://nginx.org/download/nginx-1.16.1.tar.gz

### 安装 `Nginx` 之前还需要安装其它依赖:

- 安装`Nginx`必要依赖包：gcc gcc-c++ pcre* zlib*
```shell script
#deepin 安装
iszhangsc@iszhangsc-PC:/usr/local$ sudo apt-get -y install build-essential libtool libpcre3-dev zlib1g-dev
# centos 安装
[root@VM_0_4_centos ~]# sudo yum -y install zlib zlib-devel gcc-c++ libtool
```
- 安装`Nginx`可选依赖(https支持): openssl openssl-devel
 ```shell script
# deepin 安装
iszhangsc@iszhangsc-PC:/usr/local$ sudo apt-get install openssl
# centos 安装
[root@VM_0_4_centos ~]# sudo yum -y install openssl openssl-devel
```
- 安装`fastDFS`可选依赖(http 访问文件支持)
> 安装`fastDFS`模块, 一共有三个依赖: `fastdfs`、`libfastcommon`、`fastdfs-nginx-module` 
建议拉取GitHub master，避免版本问题。[fastDFS github地址](https://github.com/happyfish100)
如果是从github上clone下来则不需要解压。否则下载zip包 则需要解压 `unzip -o fastdfs-master.zip -d /usr/local`
将下载好的fastDFS依赖放到 `/usr/local/`目录下。

```shell script
iszhangsc@iszhangsc-PC:/usr/local$ cd /usr/local/libfastcommon/
iszhangsc@iszhangsc-PC:/usr/local/libfastcommon$ sudo ./make.sh
iszhangsc@iszhangsc-PC:/usr/local/libfastcommon$ sudo  ./make.sh install
iszhangsc@iszhangsc-PC:/usr/local$ cd /usr/local/fastdfs/
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo  ./make.sh
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo ./make.sh install
```
> 拷贝配置文件
```shell script
# 将fastdfs安装目录下的conf下的文件拷贝到/etc/fdfs/下
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo cp -r conf/* /etc/fdfs/
```
> 以下是fastDFS可执行命令(自此fastDFS安装完成)
```shell script
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ ll /usr/bin/fdfs*
-rwxr-xr-x 1 root root  355680 9月   3 17:21 /usr/bin/fdfs_appender_test
-rwxr-xr-x 1 root root  355504 9月   3 17:21 /usr/bin/fdfs_appender_test1
-rwxr-xr-x 1 root root  342640 9月   3 17:21 /usr/bin/fdfs_append_file
-rwxr-xr-x 1 root root  342032 9月   3 17:21 /usr/bin/fdfs_crc32
-rwxr-xr-x 1 root root  342728 9月   3 17:21 /usr/bin/fdfs_delete_file
-rwxr-xr-x 1 root root  343568 9月   3 17:21 /usr/bin/fdfs_download_file
-rwxr-xr-x 1 root root  343008 9月   3 17:21 /usr/bin/fdfs_file_info
-rwxr-xr-x 1 root root  360728 9月   3 17:21 /usr/bin/fdfs_monitor
-rwxr-xr-x 1 root root 1234384 9月   3 17:21 /usr/bin/fdfs_storaged
-rwxr-xr-x 1 root root  364208 9月   3 17:21 /usr/bin/fdfs_test
-rwxr-xr-x 1 root root  363704 9月   3 17:21 /usr/bin/fdfs_test1
-rwxr-xr-x 1 root root  501848 9月   3 17:21 /usr/bin/fdfs_trackerd
-rwxr-xr-x 1 root root  343352 9月   3 17:21 /usr/bin/fdfs_upload_appender
-rwxr-xr-x 1 root root  344440 9月   3 17:21 /usr/bin/fdfs_upload_file
```
#### 配置并启动`trackerd`
> 修改`trackerd.conf`配置文件
```shell script
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ cd /etc/fdfs/
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo vim tracker.conf
# 将base_path=/home/yuqing/fastdfs改成base_path=/data/fastdfs
```
> 创建`trackerd`数据、日志目录即上一步的base_path目录
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo mkdir -p /data/fastdfs
```
> 启动 `trackerd`( 启动之后可以查看该应用是否正常运行 `ps -ef|grep trackerd` 或查看端口是否被监听 `sudo lsof -i:22122`,
> 如果失败则查看日志文件 `cat /data/fastdfs/logs/trackerd.log`)
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```
##### 配置并启动`storaged`
> 修改`storage.conf`配置文件
```shell script
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ cd /etc/fdfs/
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo vim storage.conf
# 将base_path=/home/yuqing/fastdfs改成base_path=/data/fastdfs
# 将store_path0=/home/yuqing/fastdfs改为：store_path0=/data/fastdfs/storage
# 将tracker_server=192.168.209.121:22122改为：tracker_server=${ip}:22122，这个ip改成自己的
```
> 创建`storage`数据、日志目录
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ mkdir -p /data/fastdfs/storage
```
> 启动`storage`( 启动之后可以查看该应用是否正常运行 `ps -ef|grep trackerd` 或查看端口是否被监听 `sudo lsof -i:22122`,
> 如果失败则查看日志文件 `cat /data/fastdfs/logs/storaged.log` )
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

#### `fastDFS`和`Nginx`整合
> `fastdfs-nginx-module`安装
```shell script
iszhangsc@iszhangsc-PC:/usr/local$ cd /usr/local/fastdfs-nginx-module/src
# 拷贝配置文件
iszhangsc@iszhangsc-PC:/usr/local/fastdfs-nginx-module/src$ sudo cp mod_fastdfs.conf /etc/fdfs/
iszhangsc@iszhangsc-PC:/usr/local/fastdfs-nginx-module/src$ sudo vim /etc/fdfs/mod_fastdfs.conf
# base_path=/tmp改成：base_path=/data/fastdfs
# tracker_server=tracker:22122改成：tracker_server=192.168.1.207:22122
# url_have_group_name = false改成：url_have_group_name = true #url中包含group名称
# store_path0=/home/yuqing/fastdfs改成：store_path0=/data/fastdfs/storage
```
> 安装nginx和第三方依赖
```shell script
iszhangsc@iszhangsc-PC:/usr/local$ cd nginx-1.16.0/
# 参数说明: --prefix：设置安装路径;
# --with-http_stub_status_module：支持nginx状态查询;
# --with-http_ssl_module：支持https;
# --with-pcre：为了支持rewrite重写功能，必须制定pcre;
# --with-http_gzip_static_module: 开启gizp压缩;
# --add-module=/usr/local/fastdfs-nginx-module/src： 安装fastDFS-nginx模块,支持nginx访问;
iszhangsc@iszhangsc-PC:/usr/local$ sudo ./configure \
--prefix=/usr/local/nginx \
--with-http_ssl_module --with-http_stub_status_module --with-pcre \
--with-http_gzip_static_module \
--add-module=/usr/local/fastdfs-nginx-module/src
iszhangsc@iszhangsc-PC:/usr/local$ sudo make
iszhangsc@iszhangsc-PC:/usr/local$ sudo make install
```
> 检查nginx模块

iszhangsc@iszhangsc-PC:/usr/local$ sudo /usr/local/nginx/sbin/nginx -V
```shell script
nginx version: nginx/1.16.0
built by gcc 7.3.0 (Debian 7.3.0-19) 
built with OpenSSL 1.1.0h  27 Mar 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_ssl_module --with-http_stub_status_module --with-pcre --with-http_gzip_static_module --add-module=/usr/local/fastdfs-nginx-module/src
```
> 修改nginx配置文件
```shell script
http {
  server {
    listen  80;
    server_name localhost;
    # fastDFS http访问配置
    location ~/group([0-9])/M00 {
       # alias /usr/local/fastDFS/storage/data;
       ngx_fastdfs_module;
    }
  }
}
```
> 创建`nginx.service`启动脚本

**iszhangsc@iszhangsc-PC:/usr/local$ sudo vim /lib/systemd/system/nginx.service**
```shell script
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
**启用该脚本**
iszhangsc@iszhangsc-PC:/usr/local$ sudo systemctl enable nginx.service
> 脚本命令:
- 启动: `sudo systemctl start nginx.service`
- 停止: `sudo systemctl stop nginx.service`
- 重启: `sudo systemctl reload nginx.service`

#### Nginx配置访问密码
> 安装`htpasswd`工具
```shell script
# contos 安装
[root@uuu ~]# yum  -y install httpd-tools
# deepin 安装
[iszhangsc@iszhangsc-PC]$ sudo apt-get install apache2-utils
```
> 设置用户名和密码，并把用户名、密码保存到指定文件中
```shell script
iszhangsc@iszhangsc-PC:/usr/local/nginx$ sudo htpasswd -c /usr/local/nginx/passwd code
New password: 
Re-type new password: 
Adding password for user code
```
> 修改nginx配置文件
```shell script
http {
  server {
    listen  80;
    server_name localhost;
    # 根目录配置
    location / {
        root    /www/web;
        index   index.html;
        try_files   $uri $uri/ /index.html;
    }

    # 开启nginx目录浏览功能（有密码限制）详细看 nginx.md
    location /file {
        autoindex   on;
        # 中文，避免乱码
        charset utf8,gbk;
        # 显示文件修改时间为服务器本地时间
        autoindex_localtime on;
        alias   /home/iszhangsc/project;
        auth_basic  "Please input password!";
        # 访问密码文件
        auth_basic_user_file    /usr/local/nginx/passwd;
    }
  }
}
```