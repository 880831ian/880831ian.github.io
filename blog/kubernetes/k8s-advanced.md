# Kubernetes (K8s) 介紹 - 進階 (Service、Ingress、StatefulSet、Deployment、ReplicaSet、ConfigMap)
前面我們在[基本篇](../k8s)中，為了使 Pod 能夠與本機連線，使用了 `port-forward`，還有另一種方法就是今天要介紹的第一個主題：Service

那我們會先複習一下 `port-forward`，再來介紹 `Service`，當然後面也會有實際操作，請大家跟我繼續一起學下去吧 ！

<br>

## port-forward

`port-forward` 簡單來說就是把本機的某一個 Port 與 Pod 所開放對外的 Port 做映射，就像是我們在 Docker 跑 container 時會使用 -p 來連結機器與 container 的 port 一樣～

使用的方法也很簡單且方便，使用 `kubectl port-forward <pod> <external-port>:<pod-port>`，我們拿 [Kubernetes 基本篇](../k8s)最後的範例，來做說明：

```sh
$ kubectl port-forward kubernetes-demo-pod 3000:3000

Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
Handling connection for 3000
```

我們把 kubernetes-demo-pod 這個 pod，用 port-forward 設定本機 3000 port 與 pod 3000 port 做映射，當我們瀏覽 `http://localhost:3000` 就可以看到 pod 裡面的內容了！

雖然很方便，可以馬上就開好要映射的 port，但缺點就是每次建立 pod 時都需要手動去打指令來設定 port，且時間久了，也會忘記本機上哪些 port 有被使用到，因此這邊推薦使用 `Service` 來取代 `port-forward`，那我們來看看 `Service` 是什麼吧。

<br>

## 什麼是 Service ?

`service` 他其實就是建立的一個網路連線通道，可以讓應用程式正確的連結到正在運行的 pods，而 `service` 又有 4 種的表現形式，我們接下來會一個一個簡單介紹：

### ClusterIP

它是 service 的預設值，所以沒有設定時，預設就是使用該方式做連線，它代表這個 service 只能在相同的 cluster 內使用，無法讓外部做使用。

<br>

### NodePort

簡單來說它可以從外部連線到內部使用。假設本機有其他服務，例如：nginx 之類的服務，還有架一個 K8s 的 cluster ，這時候只要設定好 NodePort，就可以讓本機使用 K8s cluster 來使用內部的服務。

<br>

### ExternalName

主要是為了讓不同 namespace ，以 ClusterIP 所生成的 service 可以利用 ExternalName 設定外部名稱，藉以連到指定的 namespace Service。

<br>

### LoadBalancer

這個屬性是強化版的 NodePort，除了 NodePort 可以讓外部連線的優點以外，同時也建立負載平衡的機制來分散流量，很可惜 LoadBalaner 只提供雲端服務，例如：GCP、AWS 等等都有支援，目前 minikube 要使用 LoadBalancer 需要先啟動 `tunnel` 才能做使用。`tunnel` 是什麼呢？我們後面會說明！

<br>

{{< figure src="/kubernetes/k8s-advanced/service.webp" width="300" caption="k8s service 流程圖 [Kubernetes 那些事 — Service 篇](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-service-%E7%AF%87-d19d4c6e945f)" >}}

<br>

### NodePort 實作

那我們來用 Service (NodePort) 改寫[基本篇](../k8s)的連線問題 ：([Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/Service-NodePort))

- service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-demo-service
  labels:
    app: demo
spec:
  type: NodePort
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30001
  selector:
    app: demo
```

結構與 kubernetes-demo.yaml 相同，以下簡單說明不同之處：

kind

該元件的屬性，此設定檔的類型是：Service

spec

- type：指定此 Service 要使用的方法，這邊我們使用 NodePort。
- ports.protocol：此為連線的網路協議，預設值為 `TCP`，當然也可以使用 `UDP`。
- ports.port：此為建立好的 Service 要以哪個 Port 連接到 Pod 上。
- ports.targetPort：此為目標 Pod 的 Port ，通常 port 跟 targetPort 一樣。
- ports.nodePort：此為機器上的 Port 要對應到該 Service 上，這個設定要 nodePort 形式的 Service 才會有效果，假設今天沒有設定 nodePort ，Kubernetes 會自動開一個機器上的 Port 來對應該 Service ，範圍是在 30000 - 37267 之間。
- selector.app：如果要使 Service 連接到正確的 Pod 就必須利用 `selector`，只要原封不動的把 Pod 的 `Labels` 複製上去即可。

<br>

接下來使用 `kubectl apply` 建立 service：

```sh
$ kubectl apply -f service.yaml
```

<br>

{{< figure src="/kubernetes/k8s-advanced/service-4.webp" width="1300" caption="k8s 建立 service (NodePort)" >}}

<br>

接下來取得 minikube Node IP，可以使用：

```sh
$ minikube ip

192.168.64.11
```

<br>

打開瀏覽器搜尋 `192.168.64.11:30001` 就可以看到我們可愛的柴犬囉 ><

<br>

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-1.webp" width="1000" caption="成功顯示柴犬" >}}

<br>

### LoadBalancer 實作

那我們來用 Service (LoadBalancer) 改寫[基本篇](../k8s)的連線問題 ：([Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/Service-LoadBalancer))

- service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-demo-service
  labels:
    app: demo
spec:
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  selector:
    app: demo
```

結構與 kubernetes-demo.yaml 相同，以下簡單說明不同之處：

kind

該元件的屬性，此設定檔的類型是：Service

spec

- type：指定此 Service 要使用的方法，這邊我們使用 LoadBalancer。
- ports.protocol：此為連線的網路協議，預設值為 `TCP`，當然也可以使用 `UDP`。
- ports.port：此為建立好的 Service 要以哪個 Port 連接到 Pod 上。
- ports.targetPort：此為目標 Pod 的 Port ，通常 port 跟 targetPort 一樣。
- selector.app：如果要使 Service 連接到正確的 Pod 就必須利用 `selector`，只要原封不動的把 Pod 的 `Labels` 複製上去即可。

<br>

接下來使用 `kubectl apply` 建立 service：

```sh
$ kubectl apply -f service.yaml
```

<br>

{{< figure src="/kubernetes/k8s-advanced/service-1.webp" width="1300" caption="k8s 建立 service (LoadBalancer)" >}}

可以看到 dashboard 有我們剛剛啟動的 Service，但是啟動後前面的燈是黃色的，是因為 minikube LoadBalancer 需要透過 `tunnel` 才可以使用，可以參考 minikube 官網說明：

<br>

{{< figure src="/kubernetes/k8s-advanced/service-2.webp" width="800" caption="[minikube 官網](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel) 說明 LoadBalancer minikube tunnel" >}}

所以我們需要使用 `minikube tunnel` 來啟動 tunnel

<br>

```sh
$ minikube tunnel

✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

🏃  Starting tunnel for service kubernetes-demo-service.
```

<br>

就可以看到燈號已經變成綠燈，在外部 Endpoints 多一個連結，可以直接點開

<br>

{{< figure src="/kubernetes/k8s-advanced/service-3.webp" width="1300" caption="k8s service" >}}

<br>

就可以看到我們可愛的柴犬囉 ><

<br>

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu.webp" width="1000" caption="成功顯示柴犬" >}}

<br>

## 什麼是 Ingress ?

還記得我們在 Service - NodePort 時，需要打 `<ip>:<port>` ，但現在網站除了網域以外，基本上不會需要自己去打 IP 以及 Port 了吧！那為了解決這個問題，有了 Ingreess。

Ingress 可以幫助我們統一對外的 port number，並根據 hostname 或是 pathname 來決定請求要轉發到哪一個 Service 上，之後就可以利用該 Service 連接到 Pod 來處理服務。

我們先來看一下一般的 Service ：

<br>

{{< figure src="/kubernetes/k8s-advanced/describe-service.webp" width="800" caption="Service 圖片來源：[[Day 19] 在 Kubernetes 中實現負載平衡 - Ingress Controller](https://ithelp.ithome.com.tw/articles/10196261)" >}}

可以看到當多個 Service 同時運行時，Node 都需要有對應的 port number 去對應每個 Server 的 port number。像是 GCP 這種雲端服務，每台機器都會配置屬於自己的防火牆。這也代表，不論新增、刪除 Service 物件，都必須要額外多調整防火牆的設定，Port 的管理也想對複雜。

<br>

若是使用 Ingress，我們只需要開放一個對外的 port numer，Ingree 可以在設定檔中設定不同的路徑，決定要將使用者的請求傳送到哪一個 Service 物件上：

{{< figure src="/kubernetes/k8s-advanced/describe-ingress.webp" width="900" caption="Ingress 圖片來源：[[Day 19] 在 Kubernetes 中實現負載平衡 - Ingress Controller](https://ithelp.ithome.com.tw/articles/10196261)" >}}

這樣的設計，除了讓維運人員不需要維護多個 port 或是頻繁的更改防火牆外，可以自訂條件的功能，也使得請求的導向可以更加彈性。

<br>

### Ingress 功能

- 將不同"路徑"的請求對應到不同的 Service 物件

若沒有設定網域，則該機器上的所有網域只要透過此路徑都可以連接到指定的 Service 物件。

<br>

- 將不同"網域"的請求對應到不同的 Service 物件

若沒有設定路徑，則會以 `/` 路徑連接到指定的 Service 物件。

<br>

- 支援 SSL Termination

SSL 全名是傳輸層安全性協定，而網站通常都會利用 https 進行加密以確保資料安全，但 Service 與 Pod 之間的溝通都是以無加密方式傳輸，所以 Ingress 就支援解密，讓 Service 與 Pod 可以正常溝通傳遞資料。

<br>

### minikube 啟動 Ingress

由於 minikube 預設沒有啟動 Ingress 功能，因此需要額外使用 `minikube addons enable ingress` 讓 minikube 啟動 Ingress (Ingress 也需要先安裝 Hyperfix)：

<br>

{{< figure src="/kubernetes/k8s-advanced/ingress.webp" width="900" caption="Ingress 圖片來源：[[Day 19] 在 Kubernetes 中實現負載平衡 - Ingress Controller](https://ithelp.ithome.com.tw/articles/10196261)" >}}

<br>

### 設定 /etc/hosts

加入 Ingress 基本上就需要網域才可以使用，但我們在本機上做練習，所以只要修改本機的 host 檔案就可以了(加入 minikube ip 以及想要的網域名稱)。

```sh
$ vim /etc/hosts

192.168.64.11 test.tw
192.168.64.11 test-test.tw
```

<br>

因為有兩種方法，第一種 (設定網域以及路徑)跟第二種 (只設定路徑沒有設定網域)，所以底下也會分成兩種來說明！

### 設定網域以及路徑

[Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/Ingress-Domain)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-demo-ingress
spec:
  rules:
    - host: "test.tw"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-demo-service
                port:
                  number: 3000
```

<br>

### 只設定路徑沒有設定網域

[Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/Ingress-Path)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-demo-ingress
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-demo-service
                port:
                  number: 3000
```

<br>

Ingress 整體寫法與 Service 差不多，只差在 `spec` 的細部設定，我們就來說一下 spec 設定吧：

對了～ Ingress 的 apiVersion 不像是 Pod 跟 Service 一樣使用 `v1` ，因為目前支援 Ingress 的 API 只有 networking.k8s.io/v1，詳細可以參考 [kubernetes Ingress 官網](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- rules

這代表這個 Ingress 的轉發規則，此 Ingress 所有的設定都必須寫載 `rules` 內。

- host

設定可以連接到 Service 物件的網路名稱。

- path

設定可以連接到 Service 物件的路徑名稱。

- pathType

分為 `Prefix` 和 `Exact` 兩種，Prefix：前綴符合就符合規則 ; Exact：需要完全一致才行，包含大小寫。

|  type  |  path  | request path | macth |
| :----: | :----: | :----------: | :---: |
| Prefix |   /    |   全部路徑   |  Yes  |
| Exact  |  /aa   |     /aa      |  Yes  |
| Exact  |  /bb   |     /cc      |  No   |
| Prefix |  /aa   |   /aa,/aa/   |  Yes  |
| Prefix | /aa/cc |   /aa/ccc    |  No   |

- service.name

設定連接到的 Service 名稱，這裡要填寫的就是 Service 中在 `metadata` 內寫的 `name`。

- service.port.number

設定要經由哪個 port 連接到 Service 物件，就像是 Service 的 Port 要連接到 Pod 的 targetPort。

<br>

### 建立 Ingress

一樣我們分成兩個做說明

#### 設定網域以及路徑

一樣使用 `apply` 來建立：

```shell
$ kubectl apply -f ingress.yaml

ingress.networking.k8s.io/kubernetes-demo-ingress created
```

<br>

使用 `kubectl get ing` 來查詢 Ingress 狀況：

```shell
$ kubectl  get ing

NAME                      CLASS   HOSTS     ADDRESS         PORTS   AGE
kubernetes-demo-ingress   nginx   test.tw   192.168.64.11   80      10m
```

接下來我們分別測試寫在 /etc/host 裡面的兩個網域，使用瀏覽器搜尋 `test.tw` 跟 `test-test.tw`：

<br>

- `test.tw`

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-3.webp" width="1000" caption="成功顯示柴犬" >}}

<br>

- `test-test.tw `

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-4.webp" width="800" caption="顯示 404" >}}

<br>

從上面結果可以知道，因為我們有設定 domain，所以只有符合的才會連接到 Service 。

<br>

#### 只設定路徑沒有設定網域

一樣使用 `apply` 來建立：

```shell
$ kubectl apply -f ingress.yaml

ingress.networking.k8s.io/kubernetes-demo-ingress created
```

<br>

使用 `kubectl get ing` 來查詢 Ingress 狀況：

```shell
$ kubectl  get ing

NAME                      CLASS   HOSTS   ADDRESS         PORTS   AGE
kubernetes-demo-ingress   nginx   *       192.168.64.11   80      5m56s
```

接下來我們分別測試寫在 /etc/host 裡面的兩個網域，使用瀏覽器搜尋 `test.tw` 跟 `test-test.tw`：

<br>

- `test.tw`

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-3.webp" width="1000" caption="成功顯示柴犬" >}}

<br>

- `test-test.tw `

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-5.webp" width="1000" caption="顯示 404" >}}

<br>

從上面結果可以知道，可以發現 hosts 的部分是 `*` ，這代表所有背後指向 `192.168.64.11` 這個 IP 的網域都可以直接連接到 Service 物件。

<br>

## 什麼是 Replication Controller ?

大家看到 Controller 就知道 Replication Controller 也是一種 Controller 負責控制 Replication，而 Replication 翻成中文是複製的意思，在 Kubernetes 中 Replication 代表同一種 Pod 的複製品。

這邊要帶給大家認識一個重要的設定：`replica`，`replica` 就是複製品的意思，透過這個設定我們就可以快速產生一樣內容的 Pod，舉例來說：今天設定了 `replica: 3` 就代表會產生兩個內容一樣的 Pod 出來。

<br>

{{< figure src="/kubernetes/k8s-advanced/ReplicationController.webp" width="700" caption="Kubernetes StatefulSet 架構" >}}

<br>

### Replication Controller 用途

上面有提到 Replication Controller 可以利用設定 `replica` 的方式快速建立 Pod 數量，除了建立之外 Replication Controller 也確保 Pod 數量與我們設定的 `replica` 一致，假如今天不小心刪除其中一個 Pod，這時候 Replication Controller 會自動再產生一個新的 Pod 來補齊刪除的 Pod 空缺，所以我們可以善用 Replication Controller 來讓系統更佳穩定。

<br>

### Replication Controller 寫法

我們來修改一下之前的 kubernetes-demo.yaml Pod 檔案：([Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/ReplicationController))

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubernetes-demo
spec:
  replicas: 3
  selector:
    app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: kubernetes-demo-container
          image: 880831ian/kubernetes-demo
          ports:
            - containerPort: 3000
```

這個設定檔看似複雜但其實很簡單，可以發現 `template` 區塊內的設定基本上就是 Pod 的設定，再加上一些屬於 Replication Controller 的設定。

由於我們要建立的是 Replication Controller，因此在一開始的 `spec` 要填的是 Replication Controller 的設定，所以 `replica` 會擺在第一個 `spec` 內。

可以再看到 `selector`，前面提到 Replication Controller 要控制的就是 Pod 的數量，所以這邊的 `selector` 就是要選取 Pod，就跟我們在 Service 要選取 Pod 的一樣。

最後一個新的設定：`template`，`template` 就是用來定義 Pod 的資訊，所以 Pod 的內容像是 `metadata`、`spec` 等等都會寫在 `template` 內，所以可以把 `template` 想像成不需要寫 `apiVersion` 跟 `kind` 的 Pod 壓模檔，有了這個觀念再來看 `template` 內的描述就很簡單，只是把 Pod 的內容複製過來而已，而 `template` 內的 `spec` 就是寫上 Pod 的 container 資訊。

<br>

### Replication Controller 建立

老樣子，也是使用 `apply` 來建立壓模檔：

```shell
$ kubectl apply -f kubernetes-demo.yaml

replicationcontroller/kubernetes-demo created
```

<br>

來查看一下 Replication Controller 是否有成功建立起來，可以使用 `kubectl get rc` 來查詢：

```sh
$ kubectl get rc

NAME              DESIRED   CURRENT   READY   AGE
kubernetes-demo   3         3         3       28s
```

<br>

接下來可以查看 Pod 是否有出現 3 個，所以使用 `kubectl get po` 來查詢：

```sh
$ kubectl get po

NAME                    READY   STATUS    RESTARTS   AGE
kubernetes-demo-4zkxm   1/1     Running   0          37s
kubernetes-demo-cp8jt   1/1     Running   0          37s
kubernetes-demo-pt9px   1/1     Running   0          37s
```

可以看到為了不要讓名稱重複，所以 Replication Controller 會在每一個 Pod 名稱後面加入亂數。

<br>

接下來我們用 `minikube dashboard` 來測試一下，是否刪除其中一個 Pod 後，Replication Controller 會自動建立新的：

<br>

{{< figure src="/kubernetes/k8s-advanced/rc-1.webp" width="1200" caption="刪除隨機一個 Pod" >}}

<br>

當我們隨機刪除一個 Pod 時，被刪除的 Pod 會 Terminating 準備刪除，且啟動一個新的 Pod ContainerCreating：

{{< figure src="/kubernetes/k8s-advanced/rc-2.webp" width="1200" caption="Pod 服務" >}}

<br>

當新的 Pod 啟動成功後，舊的 Pod 才會被刪除，所以可以確保我們的服務穩定度。

{{< figure src="/kubernetes/k8s-advanced/rc-3.webp" width="1200" caption="Pod 服務" >}}

<br>

綜上所述，可以知道 Replication Controller 真的會控制 Pod 數量，那我們刪掉一個 Pod 他就重生一個，這樣不會永遠都刪不完嗎？其實我們可以把 Replication Controller 砍掉就好了，而 Replication Controller 刪除時，也會自動終止底下的 Pod ，最後 Pod 都會自動刪除。

但其實 Kubernetes 官方不建議使用 Replication Controller 的方式來控制 Pod，而是建議使用 Deployment 搭配 ReplicaSet 來控制，我們接下來要介紹的主題就是：`Deployment` 跟 `ReplicaSet` 。

<br>

## 什麼是 ReplicaSet ?

看到 `Replica` 大家應該就知道這個跟控制 Pod 的數量有關係了！其實 `Replica` 跟 `Replication Controller` 很像，`ReplcaSet` 提供了更彈性的 selector，在 `Replication Controller` 中我們必須要把完整的 `key/value` Label 寫上去，在 `ReplicaSet` 不用這個麻煩，但 `ReplicaSet` 一樣也可以寫上完整的 Label ，這個就看開發者要怎麼去設計了！

<br>

### ReplcaSet 的特性

上面有提到 `ReplcaSet` 非常彈性的 selector，這邊要說的是 `ReplcaSet` 是如何讓 selector 變得更加彈性，這裡一共要介紹兩種 `ReplcaSet` 的 selector 寫法：

<br>

- matchLabels

`matchLabels` 其實跟一般的 selector 做的事情一模一樣，也要寫出完整的 Label 出來，整體大概會長像這樣：

```yaml
selector:
  matchLabels:
    app: demo
```

<br>

- matchExpressions

`matchExpressions` 則是提供更彈性的選取條件，每一筆條件主要由 key、operator、value 組成，並且使用一個 `{ }` 包起來，看起來很像 JS 的 物件型態，整體大概會長像這樣：

```yaml
selector:
  matchExpression:
    - { key: app, operator: In, values: [demo, test] }
```

看起來似乎很複雜但其實很容易理解，上面的示範中 Expression 翻成中文就是只要 Label 的 key 是 `app` 且值符合 `[demo, test]` 陣列中的其中一個值的 Pod 就會被選取到。

<br>

我們在隨便取得例子：假設有一個 `matchExpression` 長得像這樣：

```yaml
selector:
  matchExpression:
    - { key: env, operator: NotIn, values: [hello, hi] }
```

我們可以得知要選取 Label 的 key 為 `env` 且值不能是 `[hello, hi]` 其中一個值的 Pod。

<br>

## 什麼是 Deployment ?

Deployment 是一種負責管理 `ReplicaSet` 以及控制 Pod 更新的物件，在先前的文章都沒有提到 Pod 的更新，是因為 Pod 無法直接做更新，必須砍掉重建才會是新的內容，有了 Deployment 之後我們就可以很方便的進行 Pod 的更新了！

由於 `ReplicaSet` 本身也會控制 Pod，所以整個看起來會像是 Deployment 控制 Pod，但其實真正控制 Pod 的是 `ReplicaSet` ~

<br>

{{< figure src="/kubernetes/k8s-advanced/deployment.webp" width="500" caption="Deployment 與 ReplicaSet 架構圖" >}}

<br>

### Deployment 的特性

- 部署一個應用服務

上面我們提到 Deployment 可以幫助 Pod 進行更新，通常在開發一個產品的時候一定會不斷的更新，透過 Deployment 我們可以快速的更新 Pod 內部的 container，所以通常在部署應用的時候都會使用 Deployment 來進行部署。

<br>

- 在服務升級過程中可以達成無停機服務遷移 (Zero downtime deployment)

在 Deployment 幫 Pod 內部 container 進行更新的過程有一個專有名稱叫做 `RollingUpdate` ，`RollingUpdate` 翻成中文的意思是滾動更新，在更新的過程中 Deployment 會先產生一個新的 ReplicaSet 而這個 ReplicaSet 內部的 Pod 會運行新的內容，待新的 Pod 被 Kubernetes 確認可以正常運行後 Deployment 才會將舊的 ReplicaSet 進行取代的動作，這樣就完成了無停機服務遷移了。

<br>

- 可以 Rollback 到先前版本

每一次的 Deployment 在進行 RollingUpdate 的時候都會把當前的 ReplicaSet 做一個版本控制的紀錄，就像是 git commit 一樣，所以我們也可以利用這些紀錄來快速恢復成以前的版本，這些 Pod 也就會變成先前的內容。

<br>

講完基本的介紹後，接下來要介紹的是要如何撰寫以及建立一個 Deployment：

### Deployment 搭配 ReplicaSet 寫法

[Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/Deployment-ReplicaSet)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 60
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: kubernetes-demo-container
          image: 880831ian/kubernetes-demo
          ports:
            - containerPort: 3000
  selector:
    matchLabels:
      app: demo
```

<br>

看到 Deployment 是不是覺得跟 Replication Controller 非常相似呢？其實 Deployment 就多了再 RollingUpdate 時的設定以及 ReplicaSet 的設定而已，下面來說明一下這些設定：

首先一開始的 apiVersion 的值已經不是 `v1` 了，改成 `apps/v1`，由於 Kubernetes 針對 Deployment、RollingUpdate、ReplicaSet 等等設定了新的 `apiVersion` 值，通常都用 `apps/v1` 都是用來設定跟應用程式有關的架設，所以 Deployment 這邊要記得改成 `apps/v1` 歐！

<br>

在 `spec` 的地方有看到 `strategy` 新的設定值，這個主要用來設定 Deployment 更新的策略，這裡的 `strategy.type` 有兩種設定：

- RollingUpdate

此為預設值，先建立新的 ReplicaSet 並控制新內容的 Pod，待新 Pod 也可以正常運作後，才會通知 ReplicaSet 將原有的 Pod 給移除，由於過程中會有新舊兩種 Pod 同時上線，因此會有一段時間是新舊內容會隨機出現的情形發生。

這邊可以看到除了 `type` 以外還寫了 `maxSurage` 以及 `maxUnavailablle`，這兩個設定值為 RollingUpdate 過程的設定，接下來一樣說明一下兩個設定的功能：

`macSurge`：代表 ReplicaSet 可以產生比 Deployment 設定中的 `replica` 所設定的數量還多幾個出來，多新增 Pod 的好處是在 RollingUpdate 過程中可以減少舊內容顯示在頁面的機率。

`macUnavailable`：代表在 RollingUpdate 過程中，可以允許多少的 Pod 無法使用，假設 `macSurge` 設定非 0，`maxUnavailable` 也要設定非 0。

<br>

- Recreate

先通知當前 ReplicaSet 把舊的 Pod 砍掉再產生新的 ReplicaSet 並控制新內容的 Pod，由於先砍掉 Pod 才建立新的 Pod ，所以中間有一段時間伺服器會無法連線。

也因為 Recreate 會砍掉重建，因此 Recreate 無法像 RollingUpdate 設定 `maxSurge` 以及 `macUnavailable`。

<br>

講完 Deployment 的更新流程設定後，接下來要講 Deployment 完成更新後的設定，這邊有兩種設定：

- minReadySeconds

`minReadySeconds` 代表當新的 Pod 建立好並且運行的 container 沒有 crash 的情況下，最少需要多少時間可以開始接受 Request，預設為 0 秒，代表當 Pod 一被建立起來，就可以馬上開始接受 Request，假設今天 container 在剛運行的時後需要花時間做初始化，這時候就可以利用 `minReadySeconds` 讓此 Pod 不會馬上接受到 Request ，這個是選填的設定。

<br>

- revisionHistoryLimit

每次 Deployment 在進行更新的時候，都會產生一個新的 ReplicaSet 用來進行版本控制，在 Deployment 中這個專有名稱為 `revision`，所以這個設定就是要設定最多只會有多少個 `revision`，這個也是選填的設定。

<br>

### Deployment 建立

老樣子，使用 `apply` 來建立 Deployment，我們可使用 `kubectl get deploy` 來查看是否建立成功：

```sh
$ kubectl get deploy

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-deployment   3/3     3            3           8m31s
```

<br>

由於 Deployment 直接管理 ReplcaSet，因此我們可以查看 ReplcaSet 是否也有被建立起來：

```sh
$ kubectl get rs

NAME                              DESIRED   CURRENT   READY   AGE
kubernetes-deployment-dc5c59fdb   3         3         3       10m
```

可以看到 ReplcaSet 後面會自動加上一小段亂數，這邊是 Deployment 在建立 ReplicaSet 的時候加進去的，這樣之後可以更方便的利用 ReplcaSet 進行 rollback 的動作。

<br>

由於 ReplicaSet 會直接管理 Pod，因此我們也可以查看 Pod 是否有被建立起來：

```sh
$ kubectl get po

NAME                                    READY   STATUS    RESTARTS   AGE
kubernetes-deployment-dc5c59fdb-cjbz6   1/1     Running   0          12m
kubernetes-deployment-dc5c59fdb-mjvd7   1/1     Running   0          12m
kubernetes-deployment-dc5c59fdb-r92zt  1/1     Running   0          11m
```

<br>

那我們用 `minikube dashboard` 來查看一下：

<br>

{{< figure src="/kubernetes/k8s-advanced/DeploymentReplicaSet.webp" width="1200" caption="minikube dashboard 查看 Deployment 與 ReplicaSet" >}}

<br>

Deployment 的最後要來說的是要如何更新底下的 Pod 呢，大家就接著往下看囉！

### 如何更新 Deployment 內部的 Pod

大家都知道 Pod 是 Kubernetes 最小的運行單位，所以更新 Pod 的意思就是把內部運行的 container 進行更新，也就是說我們只要更新 Pod 的 image 就可以順利的讓 Pod 運行最新的內容，Deployment 就是運用這個原理才進行 Pod 的內容更新，方法也很簡單只要利用 `Set` 這個參數就可以了，可以使用 `kubectl set -h` 可以看 `set` 這個參數真正的用法。

<br>

{{< figure src="/kubernetes/k8s-advanced/set.webp" width="800" caption="kubectl set" >}}

<br>

可以看到 `set` 後面還要接 `SUBCOMMAND`，而 `SUBCOMMAND` 就是 `Available Commands` 的內容，由於我們要更新的是 image 所以這邊的 `SUBCOMMAND` 會是 `image`，完整指令是：

```sh
$ kubectl set image deployment/kubernetes-deployment kubernetes-demo-container=880831ian/kubernetes-demo:v1 --record

deployment.apps/kubernetes-deployment image updated
```

<br>

使用 set 來更新 Pod，格式是 `kubectl set image deployment/<deployment name> <pod name>=<要更新的 image>`，後面加上 `--record` 參數，這樣會紀錄每次更新的時候到底更新哪些內容，這樣日後要進行 rollback 也會比較容易知道要 rollback 回哪個 revision，由於要顯示差異，所以在 [dockerhub](https://hub.docker.com/repository/docker/880831ian/kubernetes-demo) 上又多推一個版本 `v1`，將原本柴犬的圖片改成 kubernetes logo，等於我們更新是從 `kubernetes-demo:latest` 更新至 `kubernetes-demo:v1`，我們來看看它的變化吧！

<br>

這是還沒更新 Pod 前的 Deployment 與 ReplicaSet：

{{< figure src="/kubernetes/k8s-advanced/DeploymentReplicaSet.webp" width="1200" caption="Deployment 與 ReplicaSet" >}}

<br>

我們用 set 下完更新指令後，可以查看 ReplicaSet 以及 Pod 在更新過程中的變化：

{{< figure src="/kubernetes/k8s-advanced/set-1.webp" width="1200" caption="ReplicaSet 以及 Pod 在更新過程中的變化" >}}

<br>

可以發現 strategy 為 RollingUpdate 的時候並不會把舊有的 Pod 移除反而會讓新舊 Pod 同時上線，以達到無停機服務的作用，但這樣在網頁中就有可能會同時出現新舊內容。

<br>

{{< figure src="/kubernetes/k8s-advanced/set-2.webp" width="1200" caption="因為 strategy 為 RollingUpdate，所以會同時出現新舊內容" >}}

<br>

最後等到新的 Pod 已經建立完成，且正常運作，ReplicaSet 就會把舊的 Pod 給移掉：

<br>

{{< figure src="/kubernetes/k8s-advanced/set-3.webp" width="1200" caption="新的 Pod 建立成功，並刪除舊 Pod" >}}

<br>

再次瀏覽就可以發現已經變成新的內容了！代表我們完成更新動作～

<br>

{{< figure src="/kubernetes/k8s-advanced/set-4.webp" width="600" caption="更新至 kubernetes-demo:v1" >}}

<br>

### Deployment 回朔版本

我們講完更新後，接著要講如何 rollback 回以前的版本，首先我們必須使用 `rollout` 這個參數，一樣使用 `kubectl rollout -h` 可以查看 rollout 的用法，跟 `set` 十分相似。

 <br>

{{< figure src="/kubernetes/k8s-advanced/rollout.webp" width="800" caption="更新至 kubernetes-demo:v1" >}}

<br>

由於我們這邊要 rollback ，所以 `SUBCOMMAND` 會使用 `undo`，我們上面有用 `--record` 可以查看要 rollback 的版本，所以我們這邊寫法會長像這樣：

```sh
$ kubectl rollout undo deployment/kubernetes-deployment --to-revision=1

deployment.apps/kubernetes-deployment rolled back
```

格式是 `kubectl rollout undo deployment/<deployment name> --to-revision=<--record 版本>`

<br>

就可以看到又回復成以前的柴犬囉 ><

<br>

{{< figure src="/kubernetes/k8s-advanced/Shiba-Inu-3.webp" width="1000" caption="成功還原柴犬" >}}

<br>

### 為甚麼不用 Replication Controller

最後要討論的是為什麼要不用 Replication Controller 而是改用 ReplicaSet + Deployment？

由於實際再使用 Kubernetes 時架構會比現在練習的還要複雜，所以用 ReplicaSet 讓 selector 用更彈性的方式選取 Pod 會是比較好的做法。

<br>

## 什麼是 Stateless 和 Stateful

### Stateless

Stateless 顧名思義就是無狀態，我們可以想成我們每次與伺服器要資料的過程中都不會被伺服器記錄狀態，**每一次的 Request 都是獨立的**，彼此是沒有關聯性的，也就是我們當下獲得的資料只能當下使用沒有辦法保存，靜態網頁通常都是一種 Stateless 的應用。

舉個例子來說：今天我想要查詢火車時刻表，我可以藉由 Google 搜尋火車時刻表，並點選連結，或是直接在瀏覽器輸入 `https://tip.railway.gov.tw/` ，這兩種結果最後都會一致，並不會因為我的操作不同而產生不同結果，這就是一種 Stateless 的表現。

<br>

### Stateful

Stateful 就是 Stateless 的相反，也就是每次的 Request 都會被記錄下來，日後都可以進行存取，Stateful 最常見的例子是資料庫，所以我們可以理解成 Stateful 背後一定會有一個負責更新內容的儲存空間。

幾個例子來說：今天我們想要查看 Google 雲端硬碟的內容，我們必須先登入自己的帳號才可以查看內容，這種有操作先後順序才會有結果的就是一種 Stateful 的表現。

<br>

{{< figure src="/kubernetes/k8s-advanced/stateless_ful.webp" width="700" caption="Stateless vs Stateful" >}}

<br>

那為什麼要提到 Stateless 跟 Stateful 呢？

因為跟 Pod 有很大的關係，在 Kubernetes 中 Pod 就屬於 Stateless 的，我們前面有提到 Stateless 的特性就是每次的 Request 都是獨立的，這樣有一個好處是可以快速的擴充。

在 [Kubernetes - 基本篇中的 Pod](../k8s/#pod) 有提到：Pod 是 Kubernetes 中最小的單位，由於 Pod 是屬於 Stateless 的，即便今天同一種內容的 Pod 有很多個也沒有關係，因為每次的 Request 都是獨立的，多個 Pod 就多個連線的端點而已。

<br>

### Kubernetes 的 Stateful

上面有說到 Kubernetes 的 Pod 是 Stateless 的，那難道 Kubernetes 沒有辦法做 Stateful 應用嗎？其實是可以的，Kubernetes 為了 Stateful 有特別開啟一個類別叫：`StatefulSet`

<br>

這邊會簡單說明一下 `StatefulSet` 的架構：

### StatefulSet

StatefulSet 一共有兩個重要的部分：

- Persistent Volume Claim

前面有說到 Stateful 背後有一個更新內容的儲存空間，在 Kubernetes 中負責管理儲存的空間是 `Volume`，作用與 Docker 的 Volume 幾乎一模一樣，但 Kuberntes 的 Volume 只是在 Pod 中暫時存放的儲存空間，當 Pod 移除之後這個儲存空間就會消失，為了要在 Kubernetes 中建立一個像是資料庫可以永久儲存的空間，這個 Volume 不能被包含在 Pod 中，而這個就是 `Persistent Volume (PV)`。

Persistent Volume Claim (PVC) 就是負責連接 Persistent Volume (PV) 的物件，所以可以想像一下今天有多少的 Persistent Volume 就會有多少的 Persistent Volume Claim。

<br>

- Headless Service

還記得在 [Service](#什麼是-service-) 有提到 ClusterIP 嗎？其實每個 Service 都會有自己一組的 ClusterIP (ExternalName 形式的除外)，所以 Headless 的意思其實就是不要有 ClusterIP，方法也很簡單，直接在設定檔中加入 `ClusterIP: None` 就可以了！

這麼做有什麼好處？由於 Headless Service 並沒有直接跟 Pod 有對應關係，因此 Service 本身沒有 ClusterIP，所以 Kubernetes 內部在溝通時就沒有辦法把我們設定好的 Service 名稱進行 IP 轉換，不過 Headless Service 會將內部的 Pod 的都建立屬於自己的 domain，所以我們可以自由的選擇要連接到哪一個 Pod。

這時候你會說可以用手動來連接呀？但因為 Service 一般是跟著 Pod 的 Label ，所以一個 Service 都會連接許多個 Pod，這樣我們就沒有辦法針對某個 Pod 來做事情，所以 Headless Service 在 Stateful 中也會被建立。

<br>

我們一直強調說 Kubernetes 最小的單位是 Pod，即便是 StatefulSet 也會有 Pod，只是這個 Pod 會歸 StatefulSet 管理，綜合上面所述可以知道一個 StatefulSet 裡面除了執行的 Pod 外還會有負責跟 Persistent Volume 連接的 Persistent Volume Claim，整體的 StatefulSet 架構會長得像這樣：

<br>

{{< figure src="/kubernetes/k8s-advanced/StatefulSet.webp" width="300" caption="Kubernetes StatefulSet 架構" >}}

<br>

基本上 StatefulSet 中在 Pod 的管理上都是與 Deployment 相同，基於相同的 container spec 來進行; 而其中的差別在於 **StatefulSet controller 會為每一個 Pod 產生一個固定識別資訊，不會因為 Pod 改變後而有所變動**。

<br>

### 什麼時候需要使用 StatefulSet ?

如何研判哪些 Application 需要使用 StatefulSet 來部署？只要符合以下條件，就需要使用 StatefulSet 來進行部署

- 需要穩定 & 唯一的網路識別 (pod 改變後的 pod name & hostname 都不會變動)
- 需要穩定的 Persistent storage (pod 改變後還是能存取相同的資料，基本上用 PVC 就可以解決)
- 部署 & 擴展的時候，每個 Pod 的產生都是有其順序且逐一慢慢完成的
- 進行更新操作時，也是與上面的需求相同

<br>

### StatefulSet 有什麼限制？

- v1.5 以前版本不支援，v1.5 ~ v1.9 之間是 beta，v1.9 後正式支援
- storage 的部分一定要綁定 PVC，並綁定到特定的 StorageClass or 預先配置好的 Persistent Volume，確保 Pod 被刪除後資料依然存在。
- 需要額外定義一個 Headless Service 與 StatefulSet 搭配，確保 Pod 有固定的 network identity。

{{< callout >}}
network identity：

代表可以直接透過 domain name 直接取的 Pod IP; 實現的方法則是部署一個 ClusterIP=None 的 Service，讓 Cluster 內部存取 Service 時，可以直接連到 Pod 而不是 Service IP。
{{< /callout >}}

<br>

### StatefulSet 寫法

要寫一個 StatefulSet ，有幾個重要的部分必須涵蓋：

1. Application & Persistent Volume Claim
2. Headless Service
3. `.spec.selector` 所定義的內容 (matchLabels) 必須與 `.spec.template.metadata.labels` 相同。

其他部分都與 `Deployment` 幾乎相同 ~

<br>

我們先來看看要怎麼定義 Application & Persistent Volume Claim (把 Service 跟 StatefulSet 寫在一起) ([Github 程式碼連結](https://github.com/880831ian/kubernetes-tutorial/tree/master/StatefulSet))

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

<br>

我們先打開一個 Terminal 來觀察 StatefulSet 創建 Pod 的過程：

```sh
$ kubectl get pods -w -l app=nginx
```

<br>

我們一樣使用 `kubectl apply` 來創建定義在 `web.yaml` 中的 Headless Service 和 StatefulSet。

```sh
$ kubectl apply -f web.yaml
```

<br>

我們看一下剛剛的 StatefulSet 創建 Pod 的過程，可以發現我們擁有 N 的副本的 StatefulSet ， Pod 部署時會按照 {0 .... N-1} 的序號依序創建。

```sh
NAME    READY   STATUS    RESTARTS   AGE
web-0   0/1     Pending   0          0s
web-0   0/1     Pending   0          0s
web-0   0/1     ContainerCreating   0          0s
web-0   1/1     Running             0          2s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          0s
web-1   0/1     ContainerCreating   0          0s
web-1   1/1     Running             0          3s
```

可以發現 web-1 Pod 是在 web-0 Pod 處在 Running 狀態才會被啟動。此外，可以發現就跟我們上面講的一樣，StatefulSet 中的 Pod 擁有一個獨一無二的身份標記，基於 StatefulSet 控制器分配給每個 Pod 的唯一順序索引。

Pod 名稱的格式是 < statefulset name >-< ordinal index >，像我們 `web` 這個 StatefulSet 有兩個副本，所以它創建了兩個 Pod：`web-0`、`web-1`。

<br>

### StatefulSet 測試

我們知道 StatefulSet 它有使用穩定的網路身份以及 PV 的永久儲存，那我們就分別來測試看看：

#### StatefulSet 穩定的網路身份

我們先使用 `kubectl exec` 在每個 Pod 中執行 `hostname`

```shell
$ for i in 0 1; do kubectl exec "web-$i" -- sh -c 'hostname'; done

web-0
web-1
```

<br>

再使用 `kubectl run` 來運行一個提供 `nslookup` 命令的容器，通過對 Pod 的主機名執行 `nslookup`，我們可以檢查他在集群內部的 DNS 位置。

```sh
$ kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm /bin/sh
```

<br>

啟動一個新的 shell ，並運行 `nslookup web-0.nginx ` 跟 `nslookup web-1.nginx`：

```sh
/ # nslookup web-0.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 172.17.0.5 web-0.nginx.default.svc.cluster.local

/ # nslookup web-1.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 172.17.0.6 web-1.nginx.default.svc.cluster.local
```

可以看到 Headless Service 的 CHANCE 指向 SRV 記錄 (記錄每個 Running 的 Pod)。SRV 紀錄指向一個包含 Pod IP 位址的記錄表。

<br>

我們使用 `kubectl delete pod -l app=nginx` 刪除 Pod 後，會發現 Pod 的序號、主機名、SRV 條目和記錄名稱都沒有改變！

<br>

#### StatefulSet 永久儲存

我們先查看 `web-0` 跟 `web-1` 的 PersistentVolumeClaims：

```sh
$ kubectl get pvc -l app=nginx

NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-e9a23104-d018-4d73-8cd9-89e5ea67f96c   1Gi        RWO            standard       39m
www-web-1   Bound    pvc-00dd6c87-6d95-4d04-9c1b-49b43441f4a1   1Gi        RWO            standard       38m
```

StatefulSet 控制器創建兩個 PersistentVolumeClaims，綁定兩個 PersistentVolumes，因為我們配置是動態提供 PersistentVolume，所有的 PersistentVolume 都是自動創建和綁定的。

Nginx web 服務器默認會加載位於 `/usr/share/nginx/html/index.html` 的 index 文件。因此我們在 `spec` 中的 `volumeMounts` 將 `/usr/share/nginx/html` 資料夾由一個 PersistentVolume 支持。

<br>

那我們將 Pod 主機名稱寫入 `index.html` ，再刪掉 Pod 看看寫入內容是否還會存在：

```sh
$ for i in 0 1; do kubectl exec "web-$i" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'; done
$ for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done

web-0
web-1
```

<br>

當我們刪除 Pod 後，如果沒有使用 PersistentVolumeClaims 去綁定 PersistentVolumes 的話，資料就會消失，那我們來看看有綁定的結果：

使用 `kubectl delete pod -l app=nginx` 刪除 Pod：

```sh
pod "web-0" deleted
pod "web-1" deleted
```

<br>

再次使用 `kubectl get pod -w -l app=nginx` 來檢查 Pod 的狀態：

```sh
web-0   0/1     ContainerCreating   0          0s
web-0   1/1     Running             0          2s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          0s
web-1   0/1     ContainerCreating   0          0s
web-1   1/1     Running             0          1s
```

<br>

一樣使用 `for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done` 來查看：

```sh
web-0
web-1
```

可以發現 `web-0`、`web-1` 雖然重新啟動，但依舊會監聽它們主機名，因為和它們的 PersistentVolumeClaim 相關聯的 PersistentVolume 被重新掛載到了各自的 `volumeMount` 上。

<br>

## 什麼是 ConfigMap ?

看到 Config 應該會聯想到與設定檔有關，沒錯 ConfigMap 通常都是用來存放設定檔用的，換句話來說這個物件會直接連結一個或多個檔案，而 ConfigMap 通常都是用來存放偏向部署面的設定檔，像是資料庫的初始化設定、nginx 設定檔等等，這種不用被包進去 image 內，但需要讓 container 可以使用的檔案。

<br>

### ConfigMap 特性

- 一個 ConfigMap 物件可以放一個或多個設定檔：我們上面有提到它是用來存放設定檔用的，會直接連接該設定檔。
- 無需修改程式碼，可以替換不同環境的設定檔：由於設定檔都交由 ConfigMap 管理，並不是包在 image 內，因此可以藉由修改 ConfigMap 的方式來達到不用更新 Pod 內容就可以更換設定檔的作用。
- 統一存放所有的設定檔：一個 ConfigMap 可以連結一個以上的設定檔，因此也可以將該專案會用到的所有設定檔通通存放在同一個 ConfigMap 物件中進行管理。

<br>

### 如何建立 ConfigMap？

由於 ConfigMap 可以直接存入設定檔，所以我們以現有設定檔為基準，接下來要用 `create` 這個參數來建立 ConfipMap 物件出來，這時可能會想，之前都是使用 `apply` 怎麼這次要改用 `create`呢！？ 雖然兩者都有建立的意思，但背後實作的技術完全不同：

- `create` 使用的是 [Imperative Management](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-config/)，Imperative Management 會告訴 Kubernetes 我目前的動作要做什麼，例如：`create`、`delete`、`replace` 某個物件。
- `apply` 是使用 [Declarative Management](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)，Declarative Management 是用宣告的方式來建立物件，更白話一點就是我希望這個物件要長怎麼樣，所以 `apply` 通常都會搭配 yaml 檔，而這份 yaml 檔就會在 `kind` 這個設定值告訴 Kubernetes 這個物件要長成什麼樣子。

<br>

因為我們這次要直接使用現有的設定檔來建立 ConfigMap，所以這時候不能使用 `apply` 的方式，只能使用 `create` 來建立，指令也很簡單：

```sh
$ kubectl create configmap <configmapName> --from-file=<filePath>
```

<br>

{{< figure src="/kubernetes/k8s-advanced/configmap.webp" width="1000" caption="建立 ConfigMap" >}}

<br>

建立完成使用 `get` 來查詢是否正確建立 ConfigMap：

<br>

{{< figure src="/kubernetes/k8s-advanced/configmap_1.webp" width="700" caption="使用 get 查詢是否正確 ConfigMap" >}}

<br>

最後可以下 `describe` 這個參數來查看 ConfigMap 的內容，會發現裡面就是我們設定檔的完整內容：

<br>

{{< figure src="/kubernetes/k8s-advanced/configmap_2.webp" width="800" caption="使用 describe 查看 ConfigMap 的內容" >}}

<br>

## 什麼是 Secrets ?

看到 Secrets 這個名字就知道這是非常機密的物件，相較於 ConfigMap 是用來存放部署面的檔案，Secrets 通常都是用來存機密的資料，像是使用者帳號、SSL 憑證等。

### Secrets 特性

上面 ConfigMap 提到的特性 Secrets 一樣，比較特別的是 Secrets 會將內部的資料進行 base64 編碼。因為重新編碼所以可以確保資料相較於 ConfigMap 下安全一些，所以建議如果是機密性的資料就存在 Secrets 裡面吧！

<br>

### 如何建立 Secrets？

我們一樣用現有的檔案來做基準作為示範，由於上面 ConfigMap 只存入一個檔案而已，所以這邊 Secrets 我們改成存入多個檔案：

一樣用 `create` 的參數進行 Secrets 建立，但這邊要多加一個 `SUBCOMMAND` 叫 `generic`，generic 代表意思是從本機檔案、目錄建立 Secrets，接下來只要下：

```sh
kubectl create secret generic <secretName> --from-file=<filePath>
```

<br>

{{< figure src="/kubernetes/k8s-advanced/secret.webp" width="800" caption="建立 Secrets" >}}

<br>

建立完成使用 `get` 來查詢是否正確建立 Secrets：

<br>

{{< figure src="/kubernetes/k8s-advanced/secret_1.webp" width="700" caption="使用 get 查詢是否正確 Secrets" >}}

<br>

最後一樣可以下 `describe` 這個參數來查看 Secrets 的內容，但因為加密所以不會顯示原本內容，只會看到的確有兩個檔案：

<br>

{{< figure src="/kubernetes/k8s-advanced/secret_2.webp" width="700" caption="使用 describe 查看 Secrets 的內容" >}}

<br>

## 參考資料

Kubernetes 那些事 — Service 篇：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-service-%E7%AF%87-d19d4c6e945f](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-service-%E7%AF%87-d19d4c6e945f)

Kubernetes 那些事 — Ingress 篇（一）：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-ingress-%E7%AF%87-%E4%B8%80-92944d4bf97d](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-ingress-%E7%AF%87-%E4%B8%80-92944d4bf97d)

Kubernetes 那些事 — Ingress 篇（二）：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-ingress-%E7%AF%87-%E4%BA%8C-559c7a41404b](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-ingress-%E7%AF%87-%E4%BA%8C-559c7a41404b)

Kubernetes 那些事 — Stateless 與 Stateful：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-stateless-%E8%88%87stateful-2c68cebdd635](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-stateless-%E8%88%87stateful-2c68cebdd635)

kubernetes ReplicationController：[https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)

Kubernetes 那些事 — Replication Controller：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-replication-controller-5c8592d37083](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-replication-controller-5c8592d37083)

Kubernetes 那些事 — Deployment 與 ReplicaSet（一）：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%B8%80-406234a63d43](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%B8%80-406234a63d43)

Kubernetes 那些事 — Deployment 與 ReplicaSet（二）：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%BA%8C-f60146c878e4](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%BA%8C-f60146c878e4)

Kubernetes 那些事 — Deployment 與 ReplicaSet（三）：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%B8%89-142b2863eb94](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-deployment-%E8%88%87-replicaset-%E4%B8%89-142b2863eb94)

kubernetes Deployments：[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

[Kubernetes] StatefulSet Overview：[https://godleon.github.io/blog/Kubernetes/k8s-StatefulSets-Overview/](https://godleon.github.io/blog/Kubernetes/k8s-StatefulSets-Overview/)

