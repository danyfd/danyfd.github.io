---
title: docker笔记
date: 2023-10-13 18:53:37
tags: "docker"
categories: "工具"
---

## 知识点

### Docker存在意义

#### docker作用

##### 虚拟机技术与容器化技术

- 虚拟机技术
  1. 资源占用十分多
  2. 荣誉步骤多
  3. 启动很慢
- 容器化技术
  1. 直接运行在宿主机中，容器自身无内核，也没有虚拟出硬件，所以轻便
  2. 每个容器内有自己的文件系统，互不影响

##### DevOps（开发、运维）

- 应用更快速的交付和部署

  - 传统：帮助文档安装程序
  - Docker：打包镜像发布测试一键运行

- 更便捷的升级和扩缩容

- 更简单的系统运维

  容器化后，开发和测试环境高度一致

- 更有效的计算资源利用

#### 为什么docker比vm快

1. docker抽象层比虚拟机少，无需Hypervisor实现硬件资源虚拟化，运行在docker上的程序使用实际物理机的硬件资源，故docker在效率上更有优势
2. docker利用的是宿主机的内核，而不需要Guest OS

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwNDExNzMyOS5wbmc?x-oss-process=image/format,png)

#### docker安装

##### 镜像（image）

镜像相当于一个目标，通过其创建容器服务

##### 容器（container）

镜像创建的单个应用

##### 仓库（repository）

仓库是存放镜像的地方，仓库分为公有仓库和私有仓库

```
#1.卸载旧版本
yum remove docker \
                   docker-client \
                   docker-client-latest \
                   docker-common \
                   docker-latest \
                   docker-latest-logrotate \
                   docker-logrotate \
                   docker-engine

#2.需要的安装包
yum install -y yum-utils

#3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
#更新yum软件包索引
yum makecache fast

#4.安装docker相关的docker-ce社区版	ee是企业版
yum install docker-ce docker-ce-cli containerd.io

#5.启动docker
systemctl start docker

#6.使用docker version查看是否安装成功
docker version 

#7.测试
docker run hello-world

#8.查看已下载的镜像
docker images
```

#### docker卸载

```
#1.卸载依赖
yum remove docker-ce docker-ce-cli containerd.io
#2.删除资源
yum -rf /var/lib/docker		//docker默认工作路径
```

### 阿里镜像加速

#### 登录阿里云找到容器镜像服务

![](https://s1.ax1x.com/2023/02/03/pSsYlp8.png)

#### 找到镜像加速器

![](https://s1.ax1x.com/2023/02/03/pSsYMff.png)

#### 配置使用

```
#1.创建一个目录
sudo mkdir -p /etc/docker

#2.编写配置文件
sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors": ["https://t2wwyxhb.mirror.aliyuncs.com"]
}
EOF

#3.重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Docker常用命令

1. 帮助命令

   ```
   docker version		#显示docker的版本信息
   docker info			#显示docker系统信息，包括镜像和容器的数量
   docker 命令 --help   #帮助命令
   ```

2. 镜像命令

   ```
   docker images	#查看所有本地主机上的镜像，可以使用docker image ls代替
   docker search	#搜索镜像	
   docker pull		#下载镜像	docker image pull
   docker rmi		#删除镜像	docker image rm
   
   docker search mysql --filter=STARS=3000	#搜索收藏数量大于3000的mysql
   
   docker pull tomcat:8	#若不写tag，默认是latest
   
   docker rmi -f 镜像id	#删除指定id的镜像
   docker rmi -f $(docker images -aq)	#删除全部镜像
   ```

3. 容器命令

   ```
   docker run 镜像id			#新建容器并启动
   docker ps				#列出所有运行的容器
   docker rm 容器id			#删除指定容器
   docker start 容器id		#重启容器
   docker stop	容器id		#停止当前正在运行的容器
   docker kill 容器id		#强制停止当前容器
   ```

   ```
   #新建容器并启动
   docker run [可选参数] image | docker container run [可选参数] image
   #参数说明
   --name="Name"		#容器名字 tomcat1 tomcat2用于区分容器
   -d					#后台方式运行
   -it					#使用交互方式运行，进入容器查看内容
   -p					#指定容器的端口 -p 8080(宿主机):8080(容器)
   -P(大写)				#随机指定端口
   
   #测试，启动并进入容器
   [root@iz2zeak7sgj6i7hrb2g862z ~]# docker run -it centos /bin/bash
   [root@241b5abce65e /]# ls
   bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   [root@241b5abce65e /]# exit #从容器退回主机
   
   
   #列出所有运行的容器
   docker ps 命令			#列出当前正在运行的容器
   	-a，--all			#列出当前正在运行+历史运行过的容器
   	-n=?，--last int		#列出最近创建的?个容器
   	-q，--quiet			#只列出容器的编号
   	
   
   #退出容器
   exit					#容器直接退出
   ctrl +P +Q				#容器不停止退出
   
   
   #删除容器
   docker rm 容器id			#删除指定容器，强制删除正在运行的容器-rf
   docker rm -f $(docker ps -aq)		#删除所有的容器
   docker ps -a -q | xargs docker rm	#删除所有的容器
   
   
   #启动和停止容器的操作
   docker start 容器id		#启动容器
   docker restart 容器id		#重启容器
   docker stop 容器id		#停止当前正在运行的容器
   docker kill 容器id		#强制停止当前容器
   ```

4. 其它常用命令

   **后台启动命令**

   ```
   docker run -d centos
   docker ps
   
   CONTAINER ID      IMAGE       COMMAND    CREATED     STATUS   PORTS    NAMES
   
   #发现centos停止了；常见的陷阱，若没有前台进程时，docker发现没有应用就会自动停止
   #nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
   ```

   **查看日志**

   ```
   docker logs
   #显示日志
   -tf				#显示日志信息（一直更新）
   --tail number 	#需要显示日志条数
   docker logs -t --tail n 容器id	#查看n行日志
   docker logs -ft	容器id			#跟着日志
   
   ```

   **查看容器中进程信息ps**

   **查看镜像的元数据**

   ```
   #	命令	docker top 容器id
   #	命令	docker inspect	容器id
   
   ```

   **进入当前正在运行的容器**

   docker exec -it 容器id bashshell			#进入容器后开启一个新的终端，可以在里面操作（常用）

   docker attach 容器id								#进入容器正在执行的终端

   **从容器拷贝到主机上**

   ```
   docker cp 容器id:容器内路径	主机目的路径
   
   [root@iz2zeak7sgj6i7hrb2g862z ~]# docker ps
   CONTAINER ID     IMAGE    COMMAND     CREATED         STATUS       PORTS      NAMES
   56a5583b25b4     centos   "/bin/bash" 7seconds ago    Up 6 seconds      
   
   #1. 进入docker容器内部
   [root@iz2zeak7sgj6i7hrb2g862z ~]# docker exec -it 56a5583b25b4 /bin/bash
   [root@55321bcae33d /]# ls
   bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   
   #新建一个文件
   [root@55321bcae33d /]# echo "hello" > java.java
   [root@55321bcae33d /]# cat hello.java 
   hello
   [root@55321bcae33d /]# exit
   exit
   
   #hello.java拷贝到home文件加下
   [root@iz2zeak7sgj6i7hrb2g862z /]# docker cp 56a5583b25b4:/hello.java /home 
   [root@iz2zeak7sgj6i7hrb2g862z /]# cd /home
   [root@iz2zeak7sgj6i7hrb2g862z home]# ls -l	#可以看见java.java存在
   total 8
   -rw-r--r-- 1 root root    0 May 19 22:09 haust.java
   -rw-r--r-- 1 root root    6 May 22 11:12 java.java
   drwx------ 3 www  www  4096 May  8 12:14 www
   
   ```

   ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNDIxNDMxMzk2Mi5wbmc?x-oss-process=image/format,png)

### 容器数据卷

如果数据都在容器中，那么容器删除数据就会丢失；此时容器之间需要有一个数据共享的技术，Docker中产生的数据同步到本地；**容器的持久化和同步操作，容器间实现数据共享**

卷技术：目录的挂载，将容器内的目录挂载到Linux上面

#### 方式一：直接使用命令挂载-v

```
docker run -it -v 主机目录:容器内目录  -p 主机端口:容器内端口
docker inspect 容器id

```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTE5MTY0Nzk3MC5wbmc?x-oss-process=image/format,png)

**测试文件的同步**（停止容器，在宿主机中修改文件，启动容器，数据依然同步）

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTE5MTcxODQ3MC5wbmc?x-oss-process=image/format,png)

##### 安装MySQL挂载连接测试

**思考：MySQL的数据持久化的问题**

```
#获取mysql镜像
docker pull mysql:5.7

#启动mysql并做数据挂载
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
--name 容器名字

docker run -d -p 主机端口:容器内端口 -v 主机配置文件路径:/etc/mysql/conf.d -v 主机数据文件路径:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw --name 容器名字 mysql:tag

#此时在本地新建数据库时，容器也会创建对应的数据库；若将容器内mysql删除，挂载到本地的数据卷依旧没有丢失，实现了容器数据持久化功能

```

### 具名和匿名挂载

```
# 匿名挂载（在-v中只写了容器内的路径，没有写容器外的路径）
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 查看所有的volume(卷)的情况
docker volume ls
DRIVER              VOLUME NAME # 容器内的卷名(匿名卷挂载)
local               21159a8518abd468728cdbe8594a75b204a10c26be6c36090cde1ee88965f0d0
local               b17f52d38f528893dd5720899f555caf22b31bf50b0680e7c6d5431dbda2802c

# 具名挂载 -P:表示随机映射端口
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
# 查看所有的volume(卷)的情况
$ docker volume ls                  
DRIVER              VOLUME NAME
local               21159a8518abd468728cdbe8594a75b204a10c26be6c36090cde1ee88965f0d0
local               b17f52d38f528893dd5720899f555caf22b31bf50b0680e7c6d5431dbda2802c
local               juming-nginx #多了一个名字

```

所有的docker容器内的卷，未指定目录时都是在**/var/lib/docker/volumes/自定义的卷名/_data**下

**如果指定了目录，docker volume ls 是查看不到的**

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjExNDIzMTQzNS5wbmc?x-oss-process=image/format,png)

#### 区分三种挂载方式

```
# 三种挂载： 匿名挂载、具名挂载、指定路径挂载
-v 容器内路径			#匿名挂载
-v 卷名：容器内路径		  #具名挂载
-v /宿主机路径：容器内路径 #指定路径挂载 docker volume ls 是查看不到的

```

#### 拓展

```
# 通过 -v 容器内路径： ro rw 改变读写权限
ro #readonly 只读		rw #readwrite 可读可写
docker run -d -P --name nginx05 -v juming:/etc/nginx:ro nginx
docker run -d -P --name nginx05 -v juming:/etc/nginx:rw nginx

# ro 只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作！

```

### Dockerfile

**Dockerfile 就是用来构建docker镜像的构建文件**

镜像是一层一层的，脚本是一个个的命令，每个命令都是一层

```
# 文件中的内容： 指令(大写) + 参数
$ vim dockerfile1
    FROM centos 					# 当前镜像是以centos为基础的

    VOLUME ["volume01","volume02"] 	# 挂载卷的卷目录列表(多个目录)

    CMD echo "-----end-----"		# 输出一下用于测试
    CMD /bin/bash					# 默认走bash控制台
    
$ docker build -f dockerfile1 -t dockerfiletest/centos .
# -f 代表当前文件的地址（这里是当前目录下的dockerfile1）
# -t 代表target，指目标目录（dockerfiletest镜像名前不能加斜杠'/'）
# .	表示生成在当前目录下

```

#### 容器间的数据同步

```
# 创建docker01
docker run -it --name docker01 dockerfiletest/centos
# 查看docker01的内容
ls
bin  home   lost+found	opt   run   sys  var
dev  lib    media	proc  sbin  tmp  volume01
etc  lib64  mnt		root  srv   usr  volume02
# 不关闭该容器退出
ctrl + Q + P

# 创建docker02：并让docker02继承docker01
docker run -it --name docker02 --volumes-from docker01 dockerfiletest/centos
# 查看docker02内容
ls
bin  home   lost+found	opt   run   sys  var
dev  lib    media	proc  sbin  tmp  volume01
etc  lib64  mnt		root  srv   usr  volume02

#在docker01的volume01下创建docker01.txt；在docker02中查看文件已同步

# 新建一个docker03同样继承docker01，发现volume01卷下也有docker01.txt
docker run -it --name docker03 --volumes-from docker01 dockerfiletest/centos

# 删除docker01，发现docker02和docker03中没有删除

```

##### 实现多个mysql的数据共享

```
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql01 mysql:5.7

docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=root --name mysql02 --volumes-from mysql01 mysql:5.7

# 这个时候可以实现两个容器数据同步

```

**容器之间的配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止；一旦持久化到了本地，此时本地的数据是不会删除的**

#### DockerFile构建过程

##### 基础知识

1. 每个保留关键字（指令）都必须是大写字母
2. 执行顺序从上到下
3. #表示注释
4. 每个指令都会创建提交一个新的镜像层，并提交

##### DockerFile指令

```
FROM				#from：基础镜像，一切从这里开始构建
MAINTAINER			#maintainer：镜像作者，姓名+邮箱
RUN					#run：镜像构建时需要运行的命令
ADD					#add：步骤，tomcat镜像，这个tomcat压缩包！添加内容 添加同目录
WORKDIR				#workdir：镜像的工作目录
VOLUME				#volume：挂载的目录
EXPOSE				#expose：保留端口配置
CMD					#cmd：指定该容器启动时要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT			#entrypoint：指定容器启动时运行命令，可以追加命令
ONBUILD				#onbuild：构建被继承DockerFile时运行onbuild
COPY				#copy：类似ADD，将文件拷贝到镜像中
ENV					#env：构建时设置环境变量

```

![](https://s1.ax1x.com/2023/02/03/pSsY11S.png)

##### 镜像发布

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjE3MTE1NTY2Ny5wbmc?x-oss-process=image/format,png)

### Docker网络

#### 理解Docker 0

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTIyMzIzNjc3Mi5wbmc?x-oss-process=image/format,png)

#### Docker容器互联

docker 有一个连接系统允许将多个容器连接在一起，共享连接信息，其会创建一个父子关系，父容器可以看到子容器的信息

##### 容器创建

`docker run -d -P --name coonote training/webapp python app.py`

##### 新建网络

`docker network create -d bridge test-net`

![](https://s1.ax1x.com/2023/03/20/ppNM5Hf.png)

##### 连接容器

运行一个容器并连接到新建的 test-net 网络

`docker run -itd --name test1 --network test-net ubuntu /bin/bash`

再运行一个容器并加入到 test-net 网络

`docker run -itd --name test2 --network test-net ubuntu /bin/bash`

![](https://s1.ax1x.com/2023/03/20/ppNMTUS.png)

##### 测试连接

```bash
apt-get update
apt install iputils-ping
#在 test1 容器输入以下命令
ping test2

```

##### 配置DNS

可以在宿主机的`/etc/docker/daemon.json`文件中增加以下内容来设置全部容器的 DNS：

```json
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}

```

配置完后重启docker才生效，查看容器的DNS信息：

`/etc/init.d/docker restart`

`docker run -it --rm ubuntu cat etc/resolv.conf`

###### 手动指定容器的配置

`docker run -it --rm -h host_ubuntu  --dns=114.114.114.114 --dns-search=test.com ubuntu`

- `--rm`：容器退出时自动清理内部的文件系统
- `-h 或 --hostname`：设定容器主机名，会被写到容器内的`/etc/hostname`和`/etc/hosts`
- `--dns`：添加 DNS 服务器到容器的 `/etc/resolv.conf` 中，让容器用这个服务器来解析所有不在 `/etc/hosts`中的主机名
- `--dns-search`：设定容器搜索域，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com

若容器启动时未指定 **--dns** 和 **--dns-search**，会默认用宿主主机上的`/etc/resolv.conf`来配置容器的 DNS

#### Docker网络实现

在本地主机和容器内分别创建一个虚拟接口，并让它们彼此连通

##### 创建网络参数

Docker创建一个容器时执行的操作：

- 创建一对虚拟接口，分别放在本地主机和新容器中
- 本地主机一端桥接到默认的docker0或指定网桥上，并具有一个唯一的名字
- 容器一端放到新容器中并修改名字为eth0，该接口只在容器的名字空间可见
- 从网桥可用地址段中获取一个空闲地址分配给容器的eth0，并配置默认路由到桥接网卡

完成这些之后，容器就可以使用eth0虚拟网卡连接其他容器和其他网络

也可以在`docker run`时通过`--net`指定容器的网络配置：

- `--net=bridge`：默认值，连接到默认的网桥
- `--net=host`：不将容器网络放到隔离的`namespace`中，此时容器使用本地主机的网络，若进一步使用`--privileged=true`，容器会被允许直接配置主机的网络堆栈
- `--net=container:NAME_or_ID`：将新建容器的进程放到一个已存在容器的网络栈中，新容器进程有自己的文件系统、进程列表和资源限制，但会和已存在的容器共享IP地址和端口等网络资源，两者进程可以直接通过 lo环回接口通信
- `--net=none`：将新容器放到隔离的网络栈中，但是不进行网络配置，之后由用户自己配置

##### 网络配置细节

首先启动一个 /bin/bash 容器，指定 --net=none 参数

`sudo docker run -i -t --rm --net=none base /bin/bash`

在本地主机查找容器进程id，并为它创建网络命名空间

```bash
$ sudo docker inspect -f '{{.State.Pid}}' 63f36fc01b5f
2778
$ pid=2778
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid

```

检查桥接网卡的IP和子网掩码信息

```bash
$ ip addr show docker0
21: docker0: ...
inet 172.17.42.1/16 scope global docker0
...

```

创建一对 “veth pair” 接口 A 和 B，绑定 A 到网桥 docker0，并启用它

```bash
$ sudo ip link add A type veth peer name B
$ sudo brctl addif docker0 A
$ sudo ip link set A up

```

将B放到容器的网络命名空间，命名为 eth0，启动它并配置一个可用 IP（桥接网段）和默认网关

```bash
$ sudo ip link set B netns $pid
$ sudo ip netns exec $pid ip link set dev B name eth0
$ sudo ip netns exec $pid ip link set eth0 up
$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0
$ sudo ip netns exec $pid ip route add default via 172.17.42.1

```

### Docker数据卷

即可供一个或多个容器使用的特殊目录，绕过UFS（UNIX文件系统），可提供：

- `数据卷`在容器间的共享和重用
- 对`数据卷`的修改会立马生效
- 对`数据卷`的更新不会影响镜像
- `数据卷`默认一直存在，即使容器被删除

> 数据卷的使用类似Linux下对目录或文件进行mount，镜像中被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）

#### 创建数据卷

`docker volume create my-vol`

#### 查看数据卷

`docker volume inspect my-vol`

#### 挂载数据卷

`docker run -d -P --name web -v my-vol:/usr/share/nginx/html nginx`

`docker inspect web`

```
		"Mounts": [
            {
                "Type": "volume",
                "Name": "my-vol",
                "Source": "/var/lib/docker/volumes/my-vol/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],

```

#### 删除数据卷

`docker volume rm my-vol`

### Docker Compose

实现对docker容器集群的快速编排，允许用户通过一个单独的 `docker-compose.yml` 模板文件定义一组相关联的应用容器为一个项目

`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义

`Compose` 默认管理对象是项目，通过子命令对项目中一组容器进行生命周期管理

#### 安装与卸载

##### Linux

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

```

##### window

可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在 Docker 容器中运行

##### bash命令补全

```bash
$ curl -L https://raw.githubusercontent.com/docker/compose/1.25.5/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

```

##### 卸载

若是二进制包安装的，删除二进制文件即可

`rm /usr/local/bin/docker-compose `

##### 测试安装成功

`docker-compose --version`

#### docker compose使用

```yaml
version: "3.0"
services:
  mysqldb:
    image: mysql:5.7.19
    container_name: mysql
    ports:
      - "3306:3306"
    volumes:
      - /root/mysql/conf:/etc/mysql/conf.d
      - /root/mysql/logs:/logs
      - /root/mysql/data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      - ems
    depends_on:
      - redis

  redis:
    image: redis:4.0.14
    container_name: redis
    ports:
      - "6379:6379"
    networks:
      - ems
    volumes:
      - /root/redis/data:/data
    command: redis-server
    
networks:
  ems:

```

`docker-compose up`：前台启动一组服务

每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中重复设置

- `build`：指定`dockerfile`所在文件夹的路径；也可使用`context`指定路径，使用`dockerfile`指定`dockerfile`文件名

  ```yaml
  build: ./dir
  # 或
  build:
        context: ./dir
        dockerfile: Dockerfile-alternate
        args:
          buildno: 1
  
  ```

- `command`：覆盖容器启动后默认执行的命令

- `container_name`：指定容器名称，默认使用`项目名称_服务名称_序号`格式

- `depends_on`：解决容器的依赖、启动先后问题，下列例子会先启动`redis db`再启动`web`

  ```yaml
  #web服务不会等待redis db完全启动后才启动
  services:
    web:
      build: .
      depends_on:
        - db
        - redis
  
    redis:
      image: redis
  
    db:
      image: postgres
  
  ```

- `env_file`：从文件中获取环境变量，如果有变量名称与 `environment` 指令冲突，以后者为准

- `environment` ：设置环境变量，可以使用数组或字典格式；只给定名称的变量会自动获取运行compose主机上对应变量的值

  ```yaml
  environment:
    RACK_ENV: development
    SESSION_SECRET:
  
  environment:
    - RACK_ENV=development
    - SESSION_SECRET
  
  ```

- `healthcheck`：通过命令检查容器是否健康运行

  ```yaml
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost"]
    interval: 1m30s
    timeout: 10s
    retries: 3
  
  ```

- `image`：指定为镜像名称或镜像 ID，若在本地不存在将会尝试拉取这个镜像

- `networks`：配置容器连接的网络

- `ports`：暴露端口信息，使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）

  > 当使用 HOST:CONTAINER 格式来映射端口时，若使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 YAML 会自动解析 xx:yy 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式

- `sysctls`：配置容器内核参数

- `ulimits`：指定容器的限制值

- `volumes`：数据卷挂载路径设置，可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）

  > 若路径为数据卷名称，必须在文件中配置数据卷

#### docker compose常用命令

`docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]`

##### 命令选项

- `-f，--file FILE`：指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定
- `-p，--project-name NAME`：指定项目名称，默认使用所在目录名称作为项目名
- `--x-networking`：使用docker的可拔插网络后端特性
- `--x-network-driver DRIVER`：指定网络后端的驱动，默认为`bridge`
- `--verbose`：输出更多调试信息
- `-v --version`：打印版本并退出

##### 命令

- `up`：`docker-compose up [options] [SERVICE...]`

  它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作

  链接的服务都将会被自动启动，除非已经处于运行状态

  默认情况启动的容器都在前台

  当通过 `Ctrl-C` 停止命令时，所有容器将会停止

  如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项

  默认情况下如果服务容器已经存在，将会尝试停止容器，然后重新创建，以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容

- `down`：停止`up`启动的容器，并移除网络

- `exec`：进入指定的容器

- `ps`：列出项目中目前所有的容器

  `-q`：只打印容器的ID信息

- `restart`：重启项目中的服务

  `-t`：指定重启前停止容器的超时

- `rm`：删除所有（停止状态的）服务容器

  `-f，--force`：强制删除

  `-v`：删除容器挂载的数据卷

- `start`：启动已存在的服务容器

- `stop`：停止已经处于运行状态的容器

  `-t，--timeout TIMEOUT`：停止容器时候的超时（默认为 10 秒）

- `top`：查看各个服务容器内运行的进程

- `unpause`：恢复处于暂停状态中的服务

## 实例

### 连接docker中的mysql

#### 主要步骤

##### Docker下载mysql

```
docker pull mysql

```

##### 启动mysql实例

```
docker run -d -p 13306:3306 -v /usr/local/mysql/conf:/etc/mysql/conf.d 
-v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name  mysql mysql


# --name为mysql的实例设置别名	-p 13306对外暴露的端口，3306内部端口
# -e MYSQL_ROOT_PASSWORD 初始化设置mysql登录密码 -d 表示后台运行
# 最后的mysql是镜像名称		-v为挂载（etc为配置文件，log为日志文件，lib为配置文件，将mysql容器内部文件挂载到linux中，每次变动都会在linux显示，而不用进入到容器内部查看）

```

##### 进入容器内部(mysql为容器名字)

```
docker exec -it mysql bash

```

##### 登录mysql

```
mysql -u root -p root

```

##### 创建用户授予权限

```
grant all on *.* to root@'%';
flush privileges;	#刷新权限

```

##### 防火墙

```
systemctl start firewalld.service	#开启防火墙
firewall-cmd --list-ports			#查看开启的端口号
firewall-cmd --zone=public --add-port=13306/tcp --permanent
#永久开启一个端口号
firewall-cmd --reload				#重启防火墙
firewall-cmd --list-ports			#再次查看端口号是否开启

```

##### 本地可以使用docker地址加暴露ip连接mysql

```
ip addr		#查看虚拟机ip地址，使用工具连接ip加端口

```

#### 常见问题

##### docker启动mysql失败

```
root@ecs-kc1-small-1-linux:~# docker run -p 3306:3306 mysql:8-oracle
2022-03-05 13:40:49+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.28-1.el8 started.
2022-03-05 13:40:50+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-03-05 13:40:50+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.28-1.el8 started.
2022-03-05 13:40:50+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD

```

**此问题的原因是启动命令中缺少密码，由于使用的是Linux的root用户，因此将启动命令修改为如下所示：**

```
docker run --name mysql -d -it -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql

```

### 连接docker中的elasticsearch7.6.2

#### 主要步骤

##### docker下载相关镜像

`docker pull elasticsearch:7.6.2`

##### 创建持久化文件

```
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data

```

##### 初始化配置

`echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml`

##### 启动镜像

```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms84m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.6.2

# -p:端口映射
# -e discovery.type=single-node 单点模式启动
# -e ES_JAVA_OPTS="-Xms84m -Xmx512m"：设置启动占用的内存范围（实验环境启动后可能因为云服务器内存过小而占满）
# -v 目录挂载
# -d 后台运行

```

##### 安装kibana

###### 下载同版本镜像

`docker pull kibana:7.6.2`

###### 初始化配置

```
mkdir -p /mydata/kibana
touch /mydata/kibana/kibana.yml
vim /mydata/kibana/kibana.yml

```

```yaml
server.host: 0.0.0.0
elasticsearch.hosts: http://你的ip:9200

```

###### 启动kibana

```
docker run --name kibana -v /mydata/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml -p 5601:5601 -d kibana:7.6.2 

```

#### 常见问题

##### 文件权限问题

elasticsearch启动后使用docker ps查看发现未正常启动，查看启动日志

`docker logs elasticsearch`

![](https://s1.ax1x.com/2023/02/17/pSqCzhn.png)

发现是文件拒绝访问异常,为该文件夹设置所有用户都有读写执行权限

`chmod -R 777 /mydata/elasticsearch/`

重新启动elasticsearch

`docker restart elasticsearch`

##### jdk版本过低

启动日志中也有可能报以下错误：

```
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.

```

原因是jdk版本过低，需要将jdk升级到9.0以上；去镜像网站下载jdk(这里是去华为云下载的)

```
cd /usr/local

wget https://repo.huaweicloud.com/java/jdk/9.0.1+11/jdk-9.0.1_linux-x64_bin.tar.gz

# 解压
tar -zxvf jdk-9.0.1_linux-x64_bin.tar.gz 

# 配置JAVA_HOME
vim /etc/profile
# 内容如下
export JAVA_HOME=/usr/local/jdk-9.0.1
export JRE_HOME=/usr/local/jdk-9.0.1/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

# 删除原jdk环境
# 此时执行java-version可能显示的还是jdk1.8
# 执行which java，会输出一个目录，删除之
# 执行which javac，也会输出一个目录，删除之
# 执行 ln -s $JAVA_HOME/bin/java /usr/bin/java
# 执行 ln -s $JAVA_HOME/bin/javac /usr/bin/javac
# 执行 source /etc/profile再次查看java版本即可

```

### 连接docker中的redis

#### 主要步骤

##### 拉取镜像

`docker pull redis`

##### 挂载配置文件

```
mkdir -p /home/redis/myredis
cd /home/redis/myredis
mkdir data
vim redis.conf

```

```
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
# bind 127.0.0.1
# redis.conf

protected-mode no
port 6379
tcp-backlog 511
requirepass 123456					#设置密码
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 30
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-disable-tcp-nodelay no
replica-priority 100
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly yes						#开启redis持久化
appendfilename "appendonly.aof"
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-max-len 128
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes

```

##### 启动redis容器

`docker run --restart=always --log-opt max-file=2 -p 6379:6379 --name myredis -v /home/redis/myredis/myredis.conf:/etc/redis/redis.conf -v /home/redis/myredis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes --requirepass 123456 `

##### 进入容器内部测试

`docker exec -it myredis redis-cli`

`auth 123456`

`ping`

#### 注意事项

在启动redis时不要修改防火墙，可能会出现报错