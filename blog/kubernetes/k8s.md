# Kubernetes (K8s) 介紹 - 基本
## 什麼是 Kubernetes (K8s)?

Kubernetes 也可以叫 K8s，這個名稱來源希臘語，意思是舵手或是飛行員，所以我們可以看到它的 logo 是一個船舵的標誌，之所以叫 K8s 是因為 Kubernetes 的 k 到 s 中間有 8 的英文字母，為了方便，大家常以這個名稱來稱呼他！

<br>

{{< figure src="/kubernetes/k8s/logo.webp" width="200" caption="kubernetes logo" >}}

<br>

Kubernetes 是一種開源可用來自動化部屬、擴展以及管理多個容器的系統，適用於當容器數量增加，需要穩定容器環境，以及管理資源或權限分配的狀況。

我們之前在 [Docker 介紹](../../docker/docker) 文章中，已經有介紹以往傳統虛擬機以及容器化的 Docker 差異以及優點，那當我們在管理容器時，其中一個容器出現故障，則需要啟動另一個容器，如果要用手動，會十分麻煩，所以這時就是 Kubernetes 的厲害的地方了，Kubernetes 提供：

- 服務發現和負載平衡：K8s 可以使用 DNS 名稱或是自己的 IP 位址來公開容器。如果容器流量過高，Kubernetes 能夠使用負載平衡和分配網路流量，能使部署更穩定。
- 編排儲存：Kubernetes 允許使用自動掛載來選擇儲存系統，例如使用本地儲存，或是公共雲等。
- 自動部署、刪除：可以使用 Kubernetes 來幫我們自動化部屬新的容器、刪除現有的容器並將其資源用於新容器。
- 自動打包：當我們為 Kubernetes 提供一個節點叢集，它可以用來運行容器化的任務，告訴 Kubernetes 每個容器需要多少 CPU 和 RAM。Kubernetes 可以將容器安裝到節點上，充分利用資源。
- 自動修復：Kubernetes 會重新啟動失敗的容器、替換容器、刪除不回應用戶的不健康容器，並且在容器準備好服務之前不會通知客戶端。
- 機密和配置管理：Kubernetes 允許儲存和管理敏感訊息，例如密碼、OAuth token 和 SSH 金鑰。可以部署和更新機密的應用程序配置。

<br>

{{< figure src="/kubernetes/k8s/flower.webp" width="600" caption="[kubernetes 官網](https://kubernetes.io/)" >}}

<br>

Kubernetes 很常被拿來與 Docker Swarm 做比較，兩者不同的是，Docker Swarm 必須建構在 Docker 的架構下，功能侷限、無法跳脫。

Kubernetes 則因為功能較為廣泛，而逐漸取代 Docker Swarm 在市場上的地位。下方有簡易的比較表格：

| 比較 |                                                                               Kubernetes                                                                               |                                                                    Docker Swarm                                                                    |
| :--: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------: |
| 說明 | Kubernetes 是一個開源容器的編排平台，Kubernetes 的叢集結構比 Docker Swarm 更為複雜。<br> Kubernetes 通常有建構器和工作節點，還可進一步分為 Pod、命名空間、配置映射等。 | Docker Swarm 是一個由 Docker 構建和維護的開源容器編排平台。<br>一個 Docker Swarm 叢集通常包含三個項目：Nodes、Services and tasks、Load balancers。 |
| 優點 |  它有龐大的開源社群，由 Google 支持。<br>它可以維持和管理大型架構和複雜的工作負載。<br>它是自動化，並支持自動化擴展的自我修護能力。<br>它有內建監控和廣泛的可用集成。  |              Docker Swarm 安裝簡單，它輕量化且容易學習使用。<br>Docker Swarm 與 Docker CLI 一起運作，因此不需要多運行或是安裝新的 CLI              |
| 缺點 |                                              它複雜的安裝過程以及較難學習<br>它需要安裝單獨的 CLI 工具並且學習每一項功能                                               |                它是輕量級且與 Docker API 相關聯，與 Kubernetes 相比，Docker Swarm 被限制很多功能，且自動化也沒有 Kubernetes 強大。                 |

<br>

## Kubernetes 元件介紹與說明

Kubernetes 是如何幫我們管理以及部署 Container ? 要了解 Kubernetes 如何運作，就要先了解它的元件以及架構：

那我們由小的往大的來做介紹：依序是 Pod、Worker Node、Master Node、Cluster

### Pod

Kubernetes 運作中最小的單位，一個 Pod 會對應到一個應用服務 (Application)，舉例來說一個 Pod 可能會對應到一個 NginxServer。

- 每個 Pod 都有一個定義文件，也就是屬於這個 Pod 的 `yaml` 檔。
- 一個 Pod 裡面可以有一個或多個 Container，但一般情況一個 Pod 最好只有一個 Container。
- 同一個 Pod 中的 Containers 共享相同的資源以及網路，彼此透過 local port number 溝通。

<br>

### Worker Node

Kubernetes 運作的最小硬體單位，一個 Worker Node (簡稱 Node) 對應到一台機器，可以是實體例如你的筆電、或是虛擬機例如 GCP 上的一台 Computer Engine。

每一個 Node 都有三個組件所組成：kubelet、kube-proxy、Container Runtime

#### kubelet

該 Node 的管理員，負責管理該 Node 上的所有 Pods 的狀態並負責與 Master Node 溝通。

<br>

#### kube-proxy

該 Node 的傳訊員，負責更新 Node 的 iptables，讓 Kubernetes 中不在該 Node 的其他物件可以得知該 Node 上的所有 Pods 的最新狀態。

<br>

#### Container Runtime

該 Node 真正負責容器執行的程式，K8s 預設是 `Docker`，但也支援其他 Runtime Engine，例如 Mirantis Container Runtime、CRI-O、containerd

{{< callout >}}
常見誤解：

很多人認為 Kubernetes 是 `docker container` 的管理工具，包含我一開始也是這樣認為，但其實 Kubernetes 是用來管理容器化 (`containerized applications`) **並不是專屬於 `docker`** 獨享，作為一個 `container orchestrator` 的角色，Kubernetes 希望**能夠管理所有容器化**的應用程式
{{< /callout >}}

<br>

### Master Node

Kubernetes 運作的指揮中心，可以簡化看成一個特殊化的 Node 來負責管理所有其他的 Node。

一個 Master Node (簡稱 Master) 中有四個組件組成：kube-apiserver、etcd、kube-scheduler、kube-controller-manager

#### kube-apiserver

- 管理整個 Kubernetes 所需 API 的接口 (Endpoint)，例如從 Command Line 下 kubectl 指令就會把指令送到這裡。
- 負責 Node 之間的構通橋樑，每個 Node 彼此不能直接溝通，必須要透過 apiserver 轉介。
- 負責 Kubernetes 中的請求的身份認證與授權。

<br>

#### etcd

- etcd 是兼具一制性和高可用性的分散式鍵值數據庫，可以保存 Kubernetes 所有 Cluster 的後台數據庫。

<br>

#### kube-scheduler

- 整個 Kubernetes 的 Pods 調度員，scheduler 會監視新建立但還沒有被指定要跑在哪個 Node 上的 Pod ，並根據每個 Node 上面資源規定，硬體限制等條件去協調出一個最適合放置 Node 來讓該 Pod 運行。

<br>

#### kube-controller-manager

- 負責管理並運行 Kubernetes controller 的組件，簡單來說 controller 就是 Kubernetes 裡一個負責監視 Cluster 狀態的 Process，例如：Node Controller、Replication Controller。
- 這些 Process 會在 Cluster 與預期狀態 (desire state) 不符時嘗試更新現有狀態 (current state)。例如：現在要多開一台機器以應付突然增加的流量，那我的預期狀態就會更新成 N+1，現有狀態是 N，這時相對應的 controller 就會想辦法多開一台機器。
- controller-manager 的監視與嘗試更新也都需要透過訪問 kube-apiserver 達成。

<br>

### Cluster

Cluster 也叫叢集，可以管理眾多機器的存在，在一般的系統架設中我們不會只有一台機器而已，通常都是多個機器一起運行同一種內容，在沒有 Kubernetes 的時候就必須要土法煉鋼的一台一台機器去更新，但有了 Kubernetes 我們就可以透過 Cluster 進行控管，只要更新 Master 旗下的機器，也會一併將更新的內容放上去，十分方便。在 Kubernetes 中多個 Worker Node 與 Master Node 的集合。

<br>

## 基本運作

{{< figure src="/kubernetes/k8s/k8s.webp" width="600" caption="kubernetes 組件 [Kubernetes 基礎教學（一）原理介紹](https://cwhu.medium.com/kubernetes-basic-concept-tutorial-e033e3504ec0)" >}}

接下來我們用 「Kuberntes 是如何建立一個 Pod ？」來複習一下整個 Kubernetes 的架構。

(上圖是一個簡易的 Kubernetes Cluster ，通常 Cluster 為了高穩定性都會有多個 Master 作為備援，但為了簡化我們只顯示一個。)

1. 當使用者要部署一個新的 Pod 到 Kubernetes Cluster 時，使用者要先透過 User Command (kubectl) 輸入建立 Pod 對應的指令 (後面會說明要如何實際的動手建立一個 Pod)。此時指令會經過一層確認使用者身份後，傳遞到 Master Node 中的 API Server，API Server 會把指令備份到 etcd。
2. controller-manager 會從 API Server 收到需要創建一個新的 Pod 的訊息，並檢查如果資源許可，就會建立一個新的 Pod。最後 Scheduler 在定期訪問 API Server 時，會詢問 controller-manager 是否有建置新的 Pod，如果發現新建立的 Pod 時，Scheduler 就會負責把 Pod 配送到最適合的 Node 上面。

雖然上面基本的運作看起來十分複雜，但其實我們在實際操作時，只是需入一行指令後，剩下的都是 Kubernetes 會自動幫我們完成後續的動作。

## 安裝 Kubernetes

在我們開始操作 Kubernetes 之前，需要先下載 Minikube、Hyperkit、Kubectl 套件：

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)

一個 Google 發佈的輕量級工具，讓開發者可以輕鬆體驗一個 Kubernetes Cluster。<font color='red'>(僅限開發測試環境)</font>

<br>

{{< figure src="/kubernetes/k8s/minikube.webp" width="900" caption="安裝 minikube" >}}

<br>

- (Mac 專用) [Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/)

Hyperkit 是 MacOS 系統細部設定的驅動程式。

<br>

{{< figure src="/kubernetes/k8s/hyperkit.webp" width="900" caption="安裝 hyperkit" >}}

<br>

- [Kubectl](https://kubernetes.io/docs/tasks/tools/)

Kubectl 是 Kubernetes 的 Command Line 工具，我們之後會透過 Kubectl 去操作我們的 Kubernetes Cluster。

<br>

{{< figure src="/kubernetes/k8s/kubectl.webp" width="800" caption="安裝 kubectl" >}}

<br>

## 如何建立一個 Pod

版本資訊

- Minikube：v1.25.2
- hyperkit：0.20200908
- Kubectl：Client Version：v1.22.5、Server Version：v1.23.3

下載完 Minikube 後，我們可以先透過 `Minikube` 來查詢全部的指令，由於我們前面有安裝 Hyperkit 這個驅動程式，啟動 Minikube 預設是使用 Docker，我們這邊要利用 Hyperkit 來啟動，所以使用 `Minikube  start --vm-driver=hyperkit` 來啟動 Minikube。

{{< callout >}}
Minikube 其他指令介紹：

顯示 minikube 狀態 `minikube status`

停止 minikube 運行 `minikube stop`

ssh 進入 minikube 中 `minikube ssh`

查詢 minikube 對外的 ip `minikube ip`

使用 minikube 所提供的瀏覽器 GUI `minikube dashboard ` (可以加 - - url 看網址歐)

{{< /callout >}}

我們啟動 minikube 後，我們要打做一個可以在 Pod 運行的小程式。這個小程式是一個 Node.js 的 Web 程式，他會建立一個 Server 來監聽 3000 Port，收到 request 進來後會渲染 `index.html` 這個檔案，這個檔案裡面會有一隻可愛的小柴犬。

因為本文章是在介紹 kubernetes 所以在程式部分就不多做說明，我把程式碼放在 [Github](https://github.com/880831ian/kubernetes-demo)，以及附上 [Dockerhub 的 Repository](https://hub.docker.com/repository/docker/880831ian/kubernetes-demo) 可以直接使用包好的 image 來做測試！

<br>

### Pod yaml 檔案說明

接下來我們要先撰寫一個 Pod 的定義文件 (.yaml) 檔，這個 `.yaml` 檔就可以建立出 Pod 了！

- kubernetes-demo.yaml (程式縮排要正確，不然會無法執行歐！)

```yml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-demo-pod
  labels:
    app: demo
spec:
  containers:
    - name: kubernetes-demo-container
      image: 880831ian/kubernetes-demo
      ports:
        - containerPort: 3000
```

apiVersion

該元件的版本號，必須依照 Server 上 K8s 版本來做設定 (想要知道 k8s 版本，可以使用 `kubectl version` 指令來查詢，會顯示 client 跟 server 的版本訊息，client 代表 kubectl 版本訊息，server 代表的是 master node 的 k8s 版本訊息)，目前 k8s 都使用 1.23 版本以上，所以 `apiVersion` 直接寫 `v1` 即可。

<br>

kind

該元件的屬性，用來決定此設定檔的類型，常見的有 `Pod`、`Node`、`Service`、`Namespace`、`ReplicationController` 等等

<br>

metadata

用來擺放描述性資料的地方，像是 Pod 名稱或是標籤等等都會放在此處。

- name：指定該 Pod 的名稱
- labels：指定該 Pod 的標籤

<br>

spec

用來描述物件生成的細節，像是 Pod 內其實是跑 Dokcer container，所以在 Pod 的 spec 內就會描述 container 的細節。

- container.name：指定運行的 Container 的名稱
- container.image：指定 Container 要使用哪個 Image，這裡會從 DockerHub 上搜尋
- container.ports：指定該 Container 有哪些 Port number 是允許外部存取的

<br>

### 使用 kubectl 建立 Pod

當我們有了定義文件後，我們就可以使用 kubectl 的指令來建立 Pod

```sh
kubectl create -f kubernetes-demo.yaml

kubectl apply -f kubernetes-demo.yaml
```

<br>

可以使用 `create` or `apply` 來建立 Pod ，那這兩個的差異是什麼呢？

- kubectl create

1. `kubectl create` 是所謂的 "命令式管理" (`Imperative Management`)。通過這種方式，可以告訴 Kubernets API 你要建立、更新、刪除的內容。

2. `kubectl create` 命令是先刪除所有現有的東西，重新根據 YAML 文件生成新的 Pod。所以要求 YAML 文件中的配置必須完整。

3. `kubectl create` 命令，使用同一個 YAML 文件重複建立會失敗。

<br>

- kubectl apply

1. `kubectl apply` 是 "聲明式管理" (`Declarative Management`)方法的一部分。在該方法中，即使對目標用了其他更新，也可以保持你對目標應用的更新。

2. `kubectl apply` 命令，根據配置文件裡面出來的內容，生成就有的。所以 YAML 文件的內容可以只寫需要升級的欄位。

<br>

如果看到 `pod/kubernetes-demo-pod created` 就代表我們成功建立第一個 Pod 了，接下來我們可以使用：

```sh
kubectl get pods
```

可以查看我們運行中的 Pod：

```sh
NAME                  READY   STATUS    RESTARTS   AGE
kubernetes-demo-pod   1/1     Running   0          3m5s
```

<br>

{{< callout >}}
Pod 指令介紹：

查詢現有 Pod 狀態 `kubectl get po/pod/pods`

查看該 Pod 詳細資訊 `kubectl describe pods <pod-name>`

刪除 Pod `kubectl delete pods`

查看 Pod log `kubectl logs <pod-name>`

{{< /callout >}}

<br>

也可以使用 minikube 圖形化頁面來看一下是否成功！

```sh
minikube dashboard --url

🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:55991/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

<br>

{{< figure src="/kubernetes/k8s/minikube-k8s.webp" width="1000" caption="minikube dashboard" >}}

<br>

### 連線到 Pod 的服務

當我們建立好 Pod 之後，打開瀏覽器 `localhost:3000` 會發現，什麼都沒有，是因為我們剛剛在 `.yaml` 裡面設定的是 Pod 的 Port ，它與本機的 Port 是不相通的，因此我們需要使用 `kubectl port-forward <pod> <external-port>:<pod-port>` ，來將 Pod 與本機端做 mapping。

```sh
kubectl port-forward kubernetes-demo-pod 3000:3000

Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
Handling connection for 3000
```

我們在此瀏覽 `localhost:3000`，就會看到可愛的柴犬囉！

<br>

{{< figure src="/kubernetes/k8s/Shiba-Inu.webp" width="900" caption="成功顯示柴犬" >}}

<br>

前面我們已經創建屬於我們第一個 Pod 了，但當我們 Pod 越建越多時，要怎麼快速的得知每個 Pod 在做什麼事情？除了用 Pod 的 metadata name 來命名外，還有另一種方式：

## 什麼是 Label ?

Label 顧名思義就是標籤，可以為每一個 Pod 貼上標籤，讓 Kubernetes 更方便的管控這些 Pod。

Label 的寫法很簡單，可以自己自訂一對具有辨識度的 key/value，舉我們上面的例子來說：我們可以在 labels 內加入 `app: demo`，那 Label 有什麼好處呢?

這邊要稍微提一下 `Selector`，它的功用是選取對應的物件。為了要方便選取到我們設定好的 Pod，這時候 Label 就派上用場了！

`Selector` 的寫法也很簡單，只要把我們在 Label 定義的 key/value 直接完整的貼過來就可以了～

<br>

就像這樣：

```yaml
selectors:
  app: demo
```

那選取後有什麼功用呢！請看 [Kubernetes - 進階 - Service](../../kubernetes/k8s-advanced/#%E4%BB%80%E9%BA%BC%E6%98%AF-service-)

<br>

講完 Label 後，順邊提一下跟 Label 有相似的：

## 什麼是 Annotation ?

前面提到的 Label 功用其目的是要讓 Kubernetes 知道可以去更方便管理的，那我們如果想要貼標籤但不想讓 Kubernetes 知道，要怎麼做呢？

這時我們就可以用 `Annotation`，透過 `Annotation` 可以將標籤單純給開發人員查看，那聽起來 `Annotation` 好像沒有什麼實質上的用途，因為 Kubernetes 不會採用這些標籤，但其實 `Annotation` 還是有用的歐！後續文章會再提到 ><

<br>

那既然 Label 跟 Annotation 有相似，所以寫法想必也是差不多吧：

```yaml
annotations:
  author: Pin-YI
  contact: 880831ian@gmail.com
```

一樣也是定義一組具有辨識度的 key/value ，我們這邊就先放 author、contact

<br>

那 Label 與 Annotation 要放在 Pod 的哪一處呢？

還記得我們上面說 `metadata` 是用來擺描述性資料的地方嗎，所以不管是 Label 或是 Annotation 都是放在 `metadata` 中歐！

```yaml
metadata:
  name: kubernetes-demo-pod
  labels:
    app: demo
  annotations:
    author: Pin-YI
    contact: 880831ian@gmail.com
```

<br>

## 參考資料

kubernetes 官網：[https://kubernetes.io/](https://kubernetes.io/)

Kubernetes（K8s）是什麼？基礎介紹+3 大優點解析：[https://www.sysage.com.tw/news/technology/293](https://www.sysage.com.tw/news/technology/293)

Docker Swarm vs Kubernetes: how to choose a container orchestration tool：[https://circleci.com/blog/docker-swarm-vs-kubernetes/](https://circleci.com/blog/docker-swarm-vs-kubernetes/)

Kubernetes 基礎教學（一）原理介紹：[https://cwhu.medium.com/kubernetes-basic-concept-tutorial-e033e3504ec0](https://cwhu.medium.com/kubernetes-basic-concept-tutorial-e033e3504ec0)

Kubernetes 那些事 — 基礎觀念與操作：[https://medium.com/andy-blog/kubernetes%E9%82%A3%E4%BA%9B%E4%BA%8B-%E5%9F%BA%E7%A4%8E%E8%A7%80%E5%BF%B5%E8%88%87%E6%93%8D%E4%BD%9C-97cc203a2660](https://medium.com/andy-blog/kubernetes%E9%82%A3%E4%BA%9B%E4%BA%8B-%E5%9F%BA%E7%A4%8E%E8%A7%80%E5%BF%B5%E8%88%87%E6%93%8D%E4%BD%9C-97cc203a2660)

Kubernetes 那些事 — Pod 篇：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-pod-%E7%AF%87-57475cec22f3](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-pod-%E7%AF%87-57475cec22f3)

Kubernetes 那些事 — Label 篇：[https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-label-%E7%AF%87-4186af2af556](https://medium.com/andy-blog/kubernetes-%E9%82%A3%E4%BA%9B%E4%BA%8B-label-%E7%AF%87-4186af2af556)

