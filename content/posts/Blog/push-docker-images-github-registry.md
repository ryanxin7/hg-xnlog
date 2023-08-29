---
author: Ryan
title: 使用Github Action 构建Docker镜像并上传Registry
date: 2023-08-25
lastmod: 2023-08-26
tags:
  - 个人网站
  - Hugo
  - Docker
categories:
  - Hugo
  - Github
  expirationReminder:
    enable: true
---



## 创建拥有上传权限的token

![image-20230825140715736](http://cdn1.ryanxin.live/image-20230825140715736.png)

![image-20230825140743709](http://cdn1.ryanxin.live/image-20230825140743709.png)



## 测试登录Registry



测试登录

```bash
root@harbor01[14:16:02]~ #:docker login --username ryanxin7 --password ghp_xxxxxxxx ghcr.io
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

```



## 构建镜像测试上传到Regisry



```bash
root@harbor01[14:22:43]/dockerfile/xn-blog #:docker image build -t ghcr.io/xnlog:latest ./
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  19.04MB
Step 1/4 : FROM nginx:latest
 ---> 89da1fb6dcb9
Step 2/4 : COPY public/ /usr/share/nginx/html
 ---> Using cache
 ---> 342e46da94ee
Step 3/4 : COPY default.conf /etc/nginx/conf.d/default.conf
 ---> Using cache
 ---> 56c7d4347a26
Step 4/4 : EXPOSE 8848
 ---> Using cache
 ---> 35e2e284b708
Successfully built 35e2e284b708
Successfully tagged ghcr.io/xnlog:latest
```

```bash

root@racknerd-20e7f5:~# docker pull ghcr.io/ryanxin7/xnlog:latest
latest: Pulling from ryanxin7/xnlog
52d2b7f179e3: Pull complete
fd9f026c6310: Pull complete
055fa98b4363: Pull complete
96576293dd29: Pull complete
a7c4092be904: Pull complete
e3b6889c8954: Pull complete
da761d9a302b: Pull complete
e8c074410147: Pull complete
4d2b965ac974: Pull complete
Digest: sha256:3bcffe2f09e7584d9b05da90af16c43b195c377ce645dbc013f8b9ba70ce83de
Status: Downloaded newer image for ghcr.io/ryanxin7/xnlog:latest
ghcr.io/ryanxin7/xnlog:latest
```



## 在packages中查看镜像

![image-20230825144820540](http://cdn1.ryanxin.live/image-20230825144820540.png)





## 配置action secret

![image-20230825145746677](http://cdn1.ryanxin.live/image-20230825145746677.png)





## 创建workflow文件

```yaml
mkdir workflow
name: Docker Image CI for GHCR


on:
   push

jobs:
  build_and_publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and push the image
        run: |
          docker login --username ryanxin7 --password ${{ secrets.DOCKERPACKAING }} ghcr.io
          docker build docker/. --tag ghcr.io/ryanxin7/xnlog:latest
          docker push ghcr.io/ryanxin7/xnlog:latest
```





## 测试提交代码
