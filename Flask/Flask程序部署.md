程序部署流程

### 部署方式

1. 官方推荐做法

添加setup.py文件，然后打包构建生成分发包，上传到pypi服务器。最后在远程主机通过pip命令安装。

2. 使用Git部署

大致流程：

> 1）在本地执行测试
>
> 2）将文件添加到Git仓库并提交（git add&git commit)
>
> 3）在本地将diamagnetic推送到代码托管平台（git push）
>
> 4）在远程主机上从代码托管平台复制程序仓库（git clone）
>
> 5）创建虚拟环境并安装依赖
>
> 6）创建示例文件夹，添加部署特定的配置文件或是擦黄健.env文件存储环境变量并导入
>
> 7）初始化程序数据库，创建迁移环境
>
> 8）使用Web服务器运行程序
>
> ![](.\res\deploy\gitdeploy.png)
>
> ——以上内容应用于《Flask Web开发实战：入门、进阶与原理解析》

3. 使用容器技术部署（如Docker）

### 部署到Linux服务器步骤

1. 使用xshell登录到远程服务器

2. 推送代码并初始化程序环境

   1）首先是从Git服务器复制仓库到远程主机

   ```shell
   $ git clone https://github.com/username/repository.git
   ```

   2）接着是创建虚拟环境、安装依赖并激活虚拟环境

   3）将敏感信息定义在.env文件中，程序将从中读取敏感信息

   3）创建数据库迁移环境并升级数据库

   

   ```shell
   $ flask db initdb
   $ flask db upgrade
   ```

3. 使用Gunicorn运行程序
   1）安装Gunicorn

   ```shell
   $ pip install gunicorn
   ```

   ​     Gunicorn使用下面的命令来运行一个WSGI程序：

   ```shell
   $ gunicorn[OPTION] 模块名:变量名
   ```

   ​     这里的变量名即要运行的WSGI可调用对象，也就是我们使用Flask创建的程序实例，而模块名即包含程序实例的模块

4. 使用Ngix提供反向代理

   像Gunicorn这类WSGI服务器内置了Web服务器，所以我们不需要Web服务器叶可以与客户端交换数据，处理请求和响应。但内置的Web服务器不够健壮，虽然程序已经可以运行，但更流行的部署方式是使用同一个常规的Web服务器为WSGI提供反向代理。

   这里选择Nginx，首先安装Nginx：

   ```shell
   $ sudo apt install nginx
   ```

5. 使用Supervisor管理进程

   直接使用命令来运行Gunicorn并不十分可靠。我们需要一个工具来自动在后台运行它，同时监控他的运行状况，并在系统出错或是重启时自动重启程序。

   安装Supervisor：

   ```shell
   $ sudo apt install supervisor
   ```

   

