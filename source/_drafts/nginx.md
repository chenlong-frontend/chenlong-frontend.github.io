```
 server {
    listen       8081;
    server_name  localhost;

    location / {
        root   /root/works/nxWeb;
        autoindex on;
    }
 }

 server {
    listen       8082;
    server_name  localhost;
    location / {
        root   /root/works/nxWeb/nuoxweb/html;
    }
    location /Login {
        proxy_pass  http://47.101.175.42:9001;
    }
    location /v1.0 {
        proxy_pass  http://47.101.175.42:9001;
    }
 }
```

```
/usr/local/webserver/nginx/sbin/nginx  # 启动 Nginx
/usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
/usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
/usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx
```
