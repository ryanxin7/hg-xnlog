# docker 批量删除  Exited  状态的容器




## docker 批量删除  Exited  状态的容器

```
docker rm $(docker ps -a -q -f status=exited)
```

这个命令会列出所有处于 Exited 状态的容器，并将其删除。



- `docker ps -a`：列出所有的容器，包括正在运行的和已经停止的。
- `-q`：这个选项用于静默输出容器的 ID（而不是完整的容器信息）。
- `-f status=exited`：这是一个过滤条件，它筛选出状态为 Exited 的容器。
- `docker rm`：这个命令用于删除容器。它接收的参数是一个容器 ID 或者多个容器 ID 的列表。

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/docker/docker-%E6%89%B9%E9%87%8F%E5%88%A0%E9%99%A4-exited-%E7%8A%B6%E6%80%81%E7%9A%84%E5%AE%B9%E5%99%A8/  

