title: Vapor4.0对Dockerfile文件浅析
date: 2020-03-15 10:54:37
tags:
categories: Vapor4.0
---
昨天周六，被Docker中配置Vapor搞到脑袋要炸，差一点放弃Vapor的学习。但今天感觉还好，脑炸是源于对未知的恐惧。

本身相关的包依赖下载就非常的耗时，稍个不慎还要重新下载，简直奔溃。

我的想法是先建个模版项目发布一下，熟悉整个流程后，就边加功能边发布。因为要保持环境的一致性，Vapor社区比较推荐在Docker中部署，然后再把Docker部署到服务器上，模版项目中也提供了两个Docker相关的文件 `Dockerfile` 和 `docker-compose.yml`。但是网络上关于Docker部署Vapor4的资料简直是没有，看着两个文件中的命令就头大。脑中的问题一直在飞： 本地项目怎么部署到Docker容器中？是不是要先在容器中安装Swift和Postpostgres?部署的项目和postpostrges是同一个容器，还是两个容器？这两个容器怎么交流？本地项目做了更改后，Docker中的项目会自动更改吗？这些问题怎么用命令行解决？谁想谁脑炸。

正当我要放弃的时候，社区的温暖送来了🙏。
> Deploying with docker is quite easy. Assuming you want to deploy a fresh Vapor 4 project:

>Requirements: Install latest Xcode 11.4 Beta and switch to Swift 5.2 in the preferences. (You can check in the command line using swift --version) If you want to use Docker locally on your Mac, you should install Docker Desktop for Mac. 

>Run `vapor new --branch=4 myProject` to create a fresh project from template. (You can do any changes if you want, but the fresh template already provides a Hello World example.)

>To run your application within Docker, there is already a `web.Dockerfile` provided in the template. 
We'll have to build an image first. Run `docker build -f web.Dockerfile -t my-project .`  Of course you can use whatever tag (my-project) you like.
To test your image locally use `docker run --rm -p 8080:80 my-project` (We use --rm to delete the container when we stop the app. The flag -p 8080:80 exposes the port 80 from within the container to port 8080 on our host.) You will see something like [ NOTICE ] Server starting on http://0.0.0.0:80 in the command line. Visit http://localhost:8080/ in your browser and you should see "It works!" from your Vapor application.

>If you make changes to your application, stop the container, build and run it again.

所以什么都不用管，先在项目目录下输入如下命令:

```
docker build -f Dockerfile -t tblog .
```
项目就开始在Docker中安装了。注意命令后面那个`.`别忘了，那个是当前目录的意思。

完后我按照上面大佬的命令：
```
docker run --rm -p 8080:80 tblog
```
并没有把项目跑起来，这里可以从两处做一下更改。一个是把命令行改成：
```
docker run --rm -p 8080:8080 tblog
```
因为另一个大佬说docker默认暴露的是8080端口。或者在`Dockerfile`文件中把最后一行改为：
>CMD ["serve", "--env", "production", "--hostname", "0.0.0.0", "--port", "80"]

把80端口暴露出来。

`Dockerfile`里的命令大概解释一下，就是先在Docker容器里下载Swift5.2, 然后把本地的项目拷贝到容器里，然后编译。然后再把Ubuntu拷贝下来，把编译的文件放在Ubuntu的目录结构下。然后运行。

这个时候项目跑起来是没有数据库的, `docker-compose.yml`这个文件就有用了。

运行命令：
```
 docker-compose up db
```
会安装数据库postgres，并启动数据库。

运行命令：
```
 docker-compose up migrate
```
会在数据库中创建TODO这个表。

这个时候，你再访问localhost:8080/todos这个API程序就不会报错了。这个数据库是建在另外一个容器里的。我们既可以通过运行命令：
```
docker-compose up app
```
让前一个容器里的app启动，然后通过localhost:8080/todos去访问数据库，也可以直接在Xcode中运行app，然后通过这个URL去访问数据库。数据库是同一个，但是跑起来的app却是两个。Xcode跑的那个是开发用的。容器里的这个用来发布的。


如果运行命令：
```
docker-compose build
```
它会执行`Dockerfile`里的命令。也就是说和前面docker build的命令差不多。

这些命令在`docker-compose.yml`中都有说明。

最后我发现原来docker在国内有镜像，可以加速下载images。具体方法看[这里](https://yeasy.gitbooks.io/docker_practice/install/mirror.html)。这个博客很不错，回头看看把Docker的知识补一补。

文中关于Docker的解释，我半靠理解半靠猜，但无论如何app是跑起来了，数据库也通了。后面慢慢练手，慢慢填坑。




