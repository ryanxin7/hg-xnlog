# 使用Github Action 构建Docker镜像并上传Registry




使用 GitHub Actions 来自动构建 Docker 镜像并将其上传到 Docker Registry 时，需要以下步骤进行设置：



工作流会在每次将代码推送到 `main` 分支时执行。它首先检出代码，然后设置 Docker Buildx 环境，接着登录到指定的 Docker Registry，最后构建并推送 Docker 镜像。

---



1. **创建 Dockerfile**：在你的 GitHub 仓库中创建一个名为 `Dockerfile` 的文件，用于定义镜像的构建过程和内容。

2. **设置 Secrets**：在仓库的设置中，添加三个 Secrets，分别是你的 Docker Registry 用户名、密码或访问令牌，以及 Docker Registry 的地址。

3. **创建 Workflow 文件**：在 `.github/workflows/` 目录下创建一个 `.yml` 文件（例如：`docker-build.yml`），在这个文件中定义工作流程的步骤。

4. **Workflow 配置**：在 Workflow 文件中，配置工作流程的触发条件，比如当代码被推送到特定分支时触发。然后，定义构建步骤，包括：

   - 检出代码
   - 设置 Docker Buildx 环境（用于构建多平台镜像）
   - 登录到 Docker Registry，使用之前设置的 Secrets
   - 使用 Docker 构建和推送镜像到 Registry，可以指定标签等信息。

   

   ---

   



## 创建拥有上传权限的token

![image-20230825140715736](https://cdn1.ryanxin.live/image-20230825140715736.png)

![image-20230825140743709](https://cdn1.ryanxin.live/image-20230825140743709.png)



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

![image-20230825144820540](https://cdn1.ryanxin.live/image-20230825144820540.png)





## 配置action secret

![image-20230825145746677](https://cdn1.ryanxin.live/image-20230825145746677.png)





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

```bash
xx9z@xin MINGW64 /c/xnblog/xnlog (main)
$ git add .

xx9z@xin MINGW64 /c/xnblog/xnlog (main)
$ git commit -m "blog update"
[main a5724b1] blog update
 5 files changed, 477 insertions(+)
 create mode 100644 content/posts/Blog/push-docker-images-github-registry.md
 rename "content/posts/\344\275\277\347\224\250Algolia\345\256\236\347\216\260Hugo\346\234\254\345\234\260\346\231\272\350\203\275\346\220\234\347\264\242.md" => "content/posts/Blog/\344\275\277\347\224\250Algolia\345\256\236\347\216\260Hugo\346\234\254\345\234\260\346\231\272\350\203\275\346\220\234\347\264\242.md" (100%)
 create mode 100644 content/posts/kubernetes/k8s-replace-NFS-storage.md
 create mode 100644 "content/posts/kubernetes/k8s\345\274\272\345\210\266\345\210\240\351\231\244pod&pv&pvc\345\222\214ns&namespace\346\226\271\346\263\225.md"
 create mode 100644 content/posts/kubernetes/redis-on-k8scluster.md

xx9z@xin MINGW64 /c/xnblog/xnlog (main)
$ git push origin main
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 16 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (10/10), 5.57 KiB | 2.78 MiB/s, done.
Total 10 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:ryanxin7/hg-xnlog.git
   12d766d..a5724b1  main -> main
```





![image-20230829143031062](https://cdn1.ryanxin.live/image-20230829143031062.png)

![image-20230829143215614](https://cdn1.ryanxin.live/image-20230829143215614.png)


---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/blog/push-docker-images-github-registry/  

