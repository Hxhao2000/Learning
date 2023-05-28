# Docker

> Docker学习

- Docker概述

- Docker安装

- Docker命令

   - 镜像命令

   - 容器命令

   - 操作命令

   - ......

- Docker镜像

- 容器数据卷

- DockerFile

- Docker网络原理

- IDEA整合Docker

- Docker Compose

- Docker Swarm

- CI\CD Jenkins

## Docker概述

### Docker为何而出现？

产品开发和上线通常在两套以上的环境中(开发、测试、生产)。

问题：当产品进行版本更新，对于产品运维而言比较麻烦，环境配置麻烦，每台机器都需要部署，耗时又耗力。



对比流程：

java	--	apk	--	发布(应用商店)	--	张三使用apk	--	安装即可

java	--	jar	--	打包带上环境(镜像)	--	（Docker仓库）	--	下载发布的镜像	--	运行



Docker的思想在于隔离，将每个箱子隔离起来。

### Docker的历史

Docker 公司位于旧金山，由法裔美籍开发者和企业家 Solumon Hykes 创立，其标志如下图所示。

![image-20210609114123053](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609114123053.png)



有意思的是，Docker 公司起初是一家名为 `dotCloud` 的平台即服务（Platform-as-a-Service, PaaS）提供商。

底层技术上，`dotCloud` 平台利用了 Linux 容器技术。为了方便创建和管理这些容器，dotCloud 开发了一套内部工具，之后被命名为“Docker”。Docker就是这样诞生的！

`Docker开源`

2013年，dotCloud 的 PaaS 业务并不景气，公司需要寻求新的突破。于是他们聘请了 Ben Golub 作为新的 CEO，将公司重命名为“Docker”，放弃dotCloud PaaS 平台，将Docker开源，怀揣着“将 Docker 和容器技术推向全世界”的使命，开启了一段新的征程。

如今 Docker 公司被普遍认为是一家创新型科技公司，据说其市场价值约为 10 亿美元。Docker 公司已经通过多轮融资，吸纳了来自硅谷的几家风投公司的累计超过 2.4 亿美元的投资。

几乎所有的融资都发生在公司更名为“Docker”之后。

### Docker的文档

Docker是基于Go语言开发的开源项目。

官网：https://www.docker.com/

文档地址：https://docs.docker.com/

默认仓库地址：https://hub.docker.com/

### Docker的功能

> 传统的虚拟机技术

![虚拟机技术](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/vmware-tech.svg)

**缺点：**

1. 资源占用严重

2. 冗余步骤多

3. 启动慢

> 容器化技术

==容器化技术也是一种虚拟机技术，但它并没有模拟一个完整的操作系统==

![容器化技术](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/container-tech.svg)

比较Docker和传统虚拟机的不同：

- 传统虚拟机，虚拟出硬件，运行完整的操作系统，然后在这个完整的系统上安装和运行软件
- 容器内的应用直接运行在宿主机，容器没有自己的内核，也没有独立的硬件
- 每个容器之间互相隔离，每个容器之间都有自己的文件系统，互不影响

> DevOps（开发和运维）

**应用更快速的交付和部署**

传统：一堆帮助文档，安装程序

Docker：打包镜像发布测试，一键运行

**更编辑的升级和扩缩容**

使用Docker之后，部署应用变得简单，逻辑更加清晰

项目打包为镜像之后，易于拓展

**更简单的系统运维**

容器化之后，开发、测试和生产的环境都高度一致

**更高效的资源利用**

Docker是内核级别的虚拟化，可以在一个物理机上运行多个容器实例！服务器的性能可以被完全利用。

## Docker的安装

### docker的基本组成部分

![Docker的架构图](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609131533801.png)

**镜像（images）：**

​	docker镜像如同模板，可以通过模板来创建容器服务，例如：Nginx镜像	-->	run	-->	Nginx01容器（提供服务器），可以通过这	个镜像创建多个容器 （最后的项目和服务都在容器中）

**容器（container）：**

​	Docker利用容器技术，独立运行在一个或者多个组应用，通过镜像来创建。

​	启动、停止、删除、基本命令！

​	目前可以将容器理解为简单的Lunix系统。

**仓库（repository）:**

仓库即为存放镜像的地方，仓库分共有和私有仓库，默认仓库为https://hub.docker.com/，为国外库，日常使用中，推荐使用阿里、华为、中科大的Docker库。

### 安装Docker（CentOS）

> 系统需求

需要至少CentOS 7以上的系统，具体详情见Docker的官方文档[Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/)

> 安装

```shell
# 1.卸载旧版本的docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2.安装需要的工具包
$ sudo yum install -y yum-utils

# 3.配置镜像源
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo				#默认是国外的源
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo	#推荐使用国内源

# 4.安装docker	ce为社区版	ee为企业版	
$ sudo yum install docker-ce docker-ce-cli containerd.io

# 5.启动docker
$ sudo systemctl start docker

# 6.验证是否安装成功
$ sudo docker run hello-world
```

### Docker的卸载

```shell
# 1.卸载docker的引擎及各种依赖
$ sudo yum remove docker-ce docker-ce-cli containerd.io

# 2.删除资源	/var/lib/docker是docker的默认工作路径
$ sudo rm -rf /var/lib/docker
```

### 阿里云镜像加速

1.登录阿里云，在控制台中找到容器服务

![阿里云控制台](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609151801633.png)

2.找到镜像加速地址

![阿里云镜像加速器](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609152049164.png)

### 底层原理

**Docker是怎么工作的？**

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问，Docker-Server收到Docker-Client的命令后，将执行命令。

![docker工作原理](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609153312469.png)

**Docker为什么比VM快？**

1、Docker有着更少的抽象层
2、Docker利用宿主机的内核，VM需要加载操作系统的内核
![docker与VM对比](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609153449166.png)

docker新建一个容器时，不需要重新加载一个OS的内核，通过了利用宿主机的内核，省略VM对OS内核的模拟过程。

## Docker的常用命令

### 帮助命令

```shell
docker version			# 显示docker的版本信息
docker info				# 显示docker的系统信息，包括容器和镜像
docker command --help	# 帮助命令
```

[docker命令的文档地址](https://docs.docker.com/engine/reference/)

### 镜像命令

**docker images** 查看所有本地主机上的镜像

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB

# REPOSITORY	镜像的仓库源
# TAG			镜像的标签
# IMAGE ID		镜像的id
# CREATED		镜像的创建时间
# SIZE			镜像的大小

# OPTIONS
Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

**docker search** 搜索镜像

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker search mysql
NAME			DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql			MySQL is a widely used, open-source relation…   10970     [OK]       
mariadb			MariaDB Server is a high performing open sou…   4148      [OK]
...

# OPTIONS
Usage:  docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
      
# 示例
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker search mysql --filter=STARS=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   10970     [OK]  
```

**docker pull** 下载镜像

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker pull mysql
Using default tag: latest		# 默认为latest
latest: Pulling from library/mysql
69692152171a: Pull complete		# 分层下载，docker image的核心 联合文件系统
1651b0be3df3: Pull complete 
951da7386bc8: Pull complete 
0f86c95aa242: Pull complete 
37ba2d8bd4fe: Pull complete 
6d278bb05e94: Pull complete 
497efbd93a3e: Pull complete 
f7fddf10c2c2: Pull complete 
16415d159dfb: Pull complete 
0e530ffc6b73: Pull complete 
b0a4a1a77178: Pull complete 
cd90f92aa9ef: Pull complete 
Digest: sha256:d50098d7fcb25b1fcb24e2d3247cae3fc55815d64fec640dc395840f8fa80969		# 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest	# 真实地址

# OPTIONS
Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output
```

**docker rmi** 删除镜像

![当前服务器镜像](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210609160515430.png)

```shell
# docker rmi -f 镜像id	# 删除指定id的镜像
# docker rmi -f 镜像id 容器id 容器id	# 批量删除镜像
# docker rmi -f $(docker images -a)	# 删除本地全部镜像

# OPTIONS
Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

### 容器命令

**当我们拥有镜像之后才能创建容器**

```shell
docker pull centos	# 下载centos镜像作为示例
```

**新建容器并启动**

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

# OPTIONS
--name="NAME"	# 赋予容器一个名字
-d				# 后台方式运行
-it				# 交互方式运行
-p				# 指定容器的端口
	-p 主机端口:容器端口(常用方式)
	-p ip:主机端口:容器端
	-p 容器端口
-P				# 随机端口
--add-host # 添加/etc/hosts配置

# 测试，启动并进入容器
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker run -it centos /bin/bash
[root@4bf1587cf6e5 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 从容器中退回主机
[root@4bf1587cf6e5 /]# exit
exit
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# ls
app  dump.rdb  nohup.out
```
==docker容器后台运行时，必须带有至少一个前台进程，当docker发现没有应用，就会自动停止==

**列出所有运行的容器**

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# OPTIONS
Usage:  docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes
```

**退出容器**

1、`Ctrl`+`P`+`Q`	容器不停止，退出

2、exit	直接停止并退出

**删除镜像**

```shell
docker rm 容器id		# 删除指定的容器，不能删除正在运行的容器。如果要强制删除请用 rm -f
docker -rm -f $(docker ps -a)	# 删除所有容器，无论是否在运行
docker ps -a -q|xargs docker rm	# 删除所有容器，无论是否在运行
```

**启动和停止容器**

```shell
docker start 容器id		# 启动容器
docker restart 容器id		# 重启容器
docker stop	容器id		# 停止当前正在运行的容器
docker kill	容器id		# 强制停止当前容器
```

### 其他命令

**后台启动容器**

```shell
# 命令 docker run -d 镜像名

# 示例
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]#  docker run -d centos
146723e5ec13accda97c16f1cb378fa6b4bea431df6e5e2512553d8679c741aa
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```

**查看日志**

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker logs -tf --tail 5 036f1dc6b636
2021-06-08T08:11:10.704694308Z  https://hub.docker.com/
2021-06-08T08:11:10.704696186Z 
2021-06-08T08:11:10.704697983Z For more examples and ideas, visit:
2021-06-08T08:11:10.704700395Z  https://docs.docker.com/get-started/
2021-06-08T08:11:10.704702451Z 

Usage:  docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
```

**查看容器的进程信息**

```shell
# docker top 容器id
```

**查看镜像的元数据**

```shell
# docker inspect 容器id

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

**进入当前正在运行的容器**

```shell
# 容器通常后台方式运行，但当我们需要进入容器修改一些配置时，需要进入正在运行的容器

# 方式一	进入容器后，打开新的终端，可以在里面进行操作(常用)
docker exec -it 容器id bashshell

# 方式二	进入容器中正在执行的终端
docker attach 容器id
```

**从容器内拷贝文件到主机上**

```shell
docker cp 容器id:容器内路径  目的主机路径
```

### 小结

![docker命令图解](F:\BackEndKnowledge\backendtodo\Docker\jpg\docker命令图解.jpg)

```shell
attach      Attach local standard input, output, and error streams to a running container
build       Build an image from a Dockerfile
commit      Create a new image from a container's changes
cp          Copy files/folders between a container and the local filesystem
create      Create a new container
diff        Inspect changes to files or directories on a container's filesystem
events      Get real time events from the server
exec        Run a command in a running container
export      Export a container's filesystem as a tar archive
history     Show the history of an image
images      List images
import      Import the contents from a tarball to create a filesystem image
info        Display system-wide information
inspect     Return low-level information on Docker objects
kill        Kill one or more running containers
load        Load an image from a tar archive or STDIN
login       Log in to a Docker registry
logout      Log out from a Docker registry
logs        Fetch the logs of a container
pause       Pause all processes within one or more containers
port        List port mappings or a specific mapping for the container
ps          List containers
pull        Pull an image or a repository from a registry
push        Push an image or a repository to a registry
rename      Rename a container
restart     Restart one or more containers
rm          Remove one or more containers
rmi         Remove one or more images
run         Run a command in a new container
save        Save one or more images to a tar archive (streamed to STDOUT by default)
search      Search the Docker Hub for images
start       Start one or more stopped containers
stats       Display a live stream of container(s) resource usage statistics
stop        Stop one or more running containers
tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
top         Display the running processes of a container
unpause     Unpause all processes within one or more containers
update      Update configuration of one or more containers
version     Show the Docker version information
wait        Block until one or more containers stop, then print their exit codes
```

### 作业练习

#### 练习一

> Docker安装Nginx

```shell
# 1、搜索镜像	search	建议去docker网站搜索，可以看到更多信息
# 2、下载镜像	pull
# 3、运行测试
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    d1a364dc548d   2 weeks ago    133MB
centos       latest    300e315adb2f   6 months ago   209MB

# -d 后台运行
# --name 给容器命名
# -p 宿主机端口:容器内部端口
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker run -d --name Nginx01 -p 443:80 nginx
7376d265e4b490764d1f7dc4c617f6c372ec842cb228cd9b83606809dcf3c957
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                 NAMES
7376d265e4b4   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   0.0.0.0:443->80/tcp, :::443->80/tcp   Nginx01

# 宿主机通过宿主机端口访问容器内部端口
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# curl localhost:443
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 可视化

- portainer

- Rancher(CI/CD)

  

## Docker镜像的讲解

### 镜像是什么？

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、库、环境变量、配置文件。

所有的应用，直接打包docker镜像，可以直接跑起来

**如何得到镜像：**

- 从远程仓库下载
- 拷贝
- 自己制作一个镜像DockerFile

### Docker镜像加载原理

> UnionFS(联合文件系统)

UnionFS(联合文件系统)：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

> Docker镜像加载原理

docker的镜像实际上是由一层一层的文件系统组成，这种层级的文件系统为UnionFS。

bootfs（boot file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。 

rootfs（root file system），在bootfs之上。包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu， CentOS等等。

![镜像加载原理图](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610140208121.png)

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就可以了。由此可见对于不同的linux发行版本，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs。

### 分层理解

> 分层的镜像

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
69692152171a: Already exists 
a4a46f2fd7e0: Pull complete 
bcdf6fddc3bd: Pull complete 
2902e41faefa: Pull complete 
df3e1d63cdb1: Pull complete 
fa57f005a60d: Pull complete 
Digest: sha256:7e2c6181ad5c425443b56c7c73a9cd6df24a122345847d1ea9bb86a5afc76325
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

通过下载redis镜像作为示例来理解。Docker镜像采用分层的结构，这种结构利于资源的共享，当多个镜像从相同得Base镜像构建而来，那么宿主机只需在磁盘上保留一份base镜像，同时内存中也只需要加载一份base镜像，这样就可以为所有的容器服务，节省了主机内存。

通过docker image inspect 查看镜像分层。

![镜像分层](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610152402910.png)

**理解：**

所有的Docker镜像都起始于一个基础镜像层，当进行修改或增加新的内容时，就会在当前镜像层之上，创建新的镜像层。

举一个简单的例子，假如基于Ubuntu Linux 16.04创建一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加Python包，就会在基础镜像层之上创建第二个镜像层；如果继续添加一个安全补丁，就会创建第三个镜像层。

该镜像当前已经包含3个镜像层，如下图所示（演示用的简单例子）。

![简单例子](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610152950520.png)

在添加额外的镜像层的同时，镜像始终是当前所有镜像的组合。

![添加额外的镜像](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610153903983.png)

上图中的情况下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像中。

Docker通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。

Linux上可用的存储引擎有AUFS、Overlay2、Device Mapper、Btrfs以及ZFS。顾名思义，每种存储引擎都基于Linux中对应的文件系统或者块设备技术，并且每种存储引擎都有其独有的性能特点。

Docker在Windows上仅支持windowsfilter一种存储引擎，该引擎基于NTFS文件系统之上实现了分层和CoW。

下图展示了与系统显示相同的三层镜像。所有的镜像层堆叠并合并，对外提供统一的视图。

![统一视图](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610154609837.png)

> 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部，这一层就是我们通常说的容器层，容器之下的都叫镜像层。

下图中为Nginx镜像在添加一个Nginx.conf的备份之后生成的容器提交成的镜像与原生镜像的对比。

![添加了备份的nginx镜像](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610160455345.png)

![没添加备份的原生nginx镜像](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610160534599.png)

### commit镜像

```shell
docker commit 提交容器成为一个新的副本
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```



## 容器数据卷

### 什么是容器数据卷

docker将应用和环境打包成一个镜像，如果容器被删除了，则数据也会丢失。

由此诞生新的需求：==数据可以持久化==

需求带来技术：==通过数据卷来实现数据共享==

通过目录的挂载，将容器内的目录挂载在Linux主机上，以此完成Docker容器中的数据同步到本地。

总结：容器数据卷的目的是==数据的持久化和同步操作==



### 使用数据卷

```shell
docker run -it -v 主机目录:容器内目录

# 测试
[root@iZuf66ox0kxji2tg3f4kbeZ hxh]# docker run -it -v /home/hxh/centos:/home centos /bin/bash
```

![容器元数据](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610162821343.png)

### 具名和匿名挂载

![匿名挂载](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610164541326.png)

![卷详情](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610164643647.png)

所有的docker容器内的卷，没有指定目录的情况下，默认位置为`/var/lib/docker/volumes/xxxx/_data`

通过具名挂载方便我们找到卷，因此大多数情况下都使用`具名挂载`

```shell
# 如何确定具名挂载、匿名挂载、指定路径挂载
-v 容器内路径	# 匿名挂载
-v 卷名:容器内路径	# 具名挂载
-v 主机路径:容器内路径	# 指定路径挂载
```

拓展：

```shell
# 通过 -v 容器内路径:ro/rw 改变读写权限
ro read-only #只读
rw read-write	#可读可写

# 一旦设置了容器权限，容器对我们挂载出来的内容就有限定了
docker run -d -P --name nginx02 -v xxxx:/etc/nginx:ro	nginx
docker run -d -P --name nginx02 -v xxxx:/etc/nginx:rw	nginx

# ro 内容必须通过宿主机来修改，容器内部只有读的权限
```

### 数据卷容器

>  容器同步数据

![容器同步数据](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210610171047268.png)

 ```shell
 docker run -it --name containerName --volumes-from containersName imageName:tag(imageId)
 ```

结论：

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器为止。

一旦持久化到本地，本地的数据是不会删除的。



## DockerFile

### DockerFile介绍

dockerfile是用来构建docker镜像的文件！它是一个命令参数脚本！

构建步骤为：

1、编写dockerfile文件

2、docker build 构建成为一个镜像

3、docker run 运行镜像

4、docker push 发布镜像（DockerHub、阿里云仓库）

![DockerHub-CentOS](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210615160525121.png)

![github](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210615160616397.png)

### DockerFile构建过程

**基础知识：**

1、每个保留关键字（指令）都必须是大写字母

2、执行从上到下顺序执行

3、`#`表示注释

4、每一个指令都会创建提交一个新的镜像层，并提交



Dockerfile是面向开发的，我们要发布项目，做镜像，就需要编写dockerfile文件。

步骤：开发，部署，运维

DockerFile：构建文件，定义了一切的步骤，源代码

DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品

Docker容器：镜像的实例化，运行以提供服务

### DockerFile的指令

![img](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/dockerfile-command.jpg)

```shell
FROM			# 基础镜像，从这里开始构建
MAINTAINER		# 镜像作者，姓名+邮箱
RUN				# 镜像构建时需要运行的命令
ADD				# 例如：需要tomcat镜像，则添加tomcat的压缩包
WORKDIR			# 镜像的工作目录
VOLUME			# 挂载的目录
EXPOSE			# 暴露端口
CMD				# 指定这个容器启动的时候要运行的命令，只有最后一个生效，可被替代
ENTRYPOINT		# 指定这个容器启动的时候要运行的命令，可追加参数
ONBUILD			# 当构建一个被继承 DockerFile 这个时候就会运行ONBUILD的指令
COPY			# 类似ADD，将文件拷贝到镜像中
ENV				# 构建时设置环境变量
```

### 实战测试

Docker Hub中99%的镜像大批由`scratch`这个基础镜像开始的 FROM scratch

![github](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210615160616397.png)

> 创建一个自己的CentOS

```shell
# 1、编写dockerFile的文件
FROM centos
MAINTAINER hxhao<26044330543@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim 
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "--------end---------"
CMD /bin/bash

# 2、通过dockerfile文件构建镜像
# docker build -f dockerfile文件路径 -t 镜像名:[tag]
```

![mycentos](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210615164258035.png)

我们可以列出本地进行的变更历史

![docker-history](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210615164509027.png)



### CMD 和 ENTRYPOINT 的区别

```shell
CMD				# 指定这个容器启动的时候要运行的命令，只有最后一个生效，可被替代
ENTRYPOINT		# 指定这个容器启动的时候要运行的命令，可追加参数
```

测试CMD

```shell
# 1、编写dockerfile文件
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# vim test-CMD
FROM centos
CMD ["ls","-a"]

# 2、构建镜像
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker build -f test-CMD -t cmdtest .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 300e315adb2f
Step 2/2 : CMD ["ls","-a"]
 ---> Running in 54176acf2a96
Removing intermediate container 54176acf2a96
 ---> 286cbffb0ddb
Successfully built 286cbffb0ddb
Successfully tagged cmdtest:latest

# 3、run运行镜像，生成容器
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run 286cbffb0ddb
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 尝试追加命令 -l ls -al
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run 286cbffb0ddb -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.

# cmd的情况下 -l 替换了整个CMD ["ls","-a"]命令，-l不是命令，因此报错了
```

测试ENTRYPOINT

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# vim entrypointtest
FROM centos
ENTRYPOINT ["ls","-a"]

[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker build -f entrypointtest -t entrypoint-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 300e315adb2f
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in d569be4768c2
Removing intermediate container d569be4768c2
 ---> 3e49b39489b8
Successfully built 3e49b39489b8
Successfully tagged entrypoint-test:latest
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run 3e49b39489b8
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 追加命令直接拼接在ENTRYPOINT命令后面
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run 3e49b39489b8 -l
total 0
drwxr-xr-x   1 root root   6 Jun 15 08:58 .
drwxr-xr-x   1 root root   6 Jun 15 08:58 ..
-rwxr-xr-x   1 root root   0 Jun 15 08:58 .dockerenv
lrwxrwxrwx   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root 340 Jun 15 08:58 dev
drwxr-xr-x   1 root root  66 Jun 15 08:58 etc
drwxr-xr-x   2 root root   6 Nov  3  2020 home
lrwxrwxrwx   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root   6 Dec  4  2020 lost+found
drwxr-xr-x   2 root root   6 Nov  3  2020 media
drwxr-xr-x   2 root root   6 Nov  3  2020 mnt
drwxr-xr-x   2 root root   6 Nov  3  2020 opt
dr-xr-xr-x 124 root root   0 Jun 15 08:58 proc
dr-xr-x---   2 root root 162 Dec  4  2020 root
drwxr-xr-x  11 root root 163 Dec  4  2020 run
lrwxrwxrwx   1 root root   8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root   6 Nov  3  2020 srv
dr-xr-xr-x  13 root root   0 Jun 15 08:58 sys
drwxrwxrwt   7 root root 145 Dec  4  2020 tmp
drwxr-xr-x  12 root root 144 Dec  4  2020 usr
drwxr-xr-x  20 root root 262 Dec  4  2020 var
```

### 小结

![docker流程](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/docker-flowchart.jpg)



## Docker网络

### 理解Docker0

1、查看ip地址

![测试](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616151351269.png)

2、启动docker容器

```shell
# Q:docker是如何处理容器网络访问的？

[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run -d -P --name tomcat01 tomcat

# 查看容器的内部网络地址	ip addr	容器启动的时候会得到一个形如eth0@if37的ip地址
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
36: eth0@if37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 思考linux本机是否能ping通容器内部?
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.105 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.073 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.060 ms
```

3、再次查看ip地址

![image-20210616153209017](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616153209017.png)

> 原理

- 每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡docker0，此网卡通过桥接模式连接互联网，使用的技术是evth-pair技术。

- evth-pair 带来一对一对的虚拟设备接口，它们都成对出现，一端连着协议，一端彼此相连，evth-pair充当桥梁，连接各种虚拟网络设备

- OpenStac，Docker容器之间的连接，OVS的连接，都是使用evth-pair技术

4、测试容器之间能否ping通

```shell
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat02 ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.131 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.081 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.074 ms
```



绘制网络模型图：

![docker网络模型图](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/docker-network.svg)



结论：tomcat01和tomcat02是公用的一个路由器，docker0

所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用ip



> 小结

Docker使用的是Linux的桥接，宿主机中是一个Docker容器的网桥 docker0

![linux本机总体框架](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/framework.svg)

Docker中的所有的网络接口都是虚拟的。转发效率高。

只要容器删除，对应的网桥就被删除了



### --link

> 思考一个场景，我们编写了一个微服务，database url=ip，项目不重启，数据库IP进行了更换，我们希望可以通过名字来访问容器，以此来避免IP更换的问题？

```shell
# 如何解决这个问题呢？
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known

# 通过--link 就可以解决网络连通问题
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
5b86ba491415cad6bbecb91f2de862aeeb2da9ddf6a9007fde98409c1fd99152
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.18.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.18.0.3): icmp_seq=1 ttl=64 time=0.117 ms
64 bytes from tomcat02 (172.18.0.3): icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from tomcat02 (172.18.0.3): icmp_seq=3 ttl=64 time=0.081 ms

# 反向可以ping通吗？
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known

```

探究：

![docker inspect](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616162934011.png)

```shell
# 查看 hosts 配置
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.3	tomcat02 d60dc525a578
172.18.0.4	5b86ba491415
```

本质：--link 即在容器中增加了一个 `172.18.0.3	tomcat02 d60dc525a578`

`--link`现在已经不推荐使用了

### 自定义网络

> 查看所有的docker 网络

![docker网络](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616163721139.png)

**网络模式：**

bridge：桥接docker(默认)

none：不配置网络

host：和宿主机共享网络

container：容器网络连通（使用少，局限性大）

**示例**

```shell
# 直接启动的命令中，默认为--net bridge，这个就是我们的docker0
docker run -d -P --name tomcat01 --net bridge tomcat
docker run -d -P --name tomcat01 tomcat

# 我们可以自定义一个网络
# --driver bridge
# --subnet 192.168.0.0/16
# --gateway 192.168.0.1
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
2bfd2b95e078d2fe744afc2c9fff1908794394227e3ea01286b6801e67ea7480

[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
c48dbe2ddbb7   bridge    bridge    local
4de7bd3002d1   host      host      local
2bfd2b95e078   mynet     bridge    local
3f13a404ab2e   none      null      local

[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "2bfd2b95e078d2fe744afc2c9fff1908794394227e3ea01286b6801e67ea7480",
        "Created": "2021-06-16T16:47:58.307229133+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
# 启动两个容器在自定义的网络下
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run -d -P --name tomcat01-mynet --net mynet tomcat
d265e25bcacb2e71bb759a99768f5d93c73a3f7b9f0c384387a6c52b8ed6fab5
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker run -d -P --name tomcat02-mynet --net mynet tomcat
0546cab6111770f98bdd0c3c2c5d86391f2b5aafc6f17e8aa1650a399fa39856
# 查看自定义网络详情
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "2bfd2b95e078d2fe744afc2c9fff1908794394227e3ea01286b6801e67ea7480",
        "Created": "2021-06-16T16:47:58.307229133+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0546cab6111770f98bdd0c3c2c5d86391f2b5aafc6f17e8aa1650a399fa39856": {
                "Name": "tomcat02-mynet",
                "EndpointID": "fbae04a5ce13410afb4ef31835c6240eec3bd2a9f74311a850ca9a8f1ad85a73",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "d265e25bcacb2e71bb759a99768f5d93c73a3f7b9f0c384387a6c52b8ed6fab5": {
                "Name": "tomcat01-mynet",
                "EndpointID": "6f8ccc2b599b8fee6cd3c3a5b1b355b214239a4847f572b627edbbc1f90f5e94",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 再次测试ping连接，发现已经不需要--link
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat01-mynet ping tomcat02-mynet
PING tomcat02-mynet (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02-mynet.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.104 ms
64 bytes from tomcat02-mynet.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.102 ms

[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat01-mynet ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.102 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.098 ms
```

我们自定义的网路docker已经帮我们维护好了对应的关系，推荐我们平时如此使用网络



好处：

redis - 不同的集群使用不同的网络，保证集群是安全和健康的

mysql - 不同的集群使用不同的网络，保证集群式安全和健康的



### 网络连通

```shell
# 不同网段下的容器不能ping通
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker exec -it tomcat01 ping tomcat01-mynet 
ping: tomcat01-mynet: Name or service not known
```



![连通](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616170050452.png)

![命令详情](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210616170200645.png)

```shell
# 测试连通 tomcat01 至 mynet
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network connect mynet tomcat01
[root@iZuf66ox0kxji2tg3f4kbeZ centos]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "2bfd2b95e078d2fe744afc2c9fff1908794394227e3ea01286b6801e67ea7480",
        "Created": "2021-06-16T16:47:58.307229133+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0546cab6111770f98bdd0c3c2c5d86391f2b5aafc6f17e8aa1650a399fa39856": {
                "Name": "tomcat02-mynet",
                "EndpointID": "fbae04a5ce13410afb4ef31835c6240eec3bd2a9f74311a850ca9a8f1ad85a73",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "d265e25bcacb2e71bb759a99768f5d93c73a3f7b9f0c384387a6c52b8ed6fab5": {
                "Name": "tomcat01-mynet",
                "EndpointID": "6f8ccc2b599b8fee6cd3c3a5b1b355b214239a4847f572b627edbbc1f90f5e94",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "e70708c4c3f8f69f159344e96d3ef6a08b484e77a8e237ce5774a10f9221b754": {
                "Name": "tomcat01",
                "EndpointID": "75cd37b9e8b18fe48b35fc91bb78f7ba956891e65e6711e76e578660d19a1047",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
# 可见，连通之后 将tomcat01放在mynet中
# 一个容器两个ip地址
```



## Docker Compose

### 简介

​		Docker中结合dockerfile，并通过docker build 和docker run 可以手动操作单个容器，每个容器提供一种微服务，而开发过程中，有时需要一次配置数量众多的容器，并且这些容器间还存在一定的依赖关系。在这种情况下，Compose的作用体现了出来，Docker Compose可以轻松高效地管理容器，它可以同时定义和运行多个容器。

> 官方介绍

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.

> 个人理解

Compose是Docker官方的开源项目，它并非Docker自带的命令，而是需要额外安装的。其中YAML文件是Compose的重要组成部分，一个正常的YAML文件如下所示：

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose中重要的概念：

- 服务Service，每个容器都提供的应用（web、mysql、redis）
- 项目Project，一组关联在一起的容器



### 安装

1、下载

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

![image-20210618114928193](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210618114928193.png)

2、授权

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

![image-20210618140933445](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210618140933445.png)

```shell
docker-compose version # 查看版本号
```

![image-20210618141158582](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210618141158582.png)

### yaml规则

docker-compose.yaml编写规则：

[官方文档](https://docs.docker.com/compose/compose-file/)

```shell
# 总体分为3层

# 第一层
version: ''	# 版本

# 第二层
services: '' # 服务
	service1: web
		images	# 服务配置
		build
		network
		...
	service2: redis
		...
	service3: mysql
		...
	...
	
# 第三层
# 其他配置	网络/卷、全局规则
volumes:
networks:
configs:
```

![depends_on](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/image-20210619162517834.png)

### 开源项目

[利用Compose搭建wordpress的博客系统](https://docs.docker.com/samples/wordpress/)



## 离线安装docker

```shell
# 在线环境下更换安装源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 在线环境下载docker-ce docker-ce-cli containerd.io需要的依赖安装包
sudo yumdownloader --resolve docker-ce docker-ce-cli containerd.io
# 将依赖安装包打包为tar文件
tar cf docker-ce.offline.tar *.rpm

# 将tar文件移到离线电脑中，解压tar
tar xf docker-ce.offline.tar
# 离线安装依赖
sudo rpm -ivh --replacefiles --replacepkgs *.rpm
# 启动docker服务
sudo systemctl start docker 
# 设置开机自启动
sudo systemctl enable docker
```

https://docs.docker.com/network/iptables/#integration-with-firewalld
