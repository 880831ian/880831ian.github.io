---
title: "如何在 Nginx 下實作第一個 PHP 留言板 RESTful API"
type: docs
weight: 9998
description: 如何在 Nginx 下實作第一個 PHP 留言板 RESTful API
images:
  - php-laravel/nginx-php-restful-api/og.webp
date: 2022-02-24
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

由於網路上文章幾乎都是使用 Apache 來實作第一個 PHP RESTful API，也比較沒有中文的介紹，剛好這次也還在學習，所以把學習的紀錄分享給大家，這次會搭配實作留言板來說明，那本次教學內容採用的是

<!--more-->

* macOS 11.6
* Nginx 1.21.6
* PHP 8.1.2
* 無框架
* 無物件導向

來做教學以及示範，那我們先來簡單了解一下什麼是RESTful API：

## 什麼是RESTful API
### 定義
REST ，指得是一組架構約束條件和原則，符合 REST 設計風格的Web API 稱為 RESTful API，主要以下面三點為定義

* 直觀簡單的資源網址 URL 比如：http://example.com/resources
* 對資源的操作：Web 服務在該資源上所支持的請求方法，比如：POST、GET、PUT、PATCH、DELETE
* 傳輸的資源：Web 服務接受與返回的類型，比如：JSON、XML

分別說明每一項定義

#### 直觀簡單的資源網址 URL

<br>

{{< figure src="/php-laravel/nginx-php-restful-api/REST_URL_structure.png" width="700" caption="RESTful API URL 結構 (`Best Practices for RESTful API Design`)" >}}

<br>

可以看到這張圖後面的 Endpoint，分別代表應用服務 (Application context) 、 版本 (Version)、 資源(Resource)、 參數(Parameter) 。

* 應用服務：可以取名 api 或是 restful-api 。
* 版本：可以對 API 進行版本控制，可以有升級的服務，也不會對現有的 API 造成影響。
* 資源：要使用名詞而非動詞來命名，且建議使用複數
	* 壞的命名(以我們要實作的留言板查詢、新增、修改、刪除示範)
		* 查詢 /selectmessage
		* 新增 /createmessage
		* 修改 /updatemessage
		* 刪除 /deletemessage
	* 好的命名
 		* 查詢 GET => /messages (回傳所有留言)
		* 新增 POST => /messages (新增留言)
		* 修改 PUT => /messages/1 (修改單筆留言全部資料)
		* 修改 PATCH => /messages/1 (修改單筆留言有更動資料)
		* 刪除 DELETE => /messages/1 (刪除單筆留言)

#### 對資源的操作

RESTful API 傳送時，會依照我們所定的 HTTP Request Method 請求方法，那主要有以下 5 種的 Method

* GET : 此方法只能向指定的資源要求取得資料，並不會更動內部的資料
* POST：向指定的資源要求新增資料
* PUT：向指定的資源要求修改資料內容
* PATCH：向指定的資源要求修改部分資料內容
* DELETE：向指定的資源要求刪除資料內容

#### 傳輸的資源

剛剛是客戶端對伺服器的請求，那我們也要針對他的請求給予對應的回應 Response ，那回應的格式有 JSON 、XML ，但都以 JSON 較為普遍，所以我們後續實作也會以 JSON 作為我們的 Response 。

回應的格式內容會依照文件所自訂，但基本上都會回傳狀態碼 http status code ，下面就用表格的方式來說明常用的狀態碼、以及狀態碼對應的意思跟我們在 RESTful API 使用的場景。

| 狀態碼 | 名稱 | 說明 | RESRful API 使用場景 |
| ---- | ---- | ---- | ---- |
| 200 | OK | 請求成功 | GET、PUT 方法，取得資料 |
| 201 | Created | 新的資源已建立 | POST 方法，新增資料 |
| 204 | No Content | 沒有返回任何內容 | DELETE 方法，刪除資料 |
| 400 | Bad Request | 請求不正確 | |
| 401 | Unauthorized | 用戶需要認證 | |
| 403 | Forbidden | 禁止訪問，與401 不同的是，用戶已經認證，但沒有權限 | |
| 404 | Not Found | 沒有找到指定的資源 | GET、PUT、DELETE 方法，該資料不存在
| 500 | Internal Server Error | 伺服器發生錯誤 | |

( 參考[RESTful Web API 設計指南](https://www.footmark.com.tw/news/programming-language/design/restful-webapi-design-guide/) )

### 為什麼要使用 RESTful API

1. REST 使用所有標準 HTTP 協議方法 - GET、POST、PUT、DELETE、PATCH ，以及更具體的 URL 。
2. REST 將客戶端與伺服器之間的操作分開 - 他允許開發人員使用它們希望使用的任何前端技術，包含像是 AngularJS、Bootstrap、VUE、Ember、ReactJS、CycleJS、ExtJS、PHP、.NET 等等，極大提高了可移植性。
3. REST 針對 web 進行了優化 - 因為它依賴 HTTP 協議。此外，由於它的主要數據格式是 JSON，它基本上兼容所有互聯網瀏覽器。

### RESTful API 安全性

我們在設計 API 時，要確保 RESTful API 的安全性，像是 PUT 、 DELETE 更新刪除這類型的操作並不安全，沒有權限認證下任何人都可以使用來操作，總不可能讓陌生人去更動我們的資料吧？因此安全性沒有做足夠容易成為駭客攻擊的對象。

* 要怎麼做比較安全？

	例如我們的留言板：登入後(帳號密碼) > 伺服器驗證成功並取的一組的 API token > 就可以使用這組 token 來訪問 API 資源，也就知道是誰去對這筆資料進行操作

## 實作開始

### 1. 修改 Nginx.conf

我們這邊就直接對架設好的 Nginx 設定檔來修改，如果不清楚要怎麼架設的，這方面網路上蠻多文章，如果還是不清楚，可以留言告訴我，我在另外寫一篇文章來介紹環境安裝。

為什麼要先修改 conf 呢？修改的目的就是要讓原本網址

```sh
http://localhost/api/index.php?messages=all
```
變成

```sh
http://localhost/api/messages
```
來實現我們在介紹時所說的直觀簡單的資源網址，我說明一下上面網址的關係，

* http://localhost 是因為我們本地端運行，如果已經上線的，那就是你自己的網址，

* /api 是我來放這支 api 的目錄，他位於網頁的根目錄下方(詳細的配置下方會附上)，

* index.php 是我來放這支 api 的網頁，後面的 messages 是我們這次的資源，後面可以接我想要處理的參數

那要怎麼達成讓網址變成我們想要呈現的直觀簡單的網址呢，這時候我們就要使用 Nginx.conf 來做設定，網路上的文章比較多的是 apache 的 htaccess，我一開始還以為 Nginx 也可以使用 htaccess ，試了半天才知道，在 Nginx 要改用 Nginx.conf 來設定，那我下方會附上我原本的 .htaccess 以及轉換後的 conf，那接下來我一步一步來介紹


> [!NOTE] 線上轉換工具
>網路上蠻多工具可以線上轉換，我推薦可以用 [Apache htaccess to Nginx converter
](https://winginx.com/en/htaccess)


* Apache .htaccess 檔案

```.htaccess
RewriteRule ^/api/messages$   /api/index.php?messages=all [nc,qsa]
RewriteRule ^/api/messages/(\d+)$   /api/index.php?messages=$1 [nc,qsa]
```
這是我會修改部分的 Nginx.conf ，可以看到最後兩條 location，這邊就是我們轉換過來的設定


* Nginx.conf 檔案(片段)

```nginx
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm index.php;
        }

		location = /api/messages {
		  rewrite ^(.*)$ /api/index.php?messages=all;
		}

		location /api {
		  rewrite ^/api/messages/(\d+)$ /api/index.php?messages=$1;
		}
```


他的意思代表的我們網址路徑是在 /api/messages ，我們讓原本 /api/index.php?messages 這段變成 /api/messages/ 的設定，設定好記得先下  ```sh sudo nginx -t ``` 指令來檢查一下Nginx.conf檔案是不是都正確，再用 ```sh sudo nginx -s reload ``` 指令重新啟動 Nginx  (是否成功，後面會帶到如何做測試，就繼續一起往下看吧！)


```mysql
CREATE TABLE `messages` (
  `id` int NOT NULL,
  `name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `message` text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `sendtime` datetime NOT NULL,
  `version` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```


## 參考資料

[PHP實現RESTful風格的API實例](https://www.cnblogs.com/luyucheng/p/6016801.html)

[[XAMPP][PHP] 製作Restful API
](https://cg2010studio.com/2016/08/06/xamppphp-%E8%A3%BD%E4%BD%9Crestful-api/)

[Best Practices for RESTful API Design
](https://avaldes.com/best-practices-for-restful-api-design/)

[RESTful API與MVC名詞介紹](https://ithelp.ithome.com.tw/articles/10191925)