---
title: "GKE DNS 相關注意事項及結論"
type: docs
weight: 9984
description: GKE DNS 相關注意事項及結論
images:
  - gcp/gke-cloud-dns-nodelocaldnscache/og.webp
date: 2025-08-12
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Google Cloud Platform
  - GCP
  - Kubernetes
  - K8s
  - DNS
  - KubeDNS
  - Cloud DNS
  - NodeLocal DNSCache
---

一樣先前情提要：

最近在評估要將公司內的 EndPoint 都改成 Cloud DNS 的 Private Zone (打造內部的 internal dns 服務機制)，到時候 DNS 解析的請求會比以往還要多，所以需要先測試評估 GKE 內的 DNS 解析方案，避免再次發生 [Pod 出現 cURL error 6: Could not resolve host](../../kubernetes/pod-curl-error-6-could-not-resolve-host)，此篇文章再來回顧以及整理，可以依照情境測試後的結論以及注意事項，來選擇適合 GKE DNS 組合：

[GKE KubeDNS 運作測試](../gke-kubedns/)

[GKE KubeDNS + NodeLocal DNSCache 運作測試](../gke-kubedns-nodelocaldnscache/)

[GKE Cloud DNS 運作測試](../gke-cloud-dns/)

[GKE Cloud DNS + NodeLocal DNSCache 運作測試](../gke-cloud-dns-nodelocaldnscache/)

<br>

## 測試情境的依賴性

| GKE DNS 情境 | KubeDNS | KubeDNS + NodeLocal DNSCache | Cloud DNS | Cloud DNS + NodeLocal DNSCache |
| :--- | :--- | :--- | :--- | :--- |
| 叢集內部 cluster.local<br>KubeDNS Pod 異常 | KubeDNS 切成 0 顆後，不會馬上無法解析，因為 KubeDNS 有 Liveness，不會馬上關掉，等到 KubeDNS 關成 0 後，就無法解析 | KubeDNS 切成 0 顆後，不會馬上無法解析，因為有 NodeLocal DNSCache，透過 Cache 還可以回覆 DNS 請求，但等到 TTL 時間到，就會出現無法解析 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 |
| 叢集內部 cluster.local<br>NodeLocal DNSCache Pod 異常 | - | 當 NodeLocal DNSCache 切成 0 顆後，會短暫卡住，但他會自動切換到 KubeDNS 上繼續解析 | - | 當 NodeLocal DNSCache 切成 0 顆後，會直接卡住，會出現無法解析，也不會自動切換成 KubeDNS |
| Internal DNS (Cloud DNS Private)<br>KubeDNS Pod 異常 | KubeDNS 切成 0 顆後，不會馬上無法解析，因為 KubeDNS 有 Liveness，不會馬上關掉，等到 KubeDNS 關成 0 後，就無法解析 | KubeDNS 切成 0 顆後，解析依然正常，因為 Cloud DNS Private 他不是 cluster.local，所以不會透過 KubeDNS 來做解析 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 |
| Internal DNS (Cloud DNS Private)<br>NodeLocal DNSCache Pod 異常 | - | 當 NodeLocal DNSCache 切成 0 顆後，會短暫卡住，但他會自動切換到 KubeDNS 上繼續解析 | - | 當 NodeLocal DNSCache 切成 0 顆後，會直接卡住，會出現無法解析，也不會自動切換成 KubeDNS |
| 外部 DNS (ifconfig.me)<br>KubeDNS Pod 異常 | KubeDNS 切成 0 顆後，不會馬上無法解析，因為 KubeDNS 有 Liveness，不會馬上關掉，等到 KubeDNS 關成 0 後，就無法解析 | KubeDNS 切成 0 顆後，解析依然正常，因為外部 DNS 他不是 cluster.local，所以不會透過 KubeDNS 來做解析 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 | KubeDNS 切成 0 顆後，解析依舊正常，因為 provider 已經改用 Cloud DNS，對解析不會有影響 |
| 外部 DNS (ifconfig.me)<br>NodeLocal DNSCache Pod 異常 | - | 當 NodeLocal DNSCache 切成 0 顆後，會短暫卡住，但他會自動切換到 KubeDNS 上繼續解析 | - | 當 NodeLocal DNSCache 切成 0 顆後，會直接卡住，會出現無法解析，也不會自動切換成 KubeDNS |

<br>

## 測試對於情境的依賴性 (不管 Liveness 或是有 Cache，以結果來看)

✅：代表服務異常不會對解析造成影響

❌：代表服務異常會對解析造成影響

| GKE DNS 情境 | KubeDNS | KubeDNS + NodeLocal DNSCache | Cloud DNS | Cloud DNS + NodeLocal DNSCache |
| :--- | :--- | :--- | :--- | :--- |
| 叢集內部 cluster.local<br>KubeDNS Pod 異常 | ❌ | ❌ | ✅ | ✅ |
| 叢集內部 cluster.local<br>NodeLocal DNSCache Pod 異常 | - | ✅ | - | ❌ |
| Internal DNS (Cloud DNS Private)<br>KubeDNS Pod 異常 | ❌ | ✅ | ✅ | ✅ |
| Internal DNS (Cloud DNS Private)<br>NodeLocal DNSCache Pod 異常 | - | ✅ | - | ❌ |
| 外部 DNS (ifconfig.me)<br>KubeDNS Pod 異常 | ❌ | ✅ | ✅ | ✅ |
| 外部 DNS (ifconfig.me)<br>NodeLocal DNSCache Pod 異常 | - | ✅ | - | ❌ |

<br>

## 結論與優缺點

| 模式 | 結論 | 優點 | 缺點 | /etc/resolv.conf |
| :--- | :--- | :--- | :--- | :--- |
| **KubeDNS** | 對 KubeDNS 的依賴性極高。只要壞掉，所有解析都會失效 | GKE 預設設定，能快速使用 | 容易出現單點故障，影響所有 DNS 解析。<br>GKE KubeDNS 可以將 kube-dns-autoscaler configmap 納管進版控來調整 Pod 數量 (但沒辦法動態調整)<br>Deployment 沒辦法調整，會被還原，因為有：addonmanager.kubernetes.io/mode=Reconcile 設定，由 GKE 維運 ([也可以自建 DNS](https://cloud.google.com/kubernetes-engine/docs/how-to/custom-kube-dns?hl=zh-tw)) | KubeDNS 的 IP |
| **KubeDNS + NodeLocal DNSCache** | 大幅降低對 KubeDNS 的依賴。快取可以應對 KubeDNS 短暫異常 | 提高 DNS 解析的效能和可靠性 | 需要維護 KubeDNS 的 kube-dns-autoscaler configmap 來調整 Pod 數量 | KubeDNS 的 IP |
| **Cloud DNS** | 所有的 DNS 解析都依賴 Cloud DNS。可以將 KubeDNS Pod 縮減為 0 | 高可用性。DNS 責任交由託管服務處理，節省維護成本 | 會有額外的費用，以及若 GCP DNS 出現異常會導致 GKE 服務無法解析問題 | 169.254.169.254 |
| **Cloud DNS + NodeLocal DNSCache** | 當 NodeLocal DNSCache 只要發生問題，所有的解析都會有異常。<br>所以不太建議使用 Cloud DNS + NodeLocal DNSCache 的模式 | 提供更快的 DNS 解析速度，減少延遲 | 當 NodeLocal DNSCache 出現問題時，所有解析都會失效 | 169.254.20.10 |

<br>

## 注意事項

1. 測試 KubeDNS、KubeDNS + NodeLocal DNSCache、Cloud DNS、Cloud DNS + NodeLocal DNSCache 使用 K6 簡單進行壓測，會發現除了 Cloud DNS 以外，其他的 DNS 解析 RPS 不一定會慢於直接打 IP
2. 如果有需求需要管理 KubeDNS Deployment 服務，可以參考：[自訂 kube-dns 部署作業](https://cloud.google.com/kubernetes-engine/docs/how-to/custom-kube-dns?hl=zh-tw)，但會有相對的維運成本 (因為官方預設的沒辦法調整 Deployment，會因為 addonmanager.kubernetes.io/mode=Reconcile 被 GKE 還原)
3. 所有只要經過 Cloud DNS 以及 Metadata Server (在 Node 上) 都會有 Cache 機制
4. 將 GKE 叢集的 DNS 服務從 KubeDNS + NodeLocal DNSCache 換成 Cloud DNS + NodeLocal DNSCache，且此過程會導致舊節點無法解析網域，造成服務中斷，因此建議在維護期間執行，可以參考：[從 kube-dns 遷移至 Cloud DNS 後，啟用 NodeLocal DNSCache 時 DNS 解析失敗](https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw#dns-fails-dnscache)
5. 在升級 GKE Cluster 時，因為會更新 KubeDNS，所以有可能會發生 DNS 解析失敗的情況，不管是否有開啟 NodeLocal DNSCache，都也可能發生 (通常只有一小部分節點會遇到 10%)，可以參考：[kube-dns 的效能限制](https://cloud.google.com/kubernetes-engine/docs/how-to/kube-dns?hl=zh-tw#performance_limitations_with_kube-dns)

<br>

{{< figure src="/gcp/gke-dns/1.webp" width="600" caption="" >}}

<br>

6. 將 GKE 叢集的 DNS 服務從 KubeDNS 換成 Cloud DNS VPC 範圍，必須在重建節點後才會生效，且此過程會導致舊節點無法解析網域，造成服務中斷，因此建議在維護期間執行，可以參考：[在現有叢集啟用 VPC 範圍 DNS](https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw#enable-vpc-scope-existing)

<br>

{{< figure src="/gcp/gke-dns/2.webp" width="600" caption="" >}}

<br>

7. 如果已經建立 GKE Cloud DNS VPC 範圍的 Cluster，是沒辦法切換回 KubeDNS，只能重建 Cluster，可以參考：[停用 Cloud DNS for GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw#disable-cloud-dns)

<br>

{{< figure src="/gcp/gke-dns/3.webp" width="600" caption="" >}}

<br>