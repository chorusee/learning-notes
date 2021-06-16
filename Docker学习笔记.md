## Docker学习笔记

### Docker能做什么

虚拟机技术缺点：

1. 资源占用十分多
2. 冗余步骤多
3. 启动很慢

**容器化技术不是模拟一个完整的操作系统**

Docker和虚拟机技术的不同：

+ 传统虚拟机，虚拟出一系列硬件，运行一个完整的操作系统，然后再这个系统上安装和运行软件
+ 容器内的应用程序直接运行再宿主机上，容器没有自己的内核，也没有虚拟硬件，所以轻便
+ 每个容器是相互隔离的，每个容器内都有自己的文件系统，互不影响

Docker能做什么：

+ 更快捷的交付和部署
+ 更便捷的升级和扩缩容
+ 更简单的运维系统
+ 更高效的计算资源利用

### 关键概念

#### 镜像

Docker镜像就好比一个模板，重复用这个模板来创建容器服务。通过这个镜像可以创建多个容器。

#### 容器

Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建。

基本命令：启动、停止、删除

目前可以把容器理解为一个简易的Linux系统。

#### 仓库

仓库就是存放镜像的地方。

仓库分为共有仓库和私有仓库。

### Docker的运行原理

Docker是怎么工作的？

Docker是一个Client-Server结构系统，Docker的守护进程运行再主机上，通过Socket客户端访问。

Docker接收到Docker-Client命令就执行。

Docker为什么快？

1. Docker有着比虚拟机更少的抽象层。
2. Docker利用的是宿主机的内核，VM需要Guest OS，所以说，新建一个容器的时候，Docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机要加载Guest OS，Docker利用的是宿主机的OS。

### Docker常用命令

#### 帮助命令

```bash
docker version  ## 显示Docker版本信息
docker info     ## 显示Docker系统信息，包括镜像容器数量
docker 命令 --help  ## 帮助命令
```

#### 镜像命令

```bash
docker images  ## 查看镜像

# 解释
# REPOSITORY 镜像的仓库源
# TAG        镜像的标签
# IMAGE ID   镜像的ID
# CREATED    镜像创建的时间
# SIZE       镜像的大小

# 选项
-a, --all    # 列出所有镜像
-q, --quiet  # 只显示镜像的ID


# Docker搜索
docker search mysql
# 选项
-f, --filter  # 过滤


# 下载镜像，分层下载
docker pull mysql
# 等价于
docker pull docker.io/library/mysql:latest

# 指定版本
docker pull mysql:5.7

# 删除镜像
docker rmi -f <ID>
# 删除多个
docker rmi -f <ID> <ID> ...
# 删除全部
docker rmi -f $(docker images -aq) # $()相当于将里面的命令输出传给rmi
```

#### 容器命令

```bash
# 新建容器并启动
docker pull centos
docker run [可选参数] image
# 参数说明
--name="Name"  # 容器名
-d             # 后台方式运行
-it            # 使用交互方式运行，进入容器内
-p             # 指定容器的端口
    -p 主机端口:容器端口
-P             # 随机指定端口

# 列出所有运行中的容器
docker ps
# 参数
-a         # 列出所有，包括历史运行过的容器
-n=数字     #显示最近创建的容器

# 退出容器
exit    # 停止容器并退出
Ctrl + P + Q   # 不停止容器退出

# 删除容器，不能删除正在运行的程序，如果要强制删除，加-f参数
docker rm <容器ID>

# 启动和停止容器
docker start <容器ID>
docker restart <容器ID>
docker stop <容器ID>      # 停止当前正在运行的容器
docker kill <容器ID>      # 强制停止当前容器
```

### 日志、元数据、进程的查看

```bash
# 查看日志
docker logs <容器ID>
# 可选参数
-f, --follow      # 
-t, --timestamp   # 带上时间戳
--tail            # 要显示的条数

# 查看容器中的进程信息
docker top [容器ID]

# 容器元数据
docker inspect [容器ID]

# 进入当前正在运行的容器
docker exec -it [容器ID] [bash]   # 进入容器后开启一个新的终端
docker attach [容器ID]   # 进入容器正在执行的终端

# 从容器拷贝文件到主机上
docker cp [容器ID]:[容器内的路径] [主机路径]
```

### 镜像原理

镜像是一种轻量级的，可执行的独立软件包，用来打包软件和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

如何得到镜像：

+ 从远程仓库下载
+ 拷贝
+ 自己制作一个DockerFile

#### Dokcer镜像加载原理

1. UnionFS（联合文件系统）

   UnionFS是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，UnionFS是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像，可以制作出各种具体的应用镜像。

   特性：一次性同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

2. Docker镜像加载原理

   Docker的镜像实际上是由一层一层的文件按系统组成，这种层级文件系统就是UnionFS。

   bootfs（boot file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker的最底层是bootfs。这一层与我们典型的Linux系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统会卸载bootfs。

   rootfs（root file system），在bootfs之上。包含的就是典型Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版本。

   对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令、工具和程序库就可以了。

#### commit镜像

```
docker commit -m="提交的信息描述" -a="作者" <容器ID> <目标镜像名>:[TAG]
```



### 容器数据卷

#### 什么是容器数据卷

Docker将应用和环境打包成一个镜像，而数据不应该在容器中，如果容器删除，数据就会丢失。

需求：数据可以持久化

容器之间可以有一个数据共享的技术，Docker容器产生的数据，同步到本地。

这就是卷技术。将我们容器内的目录挂载到Linux上。

#### 使用数据卷

1. 使用命令

   ```bash
   docker run -it -v [主机目录]:[容器内目录]
   ```

2. Dockerfile

   Dockerfile就是用来构建Docker的构建文件，本质上是一个命令脚本。

   ```bash
   # 匿名挂载
   VOLUME ["volume01", "volume02"]
   ```

#### 数据卷容器

```bash
docker run -it --name docker02 --volume-from docker01 mycentos
```

### Dockerfile

构建步骤：

1. 编写一个Dockerfile文件
2. `docker build`构建一个镜像
3. `docker run`运行镜像
4. `docker push`发布镜像

指令：

```bash
FROM            # 基础镜像，一切从这里开始构建
MAINTAINER      # 谁写的，姓名+邮箱
RUN             # 镜构建的时候要运行的命令
ADD             # 添加内容
WORKDIR         # 镜像的工作目录
VOLUME          # 挂载的目录
EXPOST          # 端口配置
CMD             # 指定容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT      # 指定容器启动的时候要运行的命令，可以追加
COPY            # 类似ADD，将我们的文件拷贝到镜像中
ENV             # 构建的时候设置环境变量
```



### Spring Boot应用打包Docker

```dockerfile
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
```





###  k8s

### 什么是k8s？

k8s是自动化容器操作的开源平台，这些操作包括部署、调度和结点集群间扩展。

k8s可以：

+ 自动化容器的部署和复制
+ 随时扩展或收缩容器规模
+ 将容器组织成组，并且提供容器间的负载均衡
+ 很容易地升级应用程序容器地新版本
+ 提供他容器弹性，如果容器失效就替换它

![1.png](http://dockone.io/uploads/article/20190625/d7ce07842371eab180725bab5164ec17.png)

#### Pod

包含一组容器和卷。同一个Pod里地容器共享一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。

#### Label

一个Pod有一些Label。一个Label是attach到Pod的一对键值对，用来传递用户定义的属性。

#### Replication Controller

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它。

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它

#### Service

如果Pod是短暂的，那么重启时IP地址可能会改变，怎么才能从前端容器正确可靠地指向后台容器呢？

[Service](http://kubernetes.io/v1.1/docs/user-guide/services.html)是定义一系列Pod以及访问这些Pod的策略的一层**抽象**。Service通过Label找到Pod组。因为Service是抽象的，所以在图表里通常看不到它们的存在，这也就让这一概念更难以理解。

#### Node

节点（上图橘色方框）是物理或者虚拟机器，作为Kubernetes worker，通常称为Minion。每个节点都运行如下Kubernetes关键组件：

- Kubelet：是主节点代理。
- Kube-proxy：Service使用其将链接路由到Pod，如上文所述。
- Docker或Rocket：Kubernetes使用的容器技术来创建容器。

#### Kubernetes Master

集群拥有一个Kubernetes Master（紫色方框）。Kubernetes Master提供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Replication Controller。