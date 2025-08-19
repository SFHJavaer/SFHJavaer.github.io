---
title: Linux手动搭建MinIo分布式存储步骤
catalog: true
date: 2022-05-06 02:34:17
subtitle: 
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- MinIO
categories:
- MinIO
---
#### Linux搭建MinIo环境详细步骤

> 1.去minIO官网下载安装包，或者使用wget直接进行下载,选择自己合适的版本

```shell
wget https://dl.min.io/server/minio/release/linux-amd64/minio
```

> 2.给minio文件赋予可执行的权限

```shell
chomd +x minio
```

> 3.启动minio server端（后面的路径指定具体的文件存储目录）

```shell
./minio server /usr/local/minio/dara
```

> 4.在环境变量中指定Minio登陆的用户名和密码

```shell
export MINIO_ACCESS_KEY=myminioadmin 
export MINIO_SECRET_KEY=myminioadmin
```

PS：上述步骤可直接进行环境变量的添加，也可以手动打开profile进行添加

使用 vi /etc/profile在末尾将上述代码添加即可

>  5.后台运行MioIOserver

```shell
#后台运行
nohup ./minio server /usr/software/minio/data  >  /usr/software/minio/minio.log 2>&1 &#
```

> 6.启动后前台提示下面的代码,默认端口在9000

```shell
[root@ww minio]# MINIO_ACCESS_KEY=myminioadmin MINIO_SECRET_KEY=myminioadmin ./minio server --config-dir /usr/software/minio/config /usr/software/minio/data
Attempting encryption of all config, IAM users and policies on MinIO backend
Endpoint:  http://192.168.51.80:9000http://127.0.0.1:9000    
AccessKey: myminioadmin 
SecretKey: myminioadmin 

Browser Access:
   http://192.168.51.80:9000http://127.0.0.1:9000    

Command-line Access: https://docs.min.io/docs/minio-client-quickstart-guide
   $ mc alias set myminio http://192.168.51.80:9000 myminioadmin myminioadmin

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```

> 7.打开访问地址展示下列情况
PS:如果打不开服务器具体地址，说明防火墙没有打开，如果是端口被占用那么启动时就会提示报错，所以需要手动打开防火墙，在文章末尾进行添加步骤


![9000端口](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS82MjYyYmNjNWE5YTE3YzUwNTk0Mjk2MWU5YTQxZjM0ZC94bWxub3RlLzM1Mjc1MEU4QkZFOTQ4MDdBRUU3M0NBNDU4MDE0NUIzLzc4NQ?x-oss-process=image/format,png)

登录之后展示以下界面

![登陆展示视图](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS82MjYyYmNjNWE5YTE3YzUwNTk0Mjk2MWU5YTQxZjM0ZC94bWxub3RlLzAwOTJFNDQ4QTgwRTQ5OEU4M0YyMTNBNzA4QjZBN0E5Lzc5MA?x-oss-process=image/format,png)

进入系统后，我们先要点击右下角的“+”按钮，创建一个文件桶（输入名称后，回车即可），在上传文件到这个文件桶中。Create bucket（创建文件桶）、Upload file（上传文件）。

![上传完成](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS82MjYyYmNjNWE5YTE3YzUwNTk0Mjk2MWU5YTQxZjM0ZC94bWxub3RlLzVDNzkxQTFDOUU4MjQ3MDlCQzg4NTA2QkRBNkJEODc4LzgwMQ?x-oss-process=image/format,png)

###### 在后端进行Spring和MInio的整合，将文件进行上传，会发现图片已经存储到了MInIO的Bucket中，实现了基于MinIO的分布式存储（在后端要指定资源所存储的具体Bucket位置，上传会及自动匹配上传）

> 8.将桶中的图片进行分享，注意要先将桶设置中的分享设置为public，否则为私有资源不能分享，分享的资源可以设置过期时间

![分享时间设置](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS82MjYyYmNjNWE5YTE3YzUwNTk0Mjk2MWU5YTQxZjM0ZC94bWxub3RlLzhFMjUxNTMwQTMwMjRCNjU4RjNBQUVCQzY3OTA5QUI0LzgxOQ?x-oss-process=image/format,png)

将分享链接在后端进行提取，进行业务层的运转就可以了（可以当作简单的图床）

PS：

```txt
我们的minio之前是通过命令行直接启动的，并没有指定后台运行，而且端口跟AccessKey和SecretKey都还是使用默认的，这里需要在启动的时候指定一下。
#指定环境变量
export MINIO_ACCESS_KEY=username
export MINIO_SECRET_KEY=password
#脚本中添加后台运行执行后
nohup sudo /usr/local/minio/minio server --address=0.0.0.0:9000 --config-dir /etc/minio /data/minioData > /usr/local/minio/minio.log 2>&1&
可以将上面的命令放在一个自己新建的.sh可执行文件中，记得给当前用户加上执行权限

chmod u+x *.sh
每一次启动MioIO直接启动sh脚本即可
这样我们的minio服务端基本的配置算是ok了。
```

##### 开启防火墙端口步骤;

```txt
#1、编辑ssh.xml
vim /usr/lib/firewalld/services/ssh.xml

#2、开放43855、9090端口
<port protocol="tcp" port="43855"/>
<port protocol="tcp" port="9090"/>

#3、重启防火墙 
systemctl restart firewalld

```

