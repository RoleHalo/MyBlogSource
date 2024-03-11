---

title: Docker学习笔记
date: 2023-10-14 09:14:38
tags: Docker
toc: true
---

- [Docker安装(Ubuntu环境)](#docker安装ubuntu环境)
- [Docker命令](#docker命令)
  - [帮助命令](#帮助命令)
  - [镜像命令](#镜像命令)
  - [容器命令](#容器命令)
  - [常用其他操作命令](#常用其他操作命令)
  - [可视化](#可视化)
    - [portainer](#portainer)
- [Docker镜像](#docker镜像)
  - [镜像](#镜像)
  - [Docker镜像加载原理](#docker镜像加载原理)
  - [Commmit 镜像](#commmit-镜像)
- [容器数据卷](#容器数据卷)
  - [使用容器数据卷](#使用容器数据卷)
  - [实战：安装MySQL](#实战安装mysql)
  - [具名挂载和匿名挂载](#具名挂载和匿名挂载)
  - [初识DockerFile](#初识dockerfile)
  - [数据卷容器](#数据卷容器)
- [DockerFile](#dockerfile)
- [**Docker**网络原理](#docker网络原理)
- [IDEA整合Docker](#idea整合docker)
- [集群](#集群)
  - [Docker Compose](#docker-compose)
  - [Docker Swarm](#docker-swarm)
  - [CI/CD Jenkins](#cicd-jenkins)

<!-- more -->

# Docker安装(Ubuntu环境)

参考：https://blog.csdn.net/Tester_muller/article/details/131440306

1. ubuntu下自带了Docker的库，不需要添加新的源。但是版本过低，需要使用`root`权限，卸载Ubuntu自带的低版本Docker（若有）:`sudo apt-get remove docker docker-engine docker.io containerd runc`

2. 更新软件：`sudo apt update`;`sudo apt upgrade`

3. 安装Docker依赖：`apt-get install ca-certificates curl gnupg lsb-release`

4. 添加Docker官方GPG密钥：`curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`

5. 添加Docker软件源（如：阿里云）：`sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"`

6. 安装Docker：`apt-get install docker-ce docker-ce-cli containerd.io`

7. 配置用户组（可选）：默认情况下，只有root用户和Docker组的用户才能运行Docker命令。我们可以将当前用户添加到Docker组，以避免每次使用Docker时都需要使用sudo.  `sudo usermod -aG docker $USER`

8. 运行Docker：`systemctl start docker`

9. 安装工具：`apt-get -y install apt-transport-https ca-certificates curl software-properties-common`

10. 重启Docker：`service docker restart`

11. 验证是否安装成功`sudo docker run hello-world`

12. 查看Docker版本：`docker version`

13. 查看已安装镜像：`docker images`

14. Docker执行流程图

    > ![Docker执行流程图.png](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker执行流程图.png)

​																							**Docker执行流程图**

# Docker命令

> ![Docker命令结构图](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker命令结构图.png)

​																								**Docker命令结构图**

## 帮助命令

```shell
docker version            # 查看docker版本
docker info		            # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help        # 帮助命令
```

帮助文档的地址：`https://docs.docker.com/engine/reference/commandline/`

## 镜像命令

1. `docker images`：查看所有本地已安装的镜像

   ```shell
   ubuntu@VM-8-12-ubuntu:~$ docker images
   REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
   hello-world   latest    9c7a54a9a43c   6 months ago   13.3kB
   
   # 命令解释
   REPOSITORY	镜像的仓库源
   TAG					镜像的标签
   IMAGE ID		镜像的id
   CREATED			镜像的创建时间
   SIZE				镜像的大小
   
   # 可选项
   --all, -a			# 列出所有镜像
   --quiet, -q		# 只显示镜像的id

2. `docker search 镜像名`：搜索镜像

   ```shell
   ubuntu@VM-8-12-ubuntu:~$ docker search mysql
   NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
   mysql                           MySQL is a widely used, open-source relation…   14567     [OK]       
   mariadb                         MariaDB Server is a high performing open sou…   5558      [OK]  
   
   # 可选项,通过搜索来过滤
   --filter, -f		Filter output based on conditions provided
   --format			Pretty-print search using a Go template
   --limit			Max number of search results
   --no-trunc			Don't truncate output
   
   --filter=STARS=3000  #搜索出来的镜像就是stars大于3000的
   ubuntu@VM-8-12-ubuntu:~$ docker search mysql --filter=STARS=3000
   NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
   mysql     MySQL is a widely used, open-source relation…   14567     [OK]       
   mariadb   MariaDB Server is a high performing open sou…   5558      [OK]       
   ```

   等同于在`dockerHub`中搜索镜像

3.  `docker pull` 下载镜像

   ```shell
   docker pull [OPTIONS] NAME[:TAG|@DIGEST]
   
   # 可选项
   --all-tags, -a		Download all tagged images in the repository
   --disable-content-trust		true	Skip image verification
   --platform			API 1.32+ Set platform if server is multi-platform capable
   --quiet, -q		Suppress verbose output
   ```

   ```shell
   ubuntu@VM-8-12-ubuntu:~$ docker pull mysql
   Using default tag: latest			# 不加参数默认下载最新版镜像
   latest: Pulling from library/mysql
   8e0176adc18c: Pull complete   # 分层下载，docker images的核心  联合文件系统
   2d2c52718f65: Pull complete 
   d88d03ce139b: Pull complete 
   4a7d7f11aa1e: Pull complete 
   ce5949193e4c: Pull complete 
   f7f024dfb329: Pull complete 
   5fc3c840facc: Pull complete 
   509068e49488: Pull complete 
   cbc847bab598: Pull complete 
   942bef62a146: Pull complete 
   Digest: sha256:1773f3c7aa9522f0014d0ad2bbdaf597ea3b1643c64c8ccc2123c64afd8b82b1 		# 签名
   Status: Downloaded newer image for mysql:latest
   docker.io/library/mysql:latest		#真实地址
   # 所以: docker pull mysql  <==> docker pull docker.io/library/mysql:latest
   ```

4. `docker rmi `  删除镜像

   ```shell
   docker rmi -f 容器id #删除指定的镜像
   docker rmi -f 容器id 容器id 容器id  # 删除多个镜像
   docker rmi -f $(docker images -aq)   #删除全部的镜像
   ```

## 容器命令

注：先有镜像才能创建容器，可以在Linux上安装Docker。然后再在Docker上安装一个Centos镜像进行学习。（等同于在docker上装一个Centos VM）

```shell
docker pull centos
```

创建容器并启动

```shell
 docker run [OPTIONS] IMAGE
 
 # 参数说明
   --name="name"    容器名字  tomcat01，tomcat02 用来区分容器
   -d 							后台方式运行
   -it							使用交互方式运行，进入容器查看内容
   -p								指定容器的端口 -p  8080:8080
      -p ip:主机端口:容器端口
      -p 主机端口:容器端口 （常用）
      -p 容器端口
      容器端口
   -P								随机指定端口
----------------------------------------------------------- 
 # 通过前台启动容器并进入容器（前台进程）
docker run -it centos /bin/bash

# 查看容器内容
ls

# 停止容器并退出
exit
```

查看所有运行的容器

```shell
docker ps 		 # 查看当前运行的容器
			 		-a   # 查看当前及以往运行的容器
			 		-n=? # 显示最近运行的？（几）个容器
			 		-q	 # 只显示容器的编号
```

退出容器

```shell
exit					# 停止容器并退出
Ctrl + P + Q	# 退出不停止容器
```

删除容器

```shell
docker rm 容器id   						     # 删除指定容器（未运行的容器）。需要强制删除: rm -f
docker rm -f $(docker ps -aq)      # 删除所有的容器
docker ps -a -q|xargs docker rm -f # 删除所有的容器
```

启动和停止容器

```shell
docker start 容器id						# 启动容器
docker restart 容器id					# 重启容器
docker stop 容器id						# 停止当前正在运行的容器
docker kill 容器id						# 强制停止当前容器
```

## 常用其他操作命令

1. 通过**后台**启动容器：`docker run -d 容器名`

   - Docker**常见的坑**之一：Docker容器没有运行任何应用程序会启动后立即自动kill后exited。
     - 如：`docker run -d centos`就无法正常从后台运行cenos，而`docker run -it centos /bin/bash`则可成功通过前台启动容器并进入容器。
     - **原因**：当你启动一个Docker容器时，你需要指定一个要运行的应用程序。如果你没有指定任何应用程序，容器将在启动后立即退出。

2. 查看容器的日志：`docker logs 容器id`（需要保证容器正在运行）

   - -t ：显示日志时间戳
   - -f ：显示所有日志
   - --tail  number：要显示的日志条数
   - 如：`docker logs -tf --tail 10 f83add00ab91`，即为查看id为`f83add00ab91`的容器最后的10条日志带时间戳字符串形式的详细信息
   - 没有日志信息可以通过脚本创造一些日志：`docker run -d centos /bin/sh -c "while true;do echo docker66666;sleep 1;done"`

3. 查看容器的**进程信息**：`docker top 容器id`

4. 查看镜像的**元数据**：`docker inspect 容器id`

5. 进入**当前正在运行**的容器

   ```shell
    # 通常容器都是使用后台方式进行的，需要进入容器，修改一些配置
    
    # 方式一
    docker exec -it 容器id bashShell
    # 测试
    ubuntu@VM-8-12-ubuntu:~$ docker ps
   CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
   bf64c038f8b1   centos    "/bin/sh -c 'while t…"   24 seconds ago   Up 23 seconds             upbeat_goldwasser
   ubuntu@VM-8-12-ubuntu:~$ docker exec -it bf64c038f8b1 /bin/bash
   [root@bf64c038f8b1 /]# ls
   bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   [root@bf64c038f8b1 /]# ps -ef
   UID          PID    PPID  C STIME TTY          TIME CMD
   root           1       0  0 16:16 ?        00:00:00 /bin/sh -c while true;do echo docker66666;sleep 1;done
   root         100       0  0 16:18 pts/0    00:00:00 /bin/bash
   root         250       1  0 16:20 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
   root         251     100  0 16:20 pts/0    00:00:00 ps -ef
   [root@bf64c038f8b1 /]# 
   
   # 方式二
   docker attach 容器id
   # 测试
   ubuntu@VM-8-12-ubuntu:~$ docker attach  bf64c038f8b1
   正在执行的当前代码.....
   
   
   
   # 区别
   # docker exec    # 进入容器后开启一个新的终端，可以在里边此操作（常用）
   # docker attach  # 进入容器正在执行的终端，不会启动新的进程
   ```

6. 从容器内拷贝文件到主机上

   ```shell
   # 命令
   docker cp 容器id:容器内路径 目的的主机路径
   #测试
   dockers attach bf64c038f8b1 # 进入正在执行的容器
   cd /home							# cd到容器的home下
   ls										# 查看容器home下的文件信息
   touch test.java				# 创建test.java文件
   exit									# 停止并退出容器，回到本机
   docker ps -a   				# 查看容器信息。无需容器运行，只有存在就可以进行拷贝
   docker cp bf64c038f8b1:/home/test.java /home # 将id为bf64c038f8b1的容器/home目录下的test.java文件拷贝到主机的/home目录下
   ls   									# 查看信息是否拷贝成功
   
   
   # 拷贝是一个手动过程，使用 -v 卷的技术，可以实现自动同步
   ```

1. 注：`sudo`是获取`root`权限的命令
2. 查看Docker工作目录(默认)：`ls /var/lib/docker/`
3. 删除Docker资源：`rm -rf /var/lib/docker/`
4. 停止Docker服务：`sudo systemctl stop docker`
5. 卸载Docker软件包：`sudo apt-get purge docker-ce docker-ce-cli containerd.io`
6. 删除Docker配置和数据：`sudo rm -rf /etc/docker sudo rm -rf /var/lib/docker sudo rm -rf /var/lib/containerd`
7. 删除Docker用户组：`sudo groupdel docker`
8. 删除Docker存储库：`sudo rm -f /etc/apt/sources.list.d/docker.list`
9. 更新APT缓存：`sudo apt-get update`
10. 清除Docker无用的依赖项：`sudo apt-get autoremove`

> 测试安装Nginx

```shell
# 安装nginx版本为1.24.0的容器
docker pull nginx:1.24.0

# -d  								 # 后台运行
# --name 							 # 命名运行时的名字
# -p 宿主机端口:容器端口 # 在宿主机指定端口下运行容器的指定端口      
# 则如下命令为：以后台形式运行nginx:1.24.0容器   且是在宿主机的3344端口运行容器的80端口  并命名为nginx-1.24.0
#如果最后直接写nginx 则是运行nginx的最新版本。若本机没有，则会自动下载再运行
docker run -d --name nginx-1.24.0 -p 3344:80 nginx:1.24.0

# curl 查看端口运行信息
ubuntu@VM-8-12-ubuntu:~$ curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
....
# 可以看到Welcome to nginx!等页面信息表示端口当前运行的为nginx，即nginx运行成功

# 进入容器
# docker /bin/bash:可以帮助开发者进入一个Docker容器的命令行界面，以便进行调试、查看日志、执行命令等操作。当需要进行容器内部的操作时，使用/bin/bash命令可以启动一个交互式的终端会话，使得开发者可以像在本地终端一样操作容器。
docker exec -it nginx-1.24.0 /bin/bash

# 运行进入后可查看Nginx配置文件位置
root@831f7f5b5bf1:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx  # /etc/nginx：配置文件目录
                                                                   # /usr/share/nginx：网页发布目录
root@831f7f5b5bf1:/# ls
bin   docker-entrypoint.d   home   lib64   mnt	 root  srv  usr
boot  docker-entrypoint.sh  lib    libx32  opt	 run   sys  var
dev   etc		    lib32  media   proc  sbin  tmp
root@831f7f5b5bf1:/# cd /etc/nginx
root@831f7f5b5bf1:/etc/nginx# ls
conf.d		mime.types  nginx.conf	 uwsgi_params
fastcgi_params	modules     scgi_params
# 所以我们已经可以查到配置文件，进而实现进入容器内部去修改配置文件
# 可以看出，这种方法会比较麻烦，那么我们有没有一种方式，使得可以在容器外部提供一个映射路径，达到在容器外修改文件名，容器内部自动更新修改？-v 数据卷！
```

> ![Docker端口暴露结构图](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker端口暴露结构图.png)

​																						**Docker端口暴露结构图**



> 测试安装tomcat

```shell
# 官方使用 docker run 实现下载后直接运行
docker run -it --rm tomcat:8.5
# 一般启动都是后台，停止了容器之后，容器还是可以查到  docker run -it --rm，一般用来测试，用完即删


# 先下载再启动
docker pull tomcat:8.5
# 公网测试访问结果： 
HTTP状态 404 - 未找到
类型 状态报告
描述 源服务器未能找到目标资源的表示或者是不愿公开一个已经存在的资源表示。
Apache Tomcat/8.5.95
# 以上结果虽404但也说明部署成功
# why？？↓
# 进入容器
ubuntu@VM-8-12-ubuntu:~$ docker exec -it tomcat01 /bin/bash
root@5946fce6c6f4:/usr/local/tomcat# ls
bin              lib             NOTICE         temp
BUILDING.txt     LICENSE         README.md      webapps
conf             logs            RELEASE-NOTES  webapps.dist
CONTRIBUTING.md  native-jni-lib  RUNNING.txt    work
root@5946fce6c6f4:/usr/local/tomcat# cd webapps
root@5946fce6c6f4:/usr/local/tomcat/webapps# ls
root@5946fce6c6f4:/usr/local/tomcat/webapps# 
# 由以上可以得出404的原因可能由：
1. linux命令少了（未指定版本，默认下载）
2. 没有webapps
3. 阿里云镜像默认下载是最小的镜像，所有不必要的都剔除掉（未指定版本，默认下载）
4. 只保证最小的运行环境
# 解决方案（修改webapps.dist为webapps 或者 拷贝webapps.dist/* 到webapps下）
root@5946fce6c6f4:/usr/local/tomcat# clear
root@5946fce6c6f4:/usr/local/tomcat# ls
bin              lib             NOTICE         temp
BUILDING.txt     LICENSE         README.md      webapps
conf             logs            RELEASE-NOTES  webapps.dist
CONTRIBUTING.md  native-jni-lib  RUNNING.txt    work
root@5946fce6c6f4:/usr/local/tomcat# cd webapps.dist/
root@5946fce6c6f4:/usr/local/tomcat/webapps.dist# ls
docs  examples  host-manager  manager  ROOT
root@5946fce6c6f4:/usr/local/tomcat/webapps.dist# cd..
bash: cd..: command not found
root@5946fce6c6f4:/usr/local/tomcat/webapps.dist# cd ..
root@5946fce6c6f4:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@5946fce6c6f4:/usr/local/tomcat# cd webapps
root@5946fce6c6f4:/usr/local/tomcat/webapps# ls
docs  examples  host-manager  manager  ROOT
root@5946fce6c6f4:/usr/local/tomcat/webapps# 
# 思考：如果我们每次部署项目，都要进入容器内部，就会显得十分麻烦。那么我们有没有一种方式，使得可以在容器外部提供一个映射路径，达到在容器外放置项目，容器内部自动同步？
```



>部署 ES + kibana

```shell
# es暴露的接口过多
# es需要安全挂载？？
# --net somenetwork ？？？ docker的网络


# 运行命令
docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=string-node" -e ES_JAVA_OPTS="-Xmx512m" elasticsearch[:version] # ES_JAVA_OPTS="-Xmx512m"：限制运行内存

# elasticsearch下载内存巨大 所以需要再下载的时候加内存限制
docker stats
```





## 可视化

- portainer（暂时使用）
- Rancher（CI/CD可用）

### portainer

Docker的可视化工具

```shell
# 启动并下载最新版portainer/portainer镜像
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name portainer portainer/portainer

# 查看正在运行的容器
docker ps

# 查看日志
docker logs -f portainer

# 本地运行 查看是否成功
curl localhost:8088
```

一般docker不会选择使用可视化工具，自行测试一下学学使用即可。



# Docker镜像

## 镜像

独立的软件包，用来打包软件运行环境及运行环境开发软件。

包括某个软件所需要的所有内容，包括代码、库、环境变量和配置文件。

## Docker镜像加载原理

> UnionFS(联合文件系统)

1. UnionFS(联合文件系统)：**分层**、轻量级并且高性能的文件系统。支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同的目录挂载到通用一个虚拟文件系统下。

2. Union文件系统是Docker镜像的基础。
3. 镜像可以通过分层进行继承，**基于基础镜像（没有父镜像），可以制作各种具体的应用镜像**。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会**把各个文件系统叠加起来**，这样最终的文件系统会包含所有底层的文件和目录。

> Docker镜像加载原理

bootfs：主要包括bootloader和kernel。bootloader主要是引导加载kernel，Linux行启动时就会加载bootfs文件系统，在Docker镜像的最底层是bootfs。当boot加载完成后整个内核就都在内存了，此时内存使用权已由bootfs转交给系统，此时系统也会卸载bootfs。

rootfs： 在bootfs之上，包括的就是典型如Linux系统中的/dev，/proc,/bin,/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu、CentOS等。

`Docker中的一个容器就是一个小的Linux虚拟机环境。`

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了。因为底层直接用Host的kernel，自己只需要提供rootfs就可以了。由此可见，对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版本可以用同一个bootfs。

**虚拟机是分钟级，容器是秒级！**



> ![对Docker镜像分层的理解示意图](https://gitee.com/RoleHalo/blog-image/raw/master/image/对Docker镜像分层的理解示意图.png)

​																					**Docker镜像分层的理解示意图**





> ![Docker镜像下载示意图](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker镜像下载示意图.png)

​																							**Docker镜像下载示意图**

> 特点

有些应用的某些层是相同的，则只需要下载不存在的层。

Docker镜像都是只读的，当容器启动时，一个新的*可写层*被加载到镜像的顶部。这个可写成通常被称为**容器层**。

也就是说，**容器之下都叫镜像层。**



而我们操作过后需要提交的是将容器层和原有镜像层一起打包起来的镜像层。



## Commmit 镜像

```shell
docker commit # 提交容器成为一个副本
docker commit -m="提交内容的标签信息" -a="作者" 容器id 目标镜像名:[TAG]
```



> 测试

```shell
# 启动一个默认的tomcat

# 发行tomcat下的webapps是空的，镜像的原因，官方默认webapps下是没有文件的

# 默认的webapps.list放置了基本文件。所有我们要运行有页面需要将webapps.list下的所有文件拷贝到webapps下

# 完成以上操作后commit命令打包获得一个新镜像
```

![Docker提交镜像](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker提交镜像.png)



# 容器数据卷

如果数据都在容器中，那么删除容器数据就会丢失！**需求：数据可以持久化**

---> 容器之间可以有一个**数据共享**的技术！Dokcer容器中产生的数据，**同步到本地**！

===》**卷**技术。实现**目录的挂载**，Docker中的容器数据映射到Linux主机的某个目录下进而持久化。

![Docker卷挂载机制](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker卷挂载机制.png)

​																							**Docker卷挂载机制**

**也就是说：卷技术就是容器的持久化和同步操作；容器间也是可以数据共享的。**



## 使用容器数据卷

> 方式一：直接使用命令 -v

```shell
docker run -it -v 本机目录:容器目录  容器id(或容器名)  [/bin/bash]

# 测试
# 挂载 centos镜像下 运行name为centos 的容器
docker run -it -v  /home/ceshi:/home --name centos /bin/bash
# 查看 容器centos的 运行实例id为cfa751800ae1 的信息  
# 查找Mounts信息
docker inspect cfa751800ae1
```

![Docker挂载成功后查看信息](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker挂载成功后查看信息.png)

测试文件同步

![Docker卷技术测试](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker卷技术测试.png)

> 注意：挂载启动命令

```shell
docker run -p 1000:2000 -t -i -v /src:/dst --privileged=true --name 容器名称  image_name:tag /bin/bash
注：-t -i 可以写成-it 
```

- 对该命令字段的理解：

```
docker run：启动container
ubuntu：你想要启动的image
--name="name"   # 名字
-t：进入终端
-w, --workdir string       指定工作目录
-i：获得一个交互式的连接，通过获取container的输入
--rm: 运行完成即删除 # 一般不和 -it 一块运行
-d:后台运行docker   // 与-it相反
-p: 指定端口(可以多次-p参数) 本机的1000端口映射到容器的2000端口
-P: 大写的P，随机指定端口
--net: --net=host不指定端口，与宿主机共用一套IP和端口
-v: 挂载卷，挂载宿主机的src/文件到容器的dst/目录
--privileged: 使用该参数，container内的root拥有真正的root权限。
            否则，container内的root只是外部的一个普通用户权限。
/bin/bash：在container中启动一个bash shell
```

- 测试卷技术的数据是双向的

![Docker挂载测试](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker挂载测试.png)



## 实战：安装MySQL

解决MySQL数据持久化问题！

```shell
# 获取镜像
docker pull mysql:5.7

# 运行容器 需要做数据挂载！
# 安装启动MySQL 需要配置密码！！！
# 官方测试命令  docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动本地主机的MySQL 密码123456 端口映射为3308
docker run -d -p 3308:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
# 运行后 使用本地电脑连接远程运行的mysql 连接成功则为部署成功
```

![Docker下MySQL运行后本地测试连接成功](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下MySQL运行后本地测试连接成功.png)

​                                                                                      **MySQL运行后本地测试连接成功**

- 查看挂载文件：

![Docker下MySQL挂载成功](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下MySQL挂载成功.png)

​                                                                                              **MySQL挂载成功**

- 本地创建数据库，查看数据是否成功同步

![Docker下测试MySQL数据持久化](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下测试MySQL数据持久化.png)

​                                                                                   	**测试MySQL数据持久化**



![Docker下测试容器删除后数据是否存在](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下测试容器删除后数据是否存在.png)

​																						**测试容器删除后数据仍存在**

至此，实现了MySQL数据持久化功能！



## 具名挂载和匿名挂载

```shell
# 匿名挂载

# -P:随机映射端口
# 使用-v 但不指定本地主机的映射路径  ==> 只指定了容器内的路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看数据卷信息
docker volumn ls

# 发现数据卷信息为 以下形式
ubuntu@VM-8-12-ubuntu:~$ docker volume ls
DRIVER    VOLUME NAME
local     5f4aa55e7d5d28d66fe6ebbe434c61332c01026e231048e58f8512d50a0df309

# 则此形式为 匿名挂载

```



```shell
# 具名挂载
# -v 卷名:容器内路径
# 即使用的相对路径juming-nginx 而不是/juming-nginx（从根目录开始的绝对路径）进行挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx

# 查看数据卷信息
docker volumn ls
# 信息看到有卷名
ubuntu@VM-8-12-ubuntu:~$ docker volume ls
DRIVER    VOLUME NAME
local     juming-nginx

# 查看卷名的具体位置
docker volume inspect juming-nginx
```

![Docker下具名挂载](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下具名挂载.png)  

​																		     **查看具名挂载信息**

所有的docker容器内的卷，没有指定目录，都会存在`/var/lib/docker/volumes/`下。

> 拓展一个小问题：
>
> ![Docker下查看volume出现的权限问题](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下查看volume出现的权限问题.png)
>
> ​																		**Docker下查看volume出现的权限问题**
>
> > 起初：我想要cd到/var/lib/docker，但是出现一个权限不够的错误，然后想到使用sudo cd /etc/docker时，又提示说sudo: cd：找不到命令。 
>
> 所以：该问题主要是不具root权限 且 cd 无法直接用sudo运行导致
>
> - 先来解决方案：
>
>   1. 使用`sudo -i`提高用户权限
>   2. 使用`sudo -s` 打开特殊的shell
>
>   以上命令都可以使用exit退出，也可以使用快捷键Ctrl+D退出
>
> - 报错原因分析：
>
>   - shell
>     shell是一个命令解析器所谓shell是一个交互式的应用程序。shell执行外部命令的 时候，是通过fork/exec叉一个子进程，然后执行这个程序。
>   - sudo
>     sudo 是一种程序，用于提升用户的权限，在linux中输入sudo就是调用sudo这个程序提升权限
>     sudo的意思是，以别人的权限叉起一个进程，并运行程序。
>   - cd
>     cd是shell的内部命令。
>     也就是说，是直接由shell运行的，不叉子进程。
>     你在当前进程里当然不能提升进程的权限（其实也可以，不过得编程的时候写到代码里，然后再编译，而我们的 shell没有这个功能，否则岂不是太危险了？黑客.sh
>   - cd不是一个应用程序而是Linux内建的命令，而sudo仅仅只对应用程序起作用。sudo foo只意味着以root权限运行foo程序
>
>   所以，sudo cd /etc/docker会报sudo: cd：找不到命令。

![Docker下查看所有卷信息](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下查看所有卷信息.png)																**Docker下查看所有卷信息**



- 所有的docker容器内的卷，没有指定目录的情况下，都是在`/var/lib/docker/volumes/xxxx/_data`

![Docker下找到配置文件信息](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下找到配置文件信息.png)

​																			**Docker下找到配置文件信息**

- **具名挂载**可以方便我们找到所需要的卷，这也是我们大多数情况下使用的挂载方式

> 总结：如何确定是哪种挂载形式？
>
> 1. `-v 容器内路径 `：匿名挂载
> 2. `-v 卷名:容器内路径`：具名挂载
> 3. `-v /宿主机路径:容器内路径`：指定路径挂载

拓展：

```shell
# 通过 -v 容器内路径：ro rw 改变读写权限
ro    readonly  # 只读
rw    readwrite # 可读可写

# 一旦设置了这个容器的权限，容器对我们挂载出来的内容就有限制了
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro 只要看到ro就说明了这个路径只能通过宿主机来操作，容器内部是无法操作的！
```

## 初识DockerFile

DockerFile就是用来构建docker镜像的构建文件，是一种命令脚本。

通过这个脚本可以生成镜像，镜像是分层的，而脚本的一个个命令，每个命令就是一层！

> 测试

```shell
# 创建一个dockerFile文件  名字自定义 建议使用 dockerFile
# 文件中的内容   所有指令（大写） 参数
FROM centos

VOLUME ["volume01","volume02"]  # 直接只写容器内路径 ==>  匿名挂载

CMD echo "------end------"

CMD /bin/bash

# 这里的一个命令就是镜像的一层

# 在当前目录下生成镜像名为loumeng/centos的镜像
docker build -f dockerfile1 -t loumeng/centos .
```

![Docker下编写DockerFile测试文件](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下DockerFile测试.png)

​                                                                                                    **编写DockerFile测试文件**



- 构建dockerfile文件
- 在当前目录下生成镜像名为`loumeng/centos`的镜像：`docker build -f dockerfile1 -t loumeng/centos .`

![Docker下dockerfile构建镜像测试](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下dockerfile构建镜像测试.png)

​																						**Dockerfile构建镜像测试**



- 测试：运行生成的镜像文件，查看数据卷

  ![Docker下运行测试DockerFile结果](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下运行测试DockerFile结果.png)

​																							**运行测试DockerFile结果**



```shell
# 测试数据同步
# 在容器内部创建文件container.txt
[root@3530dcbfd45b /]# cd volume01
[root@3530dcbfd45b volume01]# touch container.txt 
[root@3530dcbfd45b volume01]# ls
conrainer.txt
[root@3530dcbfd45b volume01]# 

```

```shell
# 宿主机中查看是否有容器创建的文件
root@VM-8-12-ubuntu:/home/docker-test-volume# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                     NAMES
3530dcbfd45b   76a87817841c   "/bin/bash"              6 minutes ago   Up 6 minutes                                             bold_goodall
2ade1a1ee24f   nginx          "/docker-entrypoint.…"   4 hours ago     Up 4 hours     0.0.0.0:32770->80/tcp, :::32770->80/tcp   nginx02
157272f9755b   nginx          "/docker-entrypoint.…"   5 hours ago     Up 5 hours     0.0.0.0:32768->80/tcp, :::32768->80/tcp   nginx01
root@VM-8-12-ubuntu:/home/docker-test-volume# docker inspect 3530dcbfd45b
# 可以看到如下信息
```

![Docker下Dockerfile构建镜像后查看数据卷信息](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下Dockerfile构建镜像后查看数据卷信息.png)

​																				**Dockerfile构建镜像后查看数据卷的挂载路径**



- 得到数据卷挂载宿主机的路径后，就可以查看我们刚建的数据文件有没有被同步了。

```shell
root@VM-8-12-ubuntu:/home/docker-test-volume# cd /var/lib/docker/volumes/abf6b7a64fbd98c625b979ee27cdb222a4af1863b7706c387c53489bd5080417/_data
root@VM-8-12-ubuntu:/var/lib/docker/volumes/abf6b7a64fbd98c625b979ee27cdb222a4af1863b7706c387c53489bd5080417/_data# ls
container.txt
root@VM-8-12-ubuntu:/var/lib/docker/volumes/abf6b7a64fbd98c625b979ee27cdb222a4af1863b7706c387c53489bd5080417/_data# 
# 发现宿主机中该路径下是有文件container.txt
# 则说明我们的数据文件同步是没有问题的
```



> 总结：
>
> - 这种方法我们是非常常用的。
>
> - 假设构建容器时候没有挂载卷，就要手动使用`-v 卷名:容器内路径`命令进行挂载



## 数据卷容器

`--volumes-from`

![Docker下数据卷挂载的基本原理](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下数据卷挂载的基本原理.png)

​								    													**数据卷挂载的基本原理示意图**



![Docker下数据卷容器的概念](https://gitee.com/RoleHalo/blog-image/raw/master/image/Docker下数据卷容器的概念.png)

   																		            **数据卷容器的概念示意图**



> 实现：多个MySQL实现数据共享？

```shell
# 匿名挂载运行容器mysql01
docker run -d -p 3308:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 将容器mysql02通过挂载到容器mysql02
docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7

# 此时，就可以实现两个容器的数据同步了！
```



> 结论
>
> - 容器之间配置信息的传递，数据据卷容器的生命周期一直持续到没有容器使用为止。
> - 但是一旦持久化到了本地，-v，这个时候，本地的数据是不会被删除的！



# DockerFile































































# **Docker**网络原理

# IDEA整合Docker

































































































































































































# 集群

## Docker Compose

## Docker Swarm

## CI/CD Jenkins