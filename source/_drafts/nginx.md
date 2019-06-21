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

netstat -lntup

```
cat /usr/local/webserver/nginx/conf/nginx.conf # 查看配置文件
/usr/local/webserver/nginx/sbin/nginx  # 启动 Nginx
/usr/local/webserver/nginx/sbin/nginx -t # 测试Nginx
/usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
/usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
/usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx
```

开启 gzip 并把压缩比给到最大

```
  gzip on;
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 9;
  gzip_types text/plain application/javascript  text/css text/javascript;
  gzip_vary on;

```

try_files $uri $uri/ /index.html;
