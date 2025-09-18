---
title: "GKE DNS 使用 KubeDNS 運作測試"
type: docs
weight: 9986
description: GKE DNS 使用 KubeDNS 運作測試
images:
  - gcp/gke-kubedns/og.webp
date: 2025-08-04
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
  - Google Cloud Platform
  - GCP
  - Kubernetes
  - K8s
  - DNS
  - KubeDNS
---

最近在評估要將公司內的 EndPoint 都改成 Cloud DNS 的 Private Zone (打造內部的 internal dns 服務機制)，到時候 DNS 解析的請求會比以往還要多，所以需要先測試評估 GKE 內的 DNS 解析方案，避免再次發生 [Pod 出現 cURL error 6: Could not resolve host](../../kubernetes/pod-curl-error-6-could-not-resolve-host)，此篇文章測試的是： KubeDNS 的運作。

<br>

首先，先建立一個 dns-test pod [程式連結](https://github.com/880831ian/gke-dns/blob/main/dns-test.yaml) 以及 nginx 的 pod + svc [程式連結](https://github.com/880831ian/gke-dns/blob/main/nginx.yaml)，會分別測試

1. [叢集內部 cluster.local](#叢集內部-clusterlocal) (nginx-svc.default.svc.cluster.local)

2. [internal-dns 使用 cloud dns private](#internal-dns-cloud-dns-private) (aaa.test-audit.com)

3. [外部 dns](#外部-dns-ifconfigme) (ifconfig.me)

並使用 nslookup 腳本進行確認回傳 DNS 解析，每一次測試都會重新建立 KubeDNS Pod

相關程式以及 Prometheus、Grafana 的設定可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

## 叢集內部 cluster.local

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
EXPECTED_IP="10.36.16.255"
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
10.36.16.255 是 nginx-svc 的 Cluster IP

需要先確認 `nginx-svc` 的 IP 是多少，然後修改腳本中的 `EXPECTED_IP` 變數。

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/1.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/2.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
可以觀察 kubedns_dnsmasq_hits hit 有持續上升

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/3.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/4.webp" width="500" caption="KubeDNS 關成 0 顆，過一陣子後開始噴錯" >}}

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/5.webp" width="700" caption="因為 KubeDNS Pod 不會馬上關掉，所以還能夠解析 DNS" >}}

<br>

{{< figure src="/gcp/gke-kubedns/cluster-dns/6.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，但到了 8733 筆的時候才開始出現解析失敗，觀察後發現，因為 KubeDNS 切成 0，Pod 不會馬上關掉，所以還能夠解析 DNS，最後再將 KubeDNS 切成 1 顆後，就正常可以解析了

<br>

## Internal DNS (Cloud DNS Private)

先建立一個 cloud dns private，以下範例是 aaa.test-audit.com > 10.1.1.4，並將此 use VPC 與 GKE 的 VPC 打通

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/1.webp" width="600" caption="設定 cloud dns private" >}}

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

{{< figure src="/gcp/gke-kubedns/internal-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
可以觀察 KubeDNS 內的指標 kubedns_dnsmasq_hits hit 有持續上升

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/4.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/5.webp" width="450" caption="KubeDNS 關成 0 顆，過一陣子後開始噴錯" >}}

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/6.webp" width="700" caption="因為 KubeDNS Pod 不會馬上關掉，所以還能夠解析 DNS" >}}

<br>

{{< figure src="/gcp/gke-kubedns/internal-dns/7.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，但到了 7130 筆的時候才開始出現解析失敗，觀察後發現，因為 KubeDNS 切成 0，Pod 不會馬上關掉，所以還能夠解析 DNS，最後再將 KubeDNS 切成 1 顆後，就正常可以解析了

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

{{< figure src="/gcp/gke-kubedns/external-dns/1.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/external-dns/2.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
可以觀察 KubeDNS 內的指標 kubedns_dnsmasq_hits hit 有持續上升，且有比較特別的現象是 叢集內部 cluster.local  跟 internal-dns kubedns_dnsmasq_misses 會跟 kubedns_dnsmasq_hits 差不多，但外部 DNS 卻不會
<br><br>
cluster 內 / cloud DNS private zone 流程會是
<br>
Pod → dnsmasq (miss) → kubedns 回答 → dnsmasq cache → 下次 hit
<br>
所以會是 N hit + N miss
<br><br>
外部 domain 流程會是
<br>
Pod → kubedns → upstream → dnsmasq 只 cache 回應，不算 miss
<br>
加上 dnsmasq 本身會為不同類型的查詢（例如 A/AAAA 或 CNAME chain）各記一次 hit
<br>
所以會是 2N hit，0 miss

<br>

### 模擬 KubeDNS Pod 異常

接下來會在測試中，將 KubeDNS 調整到 0 顆，再開回去，觀察此模式對於 KubeDNS 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kubedns/external-dns/3.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kubedns/external-dns/4.webp" width="500" caption="KubeDNS 關成 0 顆，過一陣子後開始噴錯" >}}

<br>

{{< figure src="/gcp/gke-kubedns/external-dns/5.webp" width="700" caption="因為 KubeDNS Pod 不會馬上關掉，所以還能夠解析 DNS" >}}

<br>

{{< figure src="/gcp/gke-kubedns/external-dns/6.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]結論
大約在 2000 筆請求時左右將 KubeDNS 關成 0 顆，但到了 8934 筆的時候才開始出現解析失敗，觀察後發現，因為 KubeDNS 切成 0，Pod 不會馬上關掉，所以還能夠解析 DNS，最後再將 KubeDNS 切成 1 顆後，就正常可以解析了

<br>

## k6 測試

額外在另一個 cluster 建立 nginx deployment 開 5 個 pod 以及 svc 改成 lb (L4)，然後在 cloud dns 的 test-audit-com 設定 nginx-lb-internal.test-audit.com 解析到內網的 svc (10.156.16.5)

<br>

使用 k6 測試 kube dns 模式下 IP 跟 DNS 的差異

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

這邊測試的 Node 是用 e2-medium 而非 c3d-standard-4

<br>

### 第一次測試

IP (avg=137.2ms / 4080 RPS)、DNS (avg=140.4ms / 4024 RPS)

<br>

{{< figure src="/gcp/gke-kubedns/k6/1.webp" width="1000" caption="IP 第一次測試" >}}

{{< figure src="/gcp/gke-kubedns/k6/2.webp" width="1000" caption="DNS 第一次測試" >}}

<br>

### 第二次測試

IP (avg=151.05ms / 3878 RPS)、DNS (avg=260.19ms / 2715 RPS)

<br>

{{< figure src="/gcp/gke-kubedns/k6/3.webp" width="1000" caption="IP 第二次測試" >}}

{{< figure src="/gcp/gke-kubedns/k6/4.webp" width="1000" caption="DNS 第二次測試" >}}

<br>

### 第三次測試

IP (avg=150.84ms / 3858 RPS)、DNS (avg=155.8ms / 3806 RPS)

<br>

{{< figure src="/gcp/gke-kubedns/k6/5.webp" width="1000" caption="IP 第三次測試" >}}

{{< figure src="/gcp/gke-kubedns/k6/6.webp" width="1000" caption="DNS 第三次測試" >}}

<br>

### 第四次測試

IP (avg=321.47ms / 2337 RPS)、DNS (avg=254.11ms / 2775 RPS)

<br>

{{< figure src="/gcp/gke-kubedns/k6/7.webp" width="1000" caption="IP 第四次測試" >}}

{{< figure src="/gcp/gke-kubedns/k6/8.webp" width="1000" caption="DNS 第四次測試" >}}

<br>

> [!TIP]結論
理論上 ip 應該會比 dns 還要快，但測試 4 次發現其實不一定

<br>

## 結論

可以發現，使用 KubeDNS 的模式下，對於 KubeDNS 的依賴性極高，只要壞掉，不管任何的解析(叢集內、internal-dns、外部 dns)都會失效。

<br>

{{< figure src="/gcp/gke-kubedns/kubeDNS.webp" width="700" caption="GKE KubeDNS 流程圖" >}}

<br>

由於官方流程圖有些細節沒有揭露的完成，我們有額外詢問 Google TAM，並畫出以下流程圖，Google TAM 也確認流程正確

{{< figure src="/gcp/gke-kubedns/kubeDNS-flow.webp" width="1200" caption="自己重新畫的 GKE KubeDNS 流程圖" >}}

<br>

以下是我們與 Google 討論的內容：

我們：

圖中的 metadata server 是畫在 GKE Cluster 層內，與 Kube DNS 上游伺服器 (kube-dns svc) 以及 Kube DNS 在同一層。而 Kube DNS + NodeLocal DNSCache 版本架構圖中的 metadata server 則是被標示在Worker Node 裡面，這兩者實質上歸屬與位置皆不同是正確的嗎？為何會這樣？

<br>

Google：

1. metadata server 是存在每一台 VM 上面

2. 如果是 Kube DNS，請求流向是 application pod -> kube-dns svc -> 隨機 kube-dns pod，如果 kube-dns 沒答案，跳轉該 kube-dns pod 上面的 VM/worker node 的 metadata server

3. (情境2)如果是 node-local-dns，請求流向會是 application pod -> node-local-dns pod
    1. 如果是 cluster.local 就會轉發到 kube-dns
    2. 如果有設定 stubDomain 就轉發到設定的 DNS
    3. 如果是其他的則轉發這台 VM/pod 的 metadata server


--

我們補充詢問：

附上的圖是 KubeDNS 的請求解析圖，我們知道 Metadata Server 是在每一個 VM (Worker Node 上)，只是圖上範例是將 Metadata Server 放到 GKE Cluster 的 Scope 內，而不是在一個 Worker Node，因為情境2. 的 KubeDNS + NodeLocal DNSCache 是將 Metadata Server 放到 Worker Node 裡面，而不是 GKE Cluster，想確認是不是 KubeDNS 請求解析圖在這一塊比較沒有畫清楚呢？

<br>
Google 回覆：

KubeDNS Only：所有請求都會先到 kube-dns pod，如果是 cluster 之外的會轉發到 kube-dns pod 上面的 VM/worker node 的 metadata server，不是每一台 worker。

KubeDNS + node-local-dns 如果不是 cluster.svc.local 就會直接轉發到該台 worker 的 metadata 了

因此用到的 Metadata server 的 worker 不太一樣，一個是指會用到 kube-dns pod 上面的 VM，一個是全部 VM/worker 都用到

同意您說的 "圖上範例是將 Metadata Server 放到 GKE Cluster 的 Scope 內" 可能會有點誤導

您畫的圖示意是正確的

--

<br>

另外也可以從官方文件中得知 KubeDNS 的 `/etc/resolv.conf` 設定值，可以知道 Pod 最開始使用的 DNS 解析 Server 是誰。

<br>

{{< figure src="/gcp/gke-kubedns/resolv.webp" width="800" caption="/etc/resolv.conf 設定值 [服務探索和 DNS /etc/resolv.conf](https://cloud.google.com/kubernetes-engine/docs/concepts/service-discovery?hl=zh-tw#conf_value)" >}}

<br>

{{< figure src="/gcp/gke-kubedns/config.webp" width="800" caption="實際查看 /etc/resolv.conf" >}}

<br>

## 參考資料

使用 KubeDNS：[https://cloud.google.com/kubernetes-engine/docs/how-to/kube-dns?hl=zh-tw](https://cloud.google.com/kubernetes-engine/docs/how-to/kube-dns?hl=zh-tw)