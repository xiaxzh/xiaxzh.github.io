---
layout: post
title: "Iter 1 Summary"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## 个人总结--夏显茁

一个神奇的Idea，我们开发的是中大活动小程序端。其中，因为我上学期使用过Golang，我被分配到了后台的任务。然而......我在写docker file、docker-compose。不过都不要紧，重要的是过程。小组第一次会议上，在讨论我们本预期开发的订餐系统时，成员之一的雨蓓提到要不我们把中大活动的活给做了，既可以完成作业，同时也可以为这个社团完成一些开发的工作。大家觉得靠谱，与其开发一个不可能会有人用的订餐系统，倒不如真刀真枪地做个有人用的东西。于是ActivityPlus+就产生了。

说说我负责的内容，docker，这里[有一篇很不错的文章](https://www.kancloud.cn/maozhenggang/docker-api/94298)，从基本的概念开始，到后续的一些集成，都说的很清晰。但如何安装docker, 我建议到docker官网上按教程一步一步来，保证无误而且不会有错。这里注意不要选错版本了，[docker CE(Community Edition)](https://docs.docker.com/install/linux/docker-ce/ubuntu/)，这是我们最常使用的版本，也是个人使用的版本，社区版。而另外的docker EE(Enterprise Edition)企业版，没有接触过。

按照上面所提供的文章装好docker以后，就开始编写docker file。

```dockerfile
# Build on golang:1.9
FROM golang:1.9

# Create a diretory to save our files
RUN mkdir -p /go/src/service-end
WORKDIR /go/src/service-end

# Copy source code
COPY . /go/src/service-end

RUN go-wrapper download && go-wrapper install

# Setting ENV Value
ENV PORT 8080

# Expose 8080 port to communicate with host

# Containner Begin
CMD ["go-wrapper", "run"]
```

如果每次要在不同的机器上使用docker build来重新创建镜像，会出现很多重复的工作量。而且与创建时机器的[网络环境有关(更换docker image source)](https://blog.csdn.net/zzy1078689276/article/details/77371782)。所以使用docker hub，该网站与github集成起来，使在github里的项目每被更新一次，便在[docker hub下自动生成一个新的镜像](https://blog.csdn.net/cszhouwei/article/details/41312671)并保存起来。以便在不同的机器下，直接拉取生成的镜像。

这样，镜像就制作好了。下面开始使用docker-compose进行实际的配置。

光有Go的镜像容器，在实战中是不够解决问题的，因为还要将数据持久化。此处我们采用[redis+mysql的方式](https://www.cnblogs.com/hellowzd/p/5163782.html)，来实现数据持久化。

实现的docker-compose.yml如下：

```dockerfile
# mysql listening on port 3306
# redis listening on port 6379
# service-end listening on port 8080
version: '2'

services:
  redis:
    image: redis
    volumes:
      - /server/data/production/redis:/data/
    networks:
      - own

  mysql:
    image: mysql:5.6
    volumes:
      - /server/data/production/mysql:/var/lib/mysql
    environment:
        MYSQL_ROOT_PASSWORD: root
    networks:
      - own

  service-end:
    image: xiaxzh/script
    environment:
      - DEVELOP=true
      - PORT=8080
      - DATABASE_ADDRESS=mysql
      - DATABASE_PORT=3306
      - REDIS_ADDRESS=redis
      - REDIS_PORT=6379
    links:
      - redis
      - mysql
    expose:
      - 8080
    networks:
      - own
```



为了同时维护稳定版的server和同时兼顾开发测试使用的server，在服务器机器上，我们使用了3个nginx来实现这一目的。最终形成的[暂稳定版本](https://github.com/sysu-SAAD-project/script)

![](https://raw.githubusercontent.com/xiaxzh/xiaxzh.github.io/master/images/server-glance.png)



最后，编写简单的shell来实现自动下载和运行

```shell
if [ ! -d "./docker-compose.yml" ];then
download="curl -s https://raw.githubusercontent.com/sysu-SAAD-project/script/master/docker-compose.yml -o docker-compose.yml"
$download
else
echo "file exits"
fi
run="docker-compose up -d"
$run
```

以上就是迭代1中，完成的任务。下一阶段，预期使用Travis CI来实现自动测试和集成。