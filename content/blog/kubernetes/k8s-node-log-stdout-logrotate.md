---
title: "K8s Node Log Stdout Logrotate 回收機制"
type: docs
weight: 74
description: K8s Node Log Stdout Logrotate 回收機制
images:
  - kubernetes/k8s-node-log-stdout-logrotate/og.webp
date: 2025-03-12
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

我們知道可觀測性的三大支柱是 Log、Metics、Trace，其中 Log 是最基礎的一環，透過 Log 我們可以了解應用程式的運作狀況，當應用程式出現問題時，透過 Log 可以快速定位問題，進而解決問題。

以我們公司為例，我們主要使用 GCP，現在也開始用 AWS 來當作 DR 的環境。在 GCP 我們原本會使用 Log Exporter 來查看服務及系統的 Log，但為了更方便的管理，我們改使用 Datadog 來統一收集 Log (AWS 也是)。

要如何傳遞給 Datadog，我們依照 Datadog 官方文件建議，將 Log Stdout，讓 Datadog agent 去收集 Log，這樣就可以統一管理 Log 了，詳細可參考 [Datadog Kubernetes log collection](https://docs.datadoghq.com/containers/kubernetes/log/?tab=datadogoperator)。

<br>

## 疑問

我之前ㄧ直以為 Container Log Stdout 不會寫實體檔，後來發現，其實它還是會寫，但當 Node 重啟，或是 Pod 被刪除時，才會移除，要怎麼查看，我們可以進到 GKE or EKS 的 node 機器上，到 `/var/log/pods/` 路徑底下，就可以看到運作在該 node 上的 pod，資料夾名稱會是以 `{namespace}_{pod name}_xxx` 來命名。

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/1.jpg" width="550" caption="How nodes handle container logs 圖片：[Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/#how-nodes-handle-container-logs)" >}}

<br>

所以就好奇，是誰來管理這些 Log 的回收機制呢？如果沒有回收機制，Log 不會把 Node 的 Disk 給寫爆嗎？

<br>

## Node Logrotate

當然，有在使用過相關 Log 工具，對 Logrotate 一定不陌生，第一個想法就想說，會不會 Node 上有對應的 Logrotate 設定，去定期的去處理 `/var/log` 的檔案，因次我們這邊針對 GKE(GCP) 以及 EKS(AWS) 來做說明：

- GKE

首先，我們先連線到 GKE Node 上，進到 `/etc/logrotate.d` 底下，可以看到以下圖片的設定，在 GKE 裡面只有一個 Logrotate 設定檔，他會針對整個 `/var/log` 底下的 log 檔案做處理。

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/2.png" width="550" caption="GKE /etc/logrotate.d 底下檔案設定" >}}

<br>

- EKS

一樣的步驟，進到 `/etc/logrotate.d` 底下，可以看到以下圖片的設定，EKS 會有多個 Logrotate 設定檔，但卻沒有對 `/var/log/pods` 底下的 log 檔案做處理，所以 EKS 需要額外去寫 DameonSet 來處理 `/var/log/pods` 底下的 log 檔案回收嗎？

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/3.png" width="500" caption="EKS /etc/logrotate.d 底下檔案設定" >}}

<br>

## 尋找答案

最後在 Kubernetes 官方文件中 [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/#log-rotation) 有看到兩個設定值，分別是 `containerLogMaxSize` (預設 10Mi)，以及 `containerLogMaxFiles`(預設 5)。kubelet 會根據這兩個設定值來決定是否要回收 Log。

如果 Log 檔案大小超過 `containerLogMaxSize`，kubelet 就會重新開始寫新的 Log 檔案，如果 Log 檔案數量超過 `containerLogMaxFiles`，kubelet 會刪除最舊的 Log 檔案。

那要怎麼查看當前 node 的 kubelet 設定值呢？可以透過以下指令：

```bash
journalctl -u kubelet --no-pager | grep "container-log-max"
```

就可以看到底下兩個設定值：（圖片上面是 EKS，下面是 GKE）

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/4.png" width="750" caption="查看 containerLogMaxSize 跟 containerLogMaxFiles 當前設定值" >}}

<br>

可以發現都跟預設值一樣。

<br>

那我們也直接進到 `/var/log/pods` 底下，來查看是否是依照這個規律去運作。

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/5.png" width="650" caption="進入 /var/log/pods" >}}

<br>

可以看到，就跟上面說的資料夾名稱是由 `{namespace}_{pod name}_xxx` 來命名，我們進去到隨機一個資料夾來查看 Log 檔案。

<br>

- GKE

<br>

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/6.png" width="650" >}}

<br>

- EKS

{{< figure src="/kubernetes/k8s-node-log-stdout-logrotate/7.png" width="650" >}}

<br>

可以看到，GKE 跟 EKS 檔案數量都是 5 個，且每個都是小於 10Mi (因為計算誤差，所以有些會顯示 11、12)，所以可以確定是依照 `containerLogMaxSize` 跟 `containerLogMaxFiles` 在運作的。

也可以看到 GKE Log，第一次查看最舊的檔案是 `0.log.20250317-081814.gz`，第二次查看最舊的檔案就變成 `0.log.20250317-081844.gz`，就代表超過 5 份 Log 檔案，kubelet 就會刪除最舊的 Log 檔案。

<br>

## 參考資料

Logging Architecture 日誌架構：[https://kubernetes.io/docs/concepts/cluster-administration/logging/#log-rotation](https://kubernetes.io/docs/concepts/cluster-administration/logging/#log-rotation)

Datadog Kubernetes log collection：[https://docs.datadoghq.com/containers/kubernetes/log/?tab=datadogoperator](https://docs.datadoghq.com/containers/kubernetes/log/?tab=datadogoperator)
