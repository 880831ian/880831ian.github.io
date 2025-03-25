---
title: "Elastic Kubernetes Service (EKS) 介紹"
type: docs
weight: 9996
date: 2024-07-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

由於公司政策調整，除了原先的 Google Cloud Platform (GCP) 之外，我們也開始使用 AWS 服務，其中包括了 EKS (Elastic Kubernetes Service)。這篇文章將會介紹 EKS 的基本概念以及一些使用上的注意事項。

後面也會有幾篇 AWS 文章，歡迎大家持續關注。

<br>

## 什麼是 EKS？

那熟悉 Kubernetes 的朋友應該對 EKS 並不陌生，EKS 是 AWS 提供的 Kubernetes 服務，讓使用者可以在 AWS 上快速建立、擴展和管理 Kubernetes 集群。就跟 GCP 的 GKE 一樣。

<br>

流程也跟 GKE 差不多，如圖：

1. 建立 Cluster：可以透過 eksctl、AWS Console、AWS CLI、Terraform 等等來建立。

   - eksctl：是一個官方專門給 EKS 的 CLI 工具，可以快速建立、更新和刪除 EKS 集群，詳細可以看：[eksctl](https://eksctl.io/)、[Amazon EKS 入門](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/getting-started-eksctl.html)。

   Mac 安裝指令：

   ```
   brew tap weaveworks/tap
   brew install weaveworks/tap/eksctl
   ```

   - AWS CLI：是官方的 CLI 工具，除了建立 EKS 集群，還可以管理其他 AWS 服務，詳細可以看：我的另一篇筆記 [設定 AWS CLI 以及 AWS CLI 指令說明](../aws-cli)。

2. 選擇運算資源的方式：可以分成 [Fargate](https://aws.amazon.com/tw/fargate/)、[Karpenter](https://karpenter.sh/)、託管節點群組和自管理節點。那我們使用主要是使用託管節點群組。其他的方式會在後面的文章介紹。
3. 設定：設定必要的控制器、驅動程式或是服務等等。
4. 部署工作負載：自訂 Kubernetes 物件，例如 Pod、Service、Deployment 等等。
5. 管理：監督工作負載，整合 AWS 服務以簡化維運並提高工作負載效能。

<br>

{{< figure src="/aws/eks-introduce/1.png" width="750" caption="在雲端中執行 Amazon EKS 的基本流程" >}}

<br>

其他比較詳細的架構以及部署選項可以參考：[Amazon EKS 架構](https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html)、[部署選項](https://docs.aws.amazon.com/eks/latest/userguide/eks-deployment-options.html)，這邊就不再贅述。

<br>

## 建立第一個 EKS：eksctl

詳細請參考：[Amazon EKS 入門：eksctl](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/getting-started-eksctl.html)

在建立 EKS 前，我們要先確認我們要建立的節點是要使用 Fargate 還是託管節點群組，我們這邊拿 GCP 來說明，託管節點群組就是我們一般建立的 Node_Pool，而 Fargate 就是 Serverless 的概念，不需要自己管理節點，只需要管理 Pod，詳細我後面也會寫一篇文章來介紹，[可以點我查看](../fargate-introduce/)。

<br>

那我們這邊就先以託管節點群組來建立 EKS，建立使用以下指令來建立：

```bash
eksctl create cluster --name {my-cluster} --region {region-code}
eksctl create cluster --name ian-test --region us-east-1
```

將 `{my-cluster}` 替換成自己要的名稱，只能英文數字字元(區分大小寫)和連字號。必須字母數字字元開頭，且長度不可以超過 100 個字元，名稱在該區域中必須是唯一的。

`{region-code}` 則是替換成自己要的區域，例如：us-east-1。可以從 [Amazon EKS endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/eks.html) 這邊查看 region-code。
如果是 Fargate 的話，只需要在指令後面加上 `--fargate`

<br>

提醒：使用這個指令建置，會需要有 CloudFormation 的相關權限，如果缺少權限，會顯示以下錯誤：

( CloudFormation 是將基礎設施視為程式碼的服務，可以對 AWS 和第三方資源進行建模、佈建和管理，詳細一樣請參考後續文章 [AWS CloudFormation 介紹](../cloudformation-introduce)。)
<br>

{{< figure src="/aws/eks-introduce/2.png" width="850" caption="使用 eksctl 建立 EKS 缺少 CloudFormation 權限" >}}

<br>

## EKS 定價

<br>

## 參考資料

What is Amazon EKS?：[https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)

Amazon EKS 入門：eksctl：[https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/getting-started-eksctl.html](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/getting-started-eksctl.html)
