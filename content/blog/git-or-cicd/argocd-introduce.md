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
---

最近在準備面試，有些公司有使用 ArgoCD，由於前公司在 CICD 工具只使用 GitLab CI，所以比較沒有接觸這塊，只知道他是 GitOps 導向的工具，剛好趁這段時間來了解跟測試看看 ArgoCD，那我們開始囉～

<br>

## GitOps 定義

首先我們先來了解 GitOps 的定義

1. 開發團隊在程式碼管理、版本控制、CI/CD 自動化等流程，延伸至基礎設施（Infrastructure）與部署流程的操作框架
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
3. 串接 Git Repository
4. 測試 CD
5. 嘗試修改線上設定，觀察自動對齊

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

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

並取得 IP 位置

```
kubectl get svc argocd-server -n argocd -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

<br>

#### Port Forwarding 轉發

使用此指令就可以在本機上進行轉發

```
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

<br>

## 參考資料

What is ArgoCD (Youtube 影片)：[https://www.youtube.com/watch?v=p-kAqxuJNik](https://www.youtube.com/watch?v=p-kAqxuJNik)

GitOps 方法論是什麼東東？GitOps 的工作流程是什麼：[https://dongdonggcp.com/2024/12/02/what-is-gitops-methodolity-what-is-gitops-process/](https://dongdonggcp.com/2024/12/02/what-is-gitops-methodolity-what-is-gitops-process/)

ArgoCD 的規劃與實踐：[https://medium.com/ikala-tech/argocd-%E7%9A%84%E8%A6%8F%E5%8A%83%E8%88%87%E5%AF%A6%E8%B8%90-93ed88303585](https://medium.com/ikala-tech/argocd-%E7%9A%84%E8%A6%8F%E5%8A%83%E8%88%87%E5%AF%A6%E8%B8%90-93ed88303585)

Argo CD 官網：[https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)
