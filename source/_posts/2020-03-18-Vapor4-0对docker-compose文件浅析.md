title: Vapor4.0对docker-compose文件浅析
date: 2020-03-18 10:46:20
tags:
---
上篇对`docker-compose.yml`的某些理解有误，这篇对某些命令进行浅解。
```
docker build -f Dockerfile -t tblog .
```
那个`.`是指上下文路径，参见[这里](https://yeasy.gitbooks.io/docker_practice/image/build.html)-**镜像构建上下文(Context)**

```
docker-compose build
```
这个命令为啥先去运行`Dockerfile`里的命令呢？因为这个文件是用来构建`tblog:latest`镜像的。在`docker-compose.xml`中的首要任务就是构建`tblog:latest`， 并指定了build所需要的上下文, docker就找到上下文中的`Dockerfile`进行编辑了。就是下面这一段。
```
...
 app:
    image: tblog:latest
    build:
      context: .
...
```
`docker-compose.xml`最上面有一段：
```
volumes:
  db_data:
  ....
```
这个是干嘛的呢？这个是创建一个磁盘。`db_data:`应该就相当于我们说的盘符。没有给这个盘指定名称，它会有个默认的名称`tblog_db_data`, 也可以这样指定磁盘的名称。

```
volumes:
  db_data:
    name: "tuchangwei"
```
磁盘是用来存数据的，它不会消失。多个容器也可以共享这个磁盘。我们的数据库就把数据放在这个磁盘里面:
```

db:
    image: postgres:12.1-alpine
    volumes:
       - db_data:/var/lib/postgresql/data/pgdata

```
对于`docker-compose.xml`文件里命令的解释，可以参考[这里](https://docs.docker.com/compose/compose-file/#depends_on)
