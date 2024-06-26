
- ### cheat sheet
```conf=
# default values

proxy_connect_timeout   60;
proxy_send_timeout      60;
proxy_read_timeout      60;
send_timeout            60;
keepalive_timeout       75;
fastcgi_connect_timeout 60;
fastcgi_read_timeout    60;
fastcgi_send_timeout    60;

client_body_buffer_size 8/16k
# 儲存 client 的 request body

proxy_buffers           8 4/8k
# cache 上游 Server 的 response

fastcgi_buffers         8 4/8k
fastcgi_buffer_size     4/8k
# fastcgi_buffer_size 會首先接收 response 的 header & body
# if buffer is not enough, 會往 fastcgi_ buffers 放
# 最後還是不夠，會以 temp file 的檔案形式暫存

```
- #### Tricks
```conf=
proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
# trace and list 經過的設備 ip

server {
    server_name _;
    # 當沒有匹配的 domain name 都會往這邊導
}
```

- ### usage of map
這邊做了兩件事情
1. 匹配 regex 取得的值，取其第一個匹配值 ( how $1 work here )
2. 從 $http_cookie 取值 map 到 #phpsessid
```conf=
http {
    (...略)
    map $http_cookie $phpsessid {
        default "-";
        "~*PHPSID=(\S+)(;.*)" $1
    }
    (...略)
}

```

- ### resvoler

```conf=
resolver 8.8.8.8;    # 避免nginx暫存DNS解析，需加上resolver且proxy_pass使用變數
location /url_path {
    set $proxy_url "www.url.path"   
    proxy_pass http://$proxy_url;
    # 與 dynamic DNS 有關
    # domain ip 會變動
    # nginx 會 cache ip，導致錯誤
}
```
