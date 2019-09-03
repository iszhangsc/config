## Nginx
### Nginx 安装
**nginx 安装目录为** `/usr/local/nginx`

[下载](http://nginx.org/en/download.html)

> 在线下载: wget -c http://nginx.org/download/nginx-1.16.1.tar.gz

#### 安装 `Nginx` 之前还需要安装其它依赖:

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

#### 安装`fastDFS`
- 安装`fastDFS`模块, 一共有三个依赖: `fastdfs`、`libfastcommon`、`fastdfs-nginx-module` 
建议拉取GitHub master，避免版本问题。[fastDFS github地址](https://github.com/happyfish100)
如果是从github上clone下来则不需要解压。否则下载zip包 则需要解压 `unzip -o fastdfs-master.zip -d /usr/local`
将下载好的fastDFS依赖放到 `/usr/local/`目录下。

> 安装`libfastcommon`-------->`deepin、centos` 通用
```shell script
iszhangsc@iszhangsc-PC:/usr/local$ cd /usr/local/libfastcommon/
iszhangsc@iszhangsc-PC:/usr/local/libfastcommon$ sudo ./make.sh
iszhangsc@iszhangsc-PC:/usr/local/libfastcommon$ sudo  ./make.sh install
```
> 安装`fastDFS`-------->`deepin、centos` 通用
```shell script
iszhangsc@iszhangsc-PC:/usr/local$ cd /usr/local/fastdfs/
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo  ./make.sh
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo ./make.sh install
```
> 拷贝配置文件-------->`deepin、centos` 通用
```shell script
# 将fastdfs安装目录下的conf下的文件拷贝到/etc/fdfs/下
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ sudo cp -r conf/* /etc/fdfs/
```
> fastDFS可执行命令(自此fastDFS安装完成)
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
##### 配置并启动`trackerd`
> 修改`trackerd.conf`配置文件
```shell script
iszhangsc@iszhangsc-PC:/usr/local/fastdfs$ cd /etc/fdfs/
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo vim tracker.conf
# 将base_path=/home/yuqing/fastdfs改成base_path=/data/fastdfs
```
> 创建`trackerd`数据、日志目录即上一步的Base_path目录
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo mkdir -p /data/fastdfs
```
> 启动 `trackerd`， 启动之后可以查看该应用是否正常运行 `ps -ef|grep trackerd` 或查看端口是否被监听 `sudo lsof -i:22122`,
> 如果失败则查看日志文件 `cat /data/fastdfs/logs/trackerd.log`
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
> 启动`storage`， 启动之后可以查看该应用是否正常运行 `ps -ef|grep trackerd` 或查看端口是否被监听 `sudo lsof -i:22122`,
> 如果失败则查看日志文件 `cat /data/fastdfs/logs/storaged.log`
```shell script
iszhangsc@iszhangsc-PC:/etc/fdfs$ sudo /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```