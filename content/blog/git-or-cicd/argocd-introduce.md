---
title: "為 Kubernetes 而生的 GitOps 工具 - ArgoCD 介紹與說明"
type: docs
weight: 9991
description: 為 Kubernetes 而生的 GitOps 工具 - ArgoCD 介紹與說明
images:
  - gcp/gke-cloud-dns-nodelocaldnscache/og.webp
date: 2025-11-15
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Git
  - GitOps
  - ArgoCD
  - CICD
  - K8s
  - Kubernetes
---

最近在準備面試，有些公司有使用 ArgoCD，由於前公司在 CICD 工具只使用 GitLab CI，所以比較沒有接觸這塊，只知道他是 GitOps 導向的工具，剛好趁這段時間來了解跟測試看看 ArgoCD，那我們開始囉～

<br>

## GitOps 定義

首先我們先來了解 GitOps 的定義

1. 開發團隊將程式碼管理、CI/CD、自動化流程延伸至基礎設施與部署操作，形成統一的操作框架
2. 將對系統的期望用「宣告式」方式來編寫成代碼，存放在 Git 儲存庫
3. 在流程內，Git 儲存庫會是單一的真實來源，所有變更都透過 Git 提供、審查、合併，最後再透過自動化同步至運行環境

<br>

那為什麼要採用 GitOps ?

1. 提高可追蹤性：因此每次的變更都會在 Git 留存紀錄，可以審核也可以 Rollback
2. 減少手動操作導致的偏差：在一開始學習雲端時，一定會使用 UI 去點擊，但當功能越多，經過不同的人，有可能大家點出來的設定會不同，也減少人工在維運時的臨時變更，通常 GitOps 工具都會有支援 **持續對齊** & **自我修正** 的功能
3. 加速部署流程：部署流程自動化後，可以更快、更可靠的將異動內容推向到測試、正式環境
4. 一致性跨環境：由於程式都透過 Git 版控，在不同環境下都可以維持一致

<br>

## 那 ArgoCD 是什麼？

我們從[官網的標題跟說明](https://argo-cd.readthedocs.io/en/stable/)就可以知道，他是設計給 Kubernetes 的宣告式 GitOps CD 工具

<br>

{{< figure src="/git-or-cicd/argocd-introduce/argocd-ui.webp" width="800" caption="ArgoCD 的概述介紹圖" >}}

<br>

### ArgoCD 的功能

1. 自動化部署到指定的目標環境
2. 支援多種組態管理/模板工具（Kustomize、Helm、Jsonnet、純 YAML）
3. 能夠管理和部署到多個集群
4. 支援 SSO 登入整合 (OIDC、OAuth2、GitHub、GitLab)
5. 用於多租戶基於 Role 的 RBAC 政策
6. 可以任意回滾到 Git repository 提交的程式設定
7. 資源健康狀況分析
8. 自動配置飄移檢測以及可視化介面
9. Webhook 整合（GitHub、BitBucket、GitLab）
10. PreSync、Sync、PostSync 鉤子，用於支援複雜的應用程式部署（例如藍綠部署和金絲雀升級）
11. 應用程式事件以及 API 呼叫的 Audit 追蹤以及 Prometheus metrics

<br>

{{< figure src="/git-or-cicd/argocd-introduce/argocd_architecture.webp" width="600" caption="ArgoCD 架構圖" >}}

<br>

## 環境說明 & 本次目標

那我們上面知道他有哪些功能了，那我們來設定本次文章想要實作的目標有哪些

1. 安裝及設定 ArgoCD
2. 註冊要部署的 Cluster
3. 從 Git 倉庫建立應用程式
4. 修改版控觀察自動部署
5. 修改線上設定觀察自動對齊

<br>

以及此文章會使用 GKE 來進行測試，並提供相關版本讓大家參考

- GKE Version：v1.33.5-gke.1125000
- ArgoCD Version：v3.2.0
- Ingress Nginx Controller Version：v1.9.5
- Cert-Manager Version：v1.13.3

<br>

## 安裝及設定 ArgoCD

根據官網的說明，需要先有幾個必要條件

<br>

{{< figure src="/git-or-cicd/argocd-introduce/1.webp" width="650" caption="ArgoCD 使用必要條件" >}}

<br>

因為是 K8s 所以一定要安裝 K8s 的工具 Kubectl，以及有 Kubeconfig 存放 K8s 的連線資訊，最後有寫到 CoreDNS 是因為 ArgoCD 會需要確保 K8s 內有 DNS 服務提供 Pod 解析到其他 Service 名稱 (如果是 GKE 預設就會安裝好，像是測試用的 microk8s 或是 EKS 就要記得安裝)

<br>

1. 使用文件的安裝指令，以及先建立好 namespace

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

<br>

{{< figure src="/git-or-cicd/argocd-introduce/2.webp" width="750" caption="ArgoCD 會安裝蠻多東西" >}}

<br>

{{< callout type="info" >}}
要注意，上面的安裝 yaml 的 ClusterRole 有寫死 namespace，所以如果要更換 namespace 記得也需要調整設定
{{< /callout >}}

<br>

2. 本機也要安裝 ArgoCD CLI，用來操作 ArgoCD 的工具

```shell
brew install argocd
```

<br>

### 連線 API Server

3. 接下來我們要存取 ArgoCD 的 API Server，但在預設的設定，API Server 沒有暴露外部 IP 位置，所以想要存取，有以下幾種方式

<br>

{{< figure src="/git-or-cicd/argocd-introduce/3.webp" width="1100" caption="ArgoCD 預設沒有開放存取" >}}

<br>

#### Service Type Load Balancer

將 Service 的 Type 改成 Load Balancer，可以使用以下指令

```shell
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

並取得 IP 位置

```shell
kubectl get svc argocd-server -n argocd -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

<br>

#### Port Forwarding 轉發

使用此指令就可以在本機上進行轉發

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

<br>

#### ingress 入口

需要設定 ArgoCD 與 Ingress，詳細可以參考 [Ingress Configuration](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/) 文件，**那本次測試會使用此方式來進行**

<br>

那因為 ArgoCD API Server 會同時執行 gRPC 給 CLI、HTTP/HTTPS 給 UI，官方文件有提到兩種做法，那我們這邊就選擇 SSL Termination at Ingress Controller，將 gRPC 跟 HTTP/HTTPS 分兩個 ingress 入口 (因為一個 ingress 只支援一個 protocol)，並停用 TLS。

<br>

1. 首先，我們要先調整 argocd-server Deployment 設定停用 TLS

```shell
kubectl -n argocd edit deployment argocd-server
```

然後找到 containers 的 args 在後面加上 `--insecure`，如下圖：

<br>

{{< figure src="/git-or-cicd/argocd-introduce/4.webp" width="550" caption="調整 API Server 設定" >}}

<br>

最後更新完，檢查一下設定是否有設定上去

<br>

{{< figure src="/git-or-cicd/argocd-introduce/5.webp" width="600" caption="檢查 API Server 設定" >}}

<br>

2. 新增兩個 ingress yaml，分別給 HTTP/HTTPS 跟 gPRC (這邊會用到 cert-manager 跟 ingress nginx controller，請先記得安裝好)

- http-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-http-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: pin-yi-letsencrypt
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
spec:
  ingressClassName: pin-yi
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: http
      host: argocd.pin-yi.me
  tls:
    - hosts:
        - argocd.pin-yi.me
      secretName: argocd-ingress-http
```

<br>

- grpc-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-grpc-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: pin-yi-letsencrypt
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
spec:
  ingressClassName: pin-yi
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: https
      host: grpc-argocd.pin-yi.me
  tls:
    - hosts:
        - grpc-argocd.pin-yi.me
      secretName: argocd-ingress-grpc
```

- cert-manager.io/cluster-issuer 是用 cert-manager 需補上，與官網範例不同
- ingressClassName 跟 host、tls.host 都需要改成自己要的 Domain、secretName 換成要儲存 TLS 的 secret

<br>

這個切換到 DNS 將 Domain 先指向到 ingress nginx 的 LB IP 上，最後再部署兩個 yaml，並觀察一下 ingress 狀況

<br>

{{< figure src="/git-or-cicd/argocd-introduce/6.webp" width="850" caption="設定 DNS 解析" >}}

<br>

都沒問題後，理論上就可以用瀏覽器開啟 https://argocd.pin-yi.me (HTTP 範例)，或是使用剛剛安裝好的 argocd 指令透過 CLI 連線 https://grpc-argocd.pin-yi.me (gRPC 範例)

<br>

## 登入 ArgoCD

那要怎麼登入呢？ArgoCD 預設的帳號是 admin，初始化的密碼需要透過以下指令顯示：

```shell
argocd admin initial-password -n argocd
```

{{< callout type="warning" >}}
它會以明文方式存在 argocd namespace 的 argocd-initial-admin-secret secret password 欄位，建議改完密碼後將該 secret 刪除
{{< /callout >}}

<br>

{{< figure src="/git-or-cicd/argocd-introduce/7.webp" width="1100" caption="查看初始密碼" >}}

<br>

- UI 登入

打開 https://argocd.pin-yi.me 輸入帳號密碼登入

<br>

{{< figure src="/git-or-cicd/argocd-introduce/8.webp" width="1100" caption="UI 登入" >}}

- CLI 登入

<br>

指令後面 Server 使用 `grpc-argocd.pin-yi.me` 輸入帳號密碼登入

{{< figure src="/git-or-cicd/argocd-introduce/9.webp" width="650" caption="CLI 登入" >}}

<br>

最後記得再使用 `argocd account update-password` 來修改密碼，就完成初始化的安裝跟設定囉(密碼也有長度限制喔)

<br>

{{< figure src="/git-or-cicd/argocd-introduce/10.webp" width="1200" caption="透過 CLI 修改密碼" >}}

<br>

## 註冊要部署的 Cluster

這邊會是選填，原因是假設我們建立 ArgoCD 的 Cluster 本身也有服務需要做部署，可以直接透過 `https://kubernetes.default.svc` 來連線，就不需要額外設定

但通常的做法都會是將 ArgoCD 獨立建立，然後透過 `argocd cluster add CONTEXTNAME` 來設定，該指令會安裝一個 ServiceAccount (argocd-manager) 到 CONTEXTNAME 的 kube-system namespace 上，並將該 SA 綁定管理員層級的 ClusterRole，後續 ArgoCD 就會使用該 SA Token 來執行管理任務 (部署/監控)

但目前手上沒有其他叢集，所以這邊就不先實作

<br>

{{< callout type="info" >}}
CONTEXTNAME 取得方式是需要先連上該叢集，接著下 `kubectl config get-contexts -o name` 將顯示的 CONTEXTNAME 貼到指令中
{{< /callout >}}

<br>

## 從 Git 倉庫建立應用程式

### ArgoCD 核心結構說明

接下來，我們看到官方的教學要我們設定 Applications，那我們就需要先了解一下 ArgoCD 裡面有兩個核心結構 Project 跟 Application

- Project
  用來定義邊界：允許的來源、或是允許的目的地、允許的資源總類等等，作用就是限制與治理

- Application
  用來定義同步的單元：來源 Git、渲染方式、部署的目的地、同步策略，作用是執行與落地

<br>

{{< callout type="info" >}}
Application 渲染方式是指 ArgoCD 在取到 Git 內的資料後，如何把這些資料轉換成最終要套用到 Kubernetes 的 純 YAML manifests，ArgoCD 不直接吃「原始設定格式」，它一定要先渲染（render）成標準 K8s YAML，才會進行同步
{{< /callout >}}

<br>

那我們後續說明會先以建立 Application 為主，然而官方的教學是使用 CLI 或是 UI 來建立，那我這邊會使用大家常用的方式，也就是 yaml 來定義 Application，所以想要先測試 CLI or UI 可以先去做官方教學的內容 [Create An Application From A Git Repository](https://argo-cd.readthedocs.io/en/stable/getting_started/#6-create-an-application-from-a-git-repository)

<br>

那我們開始前，需要先有 Git 倉庫，以及對應的 K8s 設定，好方便後續進行 Demo

那我們後面示範的程式都會放在 [880831ian/argocd-introduce](https://github.com/880831ian/argocd-introduce#) 裡面，歡迎大家使用它來測試，由於我們的 Git 倉庫是開放的，所以不需要額外設定 Key 來去訪問，那當然 ArgoCD 也支援先設定好相關的內容

<br>

{{< figure src="/git-or-cicd/argocd-introduce/11.webp" width="800" caption="ArgoCD UI 查看設定內 Repositories" >}}

<br>

### Demo Repo 目錄結構

我們先來看 [Demo Repo](https://github.com/880831ian/argocd-introduce#) 裡面的目錄架構

```shell
pin-yi:argocd-introduce/ git:(main) ✗ $ tree -L 1
.
├── application.yaml
├── grpc-ingress.yaml
├── helm-nginx-demo
├── http-ingress.yaml
├── README.md
└── yaml-nginx-demo
```

<br>

- application.yaml：用來定義 ArgoCD 的 Application 設定，裡面會包含一般 yaml 跟 helm
- grpc-ingress.yaml：是前面在連線 API Server 的 ingress yaml 檔案
- helm-nginx-demo/：裡面有用 helm 寫一個簡單的 nginx 服務 (由 yaml-nginx-demo 轉換成 helm)
- http-ingress.yaml：是前面在連線 API Server 的 ingress yaml 檔案
- yaml-nginx-demo/：裡面有用 yaml 寫一個簡單的 nginx 服務

<br>

我們接下來看 application.yaml 裡面寫了什麼

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: yaml-nginx-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/880831ian/argocd-introduce.git
    targetRevision: main
    path: yaml-nginx-demo
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy: {}
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-nginx-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/880831ian/argocd-introduce.git
    targetRevision: main
    path: helm-nginx-demo
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

這裡面其實包含了兩個 Application，分別是給 yaml 使用的 yaml-nginx-demo 以及改用 helm 的 helm-nginx-demo

來說明一下這個設定的意思是什麼：

metadata 的 name 跟 namespace 就是該 ArgoCD 的 CRD Application 所在的 namespace 跟 name

spec 底下有：

- project：指定 Project 的名稱，如果沒有指定，就會使用 default (這個 Project 也就是上面結構說的定義邊界功能)
- source：指定 Git 倉庫的設定，包含 Git 倉庫的 URL、目標版本分支、要同步的目錄
- destination：指定要同步到的目標集群的設定，包含集群的 URL、命名空間
- syncPolicy：指定同步策略

<br>

那其實仔細看，兩個設定其實差不多，就只有 name 跟 path 還有 syncPolicy 不同，其他都差不多，那這邊要特別提到 syncPolicy，這個也是 ArgoCD 的精髓

syncPolicy 在 yaml 的 Application，我們沒有去設定，有就是代表它不會有自動化的流程，都需要透過手動方式去觸發，例如有推新的版本，需要到 ArgoCD 去觸發 Sync 動作，才會把 K8s 設定實際部署到環境上

那在 helm 這邊，我們有設定 automated，他還有額外的兩個參數分別是：

- prune：比對 Git 與叢集實際狀態，移除叢集中那些「Git 已經不存在的資源」。用途在於保持宣告式一致性，不讓多餘物件殘留。
- selfHeal：比對 Git 與叢集實際狀態，當叢集中的資源被手動修改、漂移或遭到外部調整時，自動將其強制校正回 Git 宣告內容。

<br>

### 部署 Application 設定

因此有這兩個設定後，我們等等就可以來觀察是否正常運作～

<br>

那我們了解了 Application 的設定，就可以開始部署了，可以使用以下指令來 apply 該 CRD

```shell
kubectl apply -f application.yaml
```

<br>

部署上去後，可以觀察一下 Cluster 內 CRD (CustomResourceDefinition)、在 default 的 deployment 服務以及 ArgoCD UI 上面的變化

<br>

進入到 ArgoCD 的 Application CRD 後，會看到有兩個 Application 都有設定上去，只是兩個個狀態不同，一個是 Synced ; 另一個則是 OutOfSync，Deployment 部分，也只有 helm-nginx-demo 有部署上去

{{< figure src="/git-or-cicd/argocd-introduce/12.webp" width="1200" caption="CRD (CustomResourceDefinition) 跟 default 的 deployment" >}}

<br>

查看 UI 發現在 Application 頁面已經多了兩個，顯示的內容一個是綠色一個是黃色，狀態與 CRD 看得到ㄧ致

{{< figure src="/git-or-cicd/argocd-introduce/13.webp" width="1200" caption="ArgoCD UI" >}}

<br>

會有這個原因是因為我們剛剛有說，automated 設定，會影響服務同步的自動化流程，由於 helm 有設定 selfHeal，所以一部署，就會自動化完成，而 yaml 則就需要手動來觸發 sync

<br>

我們從 UI 點進去查看一下兩者差異

<br>

可以看到 helm-nginx-demo 有成功部署，且狀態都有被 ArgoCD 抓到，下方也會顯示該 Repo 裡面的相關資源架構圖

{{< figure src="/git-or-cicd/argocd-introduce/14.webp" width="1200" caption="helm-nginx-demo 成功部署" >}}

<br>

那 yaml-nginx-demo 則是因為沒有自動化，所以都卡在 OutOfSync 狀態

{{< figure src="/git-or-cicd/argocd-introduce/15.webp" width="900" caption="yaml-nginx-demo 卡在 OutOfSync" >}}

<br>

我們現在使用 UI 來觸發 yaml-nginx-demo Sync，會看到一些設定，我們先勾選 PRUNE，下方檔案保持不變，並按下 SYNCHRONIZE，其餘設定可以到 [Sync Options](https://argo-cd.readthedocs.io/en/latest/user-guide/sync-options/) 查看

<br>

{{< figure src="/git-or-cicd/argocd-introduce/16.webp" width="560" caption="手動同步" >}}

<br>

就可以發現 yaml-nginx-demo 更新囉～

<br>

{{< figure src="/git-or-cicd/argocd-introduce/17.webp" width="1200" caption="yaml-nginx-demo 狀態變成 synced" >}}

<br>

## 修改版控觀察自動部署

接下來，我們已經完成 Application 設定後，我們來模擬後續有更新 K8s 版控 ([調整 helm-nginx-demo、yaml-nginx-demo comfigmap worker_connections 數量](https://github.com/880831ian/argocd-introduce/commit/2c86357ea41150d92d413a2465aeeda515887c58))，他的自動部署會長怎樣：

<br>

{{< figure src="/git-or-cicd/argocd-introduce/18.webp" width="1200" caption="helm-nginx-demo 偵測到就會自動更新" >}}

<br>

{{< figure src="/git-or-cicd/argocd-introduce/19.webp" width="1200" caption="yaml-nginx-demo 由於沒有自動同步，則一樣會卡在 OutOfSync，可以點擊 diff 來查看異動的差異" >}}

<br>

## 修改線上設定觀察自動對齊

最後再來測試一下，假設我們不小心，或是在維運過程有去調整線上的設定，那這樣會導致線上跟版控不同，因此我們來模擬看看調整線上設定後(調整 Replica 數量)，是否會有自動對齊：

使用指令：

```shell
kubectl scale deployment <deployment name> --replicas=3
```

<br>

{{< figure src="/git-or-cicd/argocd-introduce/20.webp" width="600" caption="調整發現，數量沒有變化" >}}

<br>

{{< figure src="/git-or-cicd/argocd-introduce/helm.gif" width="1200" caption="可以看到，其實調整當下有變成 3 顆，但因為 helm 有設定 automated，所以馬上就會以版控為主給覆蓋設定" >}}

<br>

{{< figure src="/git-or-cicd/argocd-introduce/21.webp" width="600" caption="調整線上 Replica 設定成功" >}}

<br>

{{< figure src="/git-or-cicd/argocd-introduce/yaml.gif" width="1200" caption="可以看到，調整設定後，就會維持 3 個，且出現 OutOfSync" >}}

<br>

另外，如果部署 helm 有寫錯，再部署的時候也會噴錯提示呦～

<br>

{{< figure src="/git-or-cicd/argocd-introduce/22.webp" width="1200" caption="顯示錯誤原因，且不會正常部署" >}}

<br>

<br>

## 之後還可以做什麼？

本次的說明跟介紹就到這邊，ArgoCD 還是有很多東西沒有說到，因此我先設想一下之後還可以做哪些事情呢：

1. 透過 SSO 單一登入方式，整合原先常用的登入方式
2. 權限控管
3. 通知串接
4. 監控系統
5. 如何做到多 Cluster 管理
6. 嘗試將 helmfile 透過外掛擴充到 ArgoCD 🤣，有找到相關文件，可以參考看看 [Deploying Helm Charts using ArgoCD and Helmfile](https://christianhuth.de/deploying-helm-charts-using-argocd-and-helmfile/)

<br>

## 參考資料

What is ArgoCD (Youtube 影片)：[https://www.youtube.com/watch?v=p-kAqxuJNik](https://www.youtube.com/watch?v=p-kAqxuJNik)

GitOps 方法論是什麼東東？GitOps 的工作流程是什麼：[https://dongdonggcp.com/2024/12/02/what-is-gitops-methodolity-what-is-gitops-process/](https://dongdonggcp.com/2024/12/02/what-is-gitops-methodolity-what-is-gitops-process/)

ArgoCD 的規劃與實踐：[https://medium.com/ikala-tech/argocd-%E7%9A%84%E8%A6%8F%E5%8A%83%E8%88%87%E5%AF%A6%E8%B8%90-93ed88303585](https://medium.com/ikala-tech/argocd-%E7%9A%84%E8%A6%8F%E5%8A%83%E8%88%87%E5%AF%A6%E8%B8%90-93ed88303585)

Argo CD 官網：[https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)
