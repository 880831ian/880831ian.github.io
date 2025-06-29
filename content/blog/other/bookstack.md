---
title: "Bookstack 開源知識庫筆記平台安裝 (K8s + docker)"
type: docs
weight: 9996
date: 2023-04-07
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

最近剛好有公司同事離職，想要把交接的資料給整理整理，雖然部門之前有架設專用的 wiki 給 RD 使用，但覺得介面沒有到很好用，於是就在網路上尋找，可以多人編輯的筆記系統，一開始有想過用 CodiMD (HackMD)，但考量到需要多層的架構來區分文件，最後選擇 [Bookstack](https://www.bookstackapp.com/) 這個開源知識庫筆平台來作為組內的筆記系統，以下會簡單說一下 Bookstack 的特色以及使用 K8s 跟 docker 的安裝教學。

<br>

## Bookstack 介紹

介紹部分主要參考 [Bookstack 簡介](https://docs.ossii.com.tw/books/bookstack/page/bookstack)，以下列出會選擇它的三個特色：

### 簡潔的書本列表模式

{{< figure src="/other/bookstack/1.png" width="850" caption="書架分類" >}}

<br>

{{< figure src="/other/bookstack/2.png" width="850" caption="書本分類" >}}

<br>

{{< figure src="/other/bookstack/3.png" width="850" caption="頁面章節分類" >}}

<br>

最主要是因為有以上幾個不同的分層架構，在資料整理上會更方便、更好彙整，可以自訂書架以及書本、頁面或是章節的封面以及內容。

<br>

### 強大的搜尋功能

<br>

{{< figure src="/other/bookstack/4.png" width="850" caption="搜尋功能" >}}

當我們的筆記內容越來越多，雖然有上面提到的分類模式，想要找到內容還是需要花一段時間，但 Bookstack 有強大的搜尋功能，可以針對書架、書本、章節或是書面個別搜尋，可以利用時間、標籤、標題或是內容來快速找到想要的內容。

<br>

### 畫圖功能

在討論系統或是程式的架構，最好的方式就是用畫圖的來表示。在以往都是使用 [Draw.io](https://app.diagrams.net/) 來畫圖，當畫完圖後需要匯出畫好的圖以外，如果怕畫圖的原檔消失，還需要再另外下載原檔來保留，很不方便，又怕忘記下載，而 Bookstack 內建可直接編輯的功能，當畫完圖後有問題，可以直接點擊圖片來編輯，而這些功能還會搭配內建的版控，若有問題還可以還原到正確的圖片版本。

<br>

{{< figure src="/other/bookstack/5.gif" width="1000" caption="畫圖功能" >}}

<br>

## 安裝說明

那簡單說明為什麼會選擇 Bookstack 我們就來安裝它，這邊有使用 K8s + docker 來測試安裝，那我們就一起來看程式碼吧，[程式碼放在這](https://github.com/880831ian/bookstack) 👈：

### K8s

```yaml {filename="namespace.yaml"}
apiVersion: v1
kind: Namespace
metadata:
  name: bookstack
```

<br>

我習慣會將不同的服務切 namespace 來部署，大家可以依照習慣來使用，下方的 yaml 都是建在此 namespace 上。

<br>

```yaml {filename="deployment.yaml"}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstack
  namespace: bookstack
  labels:
    app: bookstack
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookstack
  template:
    metadata:
      labels:
        app: bookstack
    spec:
      containers:
        - name: bookstack
          image: linuxserver/bookstack
          ports:
            - name: http
              containerPort: 80
          env:
            - name: DB_DATABASE
              value: bookstack
            - name: DB_HOST
              value: <<更換此處>>
            - name: DB_PORT
              value: "3306"
            - name: DB_PASSWORD
              value: <<更換此處>>
            - name: DB_USERNAME
              value: <<更換此處>>
            - name: MAIL_USERNAME
              value: example@test.com
            - name: MAIL_PASSWORD
              value: mailpass
            - name: MAIL_HOST
              value: smtp.server.com
            - name: MAIL_PORT
              value: "465"
            - name: MAIL_ENCRYPTION
              value: SSL
            - name: MAIL_DRIVER
              value: smtp
            - name: MAIL_FROM
              value: no-reply@test.com
            - name: APP_URL
              value: https://<<更換此處>>
            - name: APP_LANG
              value: zh_TW
            - name: APP_TIMEZONE
              value: Asia/Taipei
          resources:
            limits:
              cpu: "0.5"
              memory: "512Mi"
```

<br>

必要更換的參數有標示 **<<更換此處>>**，其餘可以依照各組織來自行配置，Bookstack 會將內容存在 db，圖片等存在 pod 中，需要永久保存請使用 pvc + pv 或是另外掛 nas。

<br>

```yaml {filename="svc.yaml"}
apiVersion: v1
kind: Service
metadata:
  name: bookstack
  namespace: bookstack
spec:
  type: NodePort
  selector:
    app: bookstack
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

<br>

```yaml {filename="ingress.yaml"}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookstack-ingress
  namespace: bookstack
  annotations:
    kubernetes.io/ingress.class: nginx
    service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: "true"
spec:
  rules:
    - host: <<更換此處>>
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: bookstack
                port:
                  number: 80
```

<br>

最後透過 ingress 的 domain 去訪問 svc > pod 上，就完成部署拉～第一次登入要使用預設帳號及密碼 admin@admin.com/password

<br>

### docker

```yaml {filename="docker-compose.yaml"}
version: "3.8"
services:
  bookstack:
    image: lscr.io/linuxserver/bookstack
    container_name: bookstack
    environment:
      - PUID=<<更換此處>>
      - PGID=<<更換此處>>
      - DB_HOST=bookstack_db
      - DB_PORT=3306
      - DB_USER=bookstack
      - DB_PASS=bookstack
      - DB_DATABASE=bookstackapp
      - APP_LANG=zh_TW
      - APP_TIMEZONE=Asia/Taipei
      - APP_URL=<<更換此處>>
      - GOOGLE_APP_ID=<<更換此處>>
      - GOOGLE_APP_SECRET=<<更換此處>>
    volumes:
      - /bookstack/config:/config
    ports:
      - 80:80
    restart: unless-stopped
    depends_on:
      - bookstack_db
  bookstack_db:
    image: lscr.io/linuxserver/mariadb
    container_name: bookstack_db
    environment:
      - PUID=<<更換此處>>
      - PGID=<<更換此處>>
      - MYSQL_ROOT_PASSWORD=bookstack
      - TZ=Asia/Taipei
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=bookstack
    volumes:
      - /bookstack/config:/config
    restart: unless-stopped
```

<br>

docker 的部分就更簡單了，一樣是把 **<<更換此處>>** 換成對應的內容即可，env 的部分可以參考官方文件，這邊比較特別的是我們有多使用 Google 來 Oauth 登入，Bookstack 支援多種的 Oauth 登入方式，可以參考 [Third Party Authentication](https://www.bookstackapp.com/docs/admin/third-party-auth/)。

<br>

架設好 Bookstack 就可以開始寫部門或是組織內的筆記拉～ 如果有開放外網連線，要記得修改預設的管理員帳號，以及用管理員帳號登入，到功能與安全的公開存取給取消，這樣就必須要登入才可以瀏覽筆記了！那就交給大家自己去玩這個好用的筆記工具囉～～ 😍

<br>

{{< figure src="/other/bookstack/6.png" width="850" caption="功能與安全設定" >}}

<br>

## 參考資料

BookStack 簡介：[https://docs.ossii.com.tw/books/bookstack/page/bookstack](https://docs.ossii.com.tw/books/bookstack/page/bookstack)

BookStack Installation：[https://www.bookstackapp.com/docs/admin/installation/](https://www.bookstackapp.com/docs/admin/installation/)
