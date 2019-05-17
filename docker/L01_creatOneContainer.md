## 如何创建一个docker容器

docker 创建一个tomcate

一、拉去镜像

1. #：docker search tomcat
2. #：docker pull tomcat
3. #：docker images

二、通过DockerFile构建

1. $ mkdir -p ~/tomcat/webapps ~/tomcat/logs ~/tomcat/conf
  映射tomecat容器配置的应用程序目录
2. [创建tomcate 的 DockerFile](https://github.com/docker-library/tomcat/blob/9ecc2f5cbaaedbe74c91cc3ecf1bab5192e741a6/8.5/jre8/Dockerfile)
```
root@gsh:~/tomcat# ls
conf  DockerFile  logs  webapps
root@gsh:~/tomcat# docker build -t tomcatDf .
invalid argument "tomcatDf" for "-t, --tag" flag: invalid reference format: repository name must be lowercase
See 'docker build --help'.
root@gsh:~/tomcat# docker build -t tomcatdf .
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /root/tomcat/Dockerfile: no such file or directory
root@gsh:~/tomcat# ls
conf  DockerFile  logs  webapps
root@gsh:~/tomcat# mv DockerFile Dockerfile
root@gsh:~/tomcat# docker build -t tomcatdf .
Sending build context to Docker daemon  10.24kB
Step 1/20 : FROM openjdk:8-jre
8-jre: Pulling from library/openjdk
c5e155d5a1d1: Already exists
221d80d00ae9: Already exists
4250b3117dca: Already exists
```
> 过程中 tomcatDf DockerFile 中 因为大小写的原因造成了失败

## docker cmd
  $ docker exec -it [:name] /bin/bash  
  $ exit




