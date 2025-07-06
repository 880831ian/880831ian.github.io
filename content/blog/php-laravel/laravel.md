---
title: "Laravel 介紹 (使用 Laravel 從零到有開發出一個留言板功能並搭配 RESTful API 來實現 CRUD)"
type: docs
weight: 9998
description: Laravel 介紹 (使用 Laravel 從零到有開發出一個留言板功能並搭配 RESTful API 來實現 CRUD)
images:
  - php-laravel/laravel/og.webp
date: 2022-03-02
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

本文章會介紹什麼是 Laravel ，以及它有什麼特別之處，可以讓它在2015年被評為最受歡迎的 PHP  框架第一名，並說明為什麼要使用框架，最後實作一個留言板功能搭配 RESTful API 來實現 CRUD。

就像是 Laravel 官網的大標題，『 The PHP Framework for Web Artisans 為網頁藝術家創造的 PHP 框架 』，那再了解 Laravel 之前，先來介紹一下框架是什麼：

## 框架是什麼

* 框架 (Framework) 是一個被設計用來完成特定任務的規範，程式開發人員必須遵從這個規則來開發。
* 目前大多數的框架都是參考 MVC 架構的概念來做設計，原因是在早期開發網頁時，都是直接將 HTML 以及 PHP 混合再一起的方式來編寫，雖然開發上很方便，但在後面維護或是新增功能上，會十分不便利，例如：單純要修改網站畫面的元件時，還需要從混雜的程式碼中找到要修改的元件，也很容易不小心修改到其他的功能。

### MVC

* 於是有人想到把這些各自的任務區分開來，MVC 架構分別代表的是
	* M (Model) : 屬於資料的部分，可能是商用邏輯或是資料庫的存取。
	* V (View)：屬於顯示的部分，像是 HTML、CSS 等
	* C (Controllor)：會針對請求做出回應或是處理，例如從 Model 取出資料，並顯示在 View 上面

<br>

{{< figure src="/php-laravel/laravel/mvc.webp" width="700" caption="MVC 架構 [一起走向MVC(上)](https://bak.infolight.com/new/ShareDetail.aspx?DocumentID=NDUz)" >}}

<br>

## Laravel 框架

Laravel 是基於上面所說的 MVC 架構打造的框架，並且設計出許多可以讓開發者更有效率的工具。

* Artisan：提供許多指令，讓你可以使用這些指令，來快速完成許多的任務 (有列出部分指令於 [Artisan 相關指令](#artisan-相關指令))。
* Routing 路由：管理網址與頁面的路徑指定，辨識傳入的 request 傳送至對應的 Controller，回傳指定的 View。
* Blade 模板引擎 (View)：模板解析工具， 是 Laravel 所使用的模板引擎，Blade 會將 PHP 及 HTML 完整分離的工具。
* Auth 認證：透過 Laravel 預設提供的認證讓 Controller 快速解決經常會用到的驗證需求，像是：使用者註冊、使用者認證、重置密碼的 e-mail 連結、重置密碼邏輯等。
* Eloquent ORM 物件關聯對映 (Model)：Laravel 預設的 ORM，將資料庫的欄位映射成物件 (Ｍodel 模型)，只需要用 PHP 的語法，不需要撰寫 SQL 指令就可以用物件的方式讀取欄位資訊以及與資料庫的互動。
* Migration 資料庫遷移：聽起來很像是更換資料庫，但他其實算是一種資料庫的版本控制，可以讓團隊在修改資料庫結構的同時，保持彼此的進度ㄧ致，通常會結合[Schema 結構生成器](#schema-結構生成器)一起使用，可以簡單管理資料庫結構。

> [!NOTE] 小知識
>ORM：英文叫 Object Relational Mapping，翻譯成中文為物件關聯對映。在網站開發結構中，是在資料庫和Model 資料容器兩者之間，簡單來說，它是可以讓開發者更簡單、安全的方式從資料庫讀取資料，因為 ORM 的特性是可以透過物件導向程式語言去操作資料庫。
* 優點
	1. 安全性：可以避免 SQL injection，遇到奇怪的值，會自動擋掉。
	2. 簡化性：可以將原本 SQL ```Select * From users``` 等指令透過物件導向程式語言去操作資料庫。
	3. 通用性：因為 ORM 是程式語言和資料庫之間的關係，就算有要轉換資料庫，也比較不會遇到要修改程式的狀況。
* 缺點
	1. 效能：為了要達成方便性，通常都會犧牲到效能的問題，因為等於多了『把程式語言轉譯成SQL語言』這項工作。
	2. 學習曲線高：對於初學者來說，ORM 需要融合 SQL 語言以及程式語言兩種不同的概念以及語法，會比單純學 SQL 語法還較複雜。
	3. 複雜查詢維護性低：有些 SQL 語法 ORM 沒有支援，所以導致有些情況還是需要導入原生 SQL 寫法。

(資料來源：[資料庫設計概念 - ORM](https://ithelp.ithome.com.tw/articles/10207752))

## PHP 常見框架對比

雖然本次介紹的主題是 Laravel 但我們也簡單介紹一下其他的框架優點，以及附上表格讓大家可以更清楚。

| 差別 | Laravel | Symfony | CodeIgniter | CakePHP | Zend 2 |
| --- | :---: | :---: | :---: | :---: | :---: |
| 許可 License | MIT | MIT | BSD | BSD | MIT |
| 人氣排名 Popularity<br>(2022-03 Google 搜尋熱門程度)  | 第一名 | 第二名 | 第三名 | 第四名 | 第五名 |
| 表現 | 慢 | 慢 | 快 | 慢 | 慢 |
| 模板 | Blade | Twig | PHP | 內建 | PHP |
| 資料庫 | MySQL <br> PostgreSQL <br> SQLite <br> SQL Server | MySQL <br> PostgreSQL <br> SQLite <br> SQL Server <br> Oracle | MySQL <br> PostgreSQL <br> SQLite <br> SQL Server <br> Oracle | MySQL <br> PostgreSQL <br> SQLite <br> SQL Server <br> Oracle | MySQL <br> PostgreSQL <br> SQLite <br> SQL Server <br> Oracle |
| 物件關係對映 | 有 | Doctrine | 沒有內建 |  有 | Doctrine |
| 測試 | PHPUnit | PHPUnit | 內建 |  PHPUnit | PHPUnit |

### Laravel

該框架可能是Web 開發人員中最受歡迎的框架。Laravel 是一個免費的開源PHP 框架，適用於Web 應用程序開發，且適用於移動應用程序場。(上面有介紹過，這邊就簡化)

**Laravel 框架優點**
* 易於學習
* 無縫數據遷移
* 在 PHP 社群中很受歡迎
* 支持 MVC 架構
* 大量學習素材 (文件、圖檔、影片教程)
* 模板引擎
* 簡單的單元測試

### Symfony

該框架是一個廣泛的PHP MVC 框架，目前Symfony 已經成為一個可靠和成熟的平台框架。

Symfony 非常穩定、文檔齊全、性能卓越。這些特點使Symfony 成為開發大型企業項目的完美選擇。

使 Symfony 成為 PHP 框架中獨一無二的特性之一是它的可重用 PHP 組件。使用可重用組件，開發時間減少了許多模塊，如表單創建、對象配置、模板等。

可以直接從舊組件構建，節約了大量成本。Symfony 易於在大多數平台上安裝和配置，並且可以獨立於數據庫引擎。它具有高度的靈活性，可以與 Drupal 等大型項目集成

**Symfony 框架優點**

* 官方長期技術支持
* 內置測試功能
* 豐富的框架內置功能
* 官方培訓課程和認證

### CodeIgniter

該框架可能是最適合開發動態網站的PHP 框架。它是一個非常簡單的輕量級PHP 框架。

它的大小只有2 MB 左右（包括文檔）。因此，CodeIgniter 本身俱有最小的佔用空間，它允許Web 開發人員添加第三方插件來開發更複雜的功能。

CodeIgniter 還提供了幾個預構建的模塊，用於為Web 開發創建健壯的、可重用的組件。由於設置過程簡單，這個PHP 框架非常適合初學者。

**CodeIgniter 框架優點**

* MVC 架構
* Top-Notch 錯誤處理
* 提供卓越的性能
* 包中提供了幾種工具
* 內置安全工具
* 優秀的文檔

### CakePHP

該框架對個人是完全免費，並提供付費的商業用途。它將幫助您開發功能豐富且視覺上令人印象深刻的網站。

CakePHP 起初是一個簡單而優雅的工具包，在過去的15 年裡它變得更加強大。由於它的CRUD（創建、讀取、更新和刪除）框架，CakePHP 是最容易學習的框架。

使用CakePHP 部署Web 網站是"小菜一碟"，只需要一個Web 服務器和CakePHP 框架的副本。

**CakePHP 框架優點**

* 插件和組件的簡易擴展
* 適當的類繼承
* 零配置
* 現代框架
* 支持AJAX
* 快速構建
* 內置驗證


### Zend 2

該框架是一個完整的面向對象的PHP 框架。這個PHP 框架是可定制的，對於需要添加項目特定功能的開發人員來說，這是一個好處。

Zend 構建於敏捷方法之上，可幫助開發人員為大型客戶創建、高質量的Web 應用程序的框架。它非常適合複雜的企業級項目，Zend 主要關注安全性、性能和可擴展性。

Zend 框架主要受大型IT 企業和銀行等金融機構的青睞。

**Zend 2 框架優點**

* MVC 組件
* 卓越的前端技術支持工具
* 大型開發者社區
* 簡單的雲API
* 支持第三方組件
* 數據加密
* 支持AJAX
* 會話管理


## 實作

> [!NOTE] Laravel 官網
> 詳細可以參考 [Laravel 官網](https://laravel.com/docs/5.4/migrations#creating-columns) 的文件，裡面有更詳細的欄位型態說明！

### 環境設定

要安裝 Laravel 框架有一些系統的要求，要先安裝 PHP ，請參考[ PHP 官網的安裝教學](https://www.php.net/manual/zh/install.php)

安裝完後可以使用來檢查是否安裝成功

```sh
$ php --version
PHP 7.1.33 (cli) (built: Jan 20 2022 04:04:37) ( NTS )
Copyright (c) 1997-2018 The PHP Group
```
<br>

再安裝 Composer ，請參考 [ Composer 官網的安裝教學](https://getcomposer.org/download/)


```sh
$ composer --version
Composer version 2.2.7
```
<br>
都安裝好後，我們就要來安裝 Laravel 框架囉，根據 Laravel 官網安裝步驟，先用 Composer 來安裝

```sh
$ composer global require "laravel/installer"
```
<br>
安裝後一樣先來檢查是否安裝成功

```sh
$ laravel --version
zsh: command not found: laravel
```
<br>

發現 zsh 找不到 laravel 這個命令，那我們看一下官網怎麼說 ( zsh 是我所使用的 bash ，這邊會依照安裝環境而有所不同)

`Make sure to place the $HOME/.composer/vendor/bin directory (or the equivalent directory for your OS) in your $PATH so the laravel executable can be located by your system.`

<br>
代表我們要將 Laravel 的目錄，放到 $PATH ，系統才可以使用 laravel 來執行檔案。

以 macOS 來說， Laravel installer 的目錄都會放在 `$HOME/.composer/vendor/bin` ，我們在根目錄建立一個 `.bash_profile` 的檔案來放置我們的 $PATH。


```sh
export PATH="$HOME/.composer/vendor/bin:$PATH"
```

<br>
輸入上面的 PATH ，記得儲存離開後要用這個指令來更新一下檔案

```sh
$ source .bash_profile
````

<br>

我們再次檢查是否安裝好 Laravel

```sh
$ laravel --version
Laravel Installer 2.3.0
```
<br>

因為我們後續會使用到資料庫的功能，所以也可以先把他安裝好 [ＭySQL 官網的安裝教學](https://formulae.brew.sh/formula/mysql)

```sh
$ mysql -V
mysql  Ver 8.0.28 for macos11.6 on x86_64 (Homebrew)
```

<br>

#### 本次實作の版本配置

* macOS 11.6
* PHP 7.1.33
* MySQL 8.0.28
* Composer 2.2.7
* Laravel Installer 2.3.0
* Laravel Framework 5.4.36


### 新增專案

安裝成功後，就可以開始來使用 Laravel 框架自動生成檔案囉！要怎麼做呢？就接著看下去吧

可以使用 Laravel 來新增專案

```sh
$ laravel new 專案名稱
```
<br>

也可以透過 composer 指令 (可以指定框架版本，那我們這次使用的是5.4版本)

```sh
$ composer create-project --prefer-dist laravel/laravel 專案名稱 "5.4.*"
```
<br>

完成後就進入該專案目錄底下，接著可以執行該指令來檢查框架版本

```sh
$ php artisan -V
Laravel Framework 5.4.36
```
<br>

> [!NOTE] 小提醒
>Laravel 框架可使用的的版本，取決於你的 PHP 版本，所以要找相對應的歐！使用 Laravel 5.4 跟 PHP 8.1 ，他會說 PHP 版本太新，無法安裝。

### Laravel 資料夾與檔案介紹

我們可以看到這個專案目錄下，Laravel 幫我們生成了許多資料夾以及檔案，接著來簡單說明每一個資料夾與檔案的功能與用途吧

* app：主要放置 controller 的地方，提供網站的應用處理流程，包括處理用戶的行為和資料 Model 上的改變等事件之類別方法，提供給 view 來呼叫。透過 Laravel 之架構也可透過 controller 輕易建構 RESTful API。
* bootstrap：內含之 app.php 為將此框架初始化及建構起來之程式。
* config：內含此專案網站之環境設定、資料庫設定等設定程式。
* database：放置資料庫設定 (Model)。
* public：放置網站入口，透過 index.php 導入 route 之設定首頁
* resources：放置 view 的資源，用以呈現網站頁面
* routes：放置所有專案 route 的設定，其中較常用之 web.php 做為主要頁面導向及與 controller 的溝通，而 api.php 則作為專案提供 api 的設定。
* storage：主要放置些程式生成的檔案如頁面樣板 (Blade templates)、系統記錄 (logs)、file caches、file based sessions 等資料。
* tests：放置測試用的案例 (Test Case)。
* vendor：放置透過 composer 下載管理的套件。
* artisan：可於專案中透過 `php artisan` 執行 Laravel 設計好的基本操作行為，如建立 controller 等動作。
* composer.json：管理專案使用套件，詳細用法見 composer 官方說明。
* composer.lock：鎖定專案使用套件版號。
* package.json：管理 npm 套件，主要提供給前端使用。
* phpunit.xml：phpunit 測試設定檔，可規範執行單元測試之範圍，做批量測試。
* webpack.mix.js：build 設定檔，幫助我們將 resources 中之 js 及 sass 等前端設定檔 compile 成 js、css 等檔案並 deploy 到 public 中供頁面使用。

### 建立第一個 Laravel 網頁

> [!NOTE] 文章參考來源
>下面教學流程是參考ReccaChao [ Laravel 6.0 初體驗！怎麼用最新的 laravel 架網站！](https://ithelp.ithome.com.tw/articles/10213294) 的文章下去學習以及介紹，在學習過程中再加上自己的理解以及不同範例的筆記來分享給大家，大家也可以先去瀏覽大大的文章，再回來呦XD

我們經過千辛萬苦，將 Laravel 給安裝好，也設定好它所需的環境參數，打開 IED 準備開始學習如何建立第一個 Laravel 網頁吧！

在開始說明前，還要先下一個指令。在介紹 Laravel 專案下的資料夾以及檔案中，其中一個 "Artisan"，是 Laravel 專用的指令工具，可以幫我們處理很多事情。

像是我們現在要啟動內建的伺服器來瀏覽我們的網站，這時我們要下

````sh
$ poser global requirephp artisan serve
Laravel development server started: <http://127.0.0.1:8000>
[Thu Mar  3 15:13:07 2022] 127.0.0.1:52345 [200]: /favicon.ico
````

<br>
這個伺服器會在這個終端機下被建立，屬於這個專案的伺服器，如果想要關閉，可以使用 Ctrl + C 來關閉。

> [!NOTE] 小提醒
>由於此伺服器是專屬於該專案的伺服器，所以下指令時，也要在該專案的目錄下歐！

如果想要使用特定的 Port ，可以在後面加入此參數，就可以依照設定的 Port 來當作我們伺服器的位子囉！(預設是8000 Port)

````sh
$ php artisan serve --port=7777
Laravel development server started: <http://127.0.0.1:7777>
[Thu Mar  3 15:45:23 2022] 127.0.0.1:54206 [200]: /favicon.ico
````
<br>

#### 維護模式

當我們想要修改系統或是資料庫的欄位時，就可以將該網站設定成維修模式。Laravel 讓這個工作變得很容易，只需要用以下兩個指令來控制

啟動維護模式
```sh
$ php artisan down --message="Upgrading Database" --retry=60
```

<br>
還可以設定一些參數， --message 是用來顯示或紀錄客製化訊息， --retry 用來當作HTTP head 的 Retry-After 的值

<br>

{{< figure src="/php-laravel/laravel/error503.webp" width="700" caption="Laravel 維護模式顯示頁面" >}}

<br>

關閉維護模式

```sh
$ php artisan up
```

> [!NOTE] 維護模式顯示模板
>維護模式顯示的預設模板放置在 resources/views/errors/503.blade.php。你可以自由的根據需求修改。

<br>
#### Routes 路由

我們要來建立我的第一個網頁，網頁的其中一個關鍵就是路徑。那我們網站的路徑，在 Laravel 裡面，都放置在 `routes/` 裡面，打開後可以看到4個檔案有

* web.php：這個和我們現在要說明的路徑有關，我們在瀏覽器打的網址，前面是 domain name，那在 domain 之後的字串，會在我們這個檔案裡面來定義哪些字串要導向哪一個流程或是檔案，範例 url:{domain}/hello-world。
* api.php：我們在做前後端分離的專案時會使用這個檔案，與 web.php 功能差不多，預設用來管理 API 的路徑。
* channels.php：和 Broadcast 有關係，這個是 Laravel 的廣播功能，不太常使用，所以先跳過，之後有碰到再來說明。
* console.php：這個和我們指令有關，像是我們上面介紹的 php artisan ，這個檔案就是和這個部分有關。

##### web.php

那我們就先來針對 web.php 來做介紹，我們剛剛使用指令開啟的伺服器網址是 ```http://127.0.0.1:8000```，使用瀏覽器進入後，可以看到下面這個 Laravel 預設的頁面(此為 Laravel 5.4版本，新版的好像不太一樣)

<br>

{{< figure src="/php-laravel/laravel/laravel_welcome.webp" width="700" caption="Laravel 預設首頁" >}}

<br>

那我們開啟 `routes/web.php` 檔案中，可以看到下面這一段預設的路由

```php
Route::get('/', function () {
    return view('welcome');
});
```
這段的意思就是代表，如果我們網址沒有輸入其他的目錄，就會顯示 welcome 這個網頁，那 welcome 這個顯示的畫面，就放在 ```resources/view``` 底下名為 welcome.blade.php 的檔案中，打開來看就是簡單的 HTML。

<br>

那現在我們了解路由的含義，現在換我們來實作一個看看，首先我們希望網址列輸入 ```https://127.0.0.1:8000/hello-world``` 可以出現 ```hello world``` 的字樣，那我們可以到 web.php 中，新增下面這個路由

```php
Route::get('/hello-world', function () {
    return view('hello-world');
});
```
<br>

再到到 ```resources/view``` 新增 hello-world.blade.php 檔案，簡單輸入 HTML 格式，如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1> hello ~ pinyi </h1>
</body>
</html>
```
使用瀏覽器在網址列輸入 ```http://127.0.0.1/hello-world``` ，就可以看到顯示 ```hello ~ pinyi ``` 的字樣囉～

<br>


除了回傳網頁來做顯示以外，也可以單獨回傳字串，並帶入參數歐，我們來試試看要怎麼做。

首先我們先將路由修改成下方，讓我們在 hello-world 後面可以輸入我們的名稱，並且顯示在畫面上

```php
Route::get('/hello-world/{name}', function ($name) {
    return 'hello ~ ' . $name;
});
```
<br>

我們在網址列輸入 ```http://127.0.0.1/hello-world/pinyi``` 就會顯示 ```hello ~ pinyi``` ，很神奇吧，只要在括號中 {} 輸入你要的參數，並且後面的 function 也要帶入參數，就可以將資料給帶進去了 !

##### api.php

我們可以看到 api.php 檔案也是使用 Route 來運作，如下

```php
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

middleware 就是一種過濾或是防火牆的概念，這個我們後續會再說明。

<br>

我們也在此新增一個跟上面一樣的 hello-world，如下

```php
Route::get('/hello-world', function () {
    return 'Hello World ~';
});
```

在 api.php 比較特別的部分是需要在字串前面加入 api ，因為 api.php 就是預設讓我們來放 api 的地方，如範例 url:{domain}/api/hello-world，那就會回傳 ```hello world ~```

<br>

#### View Blade 模板引擎

我們已經知道要怎麼透過路由，讓使用者輸入網址後，根據網址指向想要呈現的畫面，那我們的畫面要怎麼呈現呢 !?

上面有使用簡單的 HTML 來做說明，那本章節就來介紹怎麼修改我們的前端輸出吧。

##### Layout

我們在設計網頁時，通常都會使用樣板，讓網頁相同的地方只做一次就可以顯示，不需要每一個頁面都重複寫相同的程式碼，只需要在想要顯示的地方來顯示各自的內容就好

如果聽不太懂，我以我的 Blog 來舉例，紅色框框，在每一頁都會顯示，我們可以把他寫成一個檔案，就不需要每頁都重複去寫，藍色框框，會依照每一頁的內容來做顯示。

<br>

{{< figure src="/php-laravel/laravel/layout.webp" width="700" caption="PinYi 小天地" >}}

<br>

那我們可以透過 Laravel 的 Blade Engine 輕鬆地達到這件事情，首先，到 ```resources/views/``` 目錄下建立一個叫 ```layouts``` 的資料夾。這個資料夾之後會讓我們來放不同的模板

<br>

在 ```layouts``` 下建立一個 ```header.blade.php``` 檔案，內容如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
        <h1>測試網站，我是標題</h1>

        <div class="container">
            @yield('content')
        </div>
</body>
</html>
```
<br>

可以看到我們把 "測試網站，我是標題"，看成是一個 header，然後在 container 裡面使用了 ```@yield()``` 的語法，這個語法代表我們可以在其他的網頁 call 這個 header 來做顯示，則 container 就會依照該網頁想要呈現的內容來做顯示，我們再繼續看下去

<br>

我們在 ```resources/views/``` 下，新增一個 ```test.blade.php``` 的檔案，內容如下

```php
@extends('layouts.header')

@section('content')
    <p>今天天氣很好歐！</p>
@endsection
```
這邊簡單說明一下這個檔案裡面的意思，

```@extends('layouts.header')```  是指繼承 ```layouts``` 這個資料夾底下的 ```header.blade.php``` 檔案內容，

```@section('content')``` 是指我們剛剛在 ```header.blade.php``` 檔案裡面所下的 ```@yield(content)``` 的內容。

<br>

我們再到 ```route/web.php``` 來設定路由，加入這一段程式碼

```php

Route::get('/test', function () {
    return view('test');
});
```
<br>

我們在去瀏覽 ```http://127.0.0.1:8000/test``` ，就可以看到成品囉！這代表我們成功將 ```layouts/header.blade.php``` 的模板放到 ```test.blade.php``` 裡面囉。

<br>

{{< figure src="/php-laravel/laravel/layout_show.webp" width="500" caption="PinYi 小天地" >}}

<br>

當然，我們也可以透過 route 將後端的資料傳到前端，我們先到 ```routes/web.php``` 底下修改成

```php
Route::get('/test/{name}', function ($name) {
    return view('test', ['name' => $name]);
});
```

<br>

再到， ```resources/views/test.blade.php``` 修改成以下

```php
@extends('layouts.header')

@section('content')
    <p>今天天氣很好歐！ {{$name}}</p>
@endsection
```

<br>

就可以看到我們可以輸入 ```http://127.0.0.1:8000/test/pinyi``` ，網頁前端就會顯示


<br>

{{< figure src="/php-laravel/laravel/layout_show_2.webp" width="500" caption="PinYi 小天地" >}}

<br>

我們就成功將後端的資料送至前端！ 要注意 ```resources/views/test.blade.php``` 底下需要用兩個 {{}} 來括住參數呦，只有一個 {} 他會當成一般的字串來顯示呦～

<br>

#### Controller 控制器

除了剛剛在 ```routes``` 檔案中定義所有請求以及處理外，我們在使用 Laravel 時，通常會使用 Controller 來處理邏輯的請求，我們來看一下實際案例吧 !

<br>

題目：我們有一個冷知識產生器，依照我們輸入的 id 不同，會顯示不同的冷知識，那我們要怎麼做呢 !?

<br>

首先我們先到 ```routes``` 設定好我們的路由 ```http://127.0.0.1/trivia/(輸入的id)```

```php
Route::get('/trivia/{id}', 'TriviaController@trivia');
```
前面是我們的路由，後面則是我們要對應的 Controller 位置

<br>

接下來使用 ```artisan``` 指令來建立這個 Controller

```sh
$ php artisan make:controller TriviaController
Controller created successfully.
```
<br>

Laravel 會自動產生一個 ```TriviaController.php``` 的檔案，位置在 ```app/Http/Controllers``` 底下

<br>

{{< figure src="/php-laravel/laravel/controller.webp" width="300" caption="Controller 產生檔案位置" >}}

<br>

接著我們來修改 ```TriviaController.php``` ，加入我們在 ```routes``` 宣告的函式 ```trivia```，並把剛剛在 ```routes``` 所帶的參數 ```id``` 給帶入

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TriviaController extends Controller
{
    public function trivia($id){
        return $id;
    }
}
```

使用瀏覽器輸入 ```http://127.0.0.1/trivia/{任意數字}``` 來檢查是否有將 ```routes``` 跟 ```controller``` 串再一起。

正常的話，會看到畫面顯示你所輸入的 {任意數字} (當然因為沒有限制輸入，所以輸入任何字元都可以 XD)

<br>

接下來，依照我們的題目，我們輸入不同的 ```id``` ，要顯示不同的冷知識，那我們要在哪去處理這個邏輯呢 !?

這個問題，網路上大家看法都不一樣，我針對我學習的以及 ReccaChao [Laravel 6.0 初體驗！怎麼用最新的 laravel 架網站！](https://ithelp.ithome.com.tw/articles/10213294) 的看法，將邏輯都寫在 ```Services``` 裡面在由 ```controller``` 來做控制，這樣可以把每一個元件的職責都拆分出來。

<br>

那我們就要先在 ```app``` 底下建立一個 ```Services``` 的資料夾，來存放我們的服務。再建立一個這次題目的邏輯存放的位置 ```TriviaService.php```

```php
<?php

namespace App\Services;

/**
 * Class TriviaService
 */
class TriviaService
{
    /**
     * @return string
     */
    public function trivia($id)
    {
        $arr = [
            '李小龍的動作非常快，快到看不清，所以拍電影時只好放慢膠片的速度。',
            '流沙一般都不深，根本不像電影里的那樣，所以不用擔心，除非使面部被流沙掩埋，不然根本不會有事。',
            '最早被打上條形碼的產品是箭牌口香糖。',
            '在菲律賓溜溜球曾被作為武器。',
            '打噴嚏時無法睜着眼睛。',
        ];
        return $arr[$id];
    }
}
```
<br>

以及修改 controller.php 檔案內容

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\TriviaService;

class TriviaController extends Controller
{
    public function trivia($id){
         return (new TriviaService())->trivia($id);
    }
}
```
記得要 ```use App\Services\TriviaService; ``` ，下面才可以 Call `TriviaService()` 。

這時候我們再去測試一下，就可以發現，網頁會依照我們所輸入的 ```id```，而顯示不同的冷知識。 <font color='red'>這時候如果輸入非數字或是超過陣列的值，就會出現錯誤！</font>

<br>

{{< figure src="/php-laravel/laravel/trivia.webp" width="700" caption="依照輸入不同的 ```id``` 顯示的冷知識" >}}

<br>

#### Database 資料庫

當資料量越來越大，我們不會將要顯示的內容都寫在程式碼裡面，所以我們需要建立資料庫，來存放我們的資料，在 [PHP 常見框架對比](#php-常見框架對比) 部分有提到 Laravel 支援 MySQL、PostgreSQL、SQLite 、SQL Server 資料庫，本次介紹使用 MySQL 來做說明。

首先我們要先確認資料庫是否與 PHP 有所連結，我們先打開 ```.env``` 這個隱藏檔來看，大約在第8行有一段關於資料庫的敘述

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

<br>

由於我們本次是以 Mysql 來做說明，所以 DB_CONNECTION 不需要改，DB_HOST 以及 DB_PORT 輸入你要連線的資料庫主機以及 Port 號，DB_DATABASE 輸入想要連線的資料庫名稱，以及你的 DB_USERNAME、DB_PASSWORD 帳號及密碼，都完成後，我們來看一下 database 資料夾下有哪些東西吧
* migrations：Laravel 用程式碼來代替 SQL，讓我們資料庫有版本控制的功能
* factories：他控制整個資料被填充的過程，例如：user 資料應該如何被產生？之後我們在 seeds 可以使用 factory()這個方法
* seeds：裡面放置產生假資料的檔案。

> [!NOTE] 小提醒
>在學習時，有看到大神文章底下有人留言說 DB_USERNAME、DB_PASSWORD 不可以有 `#` ，會導致無法運作。(目前沒有測試過，大家可以再多加留意)

<br>

先大概了解這三個東西所存放的東西，後續會一一介紹到！

<br>

##### Migration 資料庫遷移

在過去開發時，會有一個很頭痛的一件事情，就是要怎麼樣同步各地方的資料庫格式。如果有一個新進工程師要開始開發，只能請別的工程師匯出自己在開發中的資料庫，十分的麻煩，而且如果要修改格式，還需要告知每一個在開發的工程師。

為了解決這個問題，Laravel 框架提供了一個新的功能：資料庫遷移(Migration)。他是利用程式碼來修改資料庫格式，所以資料庫格式和其他的程式碼都使用版本控制內。這麼一來，每次有新的工程師要建構開發環境時，只需要提醒他在取得程式碼的時後，記得跑一下 migration 就可以囉～

<br>

我們來實作看看吧！我們要下 ```php artisan migrate``` 指令來把這次的 migration 讀入資料庫並建立架構，詳細的指令介紹，放在 [Artisan 相關指令 > Migration](#migration)。

<br>

執行後發現 .... 竟然有錯誤

<br>

{{< figure src="/php-laravel/laravel/mysql_1.webp" width="800" caption="MySQL 8.x 版本導致錯誤" >}}

<br>

在網路上找了一下資料，好像是Mysql 8.x 版本才會出現的問題，發現需要在我們的 ```config\database.php``` mysql 裡面多加一個標籤 modes ，就可以正常運作了！


> [!NOTE] Laravel Mysql 8.x 錯誤文章
> [Laravel mysql migrate error](https://stackoverflow.com/questions/49949526/laravel-mysql-migrate-error)

```php
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
            'engine' => null,
            'modes'  => [
                'ONLY_FULL_GROUP_BY',
                'STRICT_TRANS_TABLES',
                'NO_ZERO_IN_DATE',
                'NO_ZERO_DATE',
                'ERROR_FOR_DIVISION_BY_ZERO',
                'NO_ENGINE_SUBSTITUTION',
            ],
        ],
```
<br>

{{< figure src="/php-laravel/laravel/mysql_2.webp" width="800" caption="加入 modes 修復成功" >}}

<br>

{{< figure src="/php-laravel/laravel/migration.webp" width="800" caption="使用 migration 後的資料庫" >}}

<br>

##### 建立屬於自己的 migration


> [!NOTE] 後續實作題目
>我們會建立一個留言板的資料表，並且透過 RESTful API 的方式去查詢、新增、修改、刪除。

安裝好之後，我們來建立屬於我們的 migration ，來改變現有的資料庫格式。老樣子，一樣使用 ```artisan``` 來產生一個 migration ```create_messages_table ``` 。

```sh
$ php artisan make:migration create_messages_table
Created Migration: 2022_03_07_054858_create_messages_table
```
<br>


這樣子就建立好了，那我們來看一下這個檔案裡面寫了什麼吧

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateMessagesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        //
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```
<br>

可以看到裡面有一個 up 跟 down 的函式，這兩個代表什麼意思呢？

我們執行 ```migrate``` 的時候，會呼叫到 ```up()``` 函式的內容，如果使用到 ```migrate:rollback```  回溯時，就會呼叫 ```down()``` 函式的內容來還原 ```migrate```  所做的事情。

所以每一個 migration 裡面的 ```down()``` 都必須要能復原 ```up()``` 所做的事情

因為我們還沒有新增 ```migrate``` ，所以回溯機制等等再說明。```migrate``` 可以把他理解新增資料表，那當我們修要修改資料表，就可以使用 ```migrate:rollback```  來回溯歐！

* migrate：可以把他理解成將程式語言轉成 SQL 的功能

<br>

我先來實際操作看看，我們先在剛剛的 ```{日期時間省略}_create_messages_table``` ，裡面改成下方

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateMessagesTable extends Migration
{
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->bigIncrements('id'); //留言板編號
            $table->string('name', 20); //留言板姓名
            $table->string('content'); //留言板內容
            $table->timestamps(); //留言板建立以及編輯的時間
        });
    }

    public function down()
    {
        Schema::dropIfExists('messages');
    }
}
```
```up()``` 裡面的是透過 [Schema 結構生成器](#schema-結構生成器)，產生資料表。(Schema 後續有比較詳細的說明)

```down()``` 是指如果進行 ```migrate:rollback``` 回溯時會刪除 messages 這個資料表 。

<br>

##### 新增資料表

我們下 ```migrate``` 指令來將 ```{日期時間省略}_create_messages_table.php``` 執行 `up()`  的部分將程式轉成新增資料表，並下指令來檢查一下，是否與我們設定的 Schema 相同。

```sh
$ php artisan migrate
Migrating: 2022_03_07_054858_create_messages_table
Migrated:  2022_03_07_054858_create_messages_table
```

<br>

{{< figure src="/php-laravel/laravel/mysql_fields.webp" width="800" caption="messages 欄位" >}}

<br>

很神奇吧，透過程式就可以直接生成資料表的欄位，這也就是為什麼會說 ```migration``` 可以透過版本控制來讓新工程師建構環境的原因呦，除了生成以外，也可以透過 ```migrate:rollback``` 回溯來刪除資料表，那接著我們來看看要怎麼去刪除資料表吧！

<br>

阿對了，也可以使用 `$ php artisan migrate:status` 來查詢目前的檔案的執行狀態，檢查一下是否運作中。

<br>

{{< figure src="/php-laravel/laravel/mysql_status_1.webp" width="800" caption="Migration 是否運作中" >}}

<br>

##### 修改資料表

我們下 ```migrate:rollback``` 指令來將 ```{日期時間省略}_create_messages_table.php``` 執行 `down()`  的部分將程式轉成刪除資料表，並下指令來檢查一下，是否與我們設定的 Schema 相同。
```sh
$ php artisan migrate:rollback
Rolling back: 2022_03_07_054858_create_messages_table
Rolled back:  2022_03_07_054858_create_messages_table
```
這邊就是回溯來刪除資料表，那接著我們來看一下目前資料表的狀態吧！

<br>

{{< figure src="/php-laravel/laravel/mysql_status_2.webp" width="800" caption="Migration 是否運作中" >}}

<br>

這時我們想要修改一下資料表的欄位，想要將 message 型態從 varchar(255) 改成 varchar(20) ，以及在 message 後面加入一個 version int(1) 且預設值為 0 的欄位，這時我們只需要修改 `{日期時間省略}_create_messages_table.php` ，在執行 `php artisan migrate` 就可以囉！
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateMessagesTable extends Migration
{
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->bigIncrements('id'); //留言板編號
            $table->string('name', 20); //留言板姓名
            $table->string('contect',20); //留言板內容
            $table->integer('version')->default(0); //留言板版本
            $table->timestamps(); //留言板建立以及編輯的時間
        });
    }

    public function down()
    {
        Schema::dropIfExists('messages');
    }
}
```
```sh
$ php artisan migrate
Migrating: 2022_03_07_054858_create_messages_table
Migrated:  2022_03_07_054858_create_messages_table
```
成功後，我們一樣來檢查一下，是否與我們設定的 Schema 相同。

<br>

{{< figure src="/php-laravel/laravel/mysql_fields_1.webp" width="800" caption="修改後 message_board 欄位" >}}

<br>

##### Schema 結構生成器

前面我們使用 `Schema 結構生成器`，來產生能轉成 SQL 讀懂的程式語言，像是 `$table->string('name', 20)` 或是 `$table->timestamps()` ，如果對於 SQL 有基本的了解，大致上也可以猜的出來他在做什麼動作，那我也將常用到的資料型態使用表格列出來，讓大家可以有對照。


> [!NOTE] Laravel 官網
> 詳細可以參考 [Laravel 官方網站>資料庫>資料表>創建欄位](https://laravel.com/docs/5.4/migrations#creating-columns) 的文件，裡面有更詳細的欄位型態說明！

<br>

* 加入欄位

更新現有的資料表，可使用 `Schma::table` 方法：

```
Schema::table('users', function($table)
{
    $table->string('email');
});
```
<br>

| 指令 | 功能描述 |
| :---: | :---: |
| `$table->bigIncrements('id')`;| ID 自動累加 |
| `$table->boolean('confirmed');` | 相當於 Boolean 型態 |
| `$table->char('name', 4);` |  相當於 Char 型態，長度為 4 |
| `$table->dateTime('created_at');	` | 相當於 datetime 型態 |
| `$table->float('amount');	` | 相當於 flaot 型態 |
| `$table->integer('votes');` | 相當於 Integer 型態  |
| `$table->string('name', 100);	` | 相當於 string  型態，長度為 100 |
| `$table->timestamps();`	| 加入 created_at 和 updated_at 欄位  |
| `->nullable()` | 允許此欄位為 NULL |
| `->default($value)` | 宣告此欄位的預設值 |

* 修改欄位名稱

要修改欄位名稱，可在結構生成器內使用 `renameColumn` 方法，請確認在修改前 `composer.json` 檔案內已經加入 `doctrine/dbal`。

```
Schema::table('users', function($table)
{
    $table->renameColumn('from', 'to');
});
```
<br>

* 移除欄位

要移除欄位，可在結構生成器內使用 `dropColumn` 方法，請確認在移除前  `composer.json` 檔案內已經加入 `doctrine/dbal`。

```
Schema::table('users', function($table)
{
    $table-> dropColumn('from');
});
```

<br>

* 檢查是否存在

您可以輕鬆的檢查資料表或欄位是否存在，使用 `hasTable` 和 `hasColumn` 方法：

**檢查資料表是否存在**
```
if (Schema::hasTable('users'))
{
    //
}
```
<br>

**檢查欄位是否存在**
```
if (Schema::hasColumn('users', 'email'))
{
    //
}
```

<br>

* 加入索引

結構生成器支援多種索引類型，有兩種方法可以加入，可以在定義欄位時直接加上去，或者是分開另外加入：

`$table->string('email')->unique();`

或是分開另外加入：

| 指令 | 功能描述 |
| :---: | :---: |
| `$table->primary('id');` | 	加入主鍵 |
| `$table->primary(array('first', 'last'));` | 加入複合鍵 |
|`$table->unique('email');` |	加入唯一索引 |
|`$table->index('state');`   | 加入基本索引 |

<br>

##### Eloquent ORM 存取資料庫內容

我們在前面，已經學會了如何建立資料庫以及修改資料表欄位，還記得我們題目是要寫一個 RESTful API 來實現 CRUD ，那我們現在就要針對如何去存取資料庫內容來做介紹！

在 Laravel 中其中一個核心的物件：Eloquent Model ，他是負責存取資料庫的一組類別，實做了 Active Record 的架構。

那我們沿用先前的資料表來進行存取資料庫的實作，建立一個 `Message` 物件。

```sh
$ php artisan make:model Message
Model created successfully.
```
<br>

他會放在 `app/` 底下，我們查看 `Message.php`
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Message extends Model
{
    //
}
```
<br>

我們創建這個 model 的目的是要讓 Laravel 知道 `messages` 資料表中哪些欄位可以使用，所以我們可以參考官方預設 `User` 物件，加入 `$fillabel` 這個變數來存放可用欄位。
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Message extends Model
{
    protected $fillable = [
        'name','content','created_at','updated_at'
    ];
}
```
我們讓 Laravel 可以使用 `name`、`contect`、`created_at`、`updated_at` 的四個欄位。

<br>

我們先隨意新增一筆資料來做顯示的測試資料。
```sql
INSERT INTO `messages` (`id`, `name`, `content`, `version`, `created_at`, `updated_at`) VALUES (NULL, 'test', 'hello world', '0', '2022-03-07 14:15:22', '2022-03-07 14:15:22');
```

<br>

這次我們先看看是否能正確的讀取資料庫，我們就先不透過 Controller，直接在 route 來做設定。
```php
Route::get('/messages', function(){
    return App\Message::all();
});
```
當你瀏覽 `http://127.0.0.1:8000/api/messages` ， 應該會出現以下資訊（資料依據你輸入的會不一樣）
```json
[
	{
		"id": 1,
		"name": "test",
		"content": "hello world",
		"version": 0,
		"created_at": "2022-03-07 14:15:22",
		"updated_at": "2022-03-07 14:15:22"
	}
]
```
到這裡，代表我們已經可以成功讀到資料庫內的資料了！

##### Seeder

我們在開發時，會常常需要進行測試，那在 Laravel 中，有一個功能是 `seeder` ，他可以幫我們產生資料，讓我們在後續的開發和測試更順利。

* 建立一個 seeder

我們先來建立一個 `Message` 的 seeder
```sh
$ php artisan make:seeder MessageTableSeeder
Seeder created successfully.
```
他會被放到 `seeds` 目錄下，照慣例我們打開來看一下內容是什麼
```php
<?php

use Illuminate\Database\Seeder;

class MessageTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        //
    }
}
```

<br>

我們在 `run()` 裡面，加入可以新增資料的程式碼
```php
<?php

use Illuminate\Database\Seeder;
use App\Message;

class MessageTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $Message = new Message;
        $Message->name = 'test';
        $Message->content = 'hello world2';
        $Message->created_at = date("Y-m-d h:i:s");
        $Message->save();
    }
}
```

記得要 `use App\Message;` ，加入後，使用 `php artisan db:seed --class=MessageTableSeeder` 指令，檢查是否自動新增成功

<br>

{{< figure src="/php-laravel/laravel/seeder.webp" width="800" caption="messages 資料" >}}

<br>

### RESTful API 實現 CRUD

接下來，就來到我們本次的重點，要使用 Laravel 來開發出留言板功能，搭配使用 RESTful API 來實現 CRUD

1. 老樣子，我們先來設定我們的 `route` 路由，這次實作的是留言板，所以我們希望網址是 ```http://127.0.0.1:8000/api/messages```，因為這次是 API，先到 `routes/api.php` 底下來設定

```php
<?php
use Illuminate\Http\Request;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/
Route::get('messages', 'MessageController@getAllMessages'); // 查詢全部留言
Route::get('messages/{id}', 'MessageController@getMessages'); // 查詢 {id} 留言
Route::post('messages', 'MessageController@createMessages'); // 新增留言
Route::put('messages/{id}', 'MessageController@updateMessages'); // 修改 {id} 留言
Route::delete('messages/{id}', 'MessageController@deleteMessages'); //刪除 {id} 留言
```

已經忘記的，可以先回到 [Controller 控制器](#controller-控制器) 複習一下上面的路由是什麼意思。

接下來，我們使用 [建立屬於自己的 migration](#建立屬於自己的-migration) 來生成我們本次要使用的資料表，再來新增一個與 [ Eloquent Model ](#eloquent-orm-存取資料庫內容) 相同的物件，選擇我們要的資料表
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Message extends Model
{
    protected $table = 'Messages';
    protected $fillable = [
        'name','content','created_at','updated_at'
    ];
}
```
<br>

都好了我們來撰寫我們的 controller，我會依照每一個功能(CRUD)來做說明，要記得加入剛剛的 model `use App\Message;`。

#### 查詢全部留言
```php
class MessageController extends Controller
{
    // 查詢全部留言
    public function getAllMessages() {
        $messages = Message::get(['id','name','content','created_at','updated_at'])->toJson(JSON_PRETTY_PRINT);
        return response($messages, 200);
    }
}
```
會依照 model  選擇的資料表來做查詢，只顯示 `id`、`name`、`content `、`created_at `、`updated_at` 五個欄位，並把他用 JSON 表示，最後用 response() 來回應 status code。

<br>

#### 查詢{id}留言
```php
class MessageController extends Controller
{
    // 查詢{id}留言
    public function getMessages($id) {
        if (Message::where('id',$id)->exists()) {
            $messages = Message::where('id',$id)->get(['id','name','content','created_at','updated_at'])->toJson(JSON_PRETTY_PRINT);
            return response($messages, 200);
        } else {
            return response()->json(["messages" => "找不到訊息"], 404);
        }
    }
}
```
先判斷輸入的 id 是否存在，如果存在才顯示 `id`、`name`、`content `、`created_at `、`updated_at` 五個欄位，並把他用 JSON 表示，最後用 response() 來回應 status code，如果不存在，直接用 JSON 表示 response() 404錯誤訊息。

<br>

#### 新增留言
```php
class MessageController extends Controller
{
    // 查詢{id}留言
    public function createMessages(Request $request) {
        if (strlen($request->content)<20){
            $messages = new Message;
            $messages->name = $request->name;
            $messages->content = $request->content;
            $messages->updated_at = NULL;
            $messages->save();
            return response()->json(["messages" => "新增紀錄成功"], 201);
        }
        if (strlen($request->content)>20){
            return response()->json(["messages" => "內容長度超過20個字元"], 400);
        }
    }
}
```
我們希望輸入的內容不要超過20的字，所以我們先用 `strlen()` 來判斷輸入長度，如果小於20，就將 request 輸入的名字及內容輸入，用 `save()` 來儲存，並把他用 JSON 表示，最後用 response() 來回應 status code，如果小於20，直接用 JSON 表示 response() 400錯誤訊息。

<br>

#### 修改留言
```php
class MessageController extends Controller
{
    // 修改{id}留言
    public function updateMessages(Request $request,$id) {
        if (Message::where('id',$id)->exists() && Message::where('version',0)->lockForUpdate() ) {
            if (strlen($request->content)<20){
                $messages = Message::find($id);
                $messages->name = is_null($request->name) ? $messages->name : $request->name;
                $messages->content = is_null($request->content) ? $messages->content : $request->content;
                $messages->version = '1';
                $messages->save();
                return response()->json(["message" => "修改成功"],200);
            }
            if (strlen($request->content)>20){
                return response()->json(["messages" => "內容長度超過20個字元"], 400);
            }
        } else {
            return response()->json(["message" => "找不到訊息"],404);
        }
}
```
先判斷輸入的 id 是否存在以及是否有被上鎖，在判斷輸入的長度，如果小於20，再判斷是否有輸入，有就將 request 輸入的值輸入，用 `save()` 來儲存，並把他用 JSON 表示，最後用 response() 來回應 status code，如果小於20，直接用 JSON 表示 response() 400錯誤訊息。

<br>

#### 刪除留言
```php
class MessageController extends Controller
{
    // 刪除{id}留言
    public function deleteMessages($id) {
        if (Message::where('id',$id)->exists()) {
            $messages = Message::find($id);
            $messages->delete();
            return response()->json(["message" => "刪除成功,沒有返回任何內容"],204);
        } else {
            return response()->json(["message" => "找不到訊息"],404);
        }
    }
}
```
先判斷輸入的 id 是否存在，如果存在，就用 `delete()` 來刪除，最後用 response() 來回應 status code，如果不存在，直接用 JSON 表示 response() 404錯誤訊息。

#### Postman 測試

##### 查詢全部留言 - 無資料

<br>

{{< figure src="/php-laravel/laravel/restful-get-null.webp" width="600" caption="查詢全部留言 無資料" >}}

<br>

查詢全部留言，因為目前沒有資料，所以會顯示空陣列以及回應 200 OK

<br>

##### 查詢全部留言 - 成功

<br>

{{< figure src="/php-laravel/laravel/restful-get-all.webp" width="600" caption="查詢全部留言 成功" >}}

<br>

查詢全部留言成功，會顯示全部資料以及回應 200 OK

<br>

##### 查詢{id}留言 - 成功

<br>

{{< figure src="/php-laravel/laravel/restful-get-id.webp" width="600" caption="查詢{id}留言 成功" >}}

<br>

查詢{id}留言成功，會顯示{id}資料以及回應 200 OK

<br>

##### 新增留言 - 成功

{{< figure src="/php-laravel/laravel/restful-post-success.webp" width="600" caption="新增留言 成功" >}}

<br>

新增留言成功，會顯示新增紀錄成功以及回應 201 Created

<br>

##### 新增留言 - 失敗

{{< figure src="/php-laravel/laravel/restful-post-error.webp" width="600" caption="新增留言 失敗" >}}

<br>

新增留言失敗，因為超出20個字元，所以會顯示內容長度超過20個字元以及回應 400 Bad Request

<br>

##### 修改留言 - 成功

{{< figure src="/php-laravel/laravel/restufl-put-success.webp" width="600" caption="修改留言 成功" >}}

<br>

修改留言成功，會顯示修改成功以及回應 200 OK

<br>

##### 修改留言 - 失敗

{{< figure src="/php-laravel/laravel/restufl-put-error.webp" width="600" caption="修改留言 失敗" >}}

<br>

修改留言失敗，會顯示找不到訊息以及回應 404 Not Found

<br>

##### 刪除留言 - 成功

<br>

{{< figure src="/php-laravel/laravel/restufl-delete-success.webp" width="600" caption="刪除留言 成功" >}}

<br>

刪除留言成功，不會顯示訊息但會回應 204 No Content

<br>

##### 刪除留言 - 失敗

<br>

{{< figure src="/php-laravel/laravel/restufl-delete-error.webp" width="600" caption="刪除留言 失敗" >}}

<br>

刪除留言失敗，會顯示找不到訊息以及回應 404 Not Found

<br>


<br>

## 測試

寫完程式後，除了自己一個一個去測試，也可以寫成程式來測試是否有錯誤。會在此談到測試，是因為自動測試變得簡單也是 Laravel 這個框架的一個重要特性。

那我先來說明一下什麼是自動測試

* 簡單來說，常見的測試，就是我們在完成新功能後，會一個一個實際去測試我們寫的功能，是不是跟我們預想的那樣子運行。
* 一開始網站還小，功能不多時，還可以應付。但當網站越來越龐大，功能越來越多後，或是要使用不同權限帳號來檢查所有功能，會很辛苦的。

幸好，我們可以用自動測試的程式來解決這些瑣事！

* 如果我們每次為每個功能撰寫程式時，都寫另一段小程式來測試這個功能。
* 隨著時間過去，需求的改變，雖然功能越來越多，但是相對應的，測試這些功能的程式也會越來越多，並且不管這個功能是多久之前開發的，每個功能都有相對應的測試。

### 如何運行自動測試

在 PHP 的世界，自動測試通常使用 [PHPUnit](https://phpunit.de/) 這個工具。然後 Laravel 在安裝時，也自動幫我們安裝這個工具了～
Laravel 幫我們把 PHPUnit 安裝在 ```vendor/``` 裡面，我們來看看要怎麼使用它吧！使用的指令如下

```sh
$ vendor/phpunit/phpunit/phpunit
PHPUnit 5.7.27 by Sebastian Bergmann and contributors.

..                                                                  2 / 2 (100%)

Time: 146 ms, Memory: 10.00MB

OK (2 tests, 2 assertions)
```
<br>

奇怪，我們什麼都還沒做，為什麼會有兩個程式已經通過測試了，是因為 Laravel 預設兩個檔案來當測試的案例，分別位於 ```tests/Feature/ExampleTest.php``` 以及 ```tests/Unit/ExampleTest.php```  (單元測試)，我這邊就簡單說明 ```Feature/ExampleTest.php``` 這個測試程式做哪些事情

<br>

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     *
     * @return void
     */
    public function testBasicTest()
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

<br>

這個  ```Feature/ExampleTest.php```  的 ```testBasicTest()``` 會檢查如果連線到 ```/``` ，HTTP Status 有沒有回傳 200 (成功連線)，這個測試功能就是在檢查首頁是否存在 ～

### 實作測試程式

大概了解原理後，一樣我們來實作看看，那我們先來輸入指令，產生一個測試程式

```sh
$ php artisan make:test HelloWorldTest
Test created successfully.
```
<br>

Laravel 會自動幫我們在 ```tests/Feature``` 底下新增了 HelloWorldTest.php 檔案，內容如下(框架版本5.4)

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class HelloWorldTest extends TestCase
{
    /**
     * A basic test example.
     *
     * @return void
     */
    public function testExample()
    {
        $this->assertTrue(true);
    }
}
```
<br>

我們測試的項目是想要連線到 ```hello-world/``` 目錄，HTTP Status 會回傳 200 ，以及網站內要看到『 hello world ~ 』的文字，我們將他預設的 testExample() 的內容改成

```php
    public function testExample()
    {
        $response = $this->get('/hello-world');

        // 如果連線 hello-world/，HTTP Status 應該要顯示 200
        $response->assertStatus(200);

        //連線到網頁內，應該要可以看到「『 hello world ~』文字
        $response->assertSee('hello world ~');
    }
```
<br>

來運作一下吧，一樣使用同樣的指令來做測試，順便讓大家看一下測試錯誤的錯誤訊息吧！


```sh
$ vendor/phpunit/phpunit/phpunit
PHPUnit 5.7.27 by Sebastian Bergmann and contributors.

.F.                                                                 3 / 3 (100%)

Time: 200 ms, Memory: 10.00MB

There was 1 failure:

1) Tests\Feature\HelloWorldTest::testExample
Expected status code 200 but received 404.
Failed asserting that false is true.

FAILURES!
Tests: 3, Assertions: 3, Failures: 1.
```

我故意在 routes 下沒有放 ```hello-world/``` 路徑，所以返回顯示 404 ，而不是 200 ，因此，就可以知道要怎麼去修改程式囉  !

<br>

## Artisan 相關指令

### Controller

```sh
php artisan make:controller {檔案名稱}Controller //建立一個 Controller
```

<br>

### Migration

```sh
php artisan make:seeder  //產生seeder檔案
php artisan make:factory //產生factory 檔案
php artisan make:migration //產生migration 檔案
php artisan migrate //將這次的migration讀入資料庫建立架構
php artisan migrate:rollback //推回上一次的migration
php artisan migrate:reset //推回全數的migration
php artisan migrate:refresh //推回所有遷移並且再執行一次
php artisan migrate --seed //將這次的migration讀入資料庫建立架構並且也跑seeder
php artisan migrate:refresh --seed //推回所有遷移並且再執行一次以及seeder
php artisan db:seed //單純跑資料的seeder 填充資料
```

<br>

### Test

```sh
php artisan make:test {檔案名稱} //測試程式
```

<br>

想必大家到這步驟已經對 Laravel 不陌生了，已經熟悉上面的教學介紹後，可以再去學習下一篇 [Laravel 進階](https://pin-yi.me/laravel-advanced)，會有如何使用內建會員系統，以及如何判斷 RESTful API 是否登入等進階實作ㄡ～


<br>

## 參考資料

PHP與Laravel簡介：[https://ithelp.ithome.com.tw/articles/10237537](https://ithelp.ithome.com.tw/articles/10237537)

比較 PHP 網頁框架：[https://opensourcedoc.com/blog/comparing-php-web-frameworks/](https://opensourcedoc.com/blog/comparing-php-web-frameworks/)

Laravel 從入門到放棄 (一) – 透過 composer 架設 laravel 網站：[https://christmasq.wordpress.com/2018/12/04/php-laravel-%E5%BE%9E%E5%85%A5%E9%96%80%E5%88%B0%E6%94%BE%E6%A3%84-%E4%B8%80-%E9%80%8F%E9%81%8E-composer-%E6%9E%B6%E8%A8%AD-laravel-%E7%B6%B2%E7%AB%99/](https://christmasq.wordpress.com/2018/12/04/php-laravel-%E5%BE%9E%E5%85%A5%E9%96%80%E5%88%B0%E6%94%BE%E6%A3%84-%E4%B8%80-%E9%80%8F%E9%81%8E-composer-%E6%9E%B6%E8%A8%AD-laravel-%E7%B6%B2%E7%AB%99/)

PHP Laravel 安裝教學[https://angela52799.medium.com/php-laravel-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8-16fc4c2271ee](https://angela52799.medium.com/php-laravel-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8-16fc4c2271ee)

資料庫設計概念 - ORM：[https://ithelp.ithome.com.tw/articles/10207752](https://ithelp.ithome.com.tw/articles/10207752)

Laravel 6.0 初體驗！怎麼用最新的 laravel 架網站！：[https://ithelp.ithome.com.tw/articles/10213294](https://ithelp.ithome.com.tw/articles/10213294)

Laravel Route(路由：][https://ithelp.ithome.com.tw/articles/10217731](https://ithelp.ithome.com.tw/articles/10217731)