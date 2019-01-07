# 为了实现前后端分离的部署，希望实现如下的调用：

比如： https://api.icheesedu.com/course/courseInfo?courseId=1  提供接口给App端调用。

比如： https://www.icheesedu.com/iOS/RunLoop.html  实现h5项目的部署。

需要对Nginx进行如下的配置修改

```
upstream tomcat {
  server 127.0.0.1:8080;
}

server {
  listen 80;
  server_name  xxx.xxx.xxx.xxx  www.icheesedu.com;
}



server {

    listen       443 ssl;
    server_name  xxx.xxx.xxx.xxx www.icheesedu.com ;
    sendfile on;
    access_log /usr/local/nginx/logs/access.log combined;
    error_log  /usr/local/nginx/logs/error.log info;

    root /usr/local/nginx/html; 


    ssl_certificate   /developer/setup/nginx-1.10.2/ssl/NNSKKSS/xxx.xxx.xxx.xxx.pem;
    ssl_certificate_key  /developer/setup/nginx-1.10.2/ssl/NNSKKSS/xxx.xxx.xxx.xxx.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;

    location / {
        root /usr/local/nginx/html; 
    }
}

server {

    listen       443 ssl;
    server_name  api.icheesedu.com ;
    sendfile on;
    access_log /usr/local/nginx/logs/access.log combined;
    error_log  /usr/local/nginx/logs/error.log info;

    root /usr/local/nginx/html;


    ssl_certificate   /developer/setup/nginx-1.10.2/ssl/NNDNNND/xxxxxxxxx.pem;
    ssl_certificate_key  /developer/setup/nginx-1.10.2/ssl/NNDNNND/xxxxxxxxx.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;

    location / {
        root /usr/local/nginx/html;
        proxy_set_header  X-Forwarded-Host $host;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        expires off;
        sendfile off;
        proxy_pass http://tomcat;
   }
}
```

此时，重启nginx服务，访问上述网站即可。

至于如何申请二级域名证书：

![](https://upload-images.jianshu.io/upload_images/325120-49d1c9cecd5c41a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
![](https://upload-images.jianshu.io/upload_images/325120-a330e9ea2b2fc8db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)







