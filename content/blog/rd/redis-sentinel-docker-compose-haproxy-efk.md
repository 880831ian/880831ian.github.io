---
title: "用 EFK 收集容器日誌 (HAProxy、Redis Sentinel、Docker-compose)"
type: docs
weight: 9997
date: 2022-04-22
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---


前情提要：本篇是 [用 HAProxy 對 Redis 做負載平衡 (Redis Sentinel、Docker-compose)](../../redis/redis-sentinel-docker-compose-haproxy/)
 以及 [Redis 哨兵模式 (Sentinel) 搭配 Docker-compose 實作](../../redis/redis-sentinel-docker-compose/) 的後續文章，主要會優化原本的程式碼，並使用 EFK 來收集 LOG！

## 什麼是 EFK ?

隨著現在各種程式系統複雜度越來越高，特別是現在都往雲上開始作部署，當我們想要查看 log 的時候，不可能一個一個去登入各節點去查看 log，不僅效率低，也會有安全性的問題，所以不可能讓工程師直接去訪問每一個節點。

而且現在大規模的系統基本上都採用叢集的部署方式，意味著對每個 service，會啟動多個完全一樣的 POD 對外提供服務，每個 container 都會產生自己的 log，從產生 log 來看，你根本不知道是由哪個 POD 產生的，這樣對查看分佈式的日誌更加困難。

所以在雲時代，需要一個收集並分析 log 的解決方案。首先需要將分佈在各個角落的 log 統一收集到一個集中的地方，方便查看。收集之後，還可以進行各種統計以及分析，甚至用流行的大數據或機器學習的方法來進行分析。

<br>

所以誕生了 ELK 或是 EFK 的解決方式：

ELK 是由 Elasticsearch、Logstash、Kibana 所組成， EFK 是由 Elasticsearch、(Filebeats or Fluentd)、Kibana 所組成，兩者的差異在中間使用的開源程式，我會分別介紹一下每一個程式主要的用途：
* Elasticsearch：它是一個集中儲存 log 的地方，更重要的是它是一個全文檢索以及分析的引擎，它能讓用戶以近乎實時的方式來查看、分析海量的數據。
* Logstash、Filebeats、Fluentd：它們主要是收集分佈在各處的 log 並進行處理(Filebeats 僅收集)。
* Kibana：它是為 Elasticsearch 開發的前端 GUI，可以讓用戶很方便的以圖形化進行查詢 Elasticsearch 中儲存的數據，同時也提供各式各樣的模組可以使用。

<br>

Logstash、Filebeats、Fluentd 關係：

Filebeats 是一個輕量級收集本地 log 數據的方案，它僅能收集本地的 log，不能對 log 做處理，所以 Filebeats 通常會將 log 送到 Logstash 做進一步的處理。

那為什麼不直接使用 Logstash 來收集收集並處理 log 呢？

因為 Logstash 會消耗許多的記憶體，所以才會先透過 Filebeats 收集資料再傳給 Logstash 做處理。

另外 Filebeats、Logstash、Elasticsearch 和 Kibana 都是屬於同一家公司的開源項目：[https://www.elastic.co/guide/](https://www.elastic.co/guide/index.html)

而 Fluentd 則是另一家公司的開源項目：[https://docs.fluentd.org/](https://docs.fluentd.org/)

那我們在這邊就使用 Fluentd 來做我們的 EFK 示範：

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/efk.jpeg" width="700" caption="EFK 示意圖 [EFK Stack: Elasticsearch, Fluentd and Kibana on Docker](https://aesher9o1.medium.com/efk-stack-elasticsearch-fluentd-and-kibana-on-docker-be60597fa99)" >}}

<br>

上面這張圖代表我們會有很多 Docker 容器的 log (也就是之前的 redis 以及其他 nginx 等的容器)，會先透過 Fluentd 收集並處理各容器的 log 在傳送到 Elasticsearch 集中儲存，再使用 Kibana 圖形化介面來查詢或檢索儲在 Elasticsearch 的 log。

<br>

## 實作

那接下來會使用 Docker-compose 實作 EFK 的 LOG 分析，[範例程式連結 點我](https://github.com/880831ian/docker-compose-redis-sentinel-haproxy-efk) 😘

版本資訊
* macOS：11.6
* Docker：Docker version 20.10.12, build e91ed57
* Nginx：1.20
* PHP：7.4-fpm
* Redis：6.2.6
* HAProxy：HAProxy version 2.5.5-384c5c5 2022/03/14 - https://haproxy.org/
* Elasticsearch：8.1.3
* Fluentd：v1.14
* Kibana：8.1.3

<br>

### 檔案結構

```
.
├── docker-volume
│   ├── fluentd
│   │   └── fluent.conf
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
├── docker.sh
├── efk
│   └── Docker-compose.yaml
├── fluentd
│   └── Dockerfile
├── haproxy_sentinel
│   ├── Docker-compose.yaml
│   ├── sentinel1
│   │   └── sentinel.conf
│   ├── sentinel2
│   │   └── sentinel.conf
│   └── sentinel3
│       └── sentinel.conf
├── nginx_php_redis
│   └── Docker-compose.yaml
└── php
    └── Dockerfile
```

<br>

這是主要的結構，簡單說明一下：(檔案越來越多了XD，這次把每一個都有分類，所以結構會與之前不太相同)

*  docker-volume/fluentd/fluent.conf：fluentd 的設定檔。
*  docker-volume/haproxy/haproxy.cfg：haproxy 的設定檔。
*  docker-volume/nginx/nginx.conf：nginx 的設定檔。
*  docker-volume/php/(r.php、rw.php)：測試用檔案。
*  docker-volume/redis/redis.conf：redis 的設定檔。
*  docker-volume/redis/(redis1、redis2、redis3)：放 redis 的資料。
*  docker.sh：是我另外多寫的腳本，可以查看相對應的角色。
*  efk/Docker-compose.yaml：會放置要產生的 elasticsearch、kibana、fluentd 的容器設定檔。
*  fluentd/Dockerfile：因為 fluentd 需要另外安裝 fluent-plugin-elasticsearch 才能使用，所以用 Dockerfile 另外寫 fluent 的映像檔。
*  haproxy_sentinel/Docker-compose.yaml：會放置要產生的 haproxy、sentinel1、sentinel2、sentinel3 的容器設定檔。
*  haproxy_sentinel/(sentinel1、sentinel2、sentinel3)/.conf：哨兵的設定檔。
*  nginx_php_redis/Docker-compose.yaml：會放置要產生的 nginx、php、redis1、redis2、redis3 的容器設定檔。
*  php/Dokcerfile：因為在 php 要使用 redis 需要多安裝一些設定，所以用 Dockerfile 另外寫 PHP 的映像檔。

<br>

那我們就依照安裝的設定開始說明：(這邊只說明與上一篇不同地方，重複的就請大家回去看之前的文章)
<br>

### docker-volume/fluentd/fluent.conf

```conf
<source>
  @type forward
  bind 0.0.0.0
  port 24224
</source>

<match *.**>
  @type copy

  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>

  <store>
    @type stdout
  </store>
</match>
```
這是 fluent 的設定檔，可以在這邊做自訂的設定，例如 host、port、預設日期、tag_key 等等。

<br>


### fluentd/Dockerfile

```Dockerfile
FROM fluent/fluentd:v1.14
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch"]
```
因為我們需要先安裝 fluent-plugin-elasticsearch 才可以讓 fluentd 來使用 elasticsearch，所以多寫一個 Dockerfile 來設定。

<br>


### efk/Docker-compose.yaml

```yml
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:8.1.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false # elasticsearch 8.x版本後會自動開啟SSL
    networks:
      efk_network:
        ipv4_address: 172.20.0.3
    ports:
      - 9200:9200

  kibana:
    image: kibana:8.1.3
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    # - I18N_LOCALE=zh-CN
    networks:
      efk_network:
        ipv4_address: 172.20.0.4
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch

  fluentd:
    build: ../fluentd
    container_name: fluentd
    volumes:
      - ../docker-volume/fluentd:/fluentd/etc
    depends_on:
      - elasticsearch
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    networks:
      efk_network:
        ipv4_address: 172.20.0.5

networks:
  efk_network:
    driver: bridge
    name: efk_network
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```
這邊是設定 EFK 三個容器的檔案，比較特別的是 elasticsearch 在 8.x 版本後會自動開啟 SSL 連線，所以沒有使用的或是在測試中的要先把它關掉，不然會連不上！，kibana 它支援 I18N 多語系，但目前沒有繁體中文，所以想要看中文的可以切成 zh-CH 的簡體中文來使用，其他就是基本的設定 Posts 以及我們全部設定好 IP，在後續測試會比較方便～

<br>

### nginx_php_redis/Docker-compose.yaml

```yml
version: "3.8"

services:
  nginx:
    image: nginx
    container_name: nginx
    networks:
      efk_network:
    ports:
      - 8080:80
    volumes:
      - ../docker-volume/nginx/:/etc/nginx/conf.d/
    environment:
      - TZ=Asia/Taipei
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: nginx

  php:
    build: ../php
    container_name: php
    networks:
      efk_network:
    expose:
      - 9000
    volumes:
      - ../docker-volume/php/:/var/www/html
    environment:
      - TZ=Asia/Taipei
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: php

  redis1:
    image: redis
    container_name: redis1
    command: redis-server /usr/local/etc/redis/redis.conf --appendonly yes
    volumes:
      - ../docker-volume/redis/redis1/:/data
      - ../docker-volume/redis/:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      efk_network:
        ipv4_address: 172.20.0.11
    ports:
      - 6379:6379
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: redis1

  redis2:
    image: redis
    container_name: redis2
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis1 6379 --appendonly yes
    volumes:
      - ../docker-volume/redis/redis2/:/data
      - ../docker-volume/redis/:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      efk_network:
        ipv4_address: 172.20.0.12
    ports:
      - 6380:6379
    depends_on:
      - redis1
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: redis2

  redis3:
    image: redis
    container_name: redis3
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis1 6379 --appendonly yes
    volumes:
      - ../docker-volume/redis/redis3/:/data
      - ../docker-volume/redis/:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    networks:
      efk_network:
        ipv4_address: 172.20.0.13
    ports:
      - 6381:6379
    depends_on:
      - redis1
      - redis2
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: redis3

networks:
  efk_network:
    external:
      name: efk_network
```
那這邊基本上都與上一篇一樣，沒有修改特別的地方，只有修改網路名稱以及：

```yml
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: nginx
```
我們要把每一個容器的 log 進行收集與處理，所以我們使用 logging 然後 driver 選擇 fluentd，並且要設定 fluentd 的 IP 位置以及每一個容器可設定不同的 tag 方便我們查詢。


<br>

### haproxy_sentinel/Docker-compose.yaml

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
      efk_network:
        ipv4_address: 172.20.0.20
    ports:
      - 16379:6379
      - 8404:8404
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: haproxy

  sentinel1:
    image: redis
    container_name: redis-sentinel-1
    networks:
      efk_network:
    ports:
      - 26379:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel1:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: sentinel1

  sentinel2:
    image: redis
    container_name: redis-sentinel-2
    networks:
      efk_network:
    ports:
      - 26380:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel2:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: sentinel2

  sentinel3:
    image: redis
    container_name: redis-sentinel-3
    networks:
      efk_network:
    ports:
      - 26381:26379
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
    volumes:
      - ./sentinel3:/usr/local/etc/redis/
    environment:
      - TZ=Asia/Taipei
    logging:
      driver: "fluentd"
      options:
        fluentd-address: 172.20.0.5:24224
        tag: sentinel3

networks:
  efk_network:
    external:
      name: efk_network
```
與前面一樣，多了一個 logging 來設定 fluentd 位置以及 tag。

<br>

## 測試

我們先用 docker-compose up 來啟動 efk/Docker-compose.yaml，接著再啟動 nginx_php_redis/Docker-compose.yaml，最後啟動 haproxy_sentinel/Docker-compose.yaml：

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/efk-compose.png" width="700" caption="啟動 efk/Docker-compose.yaml" >}}

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/nginx-php-redis-compose.png" width="700" caption="啟動 nginx_php_redis/Docker-compose.yaml" >}}

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/haproxy-sentinel-compose.png" width="700" caption="啟動 efk/Docker-compose.yaml" >}}

(啟動 efk 時要等他跑完，因為他需要啟動一陣子，如果還沒等他啟動完畢就啟動下一個，會導致 fluentd 的 fluentd-address 尚未設定好，導致啟動錯誤)

<br>

這時候可以使用瀏覽器搜尋以下網址：
* `test.com:5601`：Kibana GUI 頁面。

<br>

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/gui.png" width="1000" caption="kibana 設定" >}}

<br>

如果連線成功進來，就代表我們有安裝好 Elasticsearch 以及 kibana，那我們來做一些設定，讓我們可以在 kibana 上面看到 log。

先點選左邊的欄位，點選 "Stack Management" ，可以看到目前的 kibana 的版本，再點選左邊的 "Kibana > Date Views"：

<br>

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/kibana.png" width="1000" caption="kibana 設定" >}}

然後會跳出一個視窗點選 "Create data view"，在 name 欄位輸入 fluentd* (右側有 fluentd 加時間，代表我們接 fluentd 有成功)，按下 "Create data view"

<br>

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/fluentd.png" width="1000" caption="kibana 設定" >}}

接著點選左側的欄位，點選 "Analytics > Discover" ，就可以看到我們目前所有的 log 囉！

<br>

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/fluentd-1.png" width="1000" caption="kibana 設定" >}}

{{< figure src="/rd/redis-sentinel-docker-compose-haproxy-efk/log.png" width="1000" caption="kibana Analytics > Discover" >}}

<br>

## 參考資料

elastic 官網：[https://www.elastic.co/](https://www.elastic.co/)

fluentd 官網：[https://www.fluentd.org/](https://www.fluentd.org/)

Build the EFK system used for simulating logging server on Docker：[https://stackoverflow.com/questions/71155142/build-the-efk-system-used-for-simulating-logging-server-on-docker](https://stackoverflow.com/questions/71155142/build-the-efk-system-used-for-simulating-logging-server-on-docker)

開源日誌管理方案ELK 和EFK 的區別：[https://wsgzao.github.io/post/efk/](https://wsgzao.github.io/post/efk/)

