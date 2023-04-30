# 介绍

## 什么是docker？

搬家------->搬楼

需要 Linux 内核版本3.8以上



## docker的三要素

- 镜像（image）

  - Docker镜像（Image）就是一个**只读**的模板。镜像可以创建Docker容器，**一个镜像可以创建很多容器**。它也相当于是一个root文件系统。比如官方镜像 cent os:7 就包含了一套 cent os:7 最小系统root文件系统。相当于容器的“源代码”，**docker镜像文件类似于Java的类模板，而docker容器示例类似于Java中new出来的实例对象。**

- 容器（container）

  - 面向对象角度

    Docker利用容器（container）独立运行的一个或一组应用，应用程序或服务运行在容器里面，容器就类似与一个虚拟化的运行环境，**容器是用镜像创建的运行实例**。就像是Java中的类和实例对象一样，镜像是静态的定义，容器是镜像运行时的实体。容器为镜像提供了一个标准的和隔离的运行环境，它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

  - 镜像容器角度

    **可以把容器看作简易版的Linux环境**（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

- 仓库（repository）

  - 仓库是**集中存放镜像**文件的场所。

  - 类似于

    **Maven**仓库，存放各种**jar**包的地方

    **GitHub**仓库，存放各种**git**项目的地方

    **Docker**公司提供的官方**registry**被称为**Docker Hub**，存放各种镜像模板的地方。

  - 仓库分为公开仓库（**Public**）和私有仓库（**Private**）两种形式。**最大的公开库是Docker Hub（https://hub.docker.com/）**，存放了数量庞大的镜像提供用户下载。国内的公开库包括阿里云、网易云等。

## 总结

**需要正确的理解仓库/镜像/容器这几个概念:**

**Docker** 本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是**image**镜像文件。只有通过这个镜像文件才能生成**Docker**容器实例（类似**Java**中**new**出来一个对象）。

**image** 文件可以看作是容器的模板。**Docker** 根据 **image** 文件生成容器的实例。同一个 **image** 文件，可以生成多个同时运行的容器实例。

**镜像文件**

**image** 文件生成的容器实例，本身也是一个文件，称为镜像文件。

**容器实例**

一个容器运行一种服务，当我们需要的时候，就可以通过**docker**客户端创建一个对应的运行实例，也就是我们的容器仓库

**仓库**

就是放一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候再从仓库中拉下来就可以了。



## 整体架构

1. 用户是使用Docker Client与Docker Daemon建立通信，并发送请求给后者。
2.  Docker Daemon 作为 Docker架构中的主体部分，首先提供Docker Server的功能使其可以接受Docker Client的请求。
3.  Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
4.  Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 Graph driver将下载镜像以Graph的形式存储。
5. 当需要为 Docker创建网络环境时，通过网络管理驱动 Network driver创建并配置 Docker 容器网络环境。
6. 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 Exec driver 来完成。
7.  Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。

# 安装

## 安装

删除老版本docker

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

安装yum-utils包(它提供yum-config-manager实用程序)

```bash
yum install -y yum-utils
```

设置稳定仓库（国内采用阿里云镜像）

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

建立索引

```bash
yum makecache fast
```

开启服务

```bash
systemctl start docker
```

查看进程

```bash
ps -ef | grep docker
```

查看版本

```bash
docker version
```

运行hello-world

```bash
docker run hello-world
```

安装成功



## 卸载

```bash
systemctl stop docker
yum remove docker-ce docker-ce-cli containerd.io
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```



## 配置阿里云镜像加速

```bash
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://0td1yci8.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

# 底层原理

## 为什么docker会比vm虚拟机快

- docker有着比虚拟机更少的抽象层

  docker不需要Hypervisor（虚拟机）实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

- docker利用的是宿主机的内核，而不需要加载操作系统OS内核

  当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载OS，返回新建过程是分钟级别的。而docker由于直接方系统，则省略了返回过程，因此新建一个docker容器只需要几秒钟。



# docker常用命令

## 帮助启动类命令

启动

```bash
systemctl start docker
```



停止

```bash
systemctl stop docker
```



重启

```bash
systemctl restart docker
```



查看

```bash
systemctl status docker
```



开机启动

```bash
systemctl enable docker
```



查看docker概要信息

```bash
docker info
```



查看docker总体帮助文档

```bash
docker --help
```



查看docker命令帮助文档

```bash
docker <command> --help 
```



## 镜像命令

**列出本地主机上的镜像**

```bash
docker images <OPTIONS>
```

OPTIONS

- -q：只显示镜像ID
- -a：列出本地所有镜像（含历史）

REPOSITORY：表示镜像的仓库源

TAG：镜像的标签版本号

IMAGE ID：镜像ID

CREATED：镜像创建时间

SIZE：镜像大小

同一个仓库源可以有多个TAG版本，代表这个仓库源的不同个版本，我们使用**REPOSITORY:TAG**来定义不同的镜像。

**如果不指定一个镜像的版本标签，默认安装latest镜像。**



**搜索某个镜像名字**

```bash
docker search <OPTIONS> <imagename>
```

OPTIONS：--limit 5	限制查询结果，默认25条数据

搜索的结果：

- NAME：镜像名称
- DESCRIPTION：镜像说明
- STARS：点赞说明
- OFFICIAL：是否是官方的
- AOTUMATED：是否是自动构建的



**拉取某个镜像**

下载镜像

```bash
docker pull <imagename:TAG>
```

没有TAG就是拉取最新版，等同于`docker pull redis:lastest`



**查看镜像、容器、数据卷所占空间**

```bash
docker system df
```



**删除某个镜像**

```bash
docker rmi
```

- 删除单个

  ```bash
  docker rmi -f imageID
  ```

  

- 删除多个

  ```bash
  docker rmi -f imagename1:TAG imagename2:TAG
  ```

  

- 删除全部

  ```bash
  docker rmi -f $(docker images -qa)
  ```



## 容器命令

**新建+启动容器**

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

OPTIONS说明（常用）：

`--name="容器新名字"`：为容器指定一个名称；

`-d`：后台运行容器并返回容器ID，也即启动守护式容器（后台运行）；

**`-i`：以交模式运行容器，通常与`-t`同时使用；**	interactive交互

**`-t`：为容器重新分配一个伪输入终端，通常与`-i`同时使用**；也即**启动交互式容器（前台有伪终端，等待交互）**		tty伪终端

`-P`：随机端口映射，大写P

`-p`：指定端口映射，小写p



```bash
docker run -it centos /bin/bash
```

使用镜像centos:latest**以交互式**启动一个容器，在容器内执行/bin/bash命令。



```bash
docker run -d centos
```

以**后台守护模式**启动一个容器，保护进程不被杀死。

**重点：docker容器后台运行，就比必须有一个前台进程。**

容器运行的命令如果不是那些**一直挂起的命令**（比如运行top，tail），就会自动退出

**解决方案：将要运行的程序以前台进程的形式运行，常见就是命令行模式，表示我还有交互操作。**



**列出当前所有运行的容器**

```bash
docker ps [OPTIONS]
```



**退出容器**

- exit：容器停止
- ctrl+p+q：容器不停止



**启动已经停止的容器**

```bash
docker start containerID or containername
```



**重启容器**

```bash
docker restart containerID or containername
```



**停止容器**

```bash
docker stop containerID or containername
```



**强制停止容器**

```bash
docker kill containerID or containername
```



**删除已停止的容器**

```bash
docker rm containerID
```



**查看容器日志**

```bash
docker logs cintainerID
```



**查看容器内运行的进程**

```bash
docker ps
```



**查看容器内部细节**

```bash
docker inspect containerID
```



**进入正在运行的容器并以命令行交互**

```bash
docker exec -it containerID bashShell
```

```bash
docker attach containerID
```

**区别：**

- attach直接进入容器启动命令的终端，不会启动新的进程，用exit退出，会导致容器的停止。



## 面试题：谈谈docker虚悬镜像是什么？

定义：仓库名、标签都是<none>的镜像，俗称虚悬镜像danging image





# docker镜像

## 镜像

镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。



**镜像是分层的**

------

## UnionFS（联合文件系统）

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持**对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

## Docker镜像加载原理：

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

## 分层结构的优点

镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。

比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；
同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

## 重点

**Docker镜像层都是只读的，容器层是可写的**
**当容器启动时，一个新的可写层被加载到镜像的顶部。**
**这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。**



## docker镜像 commit

docker commit提交容器副本使之成为一个新的镜像

```bash
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
```

演示ubuntu安装vim：

1. 从Hub上下载ubuntu镜像到本地并成功运行

2. **原始的默认Ubuntu镜像是不带着vim命令的**

3. 外网连通的情况下，安装vim

   1. ```bash
      apt-get update	# 更新包管理工具
      ```

   2. ```bash
      apt-get -y install vim	# 安装vim
      ```

4. 安装完成后，commit我们自己的新镜像

   1. ```bash
      docker commit -m="add vim cmd" -a="cls" 75b4d98556e2 cls/ubuntu:1.1
      ```

   2. 

5. 启动我们的新镜像并和原来的对比

6. 总结

   Docker中的镜像分层，支持通过扩展现有镜像，创建新的镜像。类似Java继承于一个Base基础类，自己再按需扩展。
   新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。

# 本地镜像发布到阿里云

https://cr.console.aliyun.com/cn-shenzhen/instance/repositories

------

# 本地镜像发布到私有库

1 官方Docker Hub地址：https://hub.docker.com/，中国大陆访问太慢了且准备被阿里云取代的趋势，不太主流。

2 Dockerhub、阿里云这样的公共镜像仓库可能不太方便，涉及机密的公司不可能提供镜像给公网，所以需要创建一个本地私人仓库供给团队使用，基于公司内部项目构建镜像。

```bash
Docker Registry # 是官方提供的工具，可以用于构建私有镜像仓库
```



- 下载镜像docker registry

  ```bash
  docker pull registry
  ```

- 运行私有库Registry，相当于本地有个私有Docker hub

  ```bash
  docker run -d -p 5000:5000  -v /root/myregistry/:/tmp/registry --privileged=true registry
  ```

  默认情况，仓库被创建在容器的/var/lib/registry目录下，建议自行用容器卷映射，方便于宿主机联调

- 演示创建一个新镜像，ubuntu安装ifconfig命令

  - ```bash
    docker -it run ubuntu /bin/bash
    ```

  - ```bash
    apt-get update
    ```

  - ```bash
    apt-get install net-tools
    ```

  - ```bash
    docker commit -m="添加net-tools功能" -a="cls" 75b4d98556e2 ubuntu:1.2
    ```

- curl验证私服库上有什么镜像

  私服库没有任何镜像上传过

  ```bash
  curl -XGET http://172.17.144.41:5000/v2/_catalog
  ```

  ![image-20220324161959078](F:/note/Docker.assets/image-20220324161959078.png)

- 将新镜像ubuntu:1.2修改符合私服规范的Tag

  - 按照公式： docker   tag   镜像:Tag   Host:Port/Repository:Tag

  - 使用命令 docker tag 将ubuntu:1.2 这个镜像修改为172.17.144.41:5000/ubuntu:1.2

    ```bash
    docker tag ubuntu:1.2 172.17.144.41:5000/ubuntu:1.2
    ```

- 修改配置文件使之支持http

  - vim命令修改至如下内容：vim /etc/docker/daemon.json

    ```json
    {
      "registry-mirrors": ["https://0td1yci8.mirror.aliyuncs.com"],
      "insecure-registries": ["172.17.144.41:5000"]
    }
    ```

    docker默认不允许http方式推送镜像，通过配置选项来取消这个限制。====> 修改完后如果不生效，建议重启docker

- push推送到私服库

  ```bash
  docker push 172.17.144.41:5000/ubuntu:1.2
  ```

- 验证私服库的镜像

  ```bash
  curl -XGET http://172.17.144.41:5000/v2/_catalog
  ```

- pull到本地并运行

  ```bash
  docker pull 172.17.144.41:5000/ubuntu:1.2
  ```


------

# Docker容器数据卷



坑：容器卷记得加入`--privileged=true`

Docker挂载主机目录访问如果出现 cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个--privileged=true参数即可

如果是CentOS7安全模块会比之前系统版本加强，不安全的会先禁止，所以目录挂载的情况被默认为不安全的行为，在SELinux里面挂载目录被禁止掉了额，如果要开启，我们一般使用`--privileged=true`命令，扩大容器的权限解决挂载目录没有权限的问题，也即使用该参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限。



数据卷：卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：
卷的设计目的就是**数据的持久化**，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷



```
docker run -d -p 5000:5000  -v /root/myregistry/:/tmp/registry --privileged=true registry
```

`-v`类似我们Redis里面的rdb和aof文件

将docker容器内的数据保存进宿主机的磁盘中

运行一个带有容器卷存储功能的容器实例：

```bash
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名
```



**案例**

- 宿主vs容器之间映射添加容器卷

  ```bash
  docker run -it --name myu3 --privileged=true -v /tmp/myHostData:/tmp/myDockerData ubuntu /bin/bash
  ```

- 查看数据卷是否**挂载**成功

  ```bash
  docker inspect 容器ID
  ```

- 容器和宿主机之间数据共享

  - 1、docker修改，主机同步获得
  - 2 、主机修改，docker同步获得
  - 3 、docker容器stop，主机修改，docker容器重启看数据是否同步

- 读写规则映射添加说明

  - 默认读写

    docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:rw      镜像名

  - 只读

    - 容器实例内部被限制，只能读取不能写
    - docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:ro      镜像名

- 卷的继承和共享

  - 容器1完成和宿主机的映射

    ```bash
    docker run -it  --privileged=true -v /mydocker/u:/tmp --name u1 ubuntu
    ```

    

  - 容器2继承容器1的卷规则

    ```bash
    docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu
    ```

    u2 继承 u1

------



# Docker常规安装简介

## 总体步骤

- 搜索镜像

  ```bash
  docker search
  ```

  

- 拉取镜像

  ```bash
  docker pull
  ```

  

- 查看镜像

  ```bash
  docker images
  ```

  

- 启动镜像

  ```bash
  docker run
  ```

  

- 停止容器

  ```bash
  docker stop
  ```

  

- 移除容器

  ```bash
  docker rm
  ```

  

## 安装tomcat

最新版首页404报错

解决：把webapps.dist目录换成webapps

```bash
docker run -d -p 8080:8080 tomcat
docker exec -it containerID /bin/bash
cd /usr/local/tomcat
```





免修改版说明：

```bash
docker pull billygoo/tomcat8-jdk8
```

```bash
docker run -d -p 8080:8080 --name mytomcat8 billygoo/tomcat8-jdk8
```



## 安装mysql

```bash
docker run -d -p 3306:3306 --privileged=true -v /root/mysql/log:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -v /root/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=CLS4838aaoc@. --name mysql mysql:5.7
```

```bash
cd /root/mysql/conf
vim my.cnf
```

```
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```

```bash
docker restart mysql
docker exec -it mysql bash
```



**结论：docker安装完MySQL并run出容器后，建议请先修改完字符集编码后再新建mysql库-表-插数据**



## 安装redis

```bash
mkdir -p /app/redis
```

将一个redis.conf文件模板拷贝进/app/redis目录下

修改redis.conf文件

- 开启redis验证（可选）

  requirepass 123

- 允许redis外地连接（**必须**）

  注释`bind 127.0.0.1` ===> `# bind 127.0.0.1`

- daemonize no

  **将daemonize yes注释起来或者 daemonize no设置，因为该配置和docker run中-d参数冲突，会导致容器一直启动失败**

- 开启redis数据持久化  appendonly yes  (可选)



```bash
docker run -p 6379:6379 --name redis -v /root/redis/data:/data -v /root/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
```





redis:6.0.8示例：

```bash
docker run -p 6379:6379 --name myr3 --privileged=true
-v /app/redis/redis.conf:/etc/redis/redis.conf
-v /app/redis/data:/data
-d redis:6.0.8 redis-server /etc/redis/redis.conf
```



------

# Docker复杂安装详说

## 安装mysql主从复制

**主从复制原理**

**主从搭建步骤**

1. 新建主服务器容器实例3307

   ```bash
   docker run -p 3307:3306 --name mysql-master \
   -v /mydata/mysql-master/log:/var/log/mysql \
   -v /mydata/mysql-master/data:/var/lib/mysql \
   -v /mydata/mysql-master/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=root  \
   -d mysql:5.7
   ```

2. 进入/mydata/mysql-master/conf目录下新建my.cnf

   1. ```bash
      cd /mydata/mysql-master/conf
      vim my.cnf
      ```

   2. ```bash
      [mysqld]
      ## 设置server_id，同一局域网中需要唯一
      server_id=101 
      ## 指定不需要同步的数据库名称
      binlog-ignore-db=mysql  
      ## 开启二进制日志功能
      log-bin=mall-mysql-bin  
      ## 设置二进制日志使用内存大小（事务）
      binlog_cache_size=1M  
      ## 设置使用的二进制日志格式（mixed,statement,row）
      binlog_format=mixed  
      ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
      expire_logs_days=7  
      ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
      ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
      slave_skip_errors=1062
      ```

3. 修改完配置后重启master实例

   ```bash
   docker restart mysql-master
   ```

4. 进入mysql-master容器

   1. ```bash
      docker exec -it mysql-master /bin/bash
      ```

   2. ```
      mysql -uroot -proot
      ```

5. master容器实例内创建数据同步用户

   1. ```bash
      CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
      ```

   2. ```
      GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
      ```

6. 新建从服务器容器实例3308

   ```bash
   docker run -p 3308:3306 --name mysql-slave \
   -v /mydata/mysql-slave/log:/var/log/mysql \
   -v /mydata/mysql-slave/data:/var/lib/mysql \
   -v /mydata/mysql-slave/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=root  \
   -d mysql:5.7
   ```

7. 进入/mydata/mysql-slave/conf目录下新建my.cnf

   1. ```bash
      cd /mydata/mysql-slave/conf
      vim my.cnf
      ```

   2. ```bash
      ## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
      log-bin=mall-mysql-slave1-bin  
      ## 设置二进制日志使用内存大小（事务）
      binlog_cache_size=1M  
      ## 设置使用的二进制日志格式（mixed,statement,row）
      binlog_format=mixed  
      ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
      expire_logs_days=7  
      ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
      ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
      slave_skip_errors=1062  
      ## relay_log配置中继日志
      relay_log=mall-mysql-relay-bin  
      ## log_slave_updates表示slave将复制事件写进自己的二进制日志
      log_slave_updates=1  
      ## slave设置为只读（具有super权限的用户除外）
      ```

8. 修改完配置后重启slave实例

   ```bash
   docker restart mysql-slave
   ```

9. 在主数据库中查看主从同步状态

   ```bash
   show master status;
   ```

10. 进入mysql-slave容器

    1. ```bash
       docker exec -it mysql-salve /bin/bash
       ```

    2. ```
       mysql -uroot -proot
       ```

11. 在从数据库中配置主从复制

    ```bash
    change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
    ```

    主从复制命令参数说明：

    - master_password：在主数据库创建的用于同步数据的用户密码；
    - master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
    - master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
    - master_connect_retry：连接失败重试的时间间隔，单位为秒。

12. 在从数据库中查看主从同步状态

    ```bash
    show slave status \G;
    ```

    Slave_IO_Running: No

    Slave_SQL_Running: No

    没开启

13. 在从数据库中开启主从同步

    ```bash
    slave start;
    ```

14. 查看从数据库状态发现已经同步

    Slave_IO_Running: Yes

    Slave_SQL_Running: Yes

15. 主从复制测试

    1. 主机新建库-使用库-新建表-插入数据，ok
    2. 从机使用库-查看记录，ok



------

## 安装redis集群

cluster(集群)模式-docker版
哈希槽分区进行亿级数据存储

### 面试题

面试题：1~2亿条数据需要缓存，请问如何设计这个存储案例

答案：单机单台100%不可能，肯定是分布式存储，用redis如何落地？

一般业界有3种解决方案：

- 哈希取余分区

  2亿条记录就是2亿个k,v，我们单机不行必须要分布式多机，假设有3台机器构成一个集群，用户每次读写操作都是根据公式：
  hash(key) % N个机器台数，计算出哈希值，用来决定数据映射到哪一个节点上。

  - 优点：
            简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡+分而治之的作用。
  - 缺点：
             原来规划好的节点，进行扩容或者缩容就比较麻烦了，不管扩缩，每次数据变动导致节点有变动，映射关系需要重新进行计算，在服务器个数固定不变时没有问题，如果需要弹性扩容或故障停机的情况下，原来的取模公式就会发生变化：Hash(key)/3会变成Hash(key) /?。此时地址经过取余运算的结果将发生很大变化，根据公式获取的服务器也会变得不可控。
    某个redis机器宕机了，由于台数数量变化，会导致hash取余全部数据重新洗牌。

- 一致性哈希算法分区

  - 是什么：

    一致性Hash算法背景
    　　一致性哈希算法在1997年由麻省理工学院中提出的，设计目标是为了解决
    分布式缓存数据变动和映射问题，某个机器宕机了，分母数量改变了，自然取余数不OK了。

  - 能干嘛：

    提出一致性Hash解决方案。
    目的是当服务器个数发生变动时，
    尽量减少影响客户端到服务器的映射关系

  - **3大步骤：**

    - 算法构建一致性哈希环

      一致性哈希环
          一致性哈希算法必然有个hash函数并按照算法产生hash值，这个算法的所有可能哈希值会构成一个全量集，这个集合可以成为一个hash空间[0,2^32-1^]，这个是一个线性空间，但是在算法中，我们通过适当的逻辑控制将它首尾相连(0 = 2^32^),这样让它逻辑上形成了一个环形空间。

      它也是按照使用取模的方法，前面笔记介绍的节点取模法是对节点（服务器）的数量进行取模。而一致性Hash算法是对2^32^取模，简单来说，**一致性Hash算法将整个哈希值空间组织成一个虚拟的圆环**，如假设某哈希函数H的值空间为 0-2^32-1^（即哈希值是一个32位无符号整形），整个哈希环如下图：**整个空间按顺时针方向组织**，圆环的正上方的点代表0，0点右侧的第一个点代表1，以此类推，2、3、4、……直到2^32-1^，也就是说0点左侧的第一个点代表2^32-1^， 0和2^32-1^在零点中方向重合，我们把这个由2^32^个点组成的圆环称为Hash环。

      ![image-20220329190609674](F:/note/Docker.assets/image-20220329190609674.png)

    - 服务器IP节点映射

      节点映射
         将集群中各个IP节点映射到环上的某一个位置。
         将各个服务器使用Hash进行一个哈希，具体可以选择服务器的IP或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。假如4个节点NodeA、B、C、D，经过IP地址的**哈希函数**计算(hash(ip))，使用IP地址哈希后在环空间的位置如下：![image-20220329190726657](F:/note/Docker.assets/image-20220329190726657.png)

    - key落到服务器的落键规则

      ​		当我们需要存储一个kv键值对时，首先计算key的hash值，hash(key)，将这个key使用相同的函数Hash计算出哈希值并确定此数据在环上的位置，**从此位置沿环顺时针“行走”**，第一台遇到的服务器就是其应该定位到的服务器，并将该键值对存储在该节点上。
      ​		如我们有Object A、Object B、Object C、Object D四个数据对象，经过哈希计算后，在环空间上的位置如下：根据一致性Hash算法，数据A会被定为到Node A上，B被定为到Node B上，C被定为到Node C上，D被定为到Node D上。

      ![image-20220329190904675](F:/note/Docker.assets/image-20220329190904675.png)

  - **优点**

    - 一致性哈希算法的**容错性**

      假设Node C宕机，可以看到此时对象A、B、D不会受到影响，只有C对象被重定位到Node D。一般的，在一致性Hash算法中，如果一台服务器不可用，则**受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据**，其它不会受到影响。简单说，就是C挂了，受到影响的只是B、C之间的数据，并且这些数据会转移到D进行存储。

      ![image-20220329191108928](F:/note/Docker.assets/image-20220329191108928.png)

    - 一致性哈希算法的**扩展性**

      ​		数据量增加了，需要增加一台节点NodeX，X的位置在A和B之间，那收到影响的也就是A到X之间的数据，重新把A到X的数据录入到X上即可，
      ​		不会导致hash取余全部数据重新洗牌。

      ![image-20220329191147967](F:/note/Docker.assets/image-20220329191147967.png)

  - 缺点

    一致性哈希算法的数据倾斜问题

    一致性Hash算法在服务节点太少时，容易因为节点分布不均匀而造成数据倾斜（被缓存的对象大部分集中缓存在某一台服务器上）问题，
    例如系统中只有两台服务器：![image-20220329194440426](F:/note/Docker.assets/image-20220329194440426.png)

  - 总结：

    为了在节点数目发生改变时尽可能少的迁移数据

    将所有的存储节点排列在收尾相接的Hash环上，每个key在计算Hash后会**顺时针**找到临近的存储节点存放。
    而当有节点加入或退出时仅影响该节点在Hash环上**顺时针相邻的后续节点**。  

    **优点**
    加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。

    **缺点** 
    数据的分布和节点的位置有关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果。

- 哈希槽分区

  - 是什么

    - 为什么出现

      一致性哈希算法的数据倾斜问题

      哈希槽实质就是一个数组，数组[0,2^14-1^]形成hash slot空间。

    - 能干什么

      解决均匀分配的问题，**在数据和节点之间又加入了一层，把这层称为哈希槽（slot），用于管理数据和节点之间的关系**，现在就相当于节点上放的是槽，槽里放的是数据。

      ![image-20220329194819023](F:/note/Docker.assets/image-20220329194819023.png)

      槽解决的是粒度问题，相当于把粒度变大了，这样便于数据移动。

      哈希解决的是映射问题，使用key的哈希值来计算所在的槽，便于数据分配。

    - 多少个hash槽

      一个集群只能有16384个槽，编号0-16383（0-2^14-1^）。这些槽会分配给集群中的所有主节点，分配策略没有要求。可以指定哪些编号的槽分配给哪个主节点。集群会记录节点和槽的对应关系。解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取余，余数是几key就落入对应的槽里。slot = CRC16(key) % 16384。以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了。

  - 哈希槽计算

    Redis 集群中内置了 16384 个哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在 Redis 集群中放置一个 key-value时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，也就是映射到某个节点上。如下代码，key之A 、B在Node2， key之C落在Node3上

    ![image-20220329194951607](F:/note/Docker.assets/image-20220329194951607.png)

    ![image-20220329194959416](F:/note/Docker.assets/image-20220329194959416.png)

### 3主3从redis集群配置

- 启动docker

  ```bash
  systemctl start docker
  ```

  

- 新建6个docker容器redis实例

  ```bash
  docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
   
  docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382
   
  docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383
   
  docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384
   
  docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385
   
  docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
  ```

  --net host ---> 使用宿主机的IP和端口，默认

  --cluster-enabled yes ---> 开启redis集群

  --appendonly yes ---> 开启持久化

  --port 6386 ---> redis端口号

- 进入容器redis-node-1并为6台机器构建集群关系

  - 进入容器

    ```bash
    docker exec -it redis-node-1 /bin/bash
    ```

  - 构建主从关系

    ```bash
    redis-cli --cluster create 172.17.144.41:6381 172.17.144.41:6382 172.17.144.41:6383 172.17.144.41:6384 172.17.144.41:6385 172.17.144.41:6386 --cluster-replicas 1
    ```

    **--cluster-replicas 1 表示为每个master创建一个slave节点**

- 链接进入6381作为切入点，**查看集群状态**

  - 链接进入6381作为切入点，查看节点状态

    ```bash
    redis-cli -p 6381
    ```

  - 查看集群信息

    ```bash
    cluster info
    ```

  - 查看集群节点

    ```bash
    cluster nodes
    ```

    

### 主从容错切换迁移案例

数据读写存储

- 启动6机构成的集群并通过exec进入

- 对6381新增两个key

- 防止路由失效加参数-c并新增两个key

  ```bash
  redis-cli -p 6381 -c
  ```

- 查看集群信息

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  

容错切换迁移

- 主6381和从机切换，先停止主机6381

  - 6381主机停了，对应的真实从机上位
  - 6381作为1号主机分配的从机以实际情况为准，具体是几号机器就是几号

- 再次查看集群信息

  ```bash
  docker exec -it redis-node-2 /bin/bash
  redis-cli -p 6382 -c
  cluster nodes
  ```

- 先还原之前的3主3从

  ```bash
  docker start redis-node-1
  docker stop redis-node-5
  # 等待心跳超时
  docker start redis-node-5
  ```

  - 先启6381
  - 再停6385
  - 再启6385
  - **主从机器分配情况以实际情况为准**

- 查看集群状态

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  

### 主从扩容案例

- 新建6387、6388两个节点+新建后启动+查看是否8节点

  ```bash
  docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387
  docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388
  docker ps
  ```

- 进入6387容器实例内部

  ```bash
  docker exec -it redis-node-7 /bin/bash
  ```

- 将新增的6387节点(空槽号)作为master节点加入原集群

  ```bash
  redis-cli --cluster add-node 172.17.144.41:6387 172.17.144.41:6381
  ```

  6387 就是将要作为master新增节点

  6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群

- 检查集群情况第1次

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  注意：新增节点的槽点为零

- 重新分派槽号

  ```bash
  redis-cli --cluster reshard 172.17.144.41:6381
  ```

- 检查集群情况第2次

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  槽号分派说明

  为什么6387是3个新的区间，以前的还是连续？
  重新分配成本太高，所以前3家各自匀出来一部分，从6381/6382/6383三个旧节点分别匀出1364个坑位给新节点6387

- 为主节点6387分配从节点6388

  ```bash
  redis-cli --cluster add-node 172.17.144.41:6388 172.17.144.41:6387 --cluster-slave --cluster-master-id 真实id
  ```

  命令：redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID

- 检查集群情况第3次

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  



### 主从缩容案例

- 目的：6387和6388下线

- 检查集群情况1获得6388的节点ID

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

- 将6388删除

  ```bash
  redis-cli --cluster del-node 172.17.144.41:6388 5d149074b7e57b802287d1797a874ed7a1a284a8
  ```

  命令：redis-cli --cluster del-node ip:从机端口 从机6388节点ID

- 将6387的槽号清空，重新分配，本例将清出来的槽号都给6381

  ```bash
  redis-cli --cluster reshard 172.17.144.41:6381
  ```

- 检查集群情况第二次

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  4096个槽位都指给6381，它变成了8192个槽位，相当于全部都给6381了，不然要输入3次，一锅端

- 将6387删除

  ```bash
  redis-cli --cluster del-node 192.168.111.147:6387 e4781f644d4a4e4d4b4d107157b9ba8144631451
  ```

  命令：redis-cli --cluster del-node ip:端口 6387节点ID

- 检查集群情况第三次

  ```bash
  redis-cli --cluster check 172.17.144.41:6381
  ```

  

