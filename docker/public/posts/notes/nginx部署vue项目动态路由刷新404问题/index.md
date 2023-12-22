# 

# nginx部署vue项目动态路由刷新404问题



动态路由 `/detail/:id` 在 `/usr/share/nginx/html/dist` 目录下

将 `location /detail/` 的 `alias` 配置修改为：

```bash
location /detail/ {
    alias /usr/share/nginx/html/dist/;
    try_files $uri $uri/ /index.html;
}
```

这样配置将尝试匹配动态路由 `/detail/:id`，如果找不到对应文件，则路由到Vue应用的 `index.html` 文件。

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/notes/nginx%E9%83%A8%E7%BD%B2vue%E9%A1%B9%E7%9B%AE%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%E5%88%B7%E6%96%B0404%E9%97%AE%E9%A2%98/  

