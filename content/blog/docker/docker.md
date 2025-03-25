---
title: "Docker 介紹 (如何使用 Docker-compose 建置 PHP+MySQl+Nginx 環境)"
type: docs
weight: 9999
date: 2024-05-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 什麼是 Docker ?

Docker 是一種軟體平台，它可以快速建立、測試和部署應用程式。為什麼可以快速建立呢？因為 Docker 會將軟體封裝到名為『容器』的標準單位。其中會包含程式庫、系統工具、程式碼、執行軟體所需的所有項目。
剛剛有提到容器 (Container)，是一種虛擬化技術，它高效率虛擬化及易於遷移和擴展的特性，非常適合現代雲端的開發及佈署。那 Container 與傳統的虛擬機有什麼差別呢？我們來看看下面這張圖

<br>

{{< figure src="/docker/docker/container-vm.png" width="700" caption="Container 與 VM 的差異" >}}

可以看到 Container 是以應用程式為單位，而 VM 則是以作業系統為單位。Container 是一個封裝了相依性資源與應用程式的執行環境 ; VM 則是一個配置好 CPU、RAM 與 Storage 的作業系統，為了更好的做區別，我把 Container、VM 兩個差別用表格來說明～

|   區別比較   |                                                                                                                 Container                                                                                                                 |                                                                                       VM                                                                                        |
| :----------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|     單位     |                                                                                                                 應用程式                                                                                                                  |                                                                                    作業系統                                                                                     |
|   適用服務   |                                                                                                              多使用於微服務                                                                                                               |                                                                                使用較大型的服務                                                                                 |
|   硬體資源   |                                                                                                    是以程式為單位，需要的硬體資源很少                                                                                                     |                                                           VM 會先佔用 CPU、RAM 等等硬體資源，不管有沒有使用都會先佔用                                                           |
|   造成衝突   |                                                                                     Container 間是彼此隔離的，因此在同一台機器可以執行不同版本的服務                                                                                      |                                                                           會因為版本不同造成環境衝突                                                                            |
| 系統支援數量 |                                                                                                            單機支援上千個容器                                                                                                             |                                                                                 一般最多幾十個                                                                                  |
|     優點     |                                               1 . Image 較小，通常都幾 MB<br>2 . 啟動速度快，通常幾秒就可以生成一個 Container<br>3 . 更新較為容易，只需要利用新的 Image 重新啟動就會更新了                                                |     1 . 因為硬體層以上都虛擬化，因此安全性相對較高<br>2 . 系統選擇較多，在 VM 可以選擇不同的 OS<br>3 . 不需要降低應用程式內服務的耦合性，不需要將程式內的服務個別拆開來部署     |
|     缺點     | 1 . 安全性較 VM 差，因為環境與硬體都與本機共用<br>2 . 在同一台機器中，每一個 Container 的 OS 都是相同的，無法一個為 Windows、一個為 Linux，還是依賴 Host OS<br>3 . Container 通常會切成微服務的方式作部署，在各元件中的網路連結會比較複雜 | 1 . Image 的大小通常 GB 以上，比 Container 大很多<br>2 . 啟動速度通常要花幾分鐘，因此服務重啟速度較慢<br>3 . 資源使用較多，因為不只程式本身，還要將一部分資源分給 VM 的作業系統 |

**總結**

- 更快速的交付和部署：對於開發和維運人員來說，最希望就是一次建立或設定，可以再任意地方正常運行。開發者可以使用一個標準的映像檔來建立一套開發容器，開發完成之後，維運人員可以直接使用這個容器來部署程式。Docker 容器很輕很快！容器的啟動時間都是幾秒中的事情，大量地節約開發、測試、部署的時間。

- 更有效率的虛擬化：Docker 容器的執行不需要額外的虛擬化支援，它是核心層級的虛擬化，因此可以實作更高的效能和效率。

- 更輕鬆的遷移和擴展：Docker 容器幾乎可以在任意的平台上執行，包括實體機器、虛擬機、公有雲、私有雲、個人電腦、伺服器等。 這種兼容性可以讓使用者把一個服務從一個平台直接遷移到另外一個。

- 更簡單的管理：使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分發和更新，從而實作自動化並且有效率的管理。

<br>

## Docker 基本概念

- Dockerfile
- 映像檔 (Image)
- 容器 (Container)
- 倉庫 (Repository)

這四個是 Docker 最基本的組成，了解後就可以知道整個 Docker 的生命週期。

### Dockerfile

開發人員在使用 Docker 時發現，大多現成的 Docker 映像檔無法滿足他們的需求，因此需要一種能夠生成映像檔的工具。Dockerfile 是一種簡易的文件檔，裡面包含了建立新映像檔所需的指令。

Dockerfile 語法主要由 Command（命令）和 Argument （參數選擇）兩大元素組成。以下是一個簡易的 Dockerfile 示意圖：

命令式語法＋選擇參數（Command + Argument）

<br>

### 映像檔 (Image)

Docker 映像檔是一個唯獨的模板。

例如：一個映像檔可在包含一個完整的 Linux 作業系統環境，裡面可以只安裝 Nginx 或使用者會使用到的其他應用程式。

我們會使用映像檔來建立 Docker 的容器 (Container) ，Docker 也提供很簡單的機制來建立映像檔或是更新現有的映像檔，也可以去下載別人已經做好的映像檔。

<br>

### 容器 (Container)

Docker 是利用容器來執行應用程式。

每一個容器都是由映像檔所建立的的執行程式。它可以被啟動、開始、停止、刪除。且每一個容器都是相互隔離的，不會相互影響。

<br>

## Docker 實作

本章節會介紹 Docker 三大組件映像檔 (image)、容器 (Container)、倉庫 (Repository) 要如何實際操作，以及他們的關聯性是什麼～
<br>

### 映像檔 (Image)

本小節會介紹有關映像檔的內容，包括：

- 如何從倉庫取得映像檔
- 如何管理本地主機上的映像檔
- 映像檔實作 (Dockerfile)

<br>

#### 如何從倉庫取得映像檔

我們可以先到 [Docker hub](https://hub.docker.com/) 上面看看有什麼服務或程式想要下載來做使用 [ 詳細介紹會放到[倉庫 (Repository) ](https://pin-yi.me/docker/#倉庫-repository-1)章節]，找到想要的服務，我們可以下 `docker pull {要下載的服務、程式名稱}` ，我們這邊就先下載 Mysql 這個映像檔。

- mysql

```sh
$ docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
15115158dd02: Pull complete
.... 省略 ....
Digest: sha256:b17a66b49277a68066559416cf44a185cfee538d0e16b5624781019bc716c122
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```

<br>

由於我們下載的沒有加任何的 `tag` ，也就是版本，所以我們都是下載最新版 latest ，如果想要下載特定版本，可以在服務名稱後面加上 `:{版本}` ，就可以下載對應的版本囉！

<br>

如果有標記 `Official Image` 就代表是官方釋出的映像檔 ~ 在穩定性以及安全上更有保障，所以大家可以優先下載歐！

<br>

{{< figure src="/docker/docker/docker-image.png" width="900" caption="Docker hub 下載 Image" >}}

<br>

#### 管理本地主機上的映像檔

##### 查看映像檔 (images)

當我們下載好映像檔後，可以使用 `docker images` 來列出本機已下載的映像檔。

```sh
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
php          latest    d6229b88aa29   3 days ago    484MB
mysql        latest    826efd84393b   6 days ago    521MB
nginx        latest    c919045c4c2b   13 days ago   142MB
```

<br>

我們來看看都列出哪些訊息吧！

- REPOSITORY：來自於哪一個倉庫，像是 php、mysql、nginx。
- TAG：映像檔的標籤，因為我們都下載最新版本所以是 latest。
- IMAGE ID：它的唯一 `ID` 號碼。
- CREATED：建立時間。
- VIRTUAL SIZE：映像檔大小。

<br>

##### 儲存映像檔 (save)

想要儲存映像檔，可以使用 `docker save {映像檔名稱} > {檔案名稱}.tar`，來做儲存。

```sh
$ docker save mysql > mysql.tar
$ ls | grep 'mysql'
mysql.tar
```

<br>

##### 刪除映像檔 (rmi)

想要刪除映像檔，可以使用 `docker rmi {映像檔名稱}` 來做刪除。

```sh
$ docker rmi demo-image
Untagged:demo-image:latest
Deleted: sha256:1f56acbcbe9ec613a37e26934a84d98bed73879059f424dc69754520086baa37
```

想要同時刪除映像檔時，可以先用 `docker images -aq` 列出全部映像檔的 IMAGE ID，再一起刪除。

```sh
docker rmi -f $(docker images -aq)
```

- 注意！刪除映像檔(Image)時，要先用 `docker rm {容器}` 去刪除所有依賴這個映像檔的容器。

<br>

接著要怎麼建立自己的映像檔呢？我們要使用 Dockerfile 來建立映像檔。

#### 映像檔實作

##### 撰寫 Dockerfile 映像檔

Dockerfile 是一種文字格式的設定檔，可以透過 Dockerfile 快速建立自訂的映像檔，換句話說，Dockerfile 就像是建置 Docker Image 的腳本。

舉個例子：可以把自己想像成一位設計師，設計好房子的格局、擺飾等，畫好設計圖 (Dockerfile) 後，最後請師傅 (Docker) 依你的構思完成就可以了。

<br>

我們就來開始實作一個 Dockerfile 吧 ~

1. 我們先創建一個來放 Dockerfile 的資料夾，可以直接在路徑下建立出映像檔

```sh
mkdir demo-dockerfile
cd demo-dockerfile/
```

<br>

Dockerfile 結構，大致可以分為四個部分

- 基礎映像檔資訊
- 維護者資訊
- 映像檔操作指令
- 容器啟動時需執行的指令

<br>

2. 我們有說過 Dockerfile 是一個文字格式的設定檔，所以我們用 vim 來編寫 Dockerfile。

```sh
vim Dockerfile
```

<br>

```sh
# 基礎映像檔資訊
FROM nginx:latest

# 維護者資訊
LABEL maintainer="880831ian@gmail.com"

# 映像檔操作指令
RUN apt-get update -y\
    && apt-get install nginx -y

# 運行時容器提供服務的通道
EXPOSE 80

# 容器啟動時需執行的指令
CMD ["nginx","-g","daemon off;"]
```

<br>

`FROM nginx:latest`

第一行為必要的指定基礎映像檔，這邊使用 nginx 作為基礎映像檔，我們用最新版本，所以是 latest。

<br>

`LABEL maintainer="880831ian@gmail.com"`

維護者資訊想不也是不可以少的，這邊也可以輸入 Email 資訊，只是要注意的是此資訊會寫入產出映像檔的 Author 名稱屬性中。

<br>

`RUN apt-get update -y\
    && apt-get install nginx -y`

這邊是最重要的部分，想要在映像檔案上設定或安裝都需要將命令寫在這，格式必須依 RUN <command> ，RUN 指令後面放 Linux 指令，如果指令太長可以使用\來換行。 -y 是在安裝 Nginx，會同意所有進行中所出現的問題。

<br>

`EXPOSE 80`

設定運行時容器提供服務的通道。

<br>

`CMD ["nginx","-g","daemon off;"]`

最後就是啟動指定容器時預設執行的指令，格式是 CMD ["executable","param1","param2"]。

<br>

{{< callout >}}
Docker 運行 Nginx 時為什麼要使用 daemon off;

因為 Docker 容器啟動時，默認會把容器內部第一個進程，作爲 docker 容器是否正常運行的依據，如果 docker 容器 pid = 1 到進程就掛了，docker 就會退出！

Docker 未執行自定義的 CMD 之前， Nginx 的 pid 是 1，執行到 CMD 之後，Nginx 就在後台運行，bash 或是 sh 的腳本就會變成 pid =1 。

所以一但執行完 CMD，Nginx 容器就會退出了，所以才需要加上 `-g daemon off;`。

在 Nginx 官方的 [Docker Repository](https://hub.docker.com/_/nginx) 也有說明，在 Complex configuration 內。
{{< /callout >}}

<br>

順便說一下使用 Dockerfile 的優點：1 . 可以進行 Git 版控，讓你管理或分享更方便 2 . 佔用容量小，因為只是純文字檔而已。

<br>

##### 使用 Dockerfile 建立映像檔

我們已經撰寫完 Dockerfile 檔案了，接下來要執行來產生映像檔，我們要使用 `docker build` 來建立，我們一起來看看吧

```sh
docker build -t demo-image .
```

因為我們在 dockerfile 的目錄下，所以直接使用 “.” 來做建立動作，也可以使用 -f 來指定 dockerfile 的路徑位置。使用 -t 來設定映像檔的名稱，我們這邊取名叫 demo-image。

```sh
[+] Building 2.7s (7/7) FINISHED
 => [internal] load build definition from Dockerfile                                                                   0.0s
 => => transferring dockerfile: 44B                                                                                    0.0s
 => [internal] load .dockerignore                                                                                      0.0s
 => => transferring context: 2B                                                                                        0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                       2.6s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                          0.0s
 => [1/2] FROM docker.io/library/nginx:latest@sha256:8ae9bafbb64f63a50caab98fd3a5e37b3eb837a3e0780b78e5218e63193961f  0.0s
 => CACHED [2/2] RUN apt-get update -y    && apt-get install nginx -y                                                  0.0s
 => exporting to image                                                                                                 0.0s
 => => exporting layers                                                                                                0.0s
 => => writing image sha256:1f56acbcbe9ec613a37e26934a84d98bed73879059f424dc69754520086baa37                           0.0s
 => => naming to docker.io/demo-image
```

<br>

完成後我們使用 `docker images` 來看看是否建立成功。

```sh
$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
demo-image   	   latest    1f56acbcbe9e   2 hours ago   166MB
php                latest    d6229b88aa29   4 days ago    484MB
mysql              latest    826efd84393b   6 days ago    521MB
nginx              latest    c919045c4c2b   13 days ago   142MB
```

執行容器的部分，我們放到容器 (Container)的章節在介紹 ~

<br>

### 容器 (Container)

我們在介紹指令前，先來了解一下 Dockerfile、Docker Image、Docker Container 這三個的關係，可以先參考以下圖片

{{< figure src="/docker/docker/container.png" width="300" caption="容器 Container 組成" >}}

<br>

我們在啟動 Container 時，會有這三個部分組成，最底層是映像檔 (Image)，這一層主要是透過撰寫 Dockerfile 之後 build 出來的 Docker Image，就像我們前面說的它是一個唯獨的檔案。執行啟動了 Docker Container，就會加上第二層，就是需要先 Init Container 的設定，例如是 hostname、環境變數、網路連接等系統設定，最後最上層再加上一層讓使用者可以在此層去讀寫資料。

<br>

有關於容器 (Container) 的指令非常多，光是簡單的 run 就有很多參數，我們先列出比較常用且基本的 Container 指令～

#### Container 執行時的操作

##### 執行容器 (run)

我們想要創建一個新的容器並運行，就可以使用 `docker run`，我們來看看他可以使用哪些參數吧！

```sh
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

- OPTIONS 說明：

|        參數         |                               描述                               |
| :-----------------: | :--------------------------------------------------------------: |
|         -d          |                           後台運行容器                           |
|         -i          |                 命令互動模式，通常與 -t 同時使用                 |
|         -t          |       為容器重新分配一個假裝的輸入終端，通常與 -i 同時使用       |
|         -p          |      指定容器與本機的 Port ，格式是 `主機 Port : 容器 Port`      |
|   --name="{名稱}"   |                          為容器設定名稱                          |
| --net="{網路類型}"  | 指定容器的網路連接類型，支援 bridge/host/none/container 四種模式 |
| --link="{其他容器}" |                       添加連接到另一個容器                       |
|     --volume,-v     |     將容器檔案路徑映射到本地端，格式是 `本機路徑：容器路徑`      |

<br>

我們啟動我們下載好的 Nginx 來試試看吧！

```sh
$ docker run -d -p 7777:80 --name="demo-nginx" -v /Users/ian_zhuang/Desktop/data:/var/www/html nginx
31a4a4a56e3ef2fb75d538c4c9eea4914ac506a84a6ff97e1fbbb6c3213cc6b7
```

我們將 Nginx 容器在背景執行，且將預設的 80 Port 與本機的 7777 Port 綁在一起，讓我們在本機瀏覽 7777 Port 會直接導向容器的 80Port，設定容器的名字叫做 “demo-nginx" ，我們 Nginx 容器的檔案路徑映射到本地端的桌面 data 資料夾，我們就可以在本機新增檔案同步到容器中。

<br>

##### 顯示容器 (ps)

使用 `docker ps` 來檢查一下是否啟動成功 (`ps` 可以顯示映像檔的基本資訊，如果沒有加 -a 只會顯示執行中的容器)

```sh
$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED              STATUS              PORTS                  NAMES
31a4a4a56e3e   demo-image   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:7777->80/tcp   demo-nginx
```

<br>

接著我們測試是否有把桌面 data 資料夾掛到容器的路徑，我們先在 data 新增一個 hello.html ，裡面隨意輸入，瀏覽一下 `http://127.0.0.1:7777/hello.html` ，看看是否成功。

<br>

{{< figure src="/docker/docker/container-v.png" width="700" caption="容器 Container -volume 測試" >}}

<br>

##### 顯示容器紀錄 (logs)

想要看到我們執行 Container 的紀錄，可以使用 `logs` 指令來顯示。

```sh
$ docker logs demo-nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
... 省略 ...
2022/03/15 09:27:51 [notice] 1#1: start worker process 33
2022/03/15 09:27:51 [notice] 1#1: start worker process 34
2022/03/15 09:27:51 [notice] 1#1: start worker process 35
```

<br>

##### 刪除容器 (rm -f)

想要刪除不需要的 Container，可以使用 `rm` 指令來做刪除，`-f` 是強制刪除容器。

```sh
$ docker rm -f demo-nginx
demo-nginx
```

<br>

##### 進入容器 (exec)

想要進入 container 來查看資料或是修改檔案，可以使用 `exec` 來進入容器中。

```sh
$ docker exec -it demo-nginx /bin/bash
root@31a4a4a56e3e:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@31a4a4a56e3e:/# cd usr/bin/
root@31a4a4a56e3e:/usr/bin# pwd
/usr/bin
```

<br>

##### 匯出檔案 (export)

我們前面有提到說，如果刪除了容器，以前寫入的資料也會不見，如果想要輸出資料，可以使用 `export` 將可讀可寫的那一層匯成檔案。

```sh
$ docker export demo-nginx > demo-nginx.tar
$ ls | grep 'demo'
demo-nginx.tar
```

<br>

{{< callout >}}
save 跟 export 的區別：

還記得我們在儲存映像檔的時候有介紹到 save 嗎，那他跟 export 的區別是什麼呢？我們可以理解成

save 是把 Docker Image 原始檔做儲存，export 是把修改 Docker Image 的內容都一併儲存。

{{< /callout >}}

<br>

##### 匯入檔案 (import)

有匯出檔案，當然也有匯入檔案拉，可以使用 `import` 將我們匯出的檔案匯入 Docker Image 裡面。

```sh
$ cat ~/Desktop/demo-nginx.tar| docker import - import-nginx
sha256:7106935f0bfbdbb84f9eb20edb8cdb2c53207f5e0963f6a4e89d8267e9d98c56

$ docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
import-nginx   latest    7106935f0bfb   19 seconds ago   140MB
```

<br>

#### Container 的狀態

##### 檢查容器狀態 (inspect)

想要查看容器的狀態數據，可以使用 `inspect` 來顯示。

```sh
[
    {
        "Id": "sha256:1f56acbcbe9ec613a37e26934a84d98bed73879059f424dc69754520086baa37",
        "RepoTags": [
            "demo-image:latest"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2022-03-15T03:02:26.8321887Z",
        "Container": "",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
.... 省略 ....
```

<br>

##### 查看容器的 CPU、記憶體及網路使用 (stats)

想要查看容器的 CPU、記憶體及網路使用，可以使用 `stats` 來顯示。

```sh
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
31a4a4a56e3e   demo-nginx   0.00%     5.797MiB / 1.939GiB   0.29%     6.29kB / 2.83kB   1.46MB / 20.5kB   5
```

<br>

### 倉庫 (Repository)

我們在映像檔的章節有使用 `docker pull` 來下載別人的映像檔來使用，那我們要如何把我們做好的上傳上去 Docker hub 呢！(由於 Docker hub 是公開平台，所以大家都可以自由的下載映像檔，所以是公司機密的映像檔，就要避免上傳歐！)

```sh
$ docker login
Authenticating with existing credentials...
Login Succeeded
```

由於我有下載桌面版的 Docker ，所以登入時不需要再另外設定！

<br>

我們在上傳到 Docker hub 之前，需要先修改 Image 的 tag ，格式 `docker tag {Image Name} {DockerHub帳號}/{想要取的 Image Name}`

```sh
$ docker images | grep 'demo-image'
demo-image   latest    1f56acbcbe9e   24 hours ago   166MB

$ docker tag demo-image 880831ian/demo-image

$ docker images | grep 'demo-image'
880831ian/demo-image   latest    1f56acbcbe9e   24 hours ago   166MB
demo-image             latest    1f56acbcbe9e   24 hours ago   166MB
```

<br>

接下來就使用 `docker push`，將映像檔上傳到 Docker hub 上！

```sh
$ docker push 880831ian/demo-image
Using default tag: latest
The push refers to repository [docker.io/880831ian/demo-image]
7722c88c8d69: Pushed
68a85fa9d77e: Mounted from library/ubuntu
latest: digest: sha256:814caacaf3dad3eccb43dc9bcad635d0473bd5946295d40ca1ec23d13a5f6d0f size: 741
```

<br>

我們也登入 Docker hub 看一下，是不是真的上傳成功了～

<br>

{{< figure src="/docker/docker/repository.png" width="1000" caption="repository" >}}

<br>

## Docker 進階

本章節會分成三個常用功能來說明

- Volumes 介紹
- Network 模式介紹和比較
- Docker-compose 介紹與實作

<br>

### Volumes 介紹

我們在先前介紹 Container 時，也有說到，Container 會分成 Image 層、Init 層以及使用者可讀可寫層的這三層。當我們將 Container 刪除後，存放在 Docker Container 上的資料也會不見，雖然可以用 `export` 來儲存，但我們應該在根本上解決問題。

所以我們可以使用兩種方式來解決！

1. 在執行 `docker run` 指令時加入 `-v` 參數，將 Container 的檔案路徑映射到本地端的檔案路徑。
2. 在撰寫 Dockerfile 時，加入 `VOLUME` 指令，可以將資料存放在實體主機上。使用這個方法還要搭配我們介紹 [Container 的狀態 > 檢查容器狀態 (inspect)](https://pin-yi.me/docker/#檢查容器狀態-inspect) ，來查詢本地端檔案的存放路徑在哪。

<br>

#### 使用 -v 指令將容器映射到本地端

在使用 `docker run` 指令時，使用 `-v` 將容器檔案路徑映射到本地的檔案路徑。

```sh
$ docker run -it -v /Users/ian_zhuang/Desktop/data:/storage centos /bin/bash
latest: Pulling from library/centos
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
[root@a99b41fba3ca /]#
```

<br>

我們在著面的 data 資料夾隨意新增檔案或是資料，再進入 Docker Container 內的 /storage (該檔案是因為使用 -v 而新增的資料夾) ，看看有沒有同步新增。

```sh
[root@a99b41fba3ca /]# cd storage
[root@a99b41fba3ca storage]# pwd
/storage
[root@a99b41fba3ca storage]# ls
1.html	2.html	3.html
```

<br>

#### Dockerfile VOLUME 使用

我們先打開上次的 [Dockerfile](https://pin-yi.me/docker/#撰寫-dockerfile-映像檔) 檔案，在基礎映像檔資訊的底下使用 `VOLUME` 指令，加入 storage 資料夾，再將 Image Build 起來，啟動 Container，在 /storage 裡面隨便新增資料，最後我們在用 `docker inspect` 指令來找映射在本地端的路徑。

<br>

1. 在基礎映像檔資訊的底下使用 `VOLUME` 指令，加入 storage 資料夾

```sh
# 基礎映像檔資訊

FROM ubuntu:latest
VOLUME ["/storage"]
.... 省略 ....
```

<br>

2. 建立映像檔

```sh
$ docker build -t demo-image:v2 .

[+] Building 1.3s (6/6) FINISHED
 => [internal] load build definition from Dockerfile                                                                                            0.0s
 => => transferring dockerfile: 44B                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                1.2s
 => [1/2] FROM docker.io/library/ubuntu:latest@sha256:8ae9bafbb64f63a50caab98fd3a5e37b3eb837a3e0780b78e5218e63193961f9                          0.0s
 => CACHED [2/2] RUN apt-get update -y    && apt-get install nginx -y                                                                           0.0s
 => exporting to image                                                                                                                          0.0s
 => => exporting layers                                                                                                                         0.0s
 => => writing image sha256:0618bb2685ecfe200d9df4a91380d482031352d0e00cbfdf70fcd063aa8654fa                                                    0.0s
 => => naming to docker.io/library/demo-image:v2
```

<br>

3. 啟動 Container，並在 /storage 內新增隨意資料

```sh
$ docker run -it demo-image:v2 /bin/bash

root@4f8712562dfe:/# echo "Hello ian" > /storage/helloworld.txt
root@4f8712562dfe:/# ll /storage
total 12
drwxr-xr-x 2 root root 4096 Mar 16 05:39 ./
drwxr-xr-x 1 root root 4096 Mar 16 05:38 ../
-rw-r--r-- 1 root root   10 Mar 16 05:39 helloworld.txt
```

<br>

4. 使用 `inspect` 指令，來找到 Volume 在本地端映射的資料夾路徑，看看裡面有沒有我們在 Container 裡面新增的資料吧

```sh
$ docker inspect -f '{{.Mounts}}' 4f8712562dfe
[{volume 4fe10ca3f...省略 /var/lib/docker/volumes/4fe10ca3f234633164d9b3c541893c68db1b4f98806525076a2edd5c1c7863c4/_data /storage local  true }]

$ docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
/ # cd /var/lib/docker/volumes/4fe10ca3f234633164d9b3c541893c68db1b4f98806525076a2edd5c1c7863c4/_data
/var/lib/docker/volumes/4fe10ca3f234633164d9b3c541893c68db1b4f98806525076a2edd5c1c7863c4/_data # ls
helloworld.txt
```

<br>

{{< callout >}}
mac OS 找不到 /var/lib/docker/volumes：
由於 macOS 下的 docker 實際是在 vm 裡又多加一層，所以沒辦法直接訪問 /var/lib/docker/volumes，必須先透過以下指令進入 VM 中。

```sh
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

詳細內容可以參考 [Where is /var/lib/docker on Mac/OS X](https://stackoverflow.com/questions/38532483/where-is-var-lib-docker-on-mac-os-x)。

{{< /callout >}}

<br>

#### 容器與容器之間資料共享

如何啟用容器與容器之間的資料共享，可以用以下方式

1. 先啟動第一個容器指令如下

```sh
docker run -it -v /data --name=container1 centos /bin/bash
[root@ a0307ce757ca /]#
```

<br>

2. 另二個容器指令如下

```sh
docker run -it --volumes-from container1 --name=container2 centos /bin/bash
[root@720d57983cd4 /]#
```

- `--volumes-from` 參數指定 container1 的資料會與 container2 做共享。

<br>

3. 我們在第一個容器，進入 /data 資料夾，隨機輸入資料

```sh
[root@a0307ce757ca /]# cd /data/
[root@a0307ce757ca data]# echo "ian~" > hello.txt
```

<br>

4. 再來看一下第二個容器 /data 資料夾，是否有我們在容器(a0307ce757ca)產生的資料

```sh
[root@720d57983cd4 /]# cat /data/hello.txt
ian~
```

<br>

### Network 模式和比較

在執行 `docker run` 其中一個參數是 `--net` ，他可以設定 Container 要使用哪一種的網路模式，以下分別說明這些網路模式

- none：在執行 Container 時，網路功能是關閉的，所以無法與此 Container 連線。
- container：使用相同的 Network Namespace ，假設 Container 1 的 IP 是 172.17.0.2，那 Container 2 的 IP 也是 172.17.0.2。
- host：Container 的網路設定和實體主機使用相同的網路設定，所以 Container 裡面也可以修改實體機器的網路設定，因此使用此模式需要考慮網路安全性上的問題。
- bridge：Docker 預設就是此網路模式，這個網路模式就像是 NAT 的網路模式，例如實體主機的 IP 是 192.168.1.10 它會對應到 Container 裡面的 172.17.0.2，在啟動 Docker 的服務時會有一個 docker0 的網路卡來做此網路的橋接。
- overlay：Container 之間可以在不同的實體機器上做連線，例如 Ｈ ost 1 有一個 Container 1 ，然後 Host 2 有一個 Container 2，Container 1 可以直接使用 overlay 的網路模式和 Container 2 做網路連線。
- macvlan：可以直接分配實體網卡的 MAC address 給特定的 Container，讓 Container 透過實體的網卡使用網路。

<br>

那我們就來實作每一個網路模式吧！

#### none

我們使用 `docker run`指令，在後面加入參數 `--net=none` ，我們建立 `jonlabelle/network-tools` (裡面有很多網路測試工具)。

```sh
$ docker run -it --net=none jonlabelle/network-tools
[docker@network-tools]$ ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
	    .... 省略 ....
```

可以看到我們使用 `ifconfig` 查詢，只有本地端的 127.0.0.1 IP，我們在使用 `ping` 來測試 google 網站吧

```sh
[docker@network-tools]$ ping www.google.com
ping: www.google.com: Try again
```

<br>

#### container

我們先啟動一個名為 container1 的容器

```sh
docker run --name container1 -it jonlabelle/network-tools
```

<br>

再開啟另一個 Terminal 來啟動 container2 的容器，並設定相同的 Network Namespace

```sh
docker run --name container2 --net=container:container1 -it jonlabelle/network-tools
```

<br>

一樣使用 `ifconfig` 查詢，可以發現兩個容器的 IP 都是相同的

```sh
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          .... 省略 ....
```

<br>

#### host

在執行 `docker run` 指令時，在後面加入參數 `--net=host` ，來測試 host 模式

```sh
docker run --net=host -it jonlabelle/network-tools
```

<br>

可以看到 Container 的網路資訊和實體主機的網路資訊是相同的結果

```sh
[docker@network-tools]$ ifconfig
br-36a27cab1817 Link encap:Ethernet  HWaddr 02:42:20:DF:DB:FD
          inet addr:172.20.0.1  Bcast:172.20.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          .... 省略 ....
```

{{< figure src="/docker/docker/host.png" width="500" caption="host 網路模式" >}}

<br>

#### bridge

在執行 `docker run` 指令時，在後面加入參數 `--net=bridge` ，來測試 bridge 模式

```sh
docker run --net=bridge -it jonlabelle/network-tools
```

<br>

docker 會新增一個虛擬網卡作為容器網路對外的出口，預設名稱為 `docker0`，docker0 會跟本機的對外網卡(圖中的 `eth0` )相連，藉此取得對外連線的能力，也因為每一個容器都會使用一個 veth device 與 docker0 相連，所以也具備對外連線的能力。

{{< figure src="/docker/docker/host.png" width="600" caption="bridge 網路模式" >}}

<br>

#### overlay

下圖是說明 Host1 實體主機裡面有 Container1，然後 Host2 實體主機裡面有 Container2，可以透過 Docker overlay 模式將 Container1 和 Container2 連接做溝通。另外還需要一個 Consol 來存連線的資料庫，在使用 overlay 時要先在 Docker 做設定，這樣才能存放 overlay 網路模式的連線資訊。

{{< figure src="/docker/docker/overlay.png" width="550" caption="overlay 網路模式" >}}

<br>

#### macvlan

macvlan 的原理就是在本機的網卡上虛擬出很多個子網卡，通過不同的 MAC 位置在數據鏈路層進行網路資料的轉發。

{{< figure src="/docker/docker/macvlan.png" width="450" caption="macvlan 網路模式" >}}

<br>

### Docker-compose

我們在執行多個容器時，需要重複的下 `run` 指令來執行，以及容器與容器之間要做關聯也要記得每一個之間要怎麼連結，會變得很麻煩且不易管理，所以有了 `docker-compose` 可以將多個容器組合起來，變成一個強大的功能。

只要寫一個 `docker-compose.yml` 把所有會使用到的 Docker image 以及每一個容器之間的關聯或是網路的設定都寫上去，最後再使用 `docker-compose up` 指令，就可以把所有的容器都執行起來囉！

<br>

我們就直接來實作我們這次的標題，要使用 docker-compose 來整合 PHP MySQL Nginx 環境。

1. 我們先開啟一個資料夾，取名叫 `docker-compose` ，來放置我們的 docker-compose 檔案
2. 接著新增 `docker-compose.yml` 檔案，要準備來撰寫我們的設定檔囉！ 由於內容有點長，所以我分段說明，([這邊有放已經寫好的檔案歐](https://github.com/880831ian/docker-compose-php-mysql-nginx))

<br>

**檔案目錄**

{{< filetree/container >}}
{{< filetree/file name="Docker-compose.yml" >}}
{{< filetree/folder name="docker-volume" >}}
{{< filetree/folder name="html" >}}
{{< filetree/file name="index.php" >}}
{{< filetree/file name="info.php" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< filetree/folder name="nginx" >}}
{{< filetree/file name="Dockerfile" >}}
{{< filetree/file name="default.conf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="php" >}}
{{< filetree/file name="Dockerfile" >}}
{{< /filetree/folder >}}
{{< /filetree/container >}}

<br>

```yml
version: "3.8"

services:
... 省略 ....
```

可以看到一開頭，會先寫版本，這邊代表的是會使用 3.8 版本的設定檔，詳細版本對照可以參考 [Compose file versions and upgrading](https://docs.docker.com/compose/compose-file/compose-versioning/)

services 可以設定用來啟動多的容器，裡面我們總共放了三個容器，分別是 nginx、php、mysql 。

那我們來看看 nginx 裡面放了什麼吧！我會依照程式碼往下說明，有不清楚的可以底下留言！

<br>

#### nginx

```yml
nginx:
  build: ./nginx/
  container_name: nginx
  ports:
    - 7777:80
  volumes:
    - ./docker-volume/log/:/var/log/nginx/
```

nginx 的 `build` 就是要執行這個 nginx 容器的映像檔，還記得我們也可以使用 Dockerfile 來撰寫映像檔案嗎！?

由於我們還要設定其他內容，所以特別另外拉一個 nginx 資料夾來放置，裡面放了兩個檔案，分別是 Dockerfile、default.conf。

Dockerfile 檔案裡面會使用 nginx 版本 1.20 ，並將 default.conf 複製到容器的 /etc/nginx/conf.d/default.conf 來取代設定。

以及我們使用 `ports` 將容器 80 Port 指向本機 7777 Port ，格式是 `本機 Port : 容器 Port`，

再使用 `volumes` 來設定我們 nginx 容器 `log` 資料夾映射到本機的 `./docker-volume/log/` 資料夾。

<br>

#### php

```yml
php:
  build: ./php/
  container_name: php
  expose:
    - 9000
  volumes:
    - ./docker-volume/html/:/var/www/html/
```

php 的 `build` 是要執行這個 php 容器的映像檔，由於我們還要設定其他內容，所以特別另外拉一個 php 資料夾來放置 Dokcerfile。

Dockerfile 檔案裡面會使用 php 版本 7.4-fpm，並且在容器執行 `docker-php-ext-install`、`mysqli`。

並將 Port 9000 發佈於本機，再使用 `volumes` 來設定 `/var/www/html` 網站根目錄映射到本機的 `./docker-volume/html/` 資料夾。

<br>

#### mysql

```yml
mysql:
  image: mysql:8.0.28
  container_name: mysql
  volumes:
    - ./docker-volume/mysql/:/var/lib/mysql
  environment:
    MYSQL_ROOT_PASSWORD: secret
    MYSQL_DATABASE: mydb
    MYSQL_USER: myuser
    MYSQL_PASSWORD: password
```

mysql 使用的映像檔是 mysql 版本是 8.0.28，我們為了要保留資料庫的資料，所以將容器的 `/var/lib/mysql` 映射到本地端 `./docker-volume/mysql` 資料夾。

最後的環境變數，設定 root 帳號的登入密碼，以及要使用的資料庫、使用者的帳號、使用者的密碼。

<br>

最後在上面的 ([這邊有放已經寫好的檔案歐](https://github.com/880831ian/docker-compose-php-mysql-nginx)) 裡面還有多一個 docker-volume/html 的資料夾，就是我們剛剛映射到本地端的資料夾，資料夾內已經放有連線測試的檔案，輸入網址 `http://127.0.0.1:7777/index.php`，如果開啟後有顯示下方畫面，就代表我們成功用 docker-compose 將 PHP MySQL Nginx 整合再一起囉！

{{< figure src="/docker/docker/localhost-7777.png" width="450" caption="測試是否成功用 docker-compose 整合 PHP MySQL Nginx" >}}

<br>

## 參考資料

Docker 官網：[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

什麼是 Docker？：[https://aws.amazon.com/tw/docker/](https://aws.amazon.com/tw/docker/)

Docker－－從入門到實踐：[https://philipzheng.gitbook.io/docker_practice/#yuan-chu-chu-ji-can-kao-zi-liao](https://philipzheng.gitbook.io/docker_practice/#yuan-chu-chu-ji-can-kao-zi-liao)

Dockerfile 建立自訂映像檔 — 架起網站快速又簡單（一）：[https://medium.com/@jackercleaninglab/dockerfile-%E5%BB%BA%E7%AB%8B%E8%87%AA%E8%A8%82%E6%98%A0%E5%83%8F%E6%AA%94-%E6%9E%B6%E8%B5%B7%E7%B6%B2%E7%AB%99%E5%BF%AB%E9%80%9F%E5%8F%88%E7%B0%A1%E5%96%AE-%E4%B8%80-22b2743f97b9](https://medium.com/@jackercleaninglab/dockerfile-%E5%BB%BA%E7%AB%8B%E8%87%AA%E8%A8%82%E6%98%A0%E5%83%8F%E6%AA%94-%E6%9E%B6%E8%B5%B7%E7%B6%B2%E7%AB%99%E5%BF%AB%E9%80%9F%E5%8F%88%E7%B0%A1%E5%96%AE-%E4%B8%80-22b2743f97b9)

用 30 天來介紹和使用 Docker：[https://ithelp.ithome.com.tw/users/20103456/ironman/1320](https://ithelp.ithome.com.tw/users/20103456/ironman/1320)

Docker 網路簡介：[https://godleon.github.io/blog/Docker/docker-network-overview/](https://godleon.github.io/blog/Docker/docker-network-overview/)

Docker-compose Giving static IP in network mode : bridge：[https://stackoverflow.com/questions/61949319/docker-compose-giving-static-ip-in-network-mode-bridge](https://stackoverflow.com/questions/61949319/docker-compose-giving-static-ip-in-network-mode-bridge)
