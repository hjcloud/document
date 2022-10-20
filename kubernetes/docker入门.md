# docker 入门

## 容器技术

将应用程序与其所有必要文件和运行时环境封装为一个隔离的容器的技术，该容器可在任何操作系统任何环境下运行，能够屏蔽环境差异。类比港口码头的集装箱，操作系统为港口码头。程序的表现只与集装箱有关，和放在哪个港口码头无关。

优势：

1. 环境隔离，即build once, run everywhere，一处构建，到处运行
2. 轻量级
3. 快速部署与拆卸

传统虚拟机的方式也可实现，但虚拟机需要装载操作系统，操作系统极为笨重，额外占用很多磁盘和内存，启动销毁缓慢。而容器只隔离应用程序的运行时环境(程序运行依赖的各种库和配置)，共享同一个操作系统。

## docker

docker 是容器技术的一种实现，是一个 go 语言开源项目。

### 基本概念

- image：镜像，软件、配置环境、文件系统的整体打包。
- container：容器，镜像运行起来的程序实例。容器实质是进程，但拥有独立的命名空间。
- repository：仓库，存储镜像的仓库。

### 安装方式

[Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)

### Dockerfile

Dockerfile 是用于构建镜像的文本文件。

#### 指令介绍

##### FROM

功能：指定基础镜像，必须是**第一条指令**。

语法：

```dockerfile
FROM scratch  // 没有基础镜像
FROM <image>
FROM <image>:<tag>
FROM <image>:<digest>
// 没有 tag 或 digest 默认为 latest
```

##### COPY

功能：从上下文目录中复制文件到指定路径，src 为目录时指挥复制目录下文件。

语法：

```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

##### ADD

功能：将文件复制到指定路径。

与 COPY 区别：复制压缩文件能自动解压到目标路径，可以从远端 URL 复制文件。相同需求推荐使用 COPY。

语法：

```dockerfile
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

##### RUN

功能：在当前镜像层运行指定命令，**docker build 时运行**。每执行一次都会新建一层镜像，即镜像嵌套。注意避免过多无意义的层（可使用 && 符号连接命令）。

语法：

```dockerfile
RUN <command>
RUN ["executable", "param1", "param2"]
```

##### CMD

功能：容器启动时的默认命令或参数，**docker run 时运行**，若 docker run 时指定了命令，则该指令被覆盖。存在多个 CMD 指令仅最后一个生效。

语法：

```dockerfile
CMD <command>
CMD ["executable", "param1", "param2"]
CMD ["param1", "param2"]  // 为 ENTRYPOINT 指令指定的程序提供默认参数
```

##### ENTRYPOINT

功能：类似 CMD 指令，但不会被 docker run 指定的命令覆盖，且 docker run 指定命令会被当做参数传递给 ENTRYPOINT 指定的程序。可通过 docker run --entrypoint 选项覆盖 ENTRYPOINT 指令。存在多个ENTRYPOINT 指令仅最后一个生效。

语法：

```dockerfile
ENTRYPOINT <command>
ENTRYPOINT ["executable", "param1", "param2"]
```

##### ENV

功能：设置环境变量。可以在后续指令使用 $key 或 ${key} 进行引用。可以使用 docker inspect 查看容器环境变量。

语法：

```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```

##### ARG

功能：同 ENV，但作用域只在 Dockerfile 生效，构建好之后不存在该环境变量。可以在 docker build 时指定变量值，--build-arg [key=value]。

语法：

```dockerfile
ARG <name>[=<default value>]
```

##### USER

功能：指定执行后续命令的用户和用户组。

语法：

```dockerfile
USER <UID>[:<GID>]
```

### 镜像管理

#### docker login

功能：登录一个 Docker 镜像仓库，如果未指定地址，默认未官方仓库 Docker Hub。

语法：

```
docker login [OPTIONS] [SERVER]
```

OPTIONS：

* -u, --username string：用户名
* -p, --password string：密码

#### docker logout

功能：登出一个 Docker 镜像仓库，如果未指定地址，默认未官方仓库 Docker Hub。

语法：

```
docker logout [SERVER]
```

#### docker pull

功能：从镜像仓库拉取或者更新指定镜像。

语法：

```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

OPTIONS：

* -a, --all-tags：拉取所有 tagged 镜像
* --disable-content-trust：忽略镜像的校验，默认开启
* --platform string：如果服务支持多平台则设置平台
* -q, --quiet：安静模式，禁止详细输出

#### docker push

功能：将镜像上传到镜像仓库，需要先登录镜像仓库。

语法：

```
docker push [OPTIONS] NAME[:TAG]
```

OPTIONS：

* -a, --all-tags：拉取所有 tagged 镜像
* --disable-content-trust：忽略镜像的校验，默认开启
* -q, --quiet：安静模式，禁止详细输出

#### docker search

功能：从 Docker Hub 查找镜像

语法：

```
docker search [OPTIONS] TERM
```

* -f, --filter filter：根据条件过滤输出
* --format string：使用 Go 模板格式化输出
* --limit int：搜索结果的最大数目，默认 25
* --no-trunc：不截断输出

#### docker images

功能：列出本地镜像。

语法：

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

* -a, --all：列出所有镜像，默认隐藏中间层镜像。
* --digests：显示镜像摘要信息。
* -f, --filter filter：根据条件过滤输出。
* --format string：使用 Go 模板格式化输出
* --no-trunc：不截断输出
* -q, --quiet：只显示镜像 ID

#### docker rmi

功能：删除一个或多个镜像

语法：

```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

* -f, --force：强制删除
* --no-prune：不移除该镜像的过程镜像，默认移除。

#### docker tag

功能：标记本地镜像，将其归入某一仓库

语法：

```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

#### docker build

功能：使用 Dockerfile 创建镜像。

语法：

```
docker build [OPTIONS] PATH | URL
```

OPTIONS：

* -f：指定要使用的 Dockerfile 路径。
* -tag 或 -t：指定镜像的名字和标签，一般为 name:tag 或 name。

注意：PATH | URL 表示本次执行的上下文路径，PATH 一般为 . 表示当前上下文。在 docker build 构建时会将该路径下的所有内容打包给 docker 服务端引擎使用。避免构建缓慢，上下文路径不要放无用的文件。

#### docker history

功能：显示指定镜像的创建历史。

语法：

```
docker history [OPTIONS] IMAGE
```

* --format string：使用 Go 模板格式化输出
* -H, --human：以可读形式显示大小和日期，默认开启。
* --no-trunc：不截断输出
* -q, --quiet：只显示镜像 ID

#### docker save

功能：将指定镜像保存为 tar 归档文件

语法：

```
docker save [OPTIONS] IMAGE [IMAGE...]
```

OPTIONS：

* -o, --output string：输出到的文件，默认为 STDOUT

#### docker load

功能：从一个 tar 归档文件加载镜像

语法：

```
docker load [OPTIONS]
```

OPTIONS：

* -i, --input string：指定加载的文件，默认 STDIN
* -q, --quiet：精简输出信息

#### docker import

功能：从归档文件创建镜像

语法：

```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

OPTIONS：

* -c, --change list：应用 docker 指令创建镜像
* -m, --message string：提交信息
* --platform string：如果服务支持多平台则设置平台

查看本机镜像：

```
docker images
```

查找镜像：

```
docker search <image>
```

删除镜像：

```
docker rmi <image>
```

设置镜像标签：

```
docker tag <image_id> <image>:<tag>
```

### 容器管理

#### docker run

功能：启动容器

语法：

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

OPTIONS：

* -d：后台运行容器，并返回容器id
* -i：以交互模式运行容器，通常与 -t 同时使用
* -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用
* --name string：为容器指定一个名称
* -p：指定端口映射，**主机(宿主)端口:容器端口**
* -P：随机端口映射，容器内部端口随机映射到主机端口
* -v, --volume：绑定一个卷

#### docker stop

功能：停止一个运行中的容器

语法：

```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

#### docker start

功能：启动已经被停止的容器

语法：

```
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

#### docker restart

功能：重启容器

语法：

```
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

#### docker kill

功能：杀掉一个或者多个运行中的容器

语法：

```
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

OPTIONS：

* -s, --signal string：向容器发送的信号，默认 KILL

#### docker rm

功能：删除一个或者多个容器

语法：

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

OPTIONS：

* -f, --force：强制删除一个运行中的容器（使用 SIGKILL 信号）
* -l, --link：移除容器间的网络连接，而非容器本身
* -v, --volume：删除与容器关联的匿名卷

#### docker pause

功能：暂停容器中所有的进程

语法：

```
docker pause CONTAINER [CONTAINER...]
```

#### docker unpause

功能：恢复容器中所有的进程

语法：

```
docker unpause CONTAINER [CONTAINER...]
```

#### docker create

功能：创建一个新的容器但不启动

语法：同 docker run

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

#### docker exec

功能：在运行的容器中执行一条命令

语法：

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

OPTIONS：

* -d, --detach：分离模式：在后台运行命令
* -i, --interactive：即使没有附加也保持 STDIN 打开
* -t, --tty：分配一个伪终端

#### docker ps

功能：列出容器

语法：

```
docker ps [OPTIONS]
```

OPTIONS：

* -a, --all：显示所有容器（默认只显示运行中的容器）
* -f, --filter filter：根据条件过滤输出
* --format string：使用 Go 模板输出容器
* -n, --last int：显示最近创建的 n 个容器
* -l, --latest：显示最近创建的容器
* --no-trunc：不截断输出
* -q, --quiet：只显示容器ID
* -s, --size：显示总的文件大小

#### docker inspect

功能：返回 Docker 对象（容器/镜像）的底层信息

语法：

```
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

OPTIONS：

* -f, --format string：使用给定的 Go 模板格式化输出
* -s, --size：如果是容器则显示总文件大小
* --type string：为指定类型返回 JSON

#### docker top

功能：显示容器中运行的进程信息

语法：

```
docker top CONTAINER [ps OPTIONS]
```

#### docker attach

功能：将本地的标准输入、输出、错误流附加到正在运行的容器。即连接到正在运行的容器。

语法：

```
docker attach [OPTIONS] CONTAINER
```

OPTIONS：

* --detach-keys string：覆盖用于分离容器的键序列
* --no-stdin：不附加到标准输入
* --sig-proxy：将所有接收信号代理到进程（默认 true）

#### docker events

功能：从服务器获取实时事件

语法：

```
docker events [OPTIONS]
```

OPTIONS：

* -f, --filter filter：根据条件过滤输出
* --format string：根据 Go 模板格式化输出
* --since string：显示指定时间戳后的所有事件
* --until string：流式显示事件直到指定时间戳

#### docker logs

功能：获取容器的日志

语法：

```
docker logs [OPTIONS] CONTAINER
```

OPTIONS：

* --details：显示提供给日志的额外详细信息
* -f, --follow：跟踪日志输出
* --since string：显示某个时间开始（e.g. 2013-01-02T13:23:37Z）或最近时间（e.g. 42m for 42minutes）的所有日志
* -n, --tail string：显示最新的 n 条日志（默认all）
* -t, --timestamps：显示时间戳
* --until string：显示日志直到某个时间（e.g. 2013-01-02T13:23:37Z）或持续多少时间（e.g. 42m for 42 minutes）

#### docker wait

功能：阻塞容器直到容器停止，并打印退出代码

语法：

```
docker wait CONTAINER [CONTAINER...]
```

#### docker export

功能：导出一个容器的文件系统作为 tar 归档文件到 STDOUT

语法：

```
docker export [OPTIONS] CONTAINER
```

OPTIONS：

* -o, --output string：写入指定文件代替 STDOUT

#### docker port

功能：列出容器的端口映射或特定映射

语法：

```
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

#### docker stats

功能：显示容器资源使用信息的实时流，包括 CPU、内存、网络 I/O 等。

语法：

```
docker stats [OPTIONS] [CONTAINER...]
```

OPTIONS：

* -a, --all：显示所有容器（默认只显示运行中的容器）
* --format string：使用 Go 模板打印输出
* --no-stream：不流式显示信息，只展示当前状态
* --no-trunc：不截断输出

