# Cloud Native

云原生应用的最大特征是**“弹性”**，这种弹性的来源是容器。所以一句话概括云原生应用就是基于容器来构建具有弹性的大规模运应用集群。

<img src="https://jimmysong.io/kubernetes-handbook/images/cloud-native-core-target.jpg" alt="云原生的核心目标" style="zoom: 33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230509182801197.png" alt="image-20230509182801197" style="zoom: 50%;" />



更进一步的**Service Mesh（服务网格）**旨在基于Kubernetes这种平台进一步管理容器之间的网络连接，以及向外提供一套统一的网关入口，由**Service Mesh（例如Istio）**来进行负载均衡等网络方面的管理。更进一步地简化了服务的部署，使得开发者只需要专注在构建一个个原子的业务逻辑。像APM、Log这些工具也会由网格来进行管理。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230509184406500.png" alt="image-20230509184406500" style="zoom: 50%;" />





## Docker



### Container



#### 容器管理操作



docker logs [container_id] -f	查看容器的运行日志

```shell
$ docker logs [OPTIONS] CONTAINER
  Options:
        --details        显示更多的信息
    -f, --follow         跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string    从日志末尾显示多少行日志， 默认是all
    -t, --timestamps     显示时间戳
        --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）
```



docker inspect [container_id]	查看容器的基本信息

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```



#### 数据卷与文件夹挂载

--volume(-v)  source:destination

-v只能用来挂载bind类型，也就是文件夹对文件夹



--mount通过type参数既可以挂载卷（volume）也可以挂载文件夹

```shell
# 挂载文件夹
docker run --name $CONTAINER_NAME -it \-v $PWD/$CONTAINER_NAME/app:/app:rw \-v $PWD/$CONTAINER_NAME/data:/data:ro \avocado-cloud:latest /bin/bash

# 挂载volume
docker run --name $CONTAINER_NAME -it \--mount type=bind,source=$PWD/$CONTAINER_NAME/app,destination=/app \--mount type=volume source=${CONTAINER_NAME}-data,destination=/data,readonly \avocado-cloud:latest /bin/bash
```







### Docker Compose

docker提供的轻量级编排服务

可以定义服务的启动步骤、批量处理、网络转发规则

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322232008538.png" alt="image-20230322232008538" style="zoom: 33%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233408286.png" alt="image-20230322233408286" style="zoom: 25%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233500393.png" alt="image-20230322233500393" style="zoom: 25%;" />



docker-compose的常用命令

在compose这个层面的粒度是比较粗的，做不到更加精细的定制

![image-20230322233231994](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233231994.png)



#### Dockerfile

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

https://docs.docker.com/engine/reference/builder/#entrypoint

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230323002327220.png" alt="image-20230323002327220" style="zoom:33%;" />



##### FROM

构建镜像基于哪个镜像



##### MAINTAINER

镜像维护者姓名或邮箱地址



##### RUN

构建镜像时运行的指令，它是层次化的，每一行的新的RUN会在原RUN的基础上新开一个Command Line，所以只能用来执行前台任务，不要用RUN去挂任何后台任务，除非是最后一步的RUN

```shell
# shell 形式
RUN /bin/bash -c 'source $HOME/.bashrc && echo $HOME'

# exec 形式 这种形式不会开启command line 也就不会解析ENV和ARG
RUN ["/bin/bash", "-c", "echo hello"]
```



##### NTRYPOINT

运行容器时执行的shell命令，不会像CMD一样被docker run后面的命令覆盖

##### CMD

运行容器时执行的shell环境，它会被docker run后面的命令覆盖，需要很注意这一点

CMD和Entrypoint可以理解为都是在意在确保执行某些命令的，一个dockerfile中必须包括一个cmd或者entrypoint，推荐写的时候弄一个cmd就够了

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230508215340472.png" alt="image-20230508215340472" style="zoom: 33%;" />



##### VOLUME

指定容器挂载点到宿主机自动生成的目录或其他容器



##### USER

为RUN、CMD、和 ENTRYPOINT 执行命令指定运行用户



##### WORKDIR

为 RUN、CMD、ENTRYPOINT、COPY 和 ADD 设置工作目录，就是切换目录

`RUN cd /app`  `RUN echo "hello" > world.txt` 这两行放在一个dockerfile里面起不到理想中的结果，每一个RUN都是一个独立的shell进程，会从系统默认点开始工作

如果你的 `WORKDIR` 指令使用的相对路径，那么所切换的路径与之前的 `WORKDIR` 有关：

```shell
WORKDIR /a

WORKDIR b

WORKDIR c


RUN pwd // output  /a/b/c
```



##### HEALTHCHECH

健康检查



##### ENV

设置容器环境变量，无论是后面的其它指令，如 `RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量

```sh
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

##### ARG

构建时指定的一些参数，在容器运行时这些参数都会被销毁，具体的用法和上面的ENV一致



##### EXPOSE

声明容器的服务端口（仅仅是声明），在容器运行时并不会因为这个声明应用就会开启这个端口的服务在容器运行时并不会因为这个声明应用就会开启这个端口的服务，真正起作用仍旧需要通过 -P 参数设置端口映射



##### ADD

拷贝文件或目录到容器中，相较于COPY更加高级

它的源路径可以是URL并且会自动下载，但是如果下载的文件是压缩包，需要解压的的得额外使用RUN命令进行，总体来说还不如单独使用RUN命令来完成更加客制化的下载过程

综上，ADD实用性不强

##### COPY

拷贝文件或目录到容器中，跟ADD类似，但不具备自动下载或解压的功能，其中的源路径可以是多个，也可以是通配符；并且通过--chown参数也能修改

COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]



##### 虚悬镜像 dangling images

在dockerfile编写过程当中，或者是docker run的时候没能正确设置name和tag的镜像将成为虚悬镜像

这类虚悬镜像将浪费资源

docker images prune命令可以删除所有的虚悬镜像



##### dockerfile案例

从一个精简版的centOS中构建一java环境

![image-20230323115934655](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230323115934655.png)

CMD echo "success build image"

CMD echo "end..."



### Docker实现原理

Docker技术的基石就是操作系统的各种级别上的虚拟化，也就是云计算的那一套：SDN、SDS和KVM

![image-20230510110602536](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230510110602536.png)







### Cases



## Kubernetes

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230508155748637.png" alt="image-20230508155748637" style="zoom:50%;" />

![Deployment evolution](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)



### 网络



#### Ingress

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230510181149125.png" alt="image-20230510181149125" style="zoom: 67%;" />



```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend
spec:
  defaultBackend:
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
  rules:
    - http:
        paths:
          - path: /icons
            pathType: ImplementationSpecific # 此外还有exact和prefix匹配方式
            backend:
              resource:
                apiGroup: k8s.example.com
                kind: StorageBucket
                name: icon-assets
                
# wildcard 通配符匹配                
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```





#### Ingress Nginx Controller

Ingress Resources要在Controller下才能生效，后者可以看作是一个执行器

INC可以简单理解为能够执行Ingress Resources文件的Nginx定制版，它能够根据资源文件里的规则进行路由的转发与匹配



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230510183016169.png" alt="image-20230510183016169" style="zoom:67%;" />



![image-20230510183049063](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230510183049063.png)





#### Gateway



### 存储



### 资源管理



#### Pod管理