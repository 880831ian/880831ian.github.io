---
title: "Amazon Virtual Private Cloud (VPC) 介紹"
type: docs
weight: 95
description: Amazon Virtual Private Cloud (VPC) 介紹
images:
  - aws/vpc-introduce/og.webp
date: 2024-09-02
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

此篇單純是我自己學習 AWS VPC 的筆記，主要是為了熟悉 AWS 的 VPC，了解 VPC 包含哪些元件，以及如何建立 VPC，並且如何設定 VPC 的相關設定。

<br>

## 武功秘笈

文章主要參考：[AWS CSA Associate 學習筆記 - VPC(Virtual Private Cloud) Part 1](https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/)，這篇文章寫得很詳細，當然，我會加上一些自己的理解跟測試範例，如果有興趣的話，可以去看看原文。

## 什麼是 VPC？

VPC 的全名是 Virtual Private Cloud，可以在 AWS 雲端內配置邏輯上隔離的虛擬網路。透過建立自己的 VPC，可以完全控制網路環境，包括定義 IP 位址範圍、子網路、路由表和連接選項的能力。

每個 AWS 帳號包含每個 AWS 區域的預設 VPC。預設 VPC 有先設置好一些設定，可以快速啟動資源的便捷選項。但是預設 VPC 沒辦法符合長期的網路需求，這時候就需要自己建立 VPC。

建立額外的 VPC 還有很多優勢，例如：可以按照部門或業務隔離工作負載，可以設定更多的安全性控制 (Security Group & Network ACLs)，可以設定更多的網路設定等等。

還可以幫 VPC 建立 VPN 連線，將地端網路與 AWS VPC 連接起來，變成一個 Hybrid Cloud 架構。

<br>

{{< figure src="/aws/vpc-introduce/1.png" width="600" caption="VPC 架構 [什麼是 Amazon VPC](https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html)" >}}

<br>

從圖片來看，可以知道：

- VPC 會在 Region 層級底下，每個 Region 可以有多個 VPC。
- VPC 是跨 Availability Zone (AZ) 的，可以在不同的 AZ 建立 Subnet。
- VPC 會透過 Internet Gateway (IGW) 連接到 Internet。

<br>

## VPC 元件

所以從上面知道 VPC 架構，那我們來詳細看一下它是由哪些元件組成的，先看細圖：

<br>

{{< figure src="/aws/vpc-introduce/2.jpg" width="800" caption="VPC Data Flow [Virtual Private Cloud](https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/)" >}}

<br>

## 參考資料

什麼是 Amazon VPC：[https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html](https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html)

AWS CSA Associate 學習筆記 - VPC(Virtual Private Cloud) Part 1：[https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/](https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/)

Virtual Private Cloud：[https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/](https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/)
