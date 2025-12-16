---
title: Windows Server服务器nginx配置
published: 2025-12-15
description: '给服务上nginx来处理各式端口需求'
image: 'pic\nginx-winserver.png'
tags: [nginx,服务器,Windows Server]
category: '菜鸟心得'
draft: false 
lang: 'zh-cn'
---
# 前言

&emsp;&emsp;众所周知，想在Windows Server服务器端部署一个网页让别人访问，其实非常简单，只需新建一个index.html，在里面写一些你想展示的文本，然后用IIS部署即可，而且IIS算是微软做的Windows专精，图形界面非常方便。然而一旦需求稍微复杂一些，比如想部署多个应用，或者在不同访问间实现跳转等等，IIS用图形界面来回操作又费时又不直观，当然直接改web.config的话当我没说（不是）。

&emsp;&emsp;因此我把目光投向了nginx，它的优点在于只要编辑nginx.conf就能解决90%问题，完全由代码块来控制逻辑清晰，此外，还有高性能、低占用、跨平台、稳定性等多项优点。正好最近刚把服务器服务的部署从IIS转成了nginx，趁经验还在脑子里热乎着赶紧留个档，一是方便以后使用，二来以资同好。

&emsp;&emsp;当然，现在很多问题拿去问AI大部分也可以解决，但想把AI用好还是得对相关领域有基础的了解，否则人工智能可能就跟个人工智障一样。AI给的代码不亲自扫一遍多半会被坑，而且使用AI还有隐私等各种问题，所以这篇博客也有其存在的意义，菜鸡的经验也许更能指导菜鸡，而且至少也是自己存在过的证明。

&emsp;&emsp;闲言少叙，正文开始，右侧目录，按需跳转。

# 安装nginx

1. 点击[https://nginx.org/en/download.html](https://nginx.org/en/download.html)自动跳转至官网，Stable version中选择nginx/Windows-1.xx.x即可，以下均以1.26.0为例。
   如有其他版本需求，请使用魔法访问[https://github.com/nginx/nginx/releases](https://github.com/nginx/nginx/releases)。
2. 将zip解压至需要的路径，以下均以D:\nginx为例

# 配置nginx.conf

nginx.conf是配置nginx的核心，其位于根目录\conf下。
nginx.conf的全部语法可详参其官方文档[https://nginx.org/en/docs/](https://nginx.org/en/docs/)，以下仅列举比较常用可能用到的部分。

## 结构简介

```nginx
# 全局设定
......
# events 块
events {

}
# http 块
http {
    # http全局设定
    ......
    # server块
    server {
        # server 配置
        ......
        # location块，
        location [PATTERN] {

        }
    }
}
```

- 格式说明

| 指令          | 参数类型              | 可用空间                                                      | 默认设置                        | 注释                                                                                                                                                                                        |
| ------------- | --------------------- | ------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `error_log` | `file` `[level]` | `main` `http` `mail` `stream` `server` `location` | error_log logs/error.log error; | 配置日志。path指定日志的路径和名称，level指定日志记录级别，记录当前等级及以上的日志，从低到高为 `debug`、 `info`、 `notice`、 `warn`、 `error`、 `crit`、 `alert`、 `emerg` |

1. `error_log`表示可以设置的指令
2. `file` `[level]`表示必须设置 `file`参数，`[level]`表示 `level`参数为可选配参数。`number`/`auto`则表示必须在 `number`和 `auto`中二选一。
3. `main` `http` `mail` `stream` `server` `location`表示该指令可以在这些部分中正常使用
4. `error_log logs/error.log error;`表示当不设置该指令时，nginx的默认设置，-表示没有默认设置，无法默认配置，或者简单理解看情况来
5. `注释`对该指令的作用和使用方法进行简要说明。

- 需要注意

1. 在语法上nginx每条语句结束有分号 `;`，而块结构的结尾不需要。
2. 合理使用 `#`号对nginx.conf进行注释。
3. 以下以实际应用的nginx.conf文档为例，分块进行说明，在每一块下对当前使用到的指令进行详解。
   **不代表该指令只能运用于该块中，指令的适用空间请参照其“可用空间”栏。**

## 全局部分

```nginx
worker_processes  auto;
error_log  logs/error.log  warn;
pid        logs/nginx.pid;
```

| 指令                 | 参数类型             | 可用空间                                                      | 默认设置                        | 注释                                                                                                                                                                                        |
| -------------------- | -------------------- | ------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `worker_processes` | `number`/`auto`  | `main`                                                      | worker_processes 1;             | 指定工作进程数。最优数值取决于CPU核心数、存储数据的硬盘数、负载模式等。不清楚可以使用auto，自动检测可用的CPU核心数                                                                          |
| `error_log`        | `file` `[level]` | `main` `http` `mail` `stream` `server` `location` | error_log logs/error.log error; | 配置日志。path指定日志的路径和名称，level指定日志记录级别，记录当前等级及以上的日志，从低到高为 `debug`、 `info`、 `notice`、 `warn`、 `error`、 `crit`、 `alert`、 `emerg` |
| `pid`              | `file`             | `main`                                                      | pid logs/nginx.pid;             | 指定文件存储主进程的ID                                                                                                                                                                      |

注意，`file`默认使用Nginx根目录的相对路径，使用绝对路径，需要在前面加上 `/`。

## events块

```nginx
events{
    worker_connections  2048;
    multi_accept        on;
    use                 epoll;
}
```

| 指令                   | 参数类型       | 可用空间   | 默认设置                | 注释                                                                                                                                                    |
| ---------------------- | -------------- | ---------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `worker_connections` | `number`     | `events` | worker_connections 512; | 设置由所有连接的最大数量，包含与客户端的连接，以及与代理服务器等的连接等。                                                                              |
| `multi_accept`       | `on`/`off` | `events` | multi_accept off;       | 同时接受一个新的连接/同时接受所有新的连接                                                                                                               |
| `use`                | `method`     | `events` | -                       | 指定要连接处理方法。nginx 默认会使用最高效的方法。可选类型有 `selelct`、`poll`、 `kqueue`、 `epoll`、 `resig`、 `/dev/poll`、 `eventport` |

## http块

写在http块下，server块外的，相当于http的全局设定，对http块下的所有server生效。

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    sendfile_max_chunk  100k;
    send_timeout    75;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout  65;
    client_max_body_size 100M;

    # 日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;


    # 防CC攻击
    limit_req_zone $binary_remote_addr zone=cc:10m rate=10r/s;

    # 防止中文出现乱码
    charset utf-8;

    server{}
    server{}
}
```

| 指令                     | 参数类型                                                                       | 可用空间                                                             | 默认设置                             | 注释                                                                                                                          |
| ------------------------ | ------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `include`              | `file`/`mask`                                                              | `any`                                                              | -                                    | 导入其他配置文件，如include mime.types导入文件扩展名和文件类型的映射，其中mime.types是相对路径，其与nginx.conf在同目录下      |
| `default_type`         | `mime-type`                                                                  | `http` `server` `location`                                    | default_type text/plain;             | 定义响应的默认类型，可以使用 `types`句法块设置类型的映射                                                                    |
| `sendfile`             | `on`/`off`                                                                 | `http` `server` `location` `if in location`                  | sendfile off;                        | 开启/关闭高效率传输文件                                                                                                       |
| `sendfile_max_chunk`   | `size`                                                                       | `http` `serverlocation`                                         | sendfile_max_chunk 2m;               | 设置单个sendfile()调用中可以传输的最大数据量。                                                                                |
| `send_timeout`         | `time`                                                                       | `http` `server` `location`                                     | send_timeout 60s;                    | 设置向客户端发送响应的超时时间                                                                                                |
| `tcp_nopush`           | `on`/`off`                                                                 | `http` `server` `location`                                     | tcp_nopush off;                      | 启用或禁用FreeBSD上的TCP_NOPUSH(或Linux上的TCP_CORK)，仅启用sendfile时生效，启用后会以完整数据包发送文件                      |
| `tcp_nodelay`          | `on`/`off`                                                                 | `http` `server` `location`                                     | tcp_nodelay on;                      | 启用或禁用TCP_NODELAY，在连接进入keep-alive时，或在SSL连接、无缓冲代理、WebSocket代理时生效                                   |
| `keepalive_timeout`    | `timeout` `[header_timeout]`                                               | `http` `server` `location`                                     | keepalive_timeout 75s;               | 第一个timeout参数设置长连接超时时间，第二个可选参数是给响应头设置超时时间，在部分浏览器中生效                                 |
| `client_max_body_size` | `size`                                                                       | `http` `server` `location`                                     | client_max_body_size 1m;             | 设置客户端请求正文允许的最大值,如果有文件传输服务请注意调整                                                                   |
| `log_format`           | `name` `[escape=default/json/none]` `string`                             | `http`                                                             | log_format combined "...";           | 自定义一个日志格式，后续使用name引用该格式。可选参数escape指定文件格式，default默认纯文本，string指定输出内容格式             |
| `access_log`           | `path` `[format [buffer=size] [gzip[=level]] [flush=time] [if=condition]]` | `http` `server` `location` `if in location` `limit_except` | access_log logs/access.log combined; | 设置日志的路径、格式、配置。format可以引用log_format的name                                                                    |
| `access_log`           | `off`                                                                        | -                                                                    | -                                    | 设置特殊参数off会取消当前级别的所有access_log                                                                                 |
| `limit_req_zone`       | `key zone=name:size rate=rate [sync]`                                        | `http`                                                             | -                                    | 设置处理请求的共享参数，$binary_remote_addr zone=cc:10m rate=10r/s表示所有状态存在名为cc的1mbit的空间里，每秒最多处理10个请求 |
| `char_set`             | `charset`/`off`                                                            | `http` `server` `location` `if in location`                  | charset off;                         | 将指定的字符集添加到“Content-Type”响应头字段                                                                                |

下表展示 `log_format`的 `string`参数编写中可能用到的参数。

| 变量名                   | 含义                                                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `$bytes_sent`          | 发送到客户端的字节数                                                                                         |
| `$connection`          | 连接序列号                                                                                                   |
| `$connection_requests` | 当前通过连接发出的请求数                                                                                     |
| `$msec`                | 日志写入时间（秒），精确到毫秒                                                                               |
| `$pipe`                | 是否为管道化请求，"p"/"-"                                                                                    |
| `$request_length`      | 请求长度（包括请求行、标头和请求正文）                                                                       |
| `$request_time`        | 请求处理时间（秒），精确到毫秒，指从客户端读取第一个字节到将最后一个字节发送到客户端后写入日志之间经过的时间 |
| `$status`              | 响应状态                                                                                                     |
| `$time_iso8601`        | ISO 8601标准格式的当地时间                                                                                   |
| `$time_local`          | 通用日志格式的本地时间                                                                                       |

## server块

大体与http块一致，以下用两个例子来说明，第一个是正常截取80端口请求，第二个是对443的https请求反向代理至本地8000端口。重复指令和参数不再赘述。

```nginx
server {
        listen 80;
        server_name your_server_name;

        root E:/yourWebsite;
        index index.html;

        error_page 404 /404.html
        error_page 500 502 503 /50x.html
  
  
        # 防护措施
  
        # 超出部分429 Too Many Requests 
        limit_req zone=cc burst=20 nodelay;

        # 禁止恶意请求方法，禁止恶意访问
        location ~ /(\.|git|svn|log|tmp|backup|config|database|ini|bat|cmd|sh|bak)$ {
            deny all;
            return 403;
        }
  
        # 图片类使用30天内缓存，不写入日志
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|eot)$ {
            expires 30d;
            add_header Cache-Control "public, max-age=2592000";
            access_log off;
        }
    }
```

```nginx
server {
        listen 443 ssl;
        server_name your_server_name;
  
        # 证书路径
        ssl_certificate      C:/certifications/LetsEncrypt-Certs/full-chain.pem;
        ssl_certificate_key  C:/certifications/LetsEncrypt-Certs/private_key.pem;
  
        # ssl安全配置
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_session_timeout 10m;
        ssl_session_cache shared:SSL:10m;
        ssl_session_tickets off;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 反向代理
  
        # 放开文件上传大小限制（Seafile大文件上传必备）
        client_max_body_size 10G;
        # 禁用缓存（避免Seafile动态内容缓存异常）
        proxy_buffering off;

        location / {
            proxy_pass         http://127.0.0.1:8000;   # 反向代理至本地8000端口
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;

            proxy_connect_timeout 3600;
            proxy_read_timeout 3600;
            proxy_send_timeout 3600;
            proxy_http_version 1.1;
        }
   

        # 防护措施
        limit_req zone=cc burst=20 nodelay;

        # # 禁止恶意请求
        location ~ /(\.|git|svn|log|tmp|backup|config|database|ini|bat|cmd|sh|bak)$ {
            deny all;
            return 403;
        }
        if ($request_method !~ ^(GET|POST|HEAD)$) {
            return 405;
        }
    }
```

常用的基本参数为 `listen`、`server_name`、`root`、`index`，详解如下

| 指令            | 参数                                                                                                                                                                                                                                                                                                                                             | 可用空间                                            | 默认设置              | 注释                                                                                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `listen`      | `address[:port]` `[default_server]` `[ssl]` `[http2/quic]` `[proxy_protocol]` `[setfib=number]` `[fastopen=number]` `[backlog=number]` `[rcvbuf=size]` `[sndbuf=size]` `[accept_filter=filter]` `[deferred]` `[bind]` `[ipv6only=on/off]` `[reuseport]` `[so_keepalive=on/off/[keepidle]:[keepintvl]:[keepcnt]]` | `server`                                          | listen *:80 / *:8000; | 设置监听地址，如listen 127.0.0.1:8000;/listen 127.0.0.1;                                                                                                               |
| `listen`      | `port` `[default_server]` `[ssl]` `[http2/quic]` `[proxy_protocol]` `[setfib=number]` `[fastopen=number]` `[backlog=number]` `[rcvbuf=size]` `[sndbuf=size]` `[accept_filter=filter]` `[deferred]` `[bind]` `[ipv6only=on/off]` `[reuseport]` `[so_keepalive=on/off/[keepidle]:[keepintvl]:[keepcnt]]`           | `server`                                          | listen *:80 / *:8000; | 设置监听端口，如listen 8000;/listen *:8000;                                                                                                                            |
| `server_name` | `name...`                                                                                                                                                                                                                                                                                                                                      | `server`                                          | server_name "";       | 基于基于请求头的"HOST"字段捕获服务器名称，可捕获多个。详参[官方说明](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen)                                     |
| `root`        | `path`                                                                                                                                                                                                                                                                                                                                         | `http` `server` `location` `if in location` | root html;            | 设置请求的根目录，path中可以包含除 `$document_root`和 `$realpath_root`之外的各种变量。如location /i/{root /data/w3;}，请求为/i/top.gif时，将发送/data/w3/i/top.gif |
| `error_page`  | `code ...` `[=[response]]` `uri`                                                                                                                                                                                                                                                                                                           | `http` `server` `location` `if in location` | -                     | 根据错误设置对应错误页                                                                                                                                                 |
| `index`       | `file ...`                                                                                                                                                                                                                                                                                                                                     | `http` `server`  `location`                   | index index.html;     | 设置作为主页的网页，实际是将所有 `"/"`的请求转到了index下                                                                                                            |
| `return`      | `code` `[text]`                                                                                                                                                                                                                                                                                                                              | `server` `location` `if`                      | -                     | 停止处理并返回指定代码及响应正文                                                                                                                                       |
| `return`      | `[code]` `URL`                                                                                                                                                                                                                                                                                                                               | `server` `location` `if`                      | -                     | 指定重定向URL（301、302、303、307、308），可以指定为本地URI                                                                                                            |
| `limit_req`   | `zone=name [burst=number] [nodelay/delay=number]`                                                                                                                                                                                                                                                                                              | `http` `server` `location`                    | -                     | 根据设置的name共享区进行判定，当请求频率超出共享区的rate限制时，按照delay设置处理burst数量的请求，超出burst部分返回429                                                 |
| `add_header`  | `name` `value` `[always]`;                                                                                                                                                                                                                                                                                                                 | `http` `server` `location` `if in location` | -                     | 响应为200,201,204,206,301,302,303,304,307,308时将指定字段添加到响应头，always为无论响应代码必然添加                                                                    |

接收https时SSL常用参数如下，其中SSL证书为必备项。

| 指令                    | 参数类型                                                                      | 可用空间            | 默认设置                       | 注释                                             |
| ----------------------- | ----------------------------------------------------------------------------- | ------------------- | ------------------------------ | ------------------------------------------------ |
| `ssl_certificate`     | `file`                                                                      | `http` `server` | -                              | 设置符合该server块的ssl证书(PEM格式，full-chain) |
| `ssl_certificate_key` | `file`                                                                      | `http` `server` | -                              | 设置符合该server块的ssl证书（PEM格式）           |
| `ssl_protocols`       | `[SSLv2]` `[SSLv3]` `[TLSv1]` `[TLSv1.1]` `[TLSv1.2]` `[TLSv1.3]` | `http` `server` | ssl_protocols TLSv1.2 TLSv1.3; | 启用指定协议                                     |
| `ssl_session_timeout` | `time`                                                                      | `http` `server` | ssl_session_timeout 5m;        | 设置客户端可以复用会话参数的时间                 |
| `ssl_session_cache`   | `off`/`none`/`[builtin[:size]]` `[shared:name:size]`                  | `http` `server` | ssl_session_cache none;        | 设置存储会话的缓存的大小                         |
| `ssl_session_tickets` | `on`/`off`                                                                | `http` `server` | ssl_session_tickets on;        | 通过 TLS 会话票证启用或禁用会话复用              |

proxy常用参数如下，常见于反向代理。

| 指令                      | 参数类型              | 可用空间                                         | 默认设置                                                              | 注释                                       |
| ------------------------- | --------------------- | ------------------------------------------------ | --------------------------------------------------------------------- | ------------------------------------------ |
| `proxy_pass`            | `URL`               | `location` `if in location` `limit_except` | -                                                                     | 设置代理服务器的协议和地址                 |
| `proxy_set_header`      | `field` `value`   | `http` `server` `location`                 | proxy_set_header Host $proxy_host; proxy_set_header Connection close; | 设置或者重新设置传递给代理服务器的请求头   |
| `proxy_connect_timeout` | `time`              | `http` `server` `location`                 | proxy_connect_timeout 60s;                                            | 设置和代理服务器连接超时时间               |
| `proxy_read_timeout`    | `time`              | `http` `server` `location`                 | proxy_read_timeout 60s;                                               | 设置和代理服务器两个成功相应之间的超时时间 |
| `proxy_send_timeout`    | `time`              | `http` `server` `location`                 | proxy_send_timeout 60s;                                               | 设置和代理服务器发送请求的超时时间         |
| `proxy_http_version`    | `1.0`/`1.1`/`2` | `http` `server` `location`                 | proxy_http_version 1.0;                                               | 设置代理使用的 HTTP 协议版本               |

注意，server块的匹配是有优先级差异的，即精准域名匹配 >> `*`开头的通配 >> `*`结尾的通配 >> 正则匹配 >> 默认兜底。同优先级下，以配置文件中的先后顺序为准。
举例来说：`server_name example.com;` >> `server_name *.example.com;` >> `server_name example.*;` >> `server_name ~^.*\.example\.com$;` >> `server_name _;`。
如果要设置兜底的server块，可以考虑 `server_name _;`同时 `listen`处添加 `default_server`参数，强制兜底。
如需学习正则，可参考[learn regex](https://github.com/ziishaned/learn-regex)

## location块

```nginx
        location ~ /(\.|git|svn|log|tmp|backup|config|database|ini|bat|cmd|sh|bak)$ {
            deny all;
            return 403;
        }
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|eot)$ {
            # 缓存静态资源30天
            expires 30d;
            add_header Cache-Control "public, max-age=2592000";
            # 关闭日志
            access_log off;
        }
```

| 指令        | 参数类型                                | 可用空间                                            | 默认设置     | 注释                                                           |
| ----------- | --------------------------------------- | --------------------------------------------------- | ------------ | -------------------------------------------------------------- |
| `deny`    | `address`/`CIDR`/`unix:`/ `all` | `location` `if in location` `limit_except`    | -            | 拒绝指定网络或地址的访问。                                     |
| `allow`   | `address`/`CIDR`/`unix:`/ `all` | `location` `if in location` `limit_except`    | -            | 允许指定网络或地址的访问。                                     |
| `expires` | `[modified]` `time`                 | `http` `server` `location` `if in location` | expires off; | 启用或禁用对响应头中 `Expires`和 `Cache-Control`的添加修改 |
| `expires` | `epoch`/`max`/`off`               | `http` `server` `location` `if in location` | expires off; | -                                                              |

# nginx相关命令

```bash
D:
cd D:/nginx
nginx -c /yourPath/yourSite/nginx.conf
nginx -t
start nginx
nginx -s stop
nginx -s reload
```

0. D:   cd D:/nginx切换到D盘下的nginx目录（nginx.exe所在目录）
1. nginx -c /yourPath/yourSite/nginx.conf设置一个新的路径配置你的nginx.conf，如不设置，默认为/conf/nginx.conf
2. nginx -t校验配置文件语法是否正确，正确则输出如下

   ```bash
   nginx: the configuration file /yourPath/yourSite/nginx.conf syntax is ok
   nginx: configuration file /yourPath/yourSite/nginx.conf test is successful
   ```
3. start nginx校验通过后，启动nginx服务
4. nginx -s
   stop表示迅速停止，quit表示优雅地停止，reload表示优雅地停止并重新加载配置文件

# 结语

&emsp;&emsp;以上仅为菜鸟的经验，注意根据需求配置server块，对不同请求进行截取，调试时合理利用日志文件。
&emsp;&emsp;另外，这篇文章中提到的ssl证书如何免费申请、设置和续约，后续有时间再专门写一篇吧。
