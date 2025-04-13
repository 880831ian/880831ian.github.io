---
title: "用 HAProxy 對 Redis 做負載平衡 (Redis Sentinel、Docker-compose)"
type: docs
weight: 9997
date: 2022-04-22
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

前情提要：本篇是 [Redis 哨兵模式 (Sentinel) 搭配 Docker-compose 實作
](../redis-sentinel-docker-compose) 的後續文章，主要會優化原本的程式碼，並加上 HAProxy 來做負載平衡！

## 什麼是 HAProxy 以及負載均衡 ?

HAProxy 是一個使用 C 語言編寫的自由及開放原始碼軟體，其提供高可用性、負載均衡，以及基於 TCP 和 HTTP 的應用程式代理。

負載平衡 (Load Balance)：

現在很多網路服務都需要服務大量使用者，以前可以砸錢擴充機器硬體設施，但隨著網路服務的用量暴增，增加伺服器硬體設備已經無法解決問題。

為了可以擴充服務，負載平衡成為主流的技術，這幾年雖然雲端與分散式儲存運算技術火紅，除非有特別的使用需求，不然在技術上負載均衡算是比較容易達成與掌握的技術。

負載平衡除了分流能力之外，有另一個很大的好處就是可以提供 High Availability，也就是傳說中的 HA 架構，好讓你一台機器掛了其他伺服器可以繼續服務，降低斷線率。

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/haproxy.png" width="700" caption="HAProxy 與 Reids Sentinel 示意圖 [selcukusta/redis-sentinel-with-haproxy](https://github.com/selcukusta/redis-sentinel-with-haproxy)" >}}

<br>

## 實作

那接下來會使用 Docker-compose 實作 Redis 哨兵模式 + HAProxy，[範例程式連結 點我](https://github.com/880831ian/docker-compose-redis-sentinel-haproxy) 😘

版本資訊
* macOS：11.6
* Docker：Docker version 20.10.12, build e91ed57
* Nginx：1.20
* PHP：7.4-fpm
* Redis：6.2.6
*  HAProxy：HAProxy version 2.5.5-384c5c5 2022/03/14 - https://haproxy.org/

<br>

### 檔案結構

```
.
├── Docker-compose.yaml
├── docker-volume
│   ├── haproxy
│   │   └── haproxy.cfg
│   ├── nginx
│   │   └── nginx.conf
│   ├── php
│   │   ├── info.php
│   │   ├── r.php
│   │   └── rw.php
│   └── redis
│       ├── redis.conf
│       ├── redis1
│       ├── redis2
│       └── redis3
├── php
│   └── Dockerfile
├── redis.sh
└── sentinel
    ├── Docker-compose.yaml
    ├── sentinel1
    │   └── sentinel.conf
    ├── sentinel2
    │   └── sentinel.conf
    └── sentinel3
        └── sentinel.conf
```

<br>

這是主要的結構，簡單說明一下：

* Docker-compose.yaml：會放置要產生的 Nginx、PHP、redis1、redis2、redis3 容器設定檔。
*  docker-volume/haproxy/haproxy.cfg：haproxy 的設定檔。
*    docker-volume/nginx/nginx.conf：nginx 的設定檔。
*    docker-volume/php/(r.php、rw.php)：測試用檔案。
*    docker-volume/redis/redis.conf：redis 的設定檔。
*    docker-volume/redis/(redis1、redis2、redis3)：放 redis 的資料。
*    php/Dokcerfile：因為在 php 要使用 redis 需要多安裝一些設定，所以用 Dockerfile 另外寫 PHP 的映像檔。
*  redis.sh：是我另外多寫的腳本，可以查看相對應的角色。
*  sentinel/Docker-compose.yaml：會放置要產生的 haproxy、sentinel1、sentinel2、sentinel3 的容器設定檔。
*  	sentinel/(sentinel1、sentinel2、sentinel3)/.conf：哨兵的設定檔。

<br>

那我們就依照安裝的設定開始說明：

### Docker-compose.yaml

```yml
version: '3.8'

services:
  nginx:
    image: nginx:1.20
    container_name: nginx
    networks:
      HAProxy_Redis:
    ports:
      - "8888:80"
    volumes:
      - ./docker-volume/nginx/:/etc/nginx/conf.d/
      - ./log/nginx/:/var/log/nginx/
    environment:
      - TZ=Asia/Taipei

  php:
    build: ./php
    container_name: php
    networks:
      HAProxy_Redis:
    expose:
      - 9000
    volumes:
      - ./docker-volume/php/:/var/www/html

  redis1:
    image: redis
    container_name: redis1
    command: redis-server /usr/local/etc/redis/redis.conf --appendonly yes
    volumes:
      - ./docker-volume/redis/redis1/:/data
      - ./docker-volume/redis/:/usr/local/etc/redis/
      - ./log/redis1:/var/log/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      HAProxy_Redis:
        ipv4_address: 172.20.0.11
    ports:
      - 6379:6379

  redis2:
    image: redis
    container_name: redis2
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis1 6379 --appendonly yes
    volumes:
      - ./docker-volume/redis/redis2/:/data
      - ./docker-volume/redis/:/usr/local/etc/redis/
      - ./log/redis2:/var/log/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      HAProxy_Redis:
        ipv4_address: 172.20.0.12
    ports:
      - 6380:6379
    depends_on:
      - redis1

  redis3:
    image: redis
    container_name: redis3
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis1 6379 --appendonly yes
    volumes:
      - ./docker-volume/redis/redis3/:/data
      - ./docker-volume/redis/:/usr/local/etc/redis/
      - ./log/redis3:/var/log/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      HAProxy_Redis:
        ipv4_address: 172.20.0.13
    ports:
      - 6381:6379
    depends_on:
      - redis1
      - redis2

networks:
  HAProxy_Redis:
    driver: bridge
    name: HAProxy_Redis
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```
一樣詳細的 Docker 設定說明，可以參考 [Docker 介紹](../../docker/docker) 內有詳細設定說明。其他比較特別的地方是：

* 幫每一個容器都設定好 IP ，方便後續測試使用。
* 有掛載 log 目錄，可以將我們設定好的 log 做收集。
*  呈上，有加入 environment 時區，這樣在看 log 的時候才知道正確時間。

<br>

### docker-volume/haproxy/haproxy.cfg

```cfg
global
  log stdout format raw local0 info

defaults
  mode http # 默認模式 { tcp | http | health }，tcp 是4層，http 是7層，health 只會返回 OK
  timeout client 10s  # 客戶端超時
  timeout connect 5s  # 連接超時
  timeout server 10s # 伺服器超時
  timeout http-request 10s
  log global

listen admin_status
  bind 0.0.0.0:8404
  mode http
  stats enable
  stats uri /redis
  stats realm Global\ statistics
  stats refresh 1s

listen rw-redis # 判斷是否為 master 並可讀可寫
  bind 0.0.0.0:16379
  mode tcp
  balance roundrobin
  option tcp-check # redis 健康检查，確保是 master
  tcp-check connect
  tcp-check send PING\r\n
  tcp-check expect string +PONG
  tcp-check send info\ replication\r\n
  tcp-check expect string role:master
  tcp-check send QUIT\r\n
  tcp-check expect string +OK
  server redis1 redis1:6379 check inter 2000
  server redis2 redis2:6379 check inter 2000
  server redis3 redis3:6379 check inter 2000

listen r-redis  # 判斷是否為 master、slave 並可讀
  bind 0.0.0.0:16380
  mode tcp
  balance roundrobin
  server redis1 redis1:6379 check inter 2000
  server redis2 redis2:6379 check inter 2000
  server redis3 redis3:6379 check inter 2000
```
這裡是本章的重點，我們會在這邊設定好 haproxy，詳細說明請看：

defaults：
一些初始值，像是 mode 我們預設 http，它主要有三種模式 { tcp | http | health }，tcp 是4層，http 是7層，health 只會返回 OK，以及客戶端、連接、伺服器、http 請求超時時間設定。

listen admin_status：
* bind：我們要開啟 HAProxy 監控平台的 port。
* mode：模式，我們使用 http 模式。
* stats ：是否要啟動平台。
* stats uri：平台網址，我們使用 redis。
* stats refresh：平台自動更新時間，我們設定 1 秒。

listen rw-redis：
* bind ：rw 使用 16379 Port 來當輸出。
* balance：使用負載平衡。

```
option tcp-check # redis 健康检查，確保是 master
  tcp-check connect
  tcp-check send PING\r\n
  tcp-check expect string +PONG
  tcp-check send info\ replication\r\n
  tcp-check expect string role:master
  tcp-check send QUIT\r\n
  tcp-check expect string +OK
  ```
  上面這些是用來判斷角色是不是 master。

 最後放我們 3 個 redis 服務：
 ```
  server redis1 redis1:6379 check inter 2000
  server redis2 redis2:6379 check inter 2000
  server redis3 redis3:6379 check inter 2000
 ```
* check：開啟健康偵測。
* inter：參數更改檢查間隔，預設是 2 秒。

<br>

### docker-volume/nginx/nginx.conf

```conf
server {
  listen 80;
  server_name default_server;
  return 404;
}

server {
  listen 80;
  server_name test.com;
  index index.php index.html;

  error_log  /var/log/nginx/error.log  warn;
  access_log /var/log/nginx/access.log;
  root /var/www/html;

  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }

  location ~ \.php$ {
      fastcgi_pass php:9000;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
  }
}
```
Nginx 設定檔案。

<br>

### docker-volume/php/rw.php

```php
<?php

$redis = new Redis();
$redis->connect('172.20.0.20', 16379);
$r = $redis->info();

echo  $r['run_id'] . '<br>' . $r['role'] . '<br><br>';

echo '<pre>', print_r($r), '</pre>';
```
跟 `r.php` 比較不同的是，使用 16379 Port，我們在 `haproxy.cfg` 有設定 rw-redis，來判斷是不是 master 並且是可讀可寫。

<br>

### docker-volume/php/r.php

```php
<?php

$redis = new Redis();
$redis->connect('172.20.0.20', 16380);
$r = $redis->info();

echo  $r['run_id'] . '<br>' . $r['role'] . '<br><br>';

echo '<pre>', print_r($r), '</pre>';
```
使用 16380 Port，在 `haproxy.cfg` 有設定 r-redis，來顯示是不是 master、slave 且可讀。

<br>

### php/Dockerfile

```Dockerfile
FROM php:7.4-fpm

RUN pecl install -o -f redis \
&&  rm -rf /tmp/pear \
&&  echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini \
&&  echo "session.save_handler = redis" >> /usr/local/etc/php/conf.d/redis.ini \
&&  echo "session.save_path = tcp://redis:6379" >> /usr/local/etc/php/conf.d/redis.ini
```
因為 PHP 要使用 Redis，會需要安裝一些套件，所以我們將 PHP 分開來，使用 Dockerfile 來設定映像檔。

<br>

### redis.sh

```sh
#!/bin/bash

green="\033[1;32m";white="\033[1;0m";red="\033[1;31m";

echo "redis1 IPAddress:"
redis1_ip=`docker inspect redis1 | grep "IPv4" | egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`
echo $redis1_ip;
echo "------------------------------"
echo "redis2 IPAddress:"
redis2_ip=`docker inspect redis2 | grep "IPv4" | egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`
echo $redis2_ip;
echo "------------------------------"
echo "redis3 IPAddress:"
redis3_ip=`docker inspect redis3 | grep "IPv4" | egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`
echo $redis3_ip;
echo "------------------------------"
echo "haproxy IPAddress:"
haproxy_ip=`docker inspect haproxy | grep "IPv4" | egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`
echo $haproxy_ip;
echo "------------------------------"

echo "redis1:"
docker exec -it redis1 redis-cli info Replication | grep role
echo "redis2:"
docker exec -it redis2 redis-cli info Replication | grep role
echo "redis3:"
docker exec -it redis3 redis-cli info Replication | grep role
```
這個是我自己所寫的腳本，可以詳細知道目前服務的角色轉移狀況。

<br>

### sentinel/Docker-compose.yaml

```yml
version: '3.8'

services:
  haproxy:
    image: haproxy
    container_name: haproxy
    volumes:
      - ../docker-volume/haproxy/:/usr/local/etc/haproxy
    environment:
      - TZ=Asia/Taipei
    networks:
      HAProxy_Redis:
        ipv4_address: 172.20.0.20
    ports:
      - 16379:6379
      - 8404:8404

  sentinel1:
    image: redis
    container_name: redis-sentinel-1
    networks:
      HAProxy_Redis:
    ports:
      - 26379:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel1:/usr/local/etc/redis/
      - ../log/sentinel1:/var/log/redis/
    environment:
      - TZ=Asia/Taipei

  sentinel2:
    image: redis
    container_name: redis-sentinel-2
    networks:
      HAProxy_Redis:
    ports:
      - 26380:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel2:/usr/local/etc/redis/
      - ../log/sentinel2:/var/log/redis/
    environment:
      - TZ=Asia/Taipei

  sentinel3:
    image: redis
    container_name: redis-sentinel-3
    networks:
      HAProxy_Redis:
    ports:
      - 26381:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel3:/usr/local/etc/redis/
      - ../log/sentinel3:/var/log/redis/
    environment:
      - TZ=Asia/Taipei

networks:
  HAProxy_Redis:
    external:
      name: HAProxy_Redis
```

<br>

### sentinel/sentine.conf

因為 sentine 內容都基本上相同，所以舉一個來說明：

```conf
port 26379

logfile "/var/log/redis/redis-sentinel.log"

protected-mode no

#設定要監控的 Master，最後的 2 代表判定客觀下線所需的哨兵數
sentinel monitor mymaster 172.20.0.11 6379 2

#哨兵 Ping 不到 Master 超過此毫秒數會認定主觀下線
sentinel down-after-milliseconds mymaster 5000
```
要設定指定的 Port sentine1 是 26379、sentine2 是 26380、sentine3 是 26381。接下來要設定要監控的 Master，最後的數字代表我們前面有提到客觀下線需要達到的哨兵數。以及主觀下線的時間跟 failover 超過的時間。

<br>

## 測試

我們先用 docker-compose up 來啟動 Docker-compose.yaml，接著再啟動 sentinel/Docker-compose.yaml：

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/yaml.png" width="700" caption="啟動 Docker-compose.yaml" >}}

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/sentinel-yaml.png" width="700" caption="啟動 sentinel/Docker-compose.yaml" >}}

由於為了測試，有先將所有容器設定好 IP，就不會像上一篇文章一樣要去抓 IP ，才能啟動 Sentinel。

<br>

這時候可以使用瀏覽器搜尋以下網址：
* `test.com:8404/redis`：HAProxy 監看平台(只取片段)。

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/haproxy8404.png" width="1200" caption="HAProxy 監控平台" >}}

<br>

* `test.com:8888/rw.php`：只會顯示 master，並且可讀可寫。

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/rw.png" width="600" caption="master  (redis1)" >}}

<br>

* `test.com:8888/r.php`：會顯示 master、slave，且可讀。

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/r1.png" width="600" caption="master  (redis1)" >}}

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/r3.png" width="600" caption="slave  (redis2)" >}}

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/r2.png" width="600" caption="slave  (redis3)" >}}

<br>

接下來可以執行：

```sh
$ sh redis.sh

redis1 IPAddress:
172.20.0.11
------------------------------
redis2 IPAddress:
172.20.0.12
------------------------------
redis3 IPAddress:
172.20.0.13
------------------------------
haproxy IPAddress:
172.20.0.20
------------------------------
redis1:
role:master
redis2:
role:slave
redis3:
role:slave
```

就會顯示三個 redis 的 IP 以及 haproxy 的 IP，這些都是已經寫在 Docker-compose.yaml 檔案內的，如果忘記的可以再往前看 ↑

接下來我們可以先一直 F5 `test.com:8888/r.php`，來模擬大量的讀取請求，如果發現網站內容一直在更換，就代表我們成功透過 HAProxy 做到負載平衡了，可以將讀取的需求分給三個服務做處理！那因為 `test.com:8888/rw.php` 他只會抓 master，所以刷新還是同一個 master。

<br>

還記得我們上次用 Redis 的哨兵模式嗎？那我們用它來搭配 Haproxy 會有什麼結果呢？

我們先使用 `docker stop` 來模擬服務中斷：

```sh
$ docker stop redis1

redis1
```

可以看到 `test.com:8404/redis` 原本綠色的 redis1 開始變成黃色，最後變成紅色：

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/haproxy8404-1.png" width="1200" caption="模擬中斷服務 HAProxy 監控平台" >}}

<br>

最後可以看到 Redis Sentinel 作動，將 master 轉移到 redis3：

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/haproxy8404-2.png" width="1200" caption="模擬中斷服務 HAProxy 監控平台" >}}

<br>

這時候我們再去看 `test.com:8888/rw.php` ，就會發現與剛剛的 master 不太一樣囉，因為已經變成 redis2 了！

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/rw-1.png" width="600" caption="master (redis2)" >}}

代表我們 HAProxy 也有成功將 master 給顯示出來！

<br>

我們再去看 `test.com:8888/r.php` ，就可以發現剩下 redis2 以及 redis 3 了，因為 redis1 被我們給暫停服務了，而且 redis2 變成 master！

<br>

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/r4.png" width="600" caption="master (redis2)" >}}

{{< figure src="/redis/redis-sentinel-docker-compose-haproxy/r2.png" width="600" caption="slave (redis3)" >}}


<br>

## 參考資料

HAProxy 首頁：[http://www.haproxy.org/](http://www.haproxy.org/)

HAproxy的安裝設定及範例：[https://tw511.com/a/01/6959.html](https://tw511.com/a/01/6959.html)

redis sentinel集群配置及haproxy配置：[https://www.cnblogs.com/tzm7614/p/5691912.html](https://www.cnblogs.com/tzm7614/p/5691912.html)

富人用 L4 Switch，窮人用 Linux HAProxy！：[https://blog.toright.com/posts/3967/%E5%AF%8C%E4%BA%BA%E7%94%A8-l4-switch%EF%BC%8C%E7%AA%AE%E4%BA%BA%E7%94%A8-linux-haproxy%EF%BC%81.html](https://blog.toright.com/posts/3967/%E5%AF%8C%E4%BA%BA%E7%94%A8-l4-switch%EF%BC%8C%E7%AA%AE%E4%BA%BA%E7%94%A8-linux-haproxy%EF%BC%81.html)

selcukusta/redis-sentinel-with-haproxy：[https://github.com/selcukusta/redis-sentinel-with-haproxy](https://github.com/selcukusta/redis-sentinel-with-haproxy)

How to Enable Health Checks in HAProxy：[https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy/](https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy/)