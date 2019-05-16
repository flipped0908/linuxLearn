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


```