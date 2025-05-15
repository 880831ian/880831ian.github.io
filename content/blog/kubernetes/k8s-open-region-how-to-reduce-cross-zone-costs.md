---
title: "Kubernetes 開啟 Region 後，如何減少跨 Zone 網路費用"
type: docs
weight: 9989
description: Kubernetes 開啟 Region 後，如何減少跨 Zone 網路費用
images:
  - kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/og.webp
date: 2025-05-15
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

由於最近公司開始導入 Multi Zone 架構，這時候會出現一個問題，就是明明我是同一個 GKE，但因為有多個 Zone，導致開始會有一些跨 Zone 的網路傳輸費用產生 (每 Gib 會收 0.01 美元)，可以參考下方圖片：

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/1.png" width="1000" caption="跨域費用說明" >}}

<br>

所以此文章會針對要如何減少跨 Zone 網路費用進行測試以及提供解決辦法。


<br>

## 建置 GKE 環境

> [!NOTE] GKE 資訊：
>Tier / Mode：Standard<br>版本：1.32.2-gke.1297002<br>Location type：Regional<br>Default node zones：[\"asia-east1-a", \"asia-east1-b"]


<br>

- 使用的 Module 請參考：[880831ian/IaC](https://github.com/880831ian/IaC/tree/master/modules/gke)

```hcl
terraform {
  source = "${get_path_to_repo_root()}/modules/gke"
}
include {
  path = find_in_parent_folders("root.hcl")
}
inputs = {
  name                   = "test"
  regional               = true
  region                 = "asia-east1"
  zones                  = ["asia-east1-a", "asia-east1-b"]
  release_channel        = "REGULAR"
  master_ipv4_cidr_block = "172.16.1.16/28"
  network_project_id     = "xxxxx"
  network                = "default"
  subnetwork             = "default-asia-east1"
  gcs_fuse_csi_driver    = false
  enable_maintenance     = false
  monitoring_enable_managed_prometheus = false
  deletion_protection                  = false
  node_pools = [
    {
      name         = "test-pool"
      machine_type = "e2-small"
      min_count    = 1
      max_count    = 1
    }
  ]
}
```

(這邊小提醒，如果使用多個 Zone，在 node 的 CA 記得要用 min_count 跟 max_count，他才是依照 Zone 去長，也就是 test-pool 會分別在 Zone A 跟 Zone B 去長歐)

<br>

## 測試連線

### 原生 Service 測試

建立完後，啟動兩個 nginx 服務，分別叫做 nginx-a 跟 nginx-b，各會有兩個 pod，並使用 podAntiAffinity 來限制同一個服務能建立在兩個 Zone 上面。

相關的程式碼都會放在 [880831ian/k8s-open-region-how-to-reduce-cross-zone-costs](https://github.com/880831ian/k8s-open-region-how-to-reduce-cross-zone-costs)

<br>

#### 程式碼

下面附上其中一個 nginx-a 服務 yaml (nginx-b 也跟 nginx-a 一樣，只有名稱改變)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-a
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-a
  template:
    metadata:
      labels:
        app: nginx-a
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx-a
              topologyKey: topology.kubernetes.io/zone
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/conf.d
      volumes:
          - name: nginx-conf
            configMap:
              name: nginx-conf
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-a
spec:
  selector:
    app: nginx-a
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  default.conf: |
    server {
      listen 80;
      location / {
        return 200 "$hostname\n";
      }
    }
```

<br>

#### 顯示部署完的服務狀態

我們使用指令，列出四個 pod 各別的 node 以及所在的 Zone

```bash
printf "%-30s %-35s %-20s\n" "POD" "NODE" "ZONE"
echo "---------------------------------------------------------------------------------------------"

kubectl get pod -o=custom-columns='POD:metadata.name,NODE:spec.nodeName' --no-headers | while read pod node; do
  zone=$(kubectl get node "$node" -o=jsonpath='{.metadata.labels.topology\.kubernetes\.io/zone}')
  printf "%-30s %-35s %-20s\n" "$pod" "$node" "$zone"
done
```

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/2.png" width="850" caption="Pod 所在 Node 以及 Zone" >}}

<br>

接著，我們從 nginx-a-98f749dc6-jzl8m (Zone A) 發起連線打 nginx-b 的 svc，觀察一下他會打到同樣是 Zone A 的 nginx-b-795d6c85b8-c2k45，還是 Zone B 的 nginx-b-795d6c85b8-tncfm，或是隨機分配呢？

我們使用以下指令打 1000 次並觀察次數

```bash
for i in {1..1000}; do
  curl -s http://nginx-b.default.svc.cluster.local
done | sort | uniq -c
```

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/3.png" width="550" caption="Pod 所在 Node 以及 Zone" >}}

<br>

為了測試準確性，我們重複測試 3 次

| 同 nginx-a Zone A 打 nginx-b svc | nginx-b Zone A | nginx-b Zone B |
|----------------------------------|----------------|----------------|
| 第一次打 1000 次 | 482           | 518              |
| 第二次打 1000 次 | 493           | 507              |
| 第三次打 1000 次 | 498           | 502              |


<br>

就上面測試結果可以得知，他是隨機去分配到 Zone A 或是 Zone B 的 Nginx-b Pod，因為 Service 本來就會依照預設的 Round-Robin 來去分配。

<br>

### 使用 Topology Aware Routing 測試

簡單說明一下，Topology Aware Routing 縮寫是 (TAR/TAH) 中文叫做：拓樸感知路由

在 K8s 1.27 以前叫作 Topology Aware Hints，所以縮寫才會是 TAH，邏輯是需要在 Service 的 annotation 加上 `service.kubernetes.io/topology-mode: Auto` （1.27 Version），以前版本設定方式會不同。

它可以幫助網路流量盡可能的保持在同一個 Zone 裡面，Pod 之前優先使用同 Zone 流量來提高可靠性、性能以及減少成本。

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/4.png" width="750" caption="設定完成後，會顯示來至 endpoint-slice-controller" >}}

<br>

當設定上去後，因為 EndpointSlice 這邊會記錄每個 Pod 是屬於哪個 Zone，當並提供給 kube-proxy 知道，進而讓流量往同樣的 Zone 走。

<br>

#### 顯示部署完的服務狀態

那我們一樣用同樣的步驟再來測試一次：

使用一樣的指令，先列出四個 pod 各別的 node 以及所在的 zone (服務都有重啟過)



```bash
printf "%-30s %-35s %-20s\n" "POD" "NODE" "ZONE"
echo "---------------------------------------------------------------------------------------------"

kubectl get pod -o=custom-columns='POD:metadata.name,NODE:spec.nodeName' --no-headers | while read pod node; do
  zone=$(kubectl get node "$node" -o=jsonpath='{.metadata.labels.topology\.kubernetes\.io/zone}')
  printf "%-30s %-35s %-20s\n" "$pod" "$node" "$zone"
done
```

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/5.png" width="850" caption="測試結果" >}}

<br>

我們從 nginx-a-fd5bdb699-6qnwb (Zone A) 發起連線打 nginx-b 的 svc，一樣觀察他會打到同樣是 zone A 的 nginx-b-795d6c85b8-2q7m2，還是 Zone B 的 nginx-b-795d6c85b8-82427？

我們使用以下指令打 1000 次並觀察次數

```bash
for i in {1..1000}; do
  curl -s http://nginx-b.default.svc.cluster.local
done | sort | uniq -c
```

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/6.png" width="550" caption="Pod 所在 Node 以及 Zone" >}}

<br>

為了公平性，我們也比照原本測試的，重複測試 3 次

| 同 nginx-a Zone A 打 nginx-b svc | nginx-b Zone A | nginx-b Zone B |
|----------------------------------|----------------|----------------|
| 第一次打 1000 次 | 1000           | 0              |
| 第二次打 1000 次 | 1000           | 0              |
| 第三次打 1000 次 | 1000           | 0              |


<br>

> [!TIP] 所以可以得知：
> 只要 Service 有加上該 annotation，就可以透過 EndpointSlice 走到同樣的 Zone。

<br>

另外，我們也來測試看看，如果打的過程中，將 nginx-b Zone A nginx-b-795d6c85b8-2q7m2 Pod 給移除，那觀察流量會不會自動轉到 Zone B 呢？！

<br>

{{< figure src="/kubernetes/k8s-open-region-how-to-reduce-cross-zone-costs/7.png" width="850" caption="Pod 所在 Node 以及 Zone" >}}

<br>

測試結果：當一開始進到 nginx-b-795d6c85b8-2q7m2 的流量，遇到 nginx-b-795d6c85b8-2q7m2 服務中斷，由於該 Nginx-b 在 Zone A 沒有其他 Pod，所以就會切回去使用 Zone B 的 Nginx-b 服務 (nginx-b-795d6c85b8-82427)，但當 Zone A 的 Nginx-b 服務好了，又會再走回 Zone A (nginx-b-795d6c85b8-w2vr4) < 是後面長出來的 Pod。

<br>

#### 使用的建議

- 確保 K8s 版本在 1.27 以上，1.27 以下需要使用 `service.kubernetes.io/topology-aware-hints`。

- 每個服務要均勻分配在每個 Zone 上面。

- 確保服務在更新時，同 Zone 上面的數量足夠，避免切到其他 Zone 導致費用產生。

<br>

## 參考資料

Google Cloud 內部 VM 之間的資料移轉定價：[https://cloud.google.com/vpc/network-pricing?hl=zh-TW](https://cloud.google.com/vpc/network-pricing?hl=zh-TW)

Topology Aware Routing：[https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/](https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/)

AWS 相關文件 (AWS 也會有類似問題)：[https://aws.amazon.com/tw/blogs/containers/exploring-the-effect-of-topology-aware-hints-on-network-traffic-in-amazon-elastic-kubernetes-service/](https://aws.amazon.com/tw/blogs/containers/exploring-the-effect-of-topology-aware-hints-on-network-traffic-in-amazon-elastic-kubernetes-service/)