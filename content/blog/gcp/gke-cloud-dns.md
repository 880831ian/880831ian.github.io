---
title: "GKE DNS 使用 Cloud DNS 運作測試"
type: docs
weight: 9986
description: GKE DNS 使用 Cloud DNS 運作測試
images:
  - gcp/gke-cloud-dns/og.webp
date: 2025-08-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Google Cloud Platform
  - GCP
  - Kubernetes
  - K8s
  - DNS
  - Cloud DNS
---

最近在評估要將公司內的 EndPoint 都改成 Cloud DNS 的 Private Zone (打造內部的 internal dns 服務機制)，到時候 DNS 解析的請求會比以往還要多，所以需要先測試評估 GKE 內的 DNS 解析方案，避免再次發生 [Pod 出現 cURL error 6: Could not resolve host](../../kubernetes/pod-curl-error-6-could-not-resolve-host)，此篇文章測試的是： Cloud DNS 的運作。

<br>

首先，先建立一個 dns-test pod [程式連結](https://github.com/880831ian/gke-dns/blob/main/dns-test.yaml) 以及 nginx 的 pod + svc [程式連結](https://github.com/880831ian/gke-dns/blob/main/nginx.yaml)，會分別測試

1. [叢集內部 cluster.local](#叢集內部-clusterlocal) (nginx-svc.default.svc.cluster.local)

2. [internal-dns 使用 cloud dns private](#internal-dns-cloud-dns-private) (aaa.test-audit.com)

3. [外部 dns](#外部-dns-ifconfigme) (ifconfig.me)

並使用 nslookup 腳本進行確認回傳 DNS 解析，每一次測試都會重新建立 KubeDNS Pod

相關程式以及 Prometheus、Grafana 的設定可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

在建立完 Cluster 後，可以先觀察在 Cloud DNS 是否新增的 Private Zone

<br>

{{< figure src="/gcp/gke-cloud-dns/cloud-dns-private.webp" width="850" caption="" >}}

<br>

由於裡面有很多的 Record，我們先搜尋下面會用到的 nginx-svc.default.svc.cluster.local 來當作範例

<br>

## 叢集內部 cluster.local

一樣會觀察 KubeDNS Pod Metrics，因為就算開啟 Cloud DNS 後，KubeDNS 還是會留著

<br>

{{< figure src="/gcp/gke-cloud-dns/cluster-dns/1.webp" width="750" caption="https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw#architecture" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
kubedns_dnsmasq_hits{job="kubedns-dns"}
kubedns_dnsmasq_misses{job="kubedns-dns"}
```

<br>

測試腳本：
```shell
#!/bin/bash

get_taiwan_time() {
  # 獲取當前 UTC 時間的 Unix 時間戳
  UTC_TIMESTAMP=$(date -u +%s)
  # 加上 8 小時 (台灣時間是 UTC+8)
  TAIPEI_TIMESTAMP=$((UTC_TIMESTAMP + 28800))
  # 將時間戳轉換為格式化後的日期時間
  date -d "@$TAIPEI_TIMESTAMP" "+%Y-%m-%d %H:%M:%S"
}

DOMAIN="nginx-svc.default.svc.cluster.local"
EXPECTED_IP="10.36.16.243"
START_TIME=$(get_taiwan_time)
COUNT=10000
SUCCESS_COUNT=0
FAIL_COUNT=0

echo "== NSLOOKUP TEST START: $START_TIME ==" | tee -a nslookup_full.log

for i in $(seq 1 "$COUNT"); do
  OUTPUT=$(nslookup "$DOMAIN" 2>&1)
  ADDR_LINE=$(echo "$OUTPUT" | grep -E '^Address:')
  if [[ "$ADDR_LINE" == *"$EXPECTED_IP"* ]]; then
    SUCCESS_COUNT=$((SUCCESS_COUNT + 1))
  else
    FAIL_COUNT=$((FAIL_COUNT + 1))
  fi

  echo "[$i] === $(get_taiwan_time) 成功: $SUCCESS_COUNT 失敗: $FAIL_COUNT"
done

END_TIME=$(get_taiwan_time)

echo "== NSLOOKUP TEST END: $END_TIME ==" | tee -a nslookup_full.log
echo "成功次數: $SUCCESS_COUNT" | tee -a nslookup_full.log
echo "失敗次數: $FAIL_COUNT" | tee -a nslookup_full.log
```
10.36.16.243 是 nginx-svc Cluster IP

需要先確認 `nginx-svc` 的 IP 是多少，然後修改腳本中的 `EXPECTED_IP` 變數。

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/cluster-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/cluster-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP] 結論
因為改用 Cloud DNS，所以 KubeDNS 的 hit 沒有往上衝高

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/cluster-dns/4.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/cluster-dns/5.webp" width="1200" caption="Prometheus 監控指標" >}}


<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，可以發現解析還是正常，代表使用 Cloud DNS 後，KubeDNS 對於叢集內部解析不會有影響

<br>

## Internal DNS (Cloud DNS Private)

先建立一個 cloud dns private，以下範例是 aaa.test-audit.com > 10.1.1.4，並將此 use VPC 與 GKE 的 VPC 打通

<br>

{{< figure src="/gcp/gke-cloud-dns/internal-dns/1.webp" width="600" caption="設定 cloud dns private" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
kubedns_dnsmasq_hits{job="kubedns-dns"}
kubedns_dnsmasq_misses{job="kubedns-dns"}
```

<br>

測試腳本：

```shell
#!/bin/bash

get_taiwan_time() {
  # 獲取當前 UTC 時間的 Unix 時間戳
  UTC_TIMESTAMP=$(date -u +%s)
  # 加上 8 小時 (台灣時間是 UTC+8)
  TAIPEI_TIMESTAMP=$((UTC_TIMESTAMP + 28800))
  # 將時間戳轉換為格式化後的日期時間
  date -d "@$TAIPEI_TIMESTAMP" "+%Y-%m-%d %H:%M:%S"
}

DOMAIN="aaa.test-audit.com"
EXPECTED_IP="10.1.1.4"
START_TIME=$(get_taiwan_time)
COUNT=10000
SUCCESS_COUNT=0
FAIL_COUNT=0

echo "== NSLOOKUP TEST START: $START_TIME ==" | tee -a nslookup_full.log

for i in $(seq 1 "$COUNT"); do
  OUTPUT=$(nslookup "$DOMAIN" 2>&1)
  ADDR_LINE=$(echo "$OUTPUT" | grep -E '^Address:')
  if [[ "$ADDR_LINE" == *"$EXPECTED_IP"* ]]; then
    SUCCESS_COUNT=$((SUCCESS_COUNT + 1))
  else
    FAIL_COUNT=$((FAIL_COUNT + 1))
  fi

  echo "[$i] === $(get_taiwan_time) 成功: $SUCCESS_COUNT 失敗: $FAIL_COUNT"
done

END_TIME=$(get_taiwan_time)

echo "== NSLOOKUP TEST END: $END_TIME ==" | tee -a nslookup_full.log
echo "成功次數: $SUCCESS_COUNT" | tee -a nslookup_full.log
echo "失敗次數: $FAIL_COUNT" | tee -a nslookup_full.log
```
10.1.1.4 是隨機亂取的 IP，只是為了確認 domain 是否能夠正常解析

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/internal-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/internal-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
因為改用 Cloud DNS，所以 KubeDNS 的 hit 沒有往上衝高

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/internal-dns/4.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/internal-dns/5.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，可以發現解析還是正常，代表使用 Cloud DNS 後，KubeDNS 對於 internal-dns 解析不會有影響

<br>

## 外部 DNS (ifconfig.me)

<br>

- 相關 Prometheus 監控指標：

```shell
kubedns_dnsmasq_hits{job="kubedns-dns"}
kubedns_dnsmasq_misses{job="kubedns-dns"}
```

<br>

測試腳本：
```shell
#!/bin/bash

get_taiwan_time() {
  # 獲取當前 UTC 時間的 Unix 時間戳
  UTC_TIMESTAMP=$(date -u +%s)
  # 加上 8 小時 (台灣時間是 UTC+8)
  TAIPEI_TIMESTAMP=$((UTC_TIMESTAMP + 28800))
  # 將時間戳轉換為格式化後的日期時間
  date -d "@$TAIPEI_TIMESTAMP" "+%Y-%m-%d %H:%M:%S"
}

DOMAIN="ifconfig.me"
EXPECTED_IP="34.160.111.145"
START_TIME=$(get_taiwan_time)
COUNT=10000
SUCCESS_COUNT=0
FAIL_COUNT=0

echo "== NSLOOKUP TEST START: $START_TIME ==" | tee -a nslookup_full.log

for i in $(seq 1 "$COUNT"); do
  OUTPUT=$(nslookup "$DOMAIN" 2>&1)
  ADDR_LINE=$(echo "$OUTPUT" | grep -E '^Address:')
  if [[ "$ADDR_LINE" == *"$EXPECTED_IP"* ]]; then
    SUCCESS_COUNT=$((SUCCESS_COUNT + 1))
  else
    FAIL_COUNT=$((FAIL_COUNT + 1))
  fi

  echo "[$i] === $(get_taiwan_time) 成功: $SUCCESS_COUNT 失敗: $FAIL_COUNT"
done

END_TIME=$(get_taiwan_time)

echo "== NSLOOKUP TEST END: $END_TIME ==" | tee -a nslookup_full.log
echo "成功次數: $SUCCESS_COUNT" | tee -a nslookup_full.log
echo "失敗次數: $FAIL_COUNT" | tee -a nslookup_full.log
```
34.160.111.145 是 ifconfig.me 的 IP，只是為了確認 domain 是否能夠正常解析

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/external-dns/1.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/external-dns/2.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
因為改用 Cloud DNS，所以 KubeDNS 的 hit 沒有往上衝高

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-cloud-dns/external-dns/3.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/external-dns/4.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，可以發現解析還是正常，代表使用 Cloud DNS 後，KubeDNS 對於外部 DNS解析不會有影響

<br>

## k6 測試

額外在另一個 cluster 建立 nginx deployment 開 5 個 pod 以及 svc 改成 lb (L4)，然後在 cloud dns 的 test-audit-com 設定 nginx-lb-internal.test-audit.com 解析到內網的 svc (10.156.16.16)

<br>

使用 k6 測試 kube dns 模式下 IP 跟 DNS 的差異

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

這邊測試的 Node 是用 e2-medium 而非 c3d-standard-4

<br>

### 第一次測試

IP (avg=179.06ms / 3479 RPS)、DNS (avg=191.56ms / 3348 RPS)

<br>

{{< figure src="/gcp/gke-cloud-dns/k6/1.webp" width="1000" caption="IP 第一次測試" >}}

{{< figure src="/gcp/gke-cloud-dns/k6/2.webp" width="1000" caption="DNS 第一次測試" >}}

<br>

### 第二次測試

IP (avg=193.67ms / 3322 RPS)、DNS (avg=302.73ms / 2430 RPS)

<br>

{{< figure src="/gcp/gke-cloud-dns/k6/3.webp" width="1000" caption="IP 第二次測試" >}}

{{< figure src="/gcp/gke-cloud-dns/k6/4.webp" width="1000" caption="DNS 第二次測試" >}}

<br>

### 第三次測試

IP (avg=174.94ms / 3529 RPS)、DNS (avg=253.71ms / 2761 RPS)

<br>

{{< figure src="/gcp/gke-cloud-dns/k6/5.webp" width="1000" caption="IP 第三次測試" >}}

{{< figure src="/gcp/gke-cloud-dns/k6/6.webp" width="1000" caption="DNS 第三次測試" >}}

<br>

### 第四次測試

IP (avg=227.56ms / 2971 RPS)、DNS (avg=316.07ms / 2361 RPS)

<br>

{{< figure src="/gcp/gke-cloud-dns/k6/7.webp" width="1000" caption="IP 第四次測試" >}}

{{< figure src="/gcp/gke-cloud-dns/k6/8.webp" width="1000" caption="DNS 第四次測試" >}}

<br>

> [!TIP]結論
有發現之前在 KubeDNS 跟 KubeDNS + NodeLocal DNSCache 會有 DNS RPS 大於 IP 的情況，但使用 Cloud DNS 後則沒有發生，推測是因為。Cloud DNS 在 Cluster 外部，所以會比較慢

<br>

## 結論

可以發現，使用 Cloud DNS 的模式下，所有的 DNS 解析都會依靠 Cloud DNS，不會使用到 KubeDNS，所以可以把 KubeDNS 關成 0 顆，避免浪費資源。

<br>

{{< figure src="/gcp/gke-cloud-dns/gke-cloud-dns-architecture.webp" width="700" caption="GKE Cloud DNS" >}}

<br>

由於官方流程圖有些細節沒有揭露的完成，我們有額外詢問 Google TAM，並畫出以下流程圖，Google TAM 也確認流程正確

<br>

{{< figure src="/gcp/gke-cloud-dns/gke-cloud-dns-architecture-flow.webp" width="1200" caption="自己重新畫的 GKE Cloud DNS" >}}

<br>

以下是我們與 Google 討論的內容：

我們：

1. 圖中粉紅框所選取的兩個 Cloud DNS<Cloud DNS & Cloud DNS (Data Plane)> 在實際的架構中，是兩個獨立的物件嗎？或者其實是同一個物件?

2. 承(1)，以下項目所述的理解是否正確？若有差異的部分請協助說明：目前我們的認知是，在 Internal DNS 上會有三個 Cloud DNS：
    1. GKE 的設定改用 Cloud DNS 的 provider，此時系統會協助我們建立一個 Cloud DNS (Data Plane)。
    2. 我們會自己建立一個 Cloud DNS(Private) For 我們內部自己要用的 Internal DNS。
    3. 進行外部 DNS 解析時，目前的理解是會從 metadata server 對 Cloud DNS 轉送解析需求，是否屬實？

3. 完成解析後，Cache 的部分是否在 metadata server 裡面進行？


Google：

1. 處理 private zone 和公有網域名稱的是不同物件，但都在 GCE DNS 服務裡

2. 1. 主要是建立 cluster-scoped private zone，進到 metadata server 後的行為與原本相同、2 跟 3 正確

3. 是，除了metadata server 外，同時也會在 GCE DNS 服務裡快取

<br>

另外也可以從官方文件中得知 KubeDNS 的 `/etc/resolv.conf` 設定值，可以知道 Pod 最開始使用的 DNS 解析 Server 是誰。

{{< figure src="/gcp/gke-cloud-dns/resolv.webp" width="800" caption="/etc/resolv.conf 設定值 [服務探索和 DNS /etc/resolv.conf](https://cloud.google.com/kubernetes-engine/docs/concepts/service-discovery?hl=zh-tw#conf_value)" >}}

<br>

{{< figure src="/gcp/gke-cloud-dns/config.webp" width="800" caption="實際查看 /etc/resolv.conf" >}}

<br>

## 參考資料

使用 GKE 適用的 Cloud DNS：[https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw](https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-dns?hl=zh-tw)