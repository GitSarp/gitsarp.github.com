[使用教程](https://yeasy.gitbooks.io/docker_practice/image/list.html)
### dokcer基础
- docker system df
可以列出虚拟机实际占用磁盘的大小，而docker images的空间和可能会很大
#### 容器操作
- docker run -d ....
-d后台**运行容器**，输入的结果不会打印到宿主机上，可以通过docker container ls查看容器信息，docker container logs查看输出信息
- docker container stop 来终止一个运行中的容器。
当 Docker 容器中指定的应用终结时，容器也自动终止。
终止状态的容器可以用 docker container ls -a查看
- docker container start 命令来启动终止状态的容器。
- docker exec -it 容器名 bash
进入容器
- docker container rm  容器名  删除终止状态的容器加-f删除运行中的容器
- docker container prune删除所有运行态容器

### 部署Web项目到docker
1. 下载镜像
docker pull tomcat[:version]
docker images列出所有已安装镜像

2. 运行容器测试
docker run -it --rm -p 8888:8080 tomcat[:version]
-i：交互式操作，-t: 终端
–rm参数表示container结束时,Docker会自动清理其所产生的数据。
http://localhost:8888访问


3. 创建容器，将服务器或本地目录(含war包)映射到镜像目录(/usr/Downloads)
docker run --name dockertomcat(容器名) -p 8888:8080 -d -v /Users/feraxjj/Strom/dockerwar:/usr/Downloads tomcat[:version]
 
4. 进入创建的容器dockertomcat
docker exec -it dockertomcat /bin/bash

5. 将war包复制到镜像的tocat目录
cp /usr/Downloads/dockertomcat.war /usr/local/tomcat/webapps/

6. 关闭
exit退出，docker stop dockertomcat关闭容器

7. 保存镜像，提交到自己的docker账户下
docker commit -a "freaxjj" -m "from tomcat latest,with a maven simple webapp" dockertomcat dockerdockermeta/helloworldwebapp:0.0.1
docker images可以看到多出刚刚创建的镜像
docker login登录账户，docker push helloworldwebapp:0.0.1就可以将镜像上传了(如果镜像的名字不是已docker ID开头的话会报错)