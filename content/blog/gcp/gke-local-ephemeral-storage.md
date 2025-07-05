---
title: "Google Kubernetes Engine Local ephemeral storage 計算方式"
type: docs
weight: 9990
description: Google Kubernetes Engine Local ephemeral storage 計算方式
images:
  - gcp/gke-local-ephemeral-storage/og.webp
date: 2024-09-02
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

此文章主要針對 Google Kubernetes Engine Local ephemeral storage 的計算方式來做介紹。

<br>

## 前情提要

為什麼會突然在看這個主題呢？

主要是這幾天在優化自己 GKE 的 Node Pool 規格時，原先是開 e2-standard-2 / 100GB 的 node，數量是 8 ~ 12 顆 node (透過自動擴展)，改成 n2-standard-4 / 100GB 的 node，數量是 6 ~ 10 顆 node (透過自動擴展)。

發現明明 CPU/MEM 資源都夠且更多，但 node 數量卻還是跟原始規格一樣長到 9 顆，使用 Cloud logging 來查看 `unschedulable`，發現與 Local ephemeral storage 有關，因此就來了解一下 Local ephemeral storage 是如何計算的。

<br>

我們在 Cloud Logging 用以下指令來查看為何 node 會需要長新的，而不是進到還有資源的 node 上：

```bash
unschedulable
severity=WARNING
"ephemeral-storage"
```

<br>

{{< figure src="/gcp/gke-local-ephemeral-storage/1.webp" width="750" caption="Cloud Logging 查詢結果" >}}

<br>

可以看到是因為 Local ephemeral storage 不足，導致無法排程，才會長新的 node。

但這個很奇怪，我們明明建立了 100GB 的 node，為什麼會不足呢？而且我們還是開 spot instance，理論上 node 會被收回，node disk 也會被清空，所以不應該會出現這個問題才對。

因此我們就先來了解 Local ephemeral storage 是如何計算的。

<br>

## Local ephemeral storage 計算方式

根據官方文件 [Local ephemeral storage reservation](https://cloud.google.com/kubernetes-engine/docs/concepts/plan-node-sizes#local_ephemeral_storage_reservation)，我們可以知道 GKE 會為每個 node 提供本地的臨時儲存空間，當這個儲存空間在 node 故障刪除時，資料也會被一起遺失。

GKE 會依照下列的方式計算本機預留的暫存空間，也就是被偷走的空間 /ᐠ .ᆺ. ᐟ\ﾉ：

```
EVICTION_THRESHOLD + SYSTEM_RESERVATION
```

`EVICTION_THRESHOLD` 是驅逐閾值 ; `SYSTEM_RESERVATION` 是系統保留空間。

<br>

### EVICTION_THRESHOLD 驅逐閥值計算

在預設情況下，臨時儲存空間是由開機磁碟的大小來決定的，驅逐閾值是開機磁碟大小的 10%。

```
EVICTION_THRESHOLD = 10% * BOOT_DISK_CAPACITY
```

<br>

### SYSTEM_RESERVATION 系統保留空間計算

系統保留空間也跟開機磁碟大小有關：

```
SYSTEM_RESERVATION = Min(50% * BOOT_DISK_CAPACITY, 6GiB + 35% * BOOT_DISK_CAPACITY, 100 GiB)
```

會計算 `50% * BOOT_DISK_CAPACITY`、`6GiB + 35% * BOOT_DISK_CAPACITY`、 `100 GiB` 三者中的最小值。奇怪的計算方式 (\*´･д･)?。

<br>

### 計算可用空間

最後將開機磁碟大小減去 `EVICTION_THRESHOLD` 與 `SYSTEM_RESERVATION` 就可以得到可用空間：

```
BOOT_DISK_CAPACITY - (EVICTION_THRESHOLD + SYSTEM_RESERVATION)
```

<br>

### 計算範例

根據上面的公式我們可以知道 ~~被偷走的空間有多少~~，下面舉例兩個不同的開機磁碟大小來計算：

1. 開機磁碟大小 100GB

先計算 EVICTION_THRESHOLD：

```
EVICTION_THRESHOLD = 10% * 100GB = 10GB
```

再計算 SYSTEM_RESERVATION：

```
SYSTEM_RESERVATION = Min(50% * 100GB, 6GiB + 35% * 100GB, 100 GiB)
                  = Min(50GB, 6GiB + 35GB, 100GB)
                  = Min(50GB, 41GB, 100GB)
                  = 41GB
```

接著將 EVICTION_THRESHOLD 與 SYSTEM_RESERVATION 相加：

```
EVICTION_THRESHOLD + SYSTEM_RESERVATION = 10GB + 41GB = 51GB
```

就可以得出被偷走的空間為 51GB，代表我們建立 100GB 的開機磁碟時，實際上只有 49GB 空間可以使用。

<br>

2. 開機磁碟大小 300GB

先計算 EVICTION_THRESHOLD：

```
EVICTION_THRESHOLD = 10% * 300GB = 30GB
```

再計算 SYSTEM_RESERVATION：

```
SYSTEM_RESERVATION = Min(50% * 300GB, 6GiB + 35% * 300GB, 100 GiB)
                  = Min(150GB, 6GiB + 105GB, 100GB)
                  = Min(150GB, 111GB, 100GB)
                  = 100GB
```

接著將 EVICTION_THRESHOLD 與 SYSTEM_RESERVATION 相加：

```
EVICTION_THRESHOLD + SYSTEM_RESERVATION = 30GB + 100GB = 130GB
```

就可以得出被偷走的空間為 130GB，代表我們建立 300GB 的開機磁碟時，實際上只有 170GB 可以使用。

<br>

了解計算方式後，那為什麼會出現 Local ephemeral storage 不足的問題呢？

後來發現，因為我的多數服務都會使用 gcsfuse 掛載 GCS bucket，而 gcsfuse 預設 ephemeral storage request 會使用 5GB，所以以 100 GB 的開機磁碟，我一個 node 只要超過 9 個 Pod，就會導致 Local ephemeral storage 不足的問題。詳細就請參考：[Configure resources for the sidecar container](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#sidecar-container-resources)

<br>

## 哪裡可以查看 Local ephemeral storage 使用量？

1. 可以從 GKE 的 Node Pool UI 介面查看：

<br>

{{< figure src="/gcp/gke-local-ephemeral-storage/2.webp" width="450" caption="GKE Node Pool 頁面" >}}

<br>

可以看到實際上 101.2 GB 的 node，可用空間只有 47.06 GB，與我們上面的計算方式差不多。

2. 可以透過 `kubectl describe node` 來查看：

<br>

{{< figure src="/gcp/gke-local-ephemeral-storage/3.webp" width="400" caption="kubectl describe node 頁面" >}}

可以看到 Capacity ephemeral-storage: 98831908Ki，換算成 GB 大約是 101.19 GB。

Allocatable ephemeral-storage: 47060071478，換算成 GB 大約是 47.06 GB。

<br>

{{< figure src="/gcp/gke-local-ephemeral-storage/4.webp" width="550" caption="kubectl describe node 頁面" >}}

<br>

也可以從 describe 看到目前 request ephemeral-storage 設定的大小，來避免 Local ephemeral storage 出現不足的問題。

## 參考資料

Local ephemeral storage reservation：[https://cloud.google.com/kubernetes-engine/docs/concepts/plan-node-sizes#local_ephemeral_storage_reservation](https://cloud.google.com/kubernetes-engine/docs/concepts/plan-node-sizes#local_ephemeral_storage_reservation)

Configure resources for the sidecar container：[https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#sidecar-container-resources](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#sidecar-container-resources)
