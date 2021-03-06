## 反向代理

Node.js是通过监听端口启动服务来运行的，访问的时候需要带上端口，除非端口是80。但一般情况下，是不会让Node.js占用80端口的。并且Node.js在处理静态资源情况时并没有太大的优势。

所以一般的处理方式是通过Nginx之类的web server来处理，静态资源请求直接让Nginx来处理，动态类的请求通过反向代理让Node.js来处理。这样也方便做负载均衡。

### 配置

nginx下的配置可以参考下面的方式：

```
server {
    listen       80;
    server_name meinv.ueapp.com;
    index index.js index.html index.htm;
    root  /Users/welefen/Develop/git/meinv.ueapp.com/www;

    if ( -f $request_filename/index.html ){
        rewrite (.*) $1/index.html break;
    }
    if ( !-f $request_filename ){
        rewrite (.*) /index.js;
    }
    location = /index.js {
        #proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_pass http://127.0.0.1:6666$request_uri;
        proxy_redirect off;
    }
    location ~ .*\.(js|css|gif|jpg|jpeg|png|bmp|swf|ico|svg|cur|ttf|woff)$ {
        expires      1000d;
    }
}   

```

需要改动3个地方：

* `server_name meinv.ueapp.com`将server_name改为项目对应的域名
* `root  /Users/welefen/Develop/git/meinv.ueapp.com/www` 配置项目的根目录，一定要到www目录下
* `proxy_pass http://127.0.0.1:6666$request_uri;`将端口`6666`改为项目里配置的端口

### 禁止端口访问

配置反向代理后，我们可能就不希望通过端口能直接访问到Node.js服务了。那么可以在`App/Conf/config.js`设置如下的配置：

```js
use_proxy: true, //是否使用代理访问，如：nginx。开启后不能通过ip+端口直接访问
```
