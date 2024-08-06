---
title: "Identity and Access Management (IAM) 介紹"
type: docs
weight: 37
date: 2024-07-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

先來說一下為什麼會寫這篇文章，主要是在寫跟測試 [Elastic Kubernetes Service (EKS) 介紹](../eks-introduce) 文章時，想要用 eksctl 來建立 EKS，它實際上是透過 CloudFormation 來建立的，詳細可以去看 EKS 文章，總之當時遇到權限的問題，找了很久，也看不懂 AWS 的 ARN 是什麼，所以就先來寫一篇 IAM 的文章 (´≖◞౪◟≖)。

<br>

## 武功秘籍

文章主要參考：[[AWS IAM] 學習重點節錄(2) - IAM Policy](https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/)

下方文章主要都圍繞官方的 [Actions, resources, and condition keys for AWS services](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html) 文件上，如果後面沒有附連結，就是在講這份，請大家可以打開網頁一起看。

<br>

## 什麼是 IAM？

IAM 的全名是 Identity and Access Management，從字面上來看就可以得知，它是用來管理 AWS 資源的身份和存取權限的服務。透過 IAM，您可以控制對 AWS 資源的存取權限，以及對這些資源的操作權限。也就是 `Who (身份) 可以 Do (操作) What (資源)` 的一個服務。

<br>

那我們先看一下 UI 妳

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

## 參考資料

[AWS IAM] 學習重點節錄(2) - IAM Policy：[https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/](https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/)

Actions, resources, and condition keys for AWS services：[https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html)
