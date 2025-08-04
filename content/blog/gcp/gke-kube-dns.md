---
title: "GKE Kube DNS 運作測試"
type: docs
weight: 9987
description: GKE Kube DNS 運作測試
images:
  - gcp/gke-kube-dns/og.webp
date: 2025-08-04
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

最近在評估要將公司內的 EndPoint 都改成 Cloud DNS 的 Private Zone，所以需要測試 GKE 內的 DNS 解析方案，避免發生 [Pod 出現 cURL error 6: Could not resolve host](../../kubernetes/pod-curl-error-6-could-not-resolve-host)，本次測試 Kube DNS 的運作。

<br>

先建立一個 dns-test pod 以及 nginx 的 pod + svc，會分別測試

1. 叢集內部 cluster.local (nginx-svc.default.svc.cluster.local)

2.  internal-dns (cloud dns private ：aaa.test-audit.com.)

3. 外部 dns ([ifconfig.me](https://ifconfig.me))

並使用腳本進行確認回傳 DNS 解析，每一次測試都會重新建立 kube-dns、node-local-dns Pod

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

## 叢集內部 cluster.local

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/1.webp" width="750" caption="" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
kubedns_dnsmasq_hits{job="kube-dns-nodelocaldns-kube-dns"}
kubedns_dnsmasq_misses{job="kube-dns-nodelocaldns-kube-dns"}
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
EXPECTED_IP="10.36.17.113"
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
10.36.17.113 是 nginx-svc Cluster IP

需要先確認 `nginx-svc` 的 IP 是多少，然後修改腳本中的 `EXPECTED_IP` 變數。

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/4.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] 可以觀察 kubedns_dnsmasq_hits hit 有持續上升

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/5.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/6.webp" width="500" caption="kube-dns 關成 0 顆" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/7.webp" width="500" caption="因為 pod 不會馬上關掉，所以還能夠解析 DNS" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/8.webp" width="500" caption="切換回來就正常可以解析" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/9.webp" width="500" caption="進入下指令查看錯誤訊息" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/10.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]<br>
大約在 2000 筆請求時左右將 kube-dns 關成 0 顆，但到了 7234 筆的時候才開始出現解析失敗，觀察後發現，因為 kube-dns 切成 0，Pod 不會馬上關掉，所以還能夠解析 DNS，可以看到有額外測試，最後再將 kube-dns 切成 1 顆後，就正常可以解析了
<br>
<br>
從 Prometheus 可以發現，kubedns_dnsmasq_hits 在 15:48 開始飆高，與 2000 筆切換時間差不多，後面就因為 kube-dns 關掉，導致沒有 metrics

<br>

{{< figure src="/gcp/gke-kube-dns/cluster-dns/11.webp" width="900" caption="資源監控" >}}

<br>

## internal-dns (cloud dns private)

先建立一個 cloud dns private，以下範例是 aaa.test-audit.com > 10.1.1.4，並將此 use VPC 與 GKE 的 VPC 打通

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/1.webp" width="600" caption="設定 cloud dns private" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/2.webp" width="750" caption="" >}}

<br>

```shell
kubedns_dnsmasq_hits{job="kube-dns-nodelocaldns-kube-dns"}
kubedns_dnsmasq_misses{job="kube-dns-nodelocaldns-kube-dns"}
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

{{< figure src="/gcp/gke-kube-dns/internal-dns/3.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/4.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/5.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] 可以觀察 kubedns_dnsmasq_hits hit 有持續上升

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/6.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/7.webp" width="450" caption="關閉 pod 時間" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/8.webp" width="450" caption="因為 pod 關閉開始失敗" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/9.webp" width="450" caption="解析恢復" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/10.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/internal-dns/11.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] <br>
大約在 2000 筆請求時左右將 kube-dns 關成 0 顆，但到了 4847 筆的時候才開始出現解析失敗，觀察後發現，因為 kube-dns 切成 0，Pod 不會馬上關掉，所以還能夠解析 DNS， 最後再將 kube-dns 切成 1 顆後，就正常可以解析了

<br>

## 外部 DNS (ifconfig.me)

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/1.webp" width="750" caption="" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
kubedns_dnsmasq_hits{job="kube-dns-nodelocaldns-kube-dns"}
kubedns_dnsmasq_misses{job="kube-dns-nodelocaldns-kube-dns"}
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

{{< figure src="/gcp/gke-kube-dns/external-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/4.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] <br>
可以觀察 kubedns_dnsmasq_hits hit 有持續上升，且有比較特別的現象是 叢集內部 cluster.local  跟 internal-dns kubedns_dnsmasq_misses 會跟 kubedns_dnsmasq_hits 差不多，但外部 DNS卻不會 (但目前網路上沒有相關的 metrics 資訊)

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/5.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/6.webp" width="500" caption="強制刪除，所以馬上無法解析" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/7.webp" width="500" caption="因為是刪除，不是切成 0 顆，所以他馬上又長一顆出來，就恢復正常" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/8.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns/external-dns/9.webp" width="900" caption="資源監控" >}}


<br>

> [!TIP] <br>
大約在 2000 筆請求時左右將 kube-dns 關成 0 顆，這次強制刪除 pod，所以在 2110 筆的時候就會失敗，但因為這次不是將 pod 關成 0 顆，他會馬上再長一顆新的，所以約 2123 就正常了，中間時間大約是 7 秒左右，就正常可以解析了

<br>

## k6 測試

額外在另一個 cluster 建立 nginx deployment 開 5 個 pod 以及 svc 改成 lb (L4)，然後在 cloud dns 的 test-audit-com 設定 nginx-lb-internal.test-audit.com 解析到內網的 svc (10.156.16.5)

<br>

使用 k6 測試 kube dns 模式下 IP 跟 DNS 的差異

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

> [!TIP]理論上 ip 應該會比 dns 還要快，但測試 4 次發現其實不一定

<br>

### 第一次測試

IP (avg=137.2ms / 4080 RPS)、DNS (avg=140.4ms / 4024 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns/k6/1.webp" width="1000" caption="IP 第一次測試" >}}

{{< figure src="/gcp/gke-kube-dns/k6/2.webp" width="1000" caption="DNS 第一次測試" >}}

<br>

### 第二次測試

IP (avg=151.05ms / 3878 RPS)、DNS (avg=260.19ms / 2715 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns/k6/3.webp" width="1000" caption="IP 第二次測試" >}}

{{< figure src="/gcp/gke-kube-dns/k6/4.webp" width="1000" caption="DNS 第二次測試" >}}

<br>

### 第三次測試

IP (avg=150.84ms / 3858 RPS)、DNS (avg=155.8ms / 3806 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns/k6/5.webp" width="1000" caption="IP 第三次測試" >}}

{{< figure src="/gcp/gke-kube-dns/k6/6.webp" width="1000" caption="DNS 第三次測試" >}}

<br>

### 第四次測試

IP (avg=321.47ms / 2337 RPS)、DNS (avg=254.11ms / 2775 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns/k6/7.webp" width="1000" caption="IP 第四次測試" >}}

{{< figure src="/gcp/gke-kube-dns/k6/8.webp" width="1000" caption="DNS 第四次測試" >}}

<br>

## 結論

可以發現，使用 kube-dns 的模式下，對於 kube-dns 的依賴性極高，只要壞掉，不管任何的解析(叢集內、internal-dns、外部 dns)都會失效。