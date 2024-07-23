---
title: "kube-dns vs CoreDNS 比較，DNS 的最佳實踐"
type: docs
weight: 75
date: 2024-07-23
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

前情提要：由於上一篇筆記 [Pod 出現 cURL error 6: Could not resolve host](../pod-curl-error-6-could-not-resolve-host) 的後續規劃中，有提出我們想研究看看 Core DNS 來取代 Kube DNS，因此，這篇筆記要來比較一下 Kube DNS 以及 Core DNS 的差異以及兩者的優缺點，另外還會說明一下 DNS 的最佳實踐做法。

<br>

前面概論介紹 K8s 內的 DNS 以及 CoreDNS 主要都是參考 [探秘K8S DNS：解密IP查詢與服務發現](https://weng-albert.medium.com/%E6%8E%A2%E7%A7%98k8s-dns-%E8%A7%A3%E5%AF%86ip%E6%9F%A5%E8%A9%A2%E8%88%87%E6%9C%8D%E5%8B%99%E7%99%BC%E7%8F%BE-034de2e72abe)、[CoreDNS簡單除錯：解決你遇到的一般問題](https://weng-albert.medium.com/coredns%E7%B0%A1%E5%96%AE%E9%99%A4%E9%8C%AF-%E8%A7%A3%E6%B1%BA%E4%BD%A0%E9%81%87%E5%88%B0%E7%9A%84%E4%B8%80%E8%88%AC%E5%95%8F%E9%A1%8C-71d255e39548)，詳細可以查看原文。

<br>

## 概論 (K8s DNS)

K8s 會有 DNS 服務，主要是因為 K8s 連線是透過 Pod IP 來連線，但每當 Pod 重啟後 IP 會變動，所以就需要透過 Service 這個 Type 先建立持久性的名稱來讓其他 Pod 連線到後端服務上。

當我們建立 Service 時，K8s DNS 就會自動產生一個對應的 A record 將 service dns name 與 IP 位址配對。之後，Pod 就可以透過 DNS 名稱來進行連線。

而 DNS 負責動態更新 A record 與 Service IP 的異動。

就如上面有提到，目前 K8s 的 DNS 已經從 kube-dns 改成 CoreDNS，那兩者都有些相同的功能，如下：

1. kube-dns svc 可以建立一個或多個 Pod
2. kube-dns svc 監控 Kubernetes API 的 service 和 endpoint 事件，並根據需要變更其 DNS 項目。當透過建立、編輯或刪除操作修改這些 Kubernetes 服務及其相關 pod 時，這些事件會自動觸發。
3. Kubelet 會將 kube-dns svc 的叢集 IP 指派給每個新 Pod 的 `/etc/resolv.conf` 名稱伺服器檔案內，以及適當的搜尋設定以允許更短的主機名稱，如下：。

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/1.png" width="900" caption="可以看到 /etc/resolv.conf 與 kube-dns 的 cluster IP ㄧ致" >}}

<br>

K8s 服務的整個 DNS A record 如下：

- Service DNS name：

```
[service].[namespace].svc.cluster.local
```

service、namespace 可以自行替換，這也是我們最常使用的方式。

<br>

當然 Pod 也有自己的 DNS name，只是因爲 Pod 每次重啟 IP 會變動，所以我們通常不會使用 Pod DNS name：

- Pod DNS name：

```
[pod cluster IP].[namespace].pod.cluster.local
```

pod cluster IP 這邊要將 IP 從 . 的分隔符號換成 -，例如：`10-166-65-136.default.pod.cluster.local`

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/2.png" width="750" caption="可以看到 curl 的 pod cluster IP 與 svc 的 endpoint ㄧ致" >}}

<br>

所以，程式可以透過簡單且一致的主機名稱來存取 cluster 其他服務或是 Pod。

也不需要使用完整的 hostname 來存取服務，因為在 `resolv.conf` 會設定好 domain 的 suffixes，如果同一個 namespace 底下服務溝通，只要設定以下即可：

```
[service]
```

跨到其他 nampesace 就要改成：

```
[service].[namespace]
```

<br>

## 應用程式 DNS 查詢流程

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/3.jpg" width="700" caption="應用程式 DNS 查詢流程 圖片：[Connecting the Dots: Understanding How Pods Talk in Kubernetes Networks](https://medium.com/@seifeddinerajhi/connecting-the-dots-understanding-how-pods-talk-in-kubernetes-networks-992fa69fbbeb)" >}}

<br>

下面列出一下應用程式在 K8s 內的 DNS 查詢流程：
1. 當 Pod 執行 DNS 查詢時，會先查詢 Pod 裡面的 `resolv.conf` 檔案，再來查詢 Pod 所在的 node 上的 `resolv.conf` 檔案，這個檔案有設定 nodelocaldns 伺服器被設定為預設遞迴 DNS 解析器，充當快取 (在 GKE 需要另外開啟)。
2. 如果此快取不包含所請求主機名稱的 IP 位址，則查詢將轉送至叢集 DNS 伺服器 (CoreDNS)，目前 GKE 是 kube-dns。
3. 此 DNS 伺服器透過查詢 Kubernetes 服務註冊表來決定 IP 位址。此註冊表包含服務名稱到對應 IP 位址的對應。這允許叢集 DNS 伺服器將正確的 IP 位址傳回給請求的 pod。
4. 任何被查詢但不在 Kubernetes 服務註冊表中的網域都會轉送到上游 DNS 伺服器。

<br>

## 概論 (kube-dns)

這邊就來說一下目前 GKE 在使用的 kube-dns 架構，主要由這三個 container 組成：

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/4.jpg" width="600" caption="kube-dns 架構 圖片：[DNS](https://feisky.gitbooks.io/kubernetes/content/components/kube-dns.html)" >}}

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/5.png" width="750" caption="目前 GKE 1.29 kube-dns" >}}

<br>

- kubedns：DNS 服務核心元件，主要由 KubeDNS 以及 SkyDNS 兩個元件組成：
  - KubeDNS 負責監聽 Kubernetes API 的 service 和 endpoint 事件，並將相關訊息更新到 SkyDNS 中。
  - SkyDNS 負責 DNS 解析，監聽在 10053 Port (tcp/udp)，也同時監聽在 10055 Port 提供 metrics。

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/6.png" width="750" caption="SkyDNS Metrics Port 以及監聽 10053 Port" >}}

<br>

- dnsmasq (有換名稱)：負責啟動 dnsmasq，並在變化時重啟 dnsmasq：
  - dnsmasq 的 Upstream DNS Server 是 SkyDNS，cluster 內部的 DNS 查詢都會由 SkyDNS 負責。

- sidecar：負責健康檢查和提供 DNS metrics 監聽在 10054 Port。

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/7.png" width="750" caption="sidecar 健康檢查以及監聽 10054 Port" >}}

<br>

## 概論 (CoreDNS)

這邊就說明一下我們想要研究的 CoreDNS 吧，其實早就應該用 CoreDNS 來取代 kube-dns，~~~只是 GKE 不知道為什麼還不支援~~~

CoreDNS 是一個用 Go 語言寫的 DNS 伺服器，它是一個 CNCF 的專案，目前已經成為 K8s 的預設 DNS 伺服器。

跟其他 DNS 比較不一樣的是他非常靈活彈性，並且所有功能都是透過 Plugins 來實現，這樣就可以根據需求來自訂自己的 DNS 伺服器。


它的特點是：
1. Plugins (插件化)
2. Service Discovery (服務發現)
3. Fast and Flexible (快速且靈活)
4. Simplicity (簡單)

這邊來說一下 常見的 Plugins 如下：
- loadbalance：提供基於 DNS 的負載均衡功能
- loop：檢測在 DNS 解析過程中出現的簡單循環問題
- cache：提供前端快取功能
- health：對 Endpoint 進行健康檢查
- kubernetes：提供對 Kubernetes API 的訪問，並將其轉換為 DNS 記錄
- etcd：提供對 etcd 的訪問，並將其轉換為 DNS 記錄
- reload：定時自動重新加載 Corefile 配置的內容
- forward：轉發域名查詢到上游 DNS 伺服器
- proxy：轉發特定的域名查詢到多個其他 DNS 伺服器，同時提供到多個 DNS 服務器的負載均衡功能
- prometheus：爲 Prometheus 系統提供採集性能指摽數據的 URL
- log：對 DNS 查詢進行日誌記錄
- errors：對 DNS 查詢錯誤進行日誌記錄

<br>

{{< figure src="/kubernetes/kube-dns-vs-core-dns/8.png" width="750" caption="CoreDNS 使用的 Plugins/架構 圖片：[A Brief Look at CoreDNS](https://livewyer.io/blog/2018/05/31/a-brief-look-at-coredns/)" >}}

<br>

CoreDNS 的 helm：[https://github.com/coredns/helm/tree/master/charts/coredns](https://github.com/coredns/helm/tree/master/charts/coredns)

<br>

## kube-dns vs CoreDNS 優缺點

### kube-dns
  - 優點：有 dnsmesq，在性能上有一定的保障。
  - 缺點： dnsmesq 如果重啟，會先刪掉 process 才會重新起服務，中間可能會出現查詢失敗。在確認內部檔案時，如果數量過多或是太頻繁更新 DNS 有可能反而導致 dnsmasq 不穩，這時候就需要重啟 dnsmasq 服務。

<br>

### CoreDNS
 - 優點：可以自行根據需求使用自訂的 Plugins，記憶體的佔用情況也比 kube-dns 好。
 - 缺點：Cache 功能不如 dnsmasq，內部解析速度可能會比 kube-dns 慢。

<br>

## GKE 上的 DNS，可以改用 CoreDNS 嗎？

目前 GKE 上的 DNS 預設是 kube-dns，或是使用 GCP 提供的另一個 Cloud DNS 服務，但是上次我們有提到，因為公司之後想走多雲架構，會開始使用 AWS，而 AWS 的 EKS 預設的 DNS 服務是 CoreDNS (K8s 現在預設也是 CoreDNS 😿)

所以我們想說，是不是可以在 GKE 上也改使用 CoreDNS，這樣之後維護可以更一致。

<br>

在爬文的過程中有發現，Google 有一篇 knowledge 文章 [How to run CoreDNS on Kubernetes Engine?](https://cloud.google.com/knowledge/kb/how-to-run-coredns-on-kubernetes-engine-000004698) 有提到 Google 預設就是使用 kube-dns，沒辦法把 kube-dns 縮小到 0 並完全替換，如果只是單純想要使用 CoreDNS 的快取功能，可以啟用 GKE 上的 NodeLocal DNSCache

如果想要把 CronDNS 解析功能加到 GKE 上，要先部署一個 core-dns Pod，並透過 service 公開它，並為 kube-dns 配置一個 stub domain，將其指向 core-dns 服務 IP。

所有與 stub domain 後綴相符的流量都會路由到 core-dns Pod。不符合的流量將繼續由 kube-dns 解析。

<br>

## DNS 的最佳實踐

(代補....)

<br>


## 參考資料

探秘K8S DNS：解密IP查詢與服務發現：[https://weng-albert.medium.com/%E6%8E%A2%E7%A7%98k8s-dns-%E8%A7%A3%E5%AF%86ip%E6%9F%A5%E8%A9%A2%E8%88%87%E6%9C%8D%E5%8B%99%E7%99%BC%E7%8F%BE-034de2e72abe](https://weng-albert.medium.com/%E6%8E%A2%E7%A7%98k8s-dns-%E8%A7%A3%E5%AF%86ip%E6%9F%A5%E8%A9%A2%E8%88%87%E6%9C%8D%E5%8B%99%E7%99%BC%E7%8F%BE-034de2e72abe)

CoreDNS簡單除錯：解決你遇到的一般問題：[https://weng-albert.medium.com/coredns%E7%B0%A1%E5%96%AE%E9%99%A4%E9%8C%AF-%E8%A7%A3%E6%B1%BA%E4%BD%A0%E9%81%87%E5%88%B0%E7%9A%84%E4%B8%80%E8%88%AC%E5%95%8F%E9%A1%8C-71d255e39548](https://weng-albert.medium.com/coredns%E7%B0%A1%E5%96%AE%E9%99%A4%E9%8C%AF-%E8%A7%A3%E6%B1%BA%E4%BD%A0%E9%81%87%E5%88%B0%E7%9A%84%E4%B8%80%E8%88%AC%E5%95%8F%E9%A1%8C-71d255e39548)

Connecting the Dots: Understanding How Pods Talk in Kubernetes Networks：[https://medium.com/@seifeddinerajhi/connecting-the-dots-understanding-how-pods-talk-in-kubernetes-networks-992fa69fbbeb](https://medium.com/@seifeddinerajhi/connecting-the-dots-understanding-how-pods-talk-in-kubernetes-networks-992fa69fbbeb)

How to run CoreDNS on Kubernetes Engine?：[https://cloud.google.com/knowledge/kb/how-to-run-coredns-on-kubernetes-engine-000004698](https://cloud.google.com/knowledge/kb/how-to-run-coredns-on-kubernetes-engine-000004698)

