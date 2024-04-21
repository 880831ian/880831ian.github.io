---
title: "正式環境上踩到 StatefulSet 的雷，拿到 P1 的教訓"
type: docs
weight: 3
---

此文章要來記錄一下前陣子在公司的正式環境踩到 StatefulSet 的雷，事情是這樣的，我們有些服務，是使用 StatefulSet 來建置，至於為什麼不用 Deployment，這個說來話長 (也不是因為需要特定的 Pod 名稱、或是網路標記等等)，我們這邊先不討論，這個 StatefulSet 服務是 Nginx + PHP-FPM，為了避免流量進入到 processes 已經被用光的 Pod 中，我們在 StatefulSet 的 PHP Container 上有設定 readiness，readiness 的設定長得像以下：

<br>

```yaml
readinessProbe:
  exec:
    command:
      - "/bin/bash"
      - "-c"
      - |
        CHECK_INFO=$(curl -s -w 'http code:\t%{http_code}\n' 127.0.0.1/status)
        HTTP_CODE=$(echo -e "${CHECK_INFO}" | awk '/http code:/ {print $3}')
        IDLE_PROCESSES=$(echo -e "${CHECK_INFO}" | awk '/idle processes:/ {print $3}')
        [[ $HTTP_CODE -eq 200 && $IDLE_PROCESSES -ge 10 ]] || exit 1
```

<br>

我們會用 curl 來打 `/status`，檢查回傳的 http code 是否為 200，以及 idle processes 是否大於等於 10，如果不符合，就會回傳 1，讓他被標記不健康，讓 Kubernetes 停止流量到不健康的容器，以確保流量被路由到其他健康的副本。

<br>

## 問題

當天遇到的情況是，我們上程式後，Pod 都一切正常，當流量開始進來後，發現 10 個 Pod 會開始偶發的噴 `Readiness probe failed`，查看監控發現 processes 越來越低，最後被反應服務有問題，我們查看 Hpa 的紀錄的確有觸發到 40 個 Pod，只是查看 Pod 數還是依樣卡在 10 個，當下我們有嘗試使用調整 yaml 在 apply，發現 StatefulSet 的 yaml 也已經更新了，但 Pod 還是一樣卡在 10 個，也有使用 kubectl 下 `kubectl scale sts [服務名稱] --replicas=0`，想要切換 Pod 數也沒有辦法。

<br>

當下我們有先 Call Google 的 Support 一起找原因，Google 是建議我們 readiness 的條件不要設的太嚴格，可以加上 `timeoutSeconds: 秒數`，但對於 Pod 卡住，還是沒有找到原因，後來我們查了一下 StatefulSet 的文件發現，StatefulSet 有一個設定 `podManagementPolicy`，預設是 `OrderedReady`，他必須等待前面的 Pod 是 Ready 狀態，才會再繼續建立新的，也就是說我們的 StatefulSet 已經卡住，導致就算 Hpa 觸發要長到 40 個 Pod 也沒有用。

<br>

## 解決辦法

當下想趕快解決 readiness 這個問題，調整 `timeoutSeconds` 後，單純 apply 是沒有用的，要記得刪掉卡住的 Pod，讓他重新建立，才會套用新的設定 (但我們當下太在意為甚麼 Pod 會卡住，沒有想到要先把 readiness 問題修掉 xD，我們當下的解法是先將流量導到地端正常的服務上)。

另外 Google 也說，假如我們還是必須使用 StatefulSet 來建立服務，建議我們把 podManagementPolicy 改成 `Parallel`，它會有點像是 Deployment 的感覺，不會等待其他 Pod 變成 Ready 狀態，所以可以讓我們就算在 readiness 卡住的情況下，也可以自動擴縮服務。

<br>

{{< callout note >}}
StatefulSet podManagementPolicy 參數說明

- OrderedReady (預設)

Pods 會按照順序一個接一個地被創建。即，n+1 號 Pod 不會在 n 號 Pod 成功創建且 Ready 之前開始創建。
在縮小 StatefulSet 的大小時，Pods 會按照反向順序一個接一個地被終止。即，n 號 Pod 不會在 n+1 號 Pod 完全終止之前開始終止。
這確保了 Pods 的啟動和終止的順序性。

<br>

- Parallel

所有 Pods 會同時地被創建或終止。
當 StatefulSet 擴展時，新的 Pods 會立即開始創建，不用等待其他 Pods 成為 Ready 狀態。
當縮小 StatefulSet 的大小時，要終止的 Pods 會立即開始終止，不用等待其他 Pods 先終止。
這種策略提供了快速的擴展和縮小操作，但缺乏順序性保證。

{{< /callout >}}

<br>

## 測試結果

最後我們就使用兩種模式來測試看看，已下是測試結果(透過 P1 才知道的設定ＱＱ)：

有將測試的 StatefulSet 放在 Github，[可以點我查看](https://github.com/880831ian/k8s-statefulset-podmanagementpolicy) (可以調整 readinessProbe 的 httpGet.Path 故意把他用壞)

<br>

### 使用 OrderedReady 模式

StatefulSet 在 podManagementPolicy 預設 OrderedReady 的模式，故意讓 readiness 卡住時 (Pod 卡住時)：

- 當下的 StatefulSet 設定：

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/1.png "StatefulSet 設定")

- Pod 狀態：

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/2.png "Pod 狀態")

<br>

#### 使用指令調整 Pod 數量

我們這時候下指令調整 Pod 數量，看看會發生什麼事：

```
kubectl scale sts my-statefulset --replicas=5
```

<br>

我們先看 StatefulSet 的 yaml 可以看到 Pod replicas 已經改變，也可以看 generation 有更新，代表 StatefulSet 本身有接收到調整設定的請求。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/3.png "下指令調整後的 StatefulSet 設定")

<br>

看了一下 Pod 數量，也是一樣卡住，且 Pod 數量也沒有變化。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/4.png "下指令調整後的 Pod 狀態")

<br>

#### 使用 yaml 調整 Pod 數量

我們直接調整 StatefulSet yaml 的 Pod 數量，看看會發生什麼事：

一樣我們先看 StatefulSet 的 yaml 可以看到 Pod replicas 已經改變(這裡應該切別的 Pod 數量，切回 3 個好像沒有意義 xD)，也可以看 generation 有更新。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/5.png "使用 yaml 調整後的 StatefulSet 設定")

<br>

看了一下 Pod 數量，也是一樣卡住，且 Pod 數量也沒有變化。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/6.png "使用 yaml 調整後的 Pod 狀態")

<br>

所以代表在 OrderedReady 的模式下，Pod 卡住時，無法對 Pod 進行任何操作，必須要手動刪除卡住的 Pod 才吃得到最新的設定。

<br>

### 使用 Parallel 模式

StatefulSet 在 podManagementPolicy Parallel 的模式，故意讓 readiness 卡住時 (Pod 卡住時)：

- 當下的 StatefulSet 設定：

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/7.png "StatefulSet 設定")

- Pod 狀態：

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/8.png "Pod 狀態")

<br>

#### 使用指令調整 Pod 數量

我們這時候下指令調整 Pod 數量，看看會發生什麼事：

```
kubectl scale sts my-statefulset --replicas=5
```

<br>

我們先看 StatefulSet 的 yaml 可以看到 Pod replicas 已經改變，也可以看 generation 有更新，代表 StatefulSet 本身有接收到調整設定的請求。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/9.png "下指令調整後的 StatefulSet 設定")

<br>

看了一下 Pod 數量，就算 my-statefulset-2 卡住，還是可以擴到 5 個 Pod。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/10.png "下指令調整後的 Pod 狀態")

<br>

#### 使用 yaml 調整 Pod 數量

我們直接調整 StatefulSet yaml 的 Pod 數量，看看會發生什麼事：

一樣我們先看 StatefulSet 的 yaml 可以看到 Pod replicas 已經改變，也可以看 generation 有更新。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/11.png "使用 yaml 調整後的 StatefulSet 設定")

<br>

看了一下 Pod 數量，也不會管其他 Pod 是否 Ready，一樣可以縮小成 2 個 Pod。

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/12.png "使用 yaml 調整後的 Pod 狀態")

<br>

## 結論

後來我們重新檢查了一下為什麼 processes 會用完，結果發現是 RD 的程式邏輯，導致每筆 Request 必須等待前一筆 Request 做完，才會開始動作，讓 processes 一直被占用，沒辦法即時消化，導致 processes 用完，又加上服務是使用 StatefulSet，預設模式的 OrderedReady，必須等待前一個 Pod 是 Ready 才可以自動擴縮，所以當我們 Hpa 想要擴縮，來增加可用的 processes 數量，也因為沒辦法擴縮，最後導致這一連串的問題 😕。

<br>

另外，如果想要從 OrderedReady 模式切成 Parallel 模式 (反正過來也是)，必須先將原本的 StatefulSet 給刪除，才可以調整：

<br>

![](/kubernetes/k8s-statefulset-podmanagementpolicy/13.png "OrderedReady 模式切成 Parallel 模式")

<br>

## 參考資料

Kubernetes — 健康檢查：[https://medium.com/learn-or-die/kubernetes-%E5%81%A5%E5%BA%B7%E6%AA%A2%E6%9F%A5-59ee2a798115](https://medium.com/learn-or-die/kubernetes-%E5%81%A5%E5%BA%B7%E6%AA%A2%E6%9F%A5-59ee2a798115)

Pod Management Policies：[https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies)
