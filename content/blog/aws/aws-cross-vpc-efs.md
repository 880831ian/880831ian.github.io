---
title: "跨不同 VPC 資源使用 EFS"
type: docs
weight: 9982
description: 跨不同 VPC 資源使用 EFS
images:
  - aws/aws-cross-vpc-efs/og.webp
date: 2025-02-27
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - AWS
  - Cross-VPC
  - VPC
  - EFS
---

EFS 在設計上，Mount targets 只支援單一 VPC 的 Subnet 來連線，有研究一下，如果有情境需要跨 VPC 使用同一座 EFS 需要怎麼調整設定。

<br>

首先，EFS 會自己建立一條 route53 的 private dns domain 會像 (以 ap-southeast-1 區域為例)：

```
fs-xxxxx.efs.ap-southeast-1.amazonaws.com
```

<br>

開頭有說，EFS 只支援單一 VPC，所以該 EFS 產生的 Domain 只有設定的 VPC 才可以使用 (就算透過 VPC Peering 都不行)，也沒辦法另外去調整 EFS 建立的 DNS Domain，因此我們需要額外再建立一條 route53 internal-dns 來使用

新的一條 route53 internal-dns domain，需要用 A Record 解析到 EFS 的 Network interfaces IP Address 上

<br>

{{< figure src="/aws/aws-cross-vpc-efs/1.webp" width="1200" caption="EFS 的 IP" >}}

<br>

{{< figure src="/aws/aws-cross-vpc-efs/2.webp" width="1200" caption="internal dns 設定新的 EFS Domain，並指向 EFS 的 Network interfaces IP" >}}

<br>

將需要使用的 VPC 都加到該 route53 internal-dns 內，接著回到 EFS，將對應的 Subnet 都設定到 SG，開啟對應 Subnet 的 2049 Port

<br>

最後再建立 PV 時，除了原本的設定

- 原設定

```
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-xxxxxx # 替換為你的 EFS ID
```

<br>

還需要加上

- 新設定

```
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-xxxxxx # 替換為你的 EFS ID
    volumeAttributes:
      mounttargetip: <自訂的 route53-internal-dns domain>
```

<br>

就可以正常使用囉，文件可以參考：[aws-efs-csi-driver/examples/kubernetes/static_provisioning/README.md at master · kubernetes-sigs/aws-efs-csi-driver](https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/examples/kubernetes/static_provisioning/README.md)