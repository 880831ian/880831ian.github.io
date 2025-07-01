---
title: "GCP Load Balancer 介紹"
type: docs
weight: 9988
description: GCP Load Balancer 介紹
images:
  - gcp/gcp-lb-introduce/og.webp
date: 2025-05-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

最近公司在導入 Multi-Zone，有發現大量的跨域費用產生，主要是不同 Cluster 或是 Cluster 跟 VM 之間的跨域流量，與 Google TAM 討論，他們有提出可以嘗試用 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy) 的 Waterfall by zone，或是之後會推出的 Zone affinity，這些都會需要使用到 Load Balancer，簡單整理一下發現 GCP 的 Load Balancer 其實有很多種，因此，這篇文章就來介紹一下 GCP 的 Load Balancer。
(如果 Service load balancing policy 測試有結果，也會再寫一篇文章來介紹)

<br>

> [!NOTE] 想知道自己用的 Load Balancer 是哪一種嗎？
>由於以下會有多種的 Load Balancer 種類，且 gcloud CLI 沒辦法直接過濾相關 LB 名稱 (UI 可以，但需要每個都進去查看)，因此有特別寫了一隻腳本，可以來掃描查詢，請參考： [gcp-load-balancer-type](https://github.com/880831ian/gcp-load-balancer-type)

<br>

GCP 的 Load Balancer 主要分為三種，分別是：

### Application Load Balancer (ALB) – 應用程式負載平衡器
1. 層級：第 7 層（L7）
2. 支援協定：HTTP、HTTPS
3. 功能：
    1. 基於內容的路由（如 URL 路徑、主機名稱）
    2. SSL/TLS 終止
    3. 整合 Cloud CDN、Cloud Armor
    4. 支援全球負載平衡（Premium Tier）
4. 適用場景：需要進行應用層路由、SSL 終止，以及全球流量分配的 Web 應用程式。

<br>

Application Load Balancer 又可以再另外分為五種 (包含內、外網以及不同的 Region)：

#### Global external Application Load Balancer

將此負載平衡器用於具有全球分散用戶或多個區域的後端服務的外部 HTTP(S) 工作負載。 (官方建議使用)

##### 資訊
- 中文：全球外部應用程式負載平衡器
- 縮寫：GLB
- 內部/外部：Public facing (external)
- 區域：Global
- Load balancing scheme：EXTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：✅
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：✅
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 與 GKE 相容，使用[閘道](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api#gatewayclass)或獨立 NEG
- 支援先進的流量管理
- 只能使用 [Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 可以跨多個專案以及區域來存取後端

<br>

{{< figure src="/gcp/gcp-lb-introduce/1.jpg" width="250" caption="Global external Application Load Balancer" >}}

<br>

#### Classic Application Load Balancer

此負載平衡器在高階層中是全球性的。在 Premium 網路服務層，此負載平衡器提供多區域負載平衡，嘗試將流量引導至具有容量的最近的健康後端，並儘可能靠近使用者終止 HTTP(S) 流量。

在 Standard 網路服務層中，此負載平衡器只能將流量分配到單一區域內的後端。(建議不要再使用該 LB)

##### 資訊
- 中文：經典應用程式負載平衡器
- 縮寫：CLB
- 內部/外部：Public facing (external)
- 區域：Global
- Load balancing scheme：EXTERNAL
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：✅
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 與 GKE 相容，使用[閘道](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api#gatewayclass)、Ingress 或獨立 NEG
- 可以選擇 [Standard 或是 Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 可以跨多個區域來存取後端 (選擇 Standrad 不能用)
- 有支援 Cloud CDN，但選擇 Standrad 則不能用

<br>

{{< figure src="/gcp/gcp-lb-introduce/2.jpg" width="250" caption="Classic Application Load Balancer" >}}

<br>

#### Regional external Application Load Balancer

此負載平衡器包含現有的經典應用程式負載平衡器， 以及先進的流量管理能力。
如果您只想從一個地理位置提供內容，請使用此負載平衡器。

##### 資訊
- 中文：區域外部應用程式負載平衡器
- 縮寫：ALB
- 內部/外部：Public facing (external)
- 區域：Regional
- Load balancing scheme：EXTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 與 GKE 相容，使用獨立 NEG
- 支援先進的流量管理
- 可以選擇 [Standard 或是 Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 可以跨多個專案，但只能選擇該區域的資源 (且沒辦法接 Bucket)

<br>

{{< figure src="/gcp/gcp-lb-introduce/3.jpg" width="250" caption="Regional external Application Load Balancer" >}}

<br>

#### Cross-Region internal Application Load Balancer

這是一個多區域負載平衡器，它基於開源 [Envoy 代理](https://www.envoyproxy.io/)實現為託管服務。跨區域模式使您能夠將流量負載平衡到全球分佈的後端服務，包括確保流量定向到最近的後端的流量管理。此負載平衡器還具有高可用性。將後端放置在多個區域有助於避免單一區域故障。如果一個區域的後端發生故障，流量可以轉移到另一個區域。

##### 資訊
- 中文：跨區域內部應用程式負載平衡器
- 縮寫：Cross-Region internal ALB
- 內部/外部：Internal
- 區域：Global
- Load balancing scheme：INTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：❌
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：✅
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 始終可在全球範圍內訪問。 VPC 中任何 Google Cloud 區域的用戶端都可以將流量傳送到負載平衡器
- 負載平衡器可以將流量傳送到任何區域的後端 (可以跨多個專案)
- 自動故障轉移到同一或不同區域的健康後端

<br>

{{< figure src="/gcp/gcp-lb-introduce/4.jpg" width="250" caption="Cross-Region internal Application Load Balancer" >}}

<br>

#### Regional internal Application Load Balancer

這是一個區域負載平衡器，它基於開源 [Envoy 代理程式](https://www.envoyproxy.io/)作為託管服務實作。區域模式可確保所有用戶端和後端都來自指定區域，這在您需要區域合規性時很有幫助。此負載平衡器具備基於 HTTP(S)參數的豐富流量控制功能。負載平衡器配置完成後，它會自動指派 Envoy 代理程式來滿足您的流量需求。

##### 資訊
- 中文：區域內部應用程式負載平衡器
- 縮寫：ILB
- 內部/外部：Internal
- 區域：Regional
- Load balancing scheme：INTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 預設無法由不同 Region 進行訪問，需要額外開啟全球訪問設定
- 負載平衡器只能將流量傳送到與負載平衡器的代理程式位於相同區域的後端 (可以跨多個專案)
- 自動故障轉移到同一區域內的健康後端

<br>

{{< figure src="/gcp/gcp-lb-introduce/5.jpg" width="250" caption="Regional internal Application Load Balancer" >}}

<br>

### Proxy Network Load Balancer (PNLB) – 代理網路負載平衡器

1. 層級：第 4 層（L4）
2. 支援協定：TCP、SSL
3. 功能：
    1. 作為反向代理，終止 TCP 或 SSL 連線
    2. 支援 SSL/TLS 終止（僅限 SSL Proxy 模式）
    3. 可選擇全球（Premium Tier）或區域性（Standard Tier）部署
4. 適用場景：需要處理加密的 TCP 流量，並在負載平衡器層級終止 SSL 的應用程式。

<br>

Proxy Network Load Balancer 又可以再另外分為五種 (包含內、外網以及不同的 Region)：

#### Global external Proxy Network Load Balancer

此負載平衡器適用於需要全球可用性和高效能的 TCP/SSL 應用程式。它支援使用 Zonal NEGs（包括 VM 和 GKE Pod）作為後端，並整合 Google Cloud Armor 進行安全防護。

##### 資訊
- 中文：全球外部代理網路負載平衡器
- 縮寫：Global Proxy NLB
- 內部/外部：Public facing (external)
- 區域：Global
- Load balancing scheme：EXTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：✅
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：✅

<br>

##### 特點
- 只能使用 [Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 支援 TCP 和 SSL 協定
- 支援 SSL/TLS 卸載
- 支援使用 Zonal NEGs（包括 VM 和 GKE Pod）作為後端
- 整合 Google Cloud Armor 進行 DDoS 防護

<br>

{{< figure src="/gcp/gcp-lb-introduce/6.jpg" width="250" caption="Global external Proxy Network Load Balancer" >}}

<br>

#### Classic Proxy Network Load Balancer

此負載平衡器適用於現有使用傳統實例群組（Instance Groups）的應用程式。在 Premium 網路服務層中，它可以作為全球性的負載平衡器；在 Standard 網路服務層中，僅限於區域性部署。它支援 TCP 和 SSL 協定，並支援 SSL/TLS 卸載。

##### 資訊
- 中文：經典代理網路負載平衡器
- 縮寫：Classic Proxy NLB
- 內部/外部：Public facing (external)
- 區域：Global
- Load balancing scheme：EXTERNAL
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：✅
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：✅

<br>

##### 特點
- 可以選擇 [Standard 或是 Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 支援 TCP 和 SSL 協定
- 支援 SSL/TLS 卸載
- 支援使用傳統實例群組作為後端
- 在 Premium Tier 中可作為全球性負載平衡器，在 Standard Tier 中僅限於區域性部署

<br>

{{< figure src="/gcp/gcp-lb-introduce/7.jpg" width="250" caption="Classic Proxy Network Load Balancer" >}}

<br>

#### Regional external Proxy Network Load Balancer

此負載平衡器適用於需要在單一區域內處理 TCP 流量的應用程式。它在該區域內提供外部 IP，並將進來的 TCP 流量轉發至後端服務。此負載平衡器支援 TCP 協定，並可選擇使用 Premium 或 Standard 網路服務層級。

##### 資訊
- 中文：區域外部代理網路負載平衡器
- 縮寫：Regional Proxy NLB
- 內部/外部：Public facing (external)
- 區域：Regional
- Load balancing scheme：EXTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：❌
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：❌
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：✅

<br>

##### 特點
- 可以選擇 [Standard 或是 Premium 等級的 Network Service](https://cloud.google.com/network-tiers?hl=zh-tw)
- 支援 TCP 協定
- 支援使用 Compute Engine 虛擬機（VM）作為後端服務
- 相較於全球性負載平衡器，區域性負載平衡器的成本較低，適合預算有限的應用程式

<br>

{{< figure src="/gcp/gcp-lb-introduce/8.jpg" width="250" caption="Regional external Proxy Network Load Balancer" >}}

<br>

#### Cross-Region internal Proxy Network Load Balancer

這是一個多區域負載平衡器，它基於開源 [Envoy 代理](https://www.envoyproxy.io/)實現為託管服務。跨區域模式可讓您將流量負載平衡到全球分佈的後端服務，包括確保流量定向到最近的後端的流量管理。此負載平衡器還具有高可用性。將後端放置在多個區域有助於避免單一區域故障。如果一個區域的後端發生故障，流量可以轉移到另一個區域。

##### 資訊
- 中文：跨區域內部代理網路負載平衡器
- 縮寫：Cross-Region internal Proxy NLB
- 內部/外部：Internal
- 區域：Global
- Load balancing scheme：INTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：❌
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：✅
- 是否支援 SSL：❌
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：✅

<br>

##### 特點
- 始終可在全球範圍內訪問。 VPC 中任何 Google Cloud 區域的用戶端都可以將流量傳送到負載平衡器
- 負載平衡器可以將流量傳送到任何區域的後端
- 自動故障轉移到同一或不同區域的健康後端

<br>

{{< figure src="/gcp/gcp-lb-introduce/9.jpg" width="250" caption="Cross-Region internal Proxy Network Load Balancer" >}}

<br>

#### Regional internal Proxy Network Load Balancer

這是一個區域負載平衡器，它基於開源 [Envoy 代理程式](https://www.envoyproxy.io/)作為託管服務實作。區域模式可確保所有用戶端和後端都來自指定區域，這在您需要區域合規性時很有幫助。

##### 資訊
- 中文：區域內部代理網路負載平衡器
- 縮寫：Regional internal Proxy NLB
- 內部/外部：Internal
- 區域：Regional
- Load balancing scheme：INTERNAL_MANAGED
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：❌
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：❌
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：✅

<br>

##### 特點
- 預設無法由不同 Region 進行訪問，需要額外開啟全球訪問設定
- 負載平衡器只能將流量傳送到與負載平衡器的代理程式位於相同區域的後端
- 自動故障轉移到同一區域內的健康後端

<br>

{{< figure src="/gcp/gcp-lb-introduce/10.jpg" width="250" caption="Regional internal Proxy Network Load Balancer" >}}

<br>

### Passthrough Network Load Balancer (NLB) – 直通網路負載平衡器

1. 層級：第 4 層（L4）
2. 支援協定：TCP、UDP、ESP、GRE、ICMP、ICMPv6 等
3. 功能：
    1. 不終止連線，將流量直接傳遞給後端
    2. 保留原始封包資訊（來源 IP、目的地 IP 等）
    3. 僅支援區域性部署
4. 適用場景：需要低延遲、高效能，並保留原始封包資訊的內部服務，如資料庫、內部微服務通訊等。

<br>

Passthrough Network Load Balancer 主要分為兩種 (內、外網)：

#### External passthrough Network Load Balancer

這是一種區域性的第 4 層（L4）負載平衡器，將來自網際網路的流量分配至同一區域內的後端服務。它不進行代理或 SSL/TLS 卸載，並保留原始的客戶端 IP 位址。

##### 資訊
- 中文：外部直通網路負載平衡器
- 縮寫：External NLB
- 內部/外部：Public facing (external)
- 區域：Regional
- Load balancing scheme：EXTERNAL
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：✅
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：✅
- 是否支援 SSL：❌
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 支援 TCP、UDP、ESP、GRE、ICMP 和 ICMPv6 等協定
- 保留原始客戶端 IP 位址，適用於需要此資訊的應用程式
- 支援使用區域性的後端服務或目標集（Target Pool）作為後端
- 可與 Google Cloud Armor 整合，提供進階的 DDoS 防護
- 支援 IPv4 和 IPv6 流量
- 適用於需要低延遲和高效能的應用場景

<br>

{{< figure src="/gcp/gcp-lb-introduce/11.jpg" width="200" caption="External passthrough Network Load Balancer" >}}

<br>

#### Internal passthrough Network Load Balancer

這是一種區域性的第 4 層（L4）負載平衡器，將流量分配至同一 VPC 網路內的後端服務。它僅在內部網路中運作，適用於內部服務之間的通訊。

##### 資訊
- 中文：內部直通網路負載平衡器
- 縮寫：Internal NLB
- 內部/外部：Internal
- 區域：Regional
- Load balancing scheme：INTERNAL
- 是否支援 [Cloud CDN](https://cloud.google.com/cdn)：❌
- 是否支援 [Cloud Armor](https://cloud.google.com/security/products/armor)：❌
- 是否支援 [Service load balancing policy](https://cloud.google.com/load-balancing/docs/service-lb-policy)：❌
- 是否支援 SSL：❌
- 是否支援 [PROXY protocol](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)：❌

<br>

##### 特點
- 支援 TCP、UDP、ICMP、ICMPv6、SCTP、ESP、AH 和 GRE 等協定
- 保留原始客戶端 IP 位址，適用於需要此資訊的內部應用程式
- 支援使用區域性的後端服務作為後端
- 可作為靜態路由的下一跳，實現更靈活的流量控制
- 支援與 Service Directory 整合，方便服務發現與管理
- 適用於需要高效能和低延遲的內部服務通訊

<br>

{{< figure src="/gcp/gcp-lb-introduce/12.jpg" width="250" caption="Internal passthrough Network Load Balancer" >}}

<br>

## 總結

| 負載平衡器類型 | 層級 | 協定 | 功能 | 適用場景 |
|----------------|------|------|------|----------|
| Application Load Balancer (ALB) | L7 | HTTP, HTTPS | 基於內容的路由、SSL/TLS 終止、整合 Cloud CDN 和 Cloud Armor | Web 應用程式需要應用層路由和 SSL 終止 |
| Proxy Network Load Balancer (PNLB) | L4 | TCP, SSL | 反向代理、SSL/TLS 終止 | 處理加密的 TCP 流量，並在負載平衡器層級終止 SSL 的應用程式 |
| Passthrough Network Load Balancer (NLB) | L4 | TCP, UDP, ESP, GRE, ICMP 等 | 不終止連線，保留原始封包資訊 | 低延遲、高效能的內部服務，如資料庫、內部微服務通訊等 |

<br>

## 參考資料

Network Service Tiers：[https://cloud.google.com/network-tiers?hl=zh-tw](https://cloud.google.com/network-tiers?hl=zh-tw)

External Application Load Balancer：[https://cloud.google.com/load-balancing/docs/https](https://cloud.google.com/load-balancing/docs/https)

Internal Application Load Balancer：[https://cloud.google.com/load-balancing/docs/l7-internal](https://cloud.google.com/load-balancing/docs/l7-internal)

Proxy Network Load Balancer：[https://cloud.google.com/load-balancing/docs/proxy-network-load-balancer](https://cloud.google.com/load-balancing/docs/proxy-network-load-balancer)

Internal proxy Network Load Balancer：[https://cloud.google.com/load-balancing/docs/tcp/internal-proxy](https://cloud.google.com/load-balancing/docs/tcp/internal-proxy)

Passthrough Network Load Balancer：[https://cloud.google.com/load-balancing/docs/passthrough-network-load-balancer](https://cloud.google.com/load-balancing/docs/passthrough-network-load-balancer)

Internal passthrough Network Load Balancer：[https://cloud.google.com/load-balancing/docs/internal](https://cloud.google.com/load-balancing/docs/internal)