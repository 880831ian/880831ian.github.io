---
title: "GKE Kube DNS + NodeLocalDNS 運作測試"
type: docs
weight: 9987
description: GKE Kube DNS + NodeLocalDNS 運作測試
images:
  - gcp/gke-kube-dns-nodelocaldns/og.webp
date: 2025-08-01
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

最近在評估要將公司內的 EndPoint 都改成 Cloud DNS 的 Private Zone，所以需要測試 GKE 內的 DNS 解析方案，避免發生 [Pod 出現 cURL error 6: Could not resolve host](../../kubernetes/pod-curl-error-6-could-not-resolve-host)，本次測試 Kube DNS + NodeLocalDNS 的運作。

<br>

先建立一個 dns-test pod 以及 nginx 的 pod + svc，會分別測試

1. 叢集內部 cluster.local (nginx-svc.default.svc.cluster.local)

2.  internal-dns (cloud dns private ：aaa.test-audit.com.)

3. 外部 dns ([ifconfig.me](https://ifconfig.me))

並使用腳本進行確認回傳 DNS 解析，每一次測試都會重新建立 kube-dns、node-local-dns Pod

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

## 叢集內部 cluster.local

Prometheus 監控設定參數：`zones="cluster.local."`

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/1.webp" width="750" caption="" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
coredns_cache_hits_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
coredns_cache_requests_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
coredns_cache_entries{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
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
EXPECTED_IP="10.36.17.66"
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

需要先確認 `nginx-svc` 的 IP 是多少，然後修改腳本中的 `EXPECTED_IP` 變數。

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/4.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] 可以觀察 node-local-dns hit 跟 request 都有持續上升

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/5.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/6.webp" width="500" caption="因為 cache 失效出現解析失敗" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/7.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]<br>
大約在 2000 筆請求時左右將 kube-dns 關成 0 顆，但到了 6104 筆的時候才開始出現解析失敗，代表中間因爲有 node-local-dns cache，所以 dns 解析還有相關紀錄可以回覆，但後面當 cache TTL 到期後，需要先訪問 kube-dns 時，就會出現解析錯誤
<br>
<br>
從 Prometheus 可以發現，前面 node-local-dns request 跟 hit 差不多，但當 18:38 線圖開始 request 大於 hit，這代表因為後面的 kube-dns 異常，導致 node-local-dns 沒辦法做 cache hit

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/8.webp" width="900" caption="資源監控" >}}

<br>

### 模擬 node-local-dns pod 異常

接下來我們測試最後一個情境，故意用壞 node-local-dns 服務，觀察此模式對於 node-local-dns 的依賴

測試情境：

先調整 COUNT 參數變成 50000

先跑 15000 筆有 node-local-dns，然後使用以下指令讓 node-local-dns 無法使用

```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":{"this-label-does-not-exist-on-any-node":"true"}}}}}'
```

等待約 30000 筆時，使用以下指令恢復 node-local-dns
```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":null}}}}'
```

<br>

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/9.webp" width="450" caption="測試結果" >}}

<br>

分別在 16:26:03 跟 16:28:45 下指令調整


{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/10.webp" width="800" caption="模擬 node-local-dns 故障" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/11.webp" width="500" caption="觀察調整讓 node-local-dns 故障 Log" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/12.webp" width="500" caption="觀察修正 node-local-dns Log" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/cluster-dns/13.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]<br>
發現當 node-local-dns 掛了後，會短暫卡住，但會直接切換到 kube-dns 上繼續進行解析，因此以結果論，如果 node-local-dns 有短暫異常，不會出現無法解析的問題
<br>
<br>
從 Prometheus 可以發現，前面 node-local-dns 正常運作，當我們在 16:26:03 調整後，變成 kube-dns 起來開始處理解析，在 16:28:45 切換讓 node-local-dns 恢復，後面又會變成由 node-local-dns 來處理解析
<br>
<br>
綠色是 node-local-dns，紫色是 kube-dns

<br>

## internal-dns (cloud dns private)

先建立一個 cloud dns private，以下範例是 aaa.test-audit.com > 10.1.1.4，並將此 use VPC 與 GKE 的 VPC 打通

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/1.webp" width="600" caption="設定 cloud dns private" >}}

<br>

Prometheus 監控設定參數：`zones="."`

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/2.webp" width="750" caption="" >}}

<br>

```shell
coredns_cache_hits_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="."}
coredns_cache_requests_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="."}
coredns_cache_entries{job="kube-dns-nodelocaldns-nodelocaldns", zones="."}
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

<br>

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/3.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/4.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/5.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] 可以觀察 node-local-dns hit 跟 request 都有持續上升

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/6.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/7.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/8.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP]<br>
觀察發現，因為 cloud dns private 不是 .cluster.local，所以就算沒有 kube-dns 也能正常運作，後續就沒有把 kube-dns 開回去 (只有 node-local-dns)

<br>

### 模擬 node-local-dns pod 異常

接下來我們測試最後一個情境，故意用壞 node-local-dns 服務，觀察此模式對於 node-local-dns 的依賴

測試情境：

先調整 COUNT 參數變成 50000

先跑 15000 筆有 node-local-dns，然後使用以下指令讓 node-local-dns 無法使用

```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":{"this-label-does-not-exist-on-any-node":"true"}}}}}'
```

等待約 30000 筆時，使用以下指令恢復 node-local-dns
```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":null}}}}'
```

<br>

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/9.webp" width="450" caption="測試結果" >}}

<br>

分別在 17:54:11 跟 17:57:23 下指令調整

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/10.webp" width="800" caption="模擬 node-local-dns 故障" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/11.webp" width="500" caption="觀察調整讓 node-local-dns 故障 Log，卡了 3 秒" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/12.webp" width="500" caption="觀察修正 node-local-dns Log，卡了 2 秒" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/internal-dns/13.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]<br>
發現當 node-local-dns 掛了後，會短暫卡住，但會直接切換到 kube-dns 上繼續進行解析，因此以結果論，如果 node-local-dns 有短暫異常，不會出現無法解析的問題
<br>
<br>
從 Prometheus 可以發現，前面 node-local-dns 正常運作，當我們在 17:54:11 調整後，變成 kube-dns 起來開始處理解析，在 17:57:23 切換讓 node-local-dns 恢復，後面又會變成由 node-local-dns 來處理解析
<br>
<br>
黃色是 node-local-dns，紫色是 kube-dns （後面黃色是因為手動來不及把新的 pod forward metrics 導致前面沒有線圖）

<br>

## 外部 DNS (ifconfig.me)

Prometheus 監控設定參數：`zones="."`

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/1.webp" width="750" caption="" >}}

<br>

- 相關 Prometheus 監控指標：

```shell
coredns_cache_hits_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
coredns_cache_requests_total{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
coredns_cache_entries{job="kube-dns-nodelocaldns-nodelocaldns", zones="cluster.local."}
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

### 測試腳本

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/2.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/3.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/4.webp" width="900" caption="資源監控" >}}

<br>

> [!TIP] 可以觀察 node-local-dns hit 跟 request 都有持續上升

<br>

### 模擬 kube-dns Pod 異常

接下來會在測試中，將 kube-dns 調整到 0 顆，再開回去，觀察此模式對於 kube-dns 的依賴

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/5.webp" width="450" caption="測試結果" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/6.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/7.webp" width="900" caption="資源監控" >}}


<br>

> [!TIP]<br>
觀察發現，因為外部 dns 不是 .cluster.local，所以就算沒有 kube-dns 也能正常運作，後續就沒有把 kube-dns 開回去 (只有 node-local-dns)

<br>

### 模擬 node-local-dns pod 異常

接下來我們測試最後一個情境，故意用壞 node-local-dns 服務，觀察此模式對於 node-local-dns 的依賴

測試情境：

先調整 COUNT 參數變成 50000

先跑 15000 筆有 node-local-dns，然後使用以下指令讓 node-local-dns 無法使用

```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":{"this-label-does-not-exist-on-any-node":"true"}}}}}'
```

等待約 30000 筆時，使用以下指令恢復 node-local-dns
```shell
kubectl patch daemonset node-local-dns -n kube-system --type='strategic' -p '{"spec":{"template":{"spec":{"nodeSelector":null}}}}'
```

<br>

測試結果：

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/8.webp" width="450" caption="測試結果" >}}

<br>

分別在 12:13:30 跟 12:15:48 下指令調整

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/9.webp" width="800" caption="模擬 node-local-dns 故障" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/10.webp" width="500" caption="觀察調整讓 node-local-dns 故障 Log，卡了 2 秒" >}}

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/external-dns/11.webp" width="1200" caption="Prometheus 監控指標" >}}

<br>

> [!TIP]<br>
發現當 node-local-dns 掛了後，會短暫卡住，但會直接切換到 kube-dns 上繼續進行解析，因此以結果論，如果 node-local-dns 有短暫異常，不會出現無法解析的問題
<br>
<br>
從 Prometheus 可以發現，前面 node-local-dns 正常運作，當我們在 12:13:30 調整後，變成 kube-dns 起來開始處理解析，在 12:15:48 切換讓 node-local-dns 恢復，後面又會變成由 node-local-dns 來處理解析
<br>
<br>
黃色是 node-local-dns，紫色是 kube-dns

<br>

## k6 測試

額外在另一個 cluster 建立 nginx deployment 開 5 個 pod 以及 svc 改成 lb (L4)，然後在 cloud dns 的 test-audit-com 設定 nginx-lb-internal.test-audit.com 解析到內網的 svc (10.156.17.230)

<br>

使用 k6 測試 kube dns + node-local-dns 模式下 IP 跟 DNS 的差異

相關程式可以參考：[https://github.com/880831ian/gke-dns](https://github.com/880831ian/gke-dns)

<br>

> [!TIP]理論上 ip 應該會比 dns 還要快，但測試 4 次發現其實不一定

<br>

### 第一次測試

IP (avg=181.7ms / 3511 RPS)、DNS (avg=335.43ms / 2278 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/1.webp" width="1000" caption="IP 第一次測試" >}}

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/2.webp" width="1000" caption="DNS 第一次測試" >}}

<br>

### 第二次測試

IP (avg=336.98ms / 2272 RPS)、DNS (avg=503.79ms / 1640 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/3.webp" width="1000" caption="IP 第二次測試" >}}

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/4.webp" width="1000" caption="DNS 第二次測試" >}}

<br>

### 第三次測試

IP (avg=128.9ms / 4310 RPS)、DNS (avg=129.23ms / 4310 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/5.webp" width="1000" caption="IP 第三次測試" >}}

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/6.webp" width="1000" caption="DNS 第三次測試" >}}

<br>

### 第四次測試

IP (avg=149.5ms / 3965 RPS)、DNS (avg=133.73ms / 4230 RPS)

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/7.webp" width="1000" caption="IP 第四次測試" >}}

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/k6/8.webp" width="1000" caption="DNS 第四次測試" >}}

<br>

## 結論

可以發現，使用 node-local-dns 的模式下，對於 kube-dns 的依賴性降低了很多，因為 node-local-dns 會先做 cache hit，這樣就算 kube-dns 異常（只有 cluster 內的，且 cache 失效才會影響），也不會影響到 pod 的 DNS 解析。

<br>

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/nodelocal-dns-cache-diagram.webp" width="900" caption="kube DNS + Nodelocaldn 架構圖<br>[https://cloud.google.com/kubernetes-engine/docs/how-to/nodelocal-dns-cache?hl=zh-tw#architecture](https://cloud.google.com/kubernetes-engine/docs/how-to/nodelocal-dns-cache?hl=zh-tw#architecture)" >}}

<br>

但是這個結果其實是因為 GKE 在這個模式下，共用 kube-dns IP，並調整 iptables 的方式來實現的。所以跟一般的 node-local-dns 邏輯，會有點不同。

{{< figure src="/gcp/gke-kube-dns-nodelocaldns/nodelocaldns.webp" width="900" caption="Using NodeLocal DNSCache in Kubernetes Clusters<br>[https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/#architecture-diagram](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/#architecture-diagram)" >}}

<br>

## 參考資料

設定 NodeLocal DNSCache：[https://cloud.google.com/kubernetes-engine/docs/how-to/nodelocal-dns-cache?hl=zh-tw](https://cloud.google.com/kubernetes-engine/docs/how-to/nodelocal-dns-cache?hl=zh-tw)