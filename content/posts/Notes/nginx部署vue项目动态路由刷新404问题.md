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