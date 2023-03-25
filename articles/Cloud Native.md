### Docker





#### Docker Compose

docker提供的轻量级编排服务

可以定义服务的启动步骤、批量处理、网络转发规则

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322232008538.png" alt="image-20230322232008538" style="zoom: 33%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233408286.png" alt="image-20230322233408286" style="zoom: 25%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233500393.png" alt="image-20230322233500393" style="zoom: 25%;" />



docker-compose的常用命令

在compose这个层面的粒度是比较粗的，做不到更加精细的定制

![image-20230322233231994](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230322233231994.png)



#### Dockerfile

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230323002327220.png" alt="image-20230323002327220" style="zoom:33%;" />

##### dockerfile关键字

- FROM

构建镜像基于哪个镜像

- MAINTAINER

镜像维护者姓名或邮箱地址

- RUN

构建镜像时运行的指令

- CMD

运行容器时执行的shell环境，它会被docker run后面的命令覆盖，需要很注意这一点

- VOLUME

指定容器挂载点到宿主机自动生成的目录或其他容器

- USER

为RUN、CMD、和 ENTRYPOINT 执行命令指定运行用户

- WORKDIR

为 RUN、CMD、ENTRYPOINT、COPY 和 ADD 设置工作目录，就是切换目录

- HEALTHCHECH

健康检查

- ARG

构建时指定的一些参数

- EXPOSE

声明容器的服务端口（仅仅是声明）

- ENV

设置容器环境变量

- ADD

拷贝文件或目录到容器中，如果是URL或压缩包便会自动下载或自动解压

- COPY

拷贝文件或目录到容器中，跟ADD类似，但不具备自动下载或解压的功能

- ENTRYPOINT

运行容器时执行的shell命令，不会像CMD一样被docker run后面的命令覆盖



##### 虚悬镜像 dangling images

在dockerfile编写过程当中，或者是docker run的时候没能正确设置name和tag的镜像将成为虚悬镜像

这类虚悬镜像将浪费资源

docker images prune命令可以删除所有的虚悬镜像



##### dockerfile案例

从一个精简版的centOS中构建一java环境

![image-20230323115934655](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230323115934655.png)

CMD echo "success build image"

CMD echo "end..."