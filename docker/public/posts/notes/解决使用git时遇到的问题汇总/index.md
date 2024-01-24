# 使用Git时遇到的问题汇总




## 1.fatal: unable to access 'xxxxxx': Failed to connect to github.com port 443 after 21074 ms: Couldn't connect to server


解决方法： 设置git端口号和梯子的端口号保持一致

![image-20240124223407707](http://cdn1.ryanxin.live/xxlog/image-20240124223407707.png)



```bash
git config --global http.proxy 127.0.0.1:7980
git config --global https.proxy 127.0.0.1:7980
```


<br>



**查看git设置**

```bash
$ git config --global -l
user.name=xxxxx
user.email=xxxxx@xxx.com
http.proxy=127.0.0.1:7890
https.proxy=127.0.0.1:7890
```

<br>


**去掉代理**

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```


<br>



**cmd窗口中刷新dns缓存**

```cmd
 ipconfig/flushdns 
```


<br>




## 2.remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

<br>


**2021.8.13起，Github要求使用基于令牌的身份验证**

必须使用个人访问令牌（personal access token），就是把你的密码替换成token！

![image-20240124224044162](http://cdn1.ryanxin.live/xxlog/image-20240124224044162.png)

![image-20240124224326566](http://cdn1.ryanxin.live/xxlog/image-20240124224326566.png)

![1706107438712](http://cdn1.ryanxin.live/xxlog/1706107438712.png)


<br>



**有两种方式使用token登录**

1. 之后用自己生成的token登录，把上面生成的token粘贴到输入密码的位置。

如果 push 等操作没有出现输入密码选项，请先输入如下命令，之后就可以看到输入密码选项了。

```text
git config --system --unset credential.helper
```

<br>


2. 把token直接添加远程仓库链接中，这样就可以避免同一个仓库每次提交代码都要输入token了：

```text
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
```

- `<your_token>`：换成你自己得到的token
- `<USERNAME>`：是你自己github的用户名
- `<REPO>`：是你的仓库名称


---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/notes/%E8%A7%A3%E5%86%B3%E4%BD%BF%E7%94%A8git%E6%97%B6%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB/  

