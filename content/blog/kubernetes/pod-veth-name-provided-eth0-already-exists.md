---
title: "部署 Pod 遇到 container veth name provided (eth0) already exists 錯誤"
type: docs
weight: 3
---

此文章要來記錄一下公司同事在正式服務上遇到的問題，會詳細說明遇到事情的經過，以及開單詢問 google support 最後討論出的暫時解決的辦法：

<br>

簡單列出正式站當下服務環境：

- gke master version：1.25.10-gke.2700
- gke node version：1.25.8-gke.1000
- 該問題發生的 node pool 有設定 taint
- 發生問題的 Pod 是用 Statefulset 建立的服務

<br>

## 事情發生的經過

1. RD 同仁反應，發現使用 Statefulset 建立的排程服務有問題，下 `kubectl delete` 指令想要刪除 Pod，讓 Pod 重新長，卻卡在 Terminating，等待一段時間後，決定下 `kubectl delete --force --grace-period=0` 來強制刪除 Pod，這時候狀態會卡在 ContainerCreating，使用 Describe 查看，會出現以下錯誤：

<br>

```
Warning
(combined from similar events): Failed to create pod sandbox: rpo error: code = Unknown desc = failed to setup network for sandbox
"14fe0cd3d688aed4ffed4c36ffab1a145230449881bcbe4cac6478a63412b0c*: plugin type=*gke" failed (add): container veth name provided (etho) already exists
```

<br>

2. 我們 SRE 協助查看後，也有嘗試去下 `kubectl delete --force --grace-period=0` 來刪除 Pod，但還是一樣卡在 ContainerCreating，最後是先開一個新的 Node 並讓該 Pod 建立到新的 Node 上，才解決問題。為了方便 google support 協助檢查出問題的 Node，先將 Node 設定成 cordon，避免其他 Pod 被調度到該問題 node 上。

<br>

{{< callout >}}
Node 設定成 cordon

Node 可以設定 cordon、drain 和 delete 三個指定都會使 Node 停止被調度，只是每個的操作暴力程度不同：

cordon：影響最小，只會將 Node 標示為 SchedulingDisabled 不可調度狀態，但不會影響到已經在該 Node 上的 Pod，使用 `kubectl cordon [node name]` 來停止調度，使用 `kubectl uncordon [node name]` 來恢復調度。

drain：會先驅逐在 Node 上的 Pod，再將 Node 標示為 SchedulingDisabled 不可調度狀態，使用 `kubectl drain [node name]  --ignore-daemonsets --delete-local-data` 來停止調度，使用 `kubectl uncordon [node name]` 來恢復調度。

delete：會先驅逐 Node 上的 Pod，再刪除 Node 節點，它是一種暴力刪除 Node 的作法，在驅逐 Pod 時，會強制 Kill 容器進程，沒辦法優雅的終止 Pod。

{{< /callout >}}

<br>

3. 我們隨後開單詢問 goolge support。

<br>

## 與 Google Support 討論內容

Google Support 經過查詢後，回覆說：這個問題是因為 Pod 被強制刪除導致，強制刪除是一種危險的操作，不建議這樣處理，下面有詳細討論。

<br>

1. 一開始卡在 Terminating 狀態，我們也有請 RD 說明一下當下遇到的問題以及處理動作：RD 當時想要刪除 Pod 是因為該程式當下有 Bug，將 redis 與 db 連線給關閉，程式找不到就會一直 retry，導致相關進程無法結束，再加上 terminationGracePeriodSeconds 我們設定 14400，也就是 4 小時，才會卡在 Terminating 狀態。
   (terminationGracePeriodSeconds 設定這麼久是希望如果有被 on call，工程師上來時，可以查看該 Pod 的錯誤原因)

2. 因為卡在 Terminating 太久，RD 有執行 `kubectl delete --force`，就是因為下了 `--force` 才造成相關資源問題 (例如 container proccess, sandbox, 以及網路資源)沒有刪乾淨。所以引起了此次的報錯 "container veth name provided (eth0) already exists"。
   (因為我們服務使用 Statefulset，Pod 名稱相同，導致 eth0 這個網路資源名稱重複，所以造成錯誤，可以用 deployment 來改善這個問題，只是資源如果沒有清理乾淨會佔用 IP，所以單純調整成 deployment 也不是最佳解)

3. Google 產品團隊建議，如果 Pod 處於 Running 狀態時，想要快速刪除 Pod 時，一開始就先使用 `kubectl delete pod --grace-period=number[秒數]` 來刪除，如果已經是 Terminating 狀態則無效。(SRE 同仁已測試過，與 Google Support 說明相同)

4. 那如果已經處於 Terminating 狀態，要怎麽讓 Pod 被順利刪除，這部分 Google Support 後續會在測試並給出建議，目前測試是：進去卡住的 Pod Container，手動刪除主進程 (pkill) 就可以了。

<br>

<img src="/kubernetes/pod-veth-name-provided-eth0-already-exists/1.png" width="800" />

<br>

## 參考

Node 節點禁止調度（平滑維護）方式- cordon，drain，delete：[https://www.cnblogs.com/kevingrace/p/14412254.html](https://www.cnblogs.com/kevingrace/p/14412254.html)
