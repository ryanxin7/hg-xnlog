![image-20231128163312010](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231128163312010.png)



## tomcat 基础镜像

```dockerfile
# Use a base image with Java 17 and Tomcat
FROM harbor.ceamg.com/baseimages/jdk17_0.9:x1

# Tomcat version to be installed
ENV TOMCAT_VERSION 10.1.16

COPY ./apache-tomcat-10.1.16.tar.gz /tmp/

# 在容器中解压缩Tomcat文件
RUN tar -xf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /opt/ && \
    mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
    rm apache-tomcat-${TOMCAT_VERSION}.tar.gz


# Set environment variables
ENV CATALINA_HOME /opt/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH


COPY ./index.html /$CATALINA_HOME/webapps/ROOT

# Expose the port that Tomcat runs on
EXPOSE 8080

# Set the working directory inside the container
WORKDIR $CATALINA_HOME

# Start Tomcat
CMD ["catalina.sh", "run"]
```





```bash
root@harbor01[14:40:49]/dockerfile/tomcat17 #:sh build_image_command.sh x1
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  12.49MB
Step 1/10 : FROM harbor.ceamg.com/baseimages/jdk17_0.9:x1
 ---> 2831b54b6401
Step 2/10 : ENV TOMCAT_VERSION 10.1.16
 ---> Using cache
 ---> b08960efe9a8
Step 3/10 : COPY ./apache-tomcat-10.1.16.tar.gz /tmp/
 ---> Using cache
 ---> 7d3ddba62f34
Step 4/10 : RUN tar -xf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /opt/ &&     mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat &&     rm -rf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz
 ---> Running in ab9db765a511
Removing intermediate container ab9db765a511
 ---> be00e79b1f2d
Step 5/10 : ENV CATALINA_HOME /opt/tomcat
 ---> Running in 8428bb9c55e7
Removing intermediate container 8428bb9c55e7
 ---> 75ee07ece0e3
Step 6/10 : ENV PATH $CATALINA_HOME/bin:$PATH
 ---> Running in fbe785a46542
Removing intermediate container fbe785a46542
 ---> c9b94571fb95
Step 7/10 : COPY ./index.html /$CATALINA_HOME/webapps/
 ---> 4c1b4b949844
Step 8/10 : EXPOSE 8080
 ---> Running in c66aafe5856d
Removing intermediate container c66aafe5856d
 ---> 466e08e303eb
Step 9/10 : WORKDIR $CATALINA_HOME
 ---> Running in 2a6b3b867c19
Removing intermediate container 2a6b3b867c19
 ---> 79cf969913dc
Step 10/10 : CMD ["catalina.sh", "run"]
 ---> Running in d465b3338a80
Removing intermediate container d465b3338a80
 ---> 3b7659ab3a5c
Successfully built 3b7659ab3a5c
Successfully tagged harbor.ceamg.com/baseimages/webapp-tomcat:x1
The push refers to repository [harbor.ceamg.com/baseimages/webapp-tomcat]
83f5ae443ab3: Pushed
ead28053bfb3: Pushed
a256fbd2326d: Pushed
c807951d4c31: Mounted from baseimages/jdk17_0.9
b524ae647412: Mounted from baseimages/jdk17_0.9
6515074984c6: Mounted from baseimages/jdk17_0.9
x1: digest: sha256:c0fae36668191a3950f37e6a54dbacb090b5ccddaef16e8c74083df9c534c4ac size: 1580
```





```bash
## 测试运行tomcat镜像
docker run --rm -d -p 8080:8080 harbor.ceamg.com/baseimages/webapp-tomcat:x1

##停止并删除这个临时运行的容器
docker stop 593673c9d

##进入容器
docker exec -it 593673c9d bash
```

