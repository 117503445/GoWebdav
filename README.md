# GoWebdav

> A Simple, Powerful WebDAV Server By Golang.

## 特性

- 基于 Golang 实现，性能高。

- 最终编译为单文件，不需要 Apache 等环境，更加稳定。

- 支持浏览器在线访问。

- 可以在同个端口下启用多个 WebDAV 服务，各自有不同的挂载目录、用户名、密码。

- Docker 支持良好。

## 运行方法

```sh
docker rm go_webdav -f
docker rmi 117503445/go_webdav
docker run -it --name go_webdav -d -v /root/dir1:/root/dir1 -v /root/dir2:/root/dir2 -e dav="/dav1,/root/dir1,user1,pass1,true;/dav2,/root/dir2,null,null,false" -p 80:80 --restart=always 117503445/go_webdav:latest
```

其中

```sh
-e dav="/dav1,/root/dir1,user1,pass1,true;/dav2,/root/dir2,null,null,false"
```

表示将配置字符串传入 Docker 镜像。

然后在浏览器中观察 <http://localhost/dav1> 和 <http://localhost/dav2> 可以映射本地文件了。

## 配置字符串说明

使用分号将每个 WebDAV 服务配置分隔，也就是说上述字符串描述了 2 个服务，分别是

> /dav1,/root/dir1,user1,pass1,true

和

> /dav2,/root/dir2,null,null,false

第一个服务会在 /dav1 下挂载 镜像的 /root/dir1 目录，访问需要的用户名和密码分别为 user1 和 pass1 。

再根据前面的  -v /root/dir1:/root/dir1 就可以和物理机的 /root/dir1 完成映射关系，进行访问了。

第 5 个参数 true 表示这是个只读的服务，只支持 GET，不支持 增删改。

第二个服务会在 /dav2 下挂载 镜像的 /root/dir2 目录，访问需要的用户名和密码分别为 null 和 null，这时候表示不需要密码就可以访问这个服务了。

再根据前面的  -v /root/dir2:/root/dir2 就可以和物理机的 /root/dir2 完成映射关系，进行访问了。

第 5 个参数 false 表示这是个非只读的服务，支持 增删改查。

对于无保密性要求的文件分享，建议把账号密码设成 null null，再把 readonly 开成 True。

## 背景介绍

用于搭建基于 WebDAV 的文件共享服务器。

### 使用 WebDAV 的原因

1. Samba 在 Windows 的客户端上使用不方便，难以使用非默认端口。

2. FTP 挂载麻烦。

3. NextCloud 太重了，而且难以实现 共享出来服务器上的文件。

### 再造一个 WebDAV Server 轮子的原因

没看到能满足上述特性的服务端实现。

## 本地调试

把 config.yml.example 重命名为 config.yml， 在 config.yml 文件中配置，再按照常规操作运行

使用了分层构建，在 build 层 通过 go build 构筑了 可执行文件 app，再在 prod 层 进行运行。如果以后需要修改配置文件的结构，也需要修改 Dockerfile。

## 本地 Docker 运行

```sh
docker rm go_webdav -f
docker rmi 117503445/go_webdav

docker build -t 117503445/go_webdav . # 国外
docker build -t 117503445/go_webdav -f Dockerfile_cn . #国内,启用 goproxy.cn 镜像

docker run -it --name go_webdav -d -e dav="/dav1,./TestDir1,user1,pass1;/dav2,./TestDir2,user2,pass2" -p 80:80 --restart=always 117503445/go_webdav:latest
```

## 安全性

使用 HTTP Basic Auth 进行验证，账号密码明文发送，可以说是毫无安全性。如果使用了密码，请务必用 Nginx 套一层 HTTPS 。
