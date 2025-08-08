---
title: "Google Kubernetes Engine CronJob 會有短暫時間沒有執行 Job"
type: docs
weight: 9994
date: 2023-02-10
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Google Cloud Platform
  - GCP
  - Kubernetes
  - K8s
  - CronJob
  - GKE
  - Google Kubernetes Engine
  - Job
---

前陣子公司建立在 Google Kubernetes Engine 叢集上的 CronJob 服務會有短暫時間沒有執行 Job。先前情提要一下，此 CronJob 的設定是每分鐘都會執行 (圖一)，所以理當來說 Log 應該要可以看到每分鐘都有此 CronJob 的紀錄，但有時候會發生 CronJob 短暫時間都沒有執行的狀況，找了一陣子都沒有找到原因，最後開支援單請 Google 那邊協助查看，終於找到原因拉 😍。那就跟我一起看一下發生的過程，以及 Google 幫我們找到的原因，以及要如何解決等等～

<br>

{{< figure src="/gcp/gke-cronjob-not-working/1.webp" width="600" caption="(圖一) CronJob schedule 時間為每分鐘執行" >}}

<br>

## 問題發生以及問題原因

我們使用 Google Cloud Platform 裡面的記錄功能，可以看到 (圖二)，在每分鐘執行的 Log 中，有短暫時間沒有執行 Job，但這個時間除了 CronJob 以外，其他的服務都是好的。

<br>

{{< figure src="/gcp/gke-cronjob-not-working/2.webp" width="700" caption="(圖二) Google Cloud Platform 記錄有短暫沒有執行" >}}

<br>

找了一陣子都沒有找到原因，於是我們開支援單請 Google 那邊協助查看，Google 那邊找了一陣子後終於找到原因拉！！！我們一起來看看吧 (圖三)

<br>

{{< figure src="/gcp/gke-cronjob-not-working/3.webp" width="700" caption="(圖三) Google 支援單回覆" >}}

<br>

就如同 Google 所說，使用提供的指令參數來查詢，叢集在該時段有發生 master control plane 的升級，如 (圖四)，再加上我們建的這個 cluster 是使用 zonal cluster (圖五)，所以叢集只有一個 control plane，當 control plane 在更新時，會無法部署新的 workload，導致該 CronJob 沒有執行 Job。參考資料 [1]

<br>

{{< figure src="/gcp/gke-cronjob-not-working/4.webp" width="700" caption="(圖四) 發生 master control plane 的升級" >}}

{{< figure src="/gcp/gke-cronjob-not-working/5.webp" width="450" caption="(圖五) 有問題的叢集位置類型" >}}

<br>

Google 的建議是可以考慮使用另一個 regional cluster，讓 master node 在更新時不會只在單一地區，或是一樣使用舊的 zonal cluster，透過設定 Maintenance window 或者 Maintenance exclusions 來降低服務受到 workload 的影響。參考資料 [2]

<br>

{{< callout tip >}}

1. 就算把 node_pool 裡面的自動升級給停掉，也沒有辦法解決此問題！因為此 master control plane (也就是 master node) 的升級，不是 worker node 的 node pool 升級，是由 GKE 負責維護的，所以他們會定期升級 control plane，也沒辦法停止此類的升級。

2. 若已經建立好 zonal cluster 後，想要改成 regional cluster ，是沒有辦法使用修改的方式，一定只能重建 cluster，所以大家在建立時要注意～
   {{< /callout >}}

<br>

## 解決問題

最後我們選擇將叢集給整個重建，來確保 CronJob 不會有沒有執行到的狀況發生，重建叢集跟搬服務的過程很辛苦的 😰 希望大家不要發生 QQ，最後我們來看一下重建完後，在 master control plane 更新的時候，還會不會有 CronJob 沒有執行的情況發生。

<br>

新叢集使用 regional cluster 來建立，在 2/1 也有 master control plane 的升級。 (圖六)

<br>

{{< figure src="/gcp/gke-cronjob-not-working/6.webp" width="700" caption="(圖六) 發生 master control plane 的升級" >}}

<br>

查看 CronJob 執行的紀錄可以發現並沒有 Job 沒有執行的情況發生。 (圖七)

<br>

{{< figure src="/gcp/gke-cronjob-not-working/7.webp" width="700" caption="(圖七) 叢集位置類型" >}}

{{< figure src="/gcp/gke-cronjob-not-working/8.webp" width="450" caption="(圖八) 新叢集位置類型" >}}

<br>

## 參考資料

[1] Standard cluster upgrades：[https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-upgrades](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-upgrades)

[2] maintenance-windows-and-exclusions：[https://cloud.google.com/kubernetes-engine/docs/concepts/maintenance-windows-and-exclusions](https://cloud.google.com/kubernetes-engine/docs/concepts/maintenance-windows-and-exclusions)
