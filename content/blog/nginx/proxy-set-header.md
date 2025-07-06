---
title: "Nginx Proxy Set Header 注意事項"
type: docs
weight: 9997
description: Nginx Proxy Set Header 注意事項
images:
  - nginx/proxy-set-header/og.webp
date: 2024-08-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 說明

在 Nginx 中，我們可以透過 `proxy_set_header` 來設定 header，這樣子可以讓我們在 proxy 的過程中，將一些 header 帶到後端的服務上，除了新增，也可以蓋掉前面原有的 header。

通常都長得像：

```nginx
server {
  listen 80;
  server_name localhost;

  location / {
    proxy_pass http://your_service;
    proxy_set_header Host $host;
  }
}
```

## 注意事項

那假設我有很多個 path 都需要帶同樣的 header，例如：

```nginx
server {
  listen 80;
  server_name localhost;

  location /api {
    proxy_pass http://api-svc;
    proxy_set_header Host $host;
  }

  location /blog {
    proxy_pass http://blog-svc;
    proxy_set_header Host $host;
  }

  location /ian {
    proxy_pass http://ian-svc;
    proxy_set_header Host $host;
  }
}
```

<br>

那我們可以將 `proxy_set_header` 改寫到 `server` block 中，這樣就可以避免重複寫很多次。

```nginx
server {
  listen 80;
  server_name localhost;

  proxy_set_header Host $host;

  location /api {
    proxy_pass http://api-svc;
  }

  location /blog {
    proxy_pass http://blog-svc;
  }

  location /ian {
    proxy_pass http://ian-svc;
  }
}
```

<br>

官方文件說明：

> Allows redefining or appending fields to the request header passed to the proxied server. The value can contain text, variables, and their combinations.

<br>

因爲 `proxy_set_header` 是一個全域設定，它可以寫在 `http`、`server`、`location` block 中，所以如果有多台 server 都需要帶同樣的 header，可以寫在 `http` block 中。

<br>

{{< figure src="/nginx/proxy-set-header/1.webp" width="800" caption="官方文件介紹 proxy_set_header" >}}

<br>

<b>但是</b> (っ・Д・)っ 如果原本的 `location` 有設定 `proxy_set_header`，那麼在 `server` block 中的 `proxy_set_header` <b>是沒有辦法同時使用</b>，後端只會收到 `location` 的 `proxy_set_header`，所以要注意這個地方。

<br>

```nginx
server {
  listen 80;
  server_name localhost;

  proxy_set_header Host $host;

  location /api {
    proxy_pass http://api-svc;
  }

  location /blog {
    proxy_pass http://blog-svc;
  }

  location /ian {
    proxy_pass http://ian-svc;
    proxy_set_header Test helloworld;
  }
}
```

上面這個例子，`/api` 跟 `/blog` 都會帶 `Host` header，但是 `/ian` <b>只會帶 `Test` header</b>。

<br>

官方文件說明：

> These directives are inherited from the previous configuration level if and only if there are no proxy_set_header directives defined on the current level. By default, only two fields are redefined:

<br>

<b>只有當前層級沒有定義 `proxy_set_header`，才會繼承上一層級，所以如果當前層級有定義，就繼承不到上一層級的設定。</b>

<br>

## 參考資料

关于 proxy_set_header 的注意点：[https://keepmoving.ren/nginx/about-proxy-set-header/](https://keepmoving.ren/nginx/about-proxy-set-header/)

Module ngx_http_proxy_module#proxy_set_header：[https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)
