---
title: "Amazon VPC CNI 介紹"
type: docs
weight: 9984
description: Amazon VPC CNI 介紹
images:
  - aws/vpc-cni/og.webp
date: 2025-06-01
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

此文件是有同事詢問我 EKS 內的 IP 是怎麼分配的，於是我就寫了這篇文章來介紹一下 Amazon VPC CNI 的運作方式。

## VPC CNI 介紹

Amazon EKS 透過 Amazon VPC 容器網路介面實現集群聯網插件，也稱為 VPC CNI。 CNI 插件允許 Kubernetes Pod 擁有與 VPC 網路上相同的 IP 位址。更具體地說，Pod 內的所有容器共享一個網路命名空間，並且它們可以使用本地連接埠相互通訊。

Amazon VPC CNI 由兩個組件組成：

1. CNI 二進位檔（CNI Binary）是用來設定 Pod 網路的關鍵元件，負責讓 Pod 之間可以正常通訊。它會部署在 Node 的根檔案系統上，並在每次有新的 Pod 建立或現有 Pod 被移除時，由 kubelet 呼叫執行，以完成對應的網路配置或清除。

2. ipamd （IP Address Management Daemon) 是一個長期運行的 Node 本地 IP 位址管理 (IPAM) 守護進程，管理 Node 上的 Elastic Network Interface(ENI)，以及維護可用 IP 位址或前綴的暖池 (Warm Pool)，以便在 Pod 啟動時快速分配 IP 位址。

<br>

CNI 插件管理 Node 上的彈性網路介面 (ENI)。當配置 Node 時，CNI 插件會自動從 Node 的子網路向主 ENI 指派一個插槽池（IP 或前綴）。此池稱為熱池 ，其大小由 Node 的實例類型決定。根據 CNI 設置，插槽可能是 IP 位址或前綴。當 ENI 上的插槽被指派後，CNI 可能會將具有熱插槽池的其他 ENI 附加到 Node 。這些額外的 ENI 稱為輔助 ENI。根據實例類型，每個 ENI 只能支援一定數量的插槽。 CNI 根據所需的插槽數量（通常與 Pod 的數量相對應）將更多 ENI 附加到實​​例。這個過程持續進行，直到 Node 不再支援額外的 ENI。 CNI 也預先分配「熱」 ENI 和插槽，以加快 Pod 啟動速度。請注意，每種實例類型都有可附加的最大 ENI 數量。除了運算資源之外，這是對 Pod 密度（每個 Node 的 Pod 數量）的一個限制。

<br>

{{< figure src="/aws/vpc-cni/1.jpg" width="750" caption="" >}}

<br>
