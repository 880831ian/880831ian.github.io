---
title: "Amazon Virtual Private Cloud (VPC) 介紹"
type: docs
weight: 9995
description: Amazon Virtual Private Cloud (VPC) 介紹
images:
  - aws/vpc-introduce/og.webp
date: 2024-09-02
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

此篇單純是我自己學習 AWS VPC 的筆記，主要是為了熟悉 AWS 的 VPC，了解 VPC 包含哪些元件，以及如何建立 VPC、設定 VPC 等等。

<br>

## 武功秘笈

文章主要參考：[AWS CSA Associate 學習筆記 - VPC(Virtual Private Cloud) Part 1](https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/)，這篇文章寫得很詳細，當然，我會加上一些自己的理解跟測試範例，如果有興趣的話，可以去看看原文。

<br>

## 什麼是 VPC？

VPC 的全名是 Virtual Private Cloud，可以在 AWS 雲端內配置邏輯上隔離的虛擬網路。透過建立自己的 VPC，可以完全控制網路環境，包括定義 IP 位址範圍、子網路、路由表和連接選項的能力。

每個 AWS 帳號包含每個 AWS 區域的預設 VPC。預設 VPC 有先設置好一些設定，方便我們可以快速啟動資源的便捷選項。但是預設 VPC 沒辦法符合長期的網路需求，這時候就需要自己建立 VPC。

建立額外的 VPC 還有很多優勢，例如：可以按照部門或業務隔離工作負載，可以設定更多的安全性控制 (Network ACL & Security Group)，可以設定更多詳細的網路設定。

還可以幫 VPC 建立 VPN 連線，將地端網路與 AWS VPC 連接起來，變成一個 Hybrid Cloud 架構。

<br>

{{< figure src="/aws/vpc-introduce/1.webp" width="550" caption="VPC 架構 [什麼是 Amazon VPC](https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html)" >}}

<br>

從圖片來看，可以知道：

- VPC 會在 Region 層級底下，每個 Region 可以有多個 VPC。
- VPC 是跨 Availability Zone (AZ) 的，可以在不同的 AZ 建立 Subnet。
- VPC 會透過 Internet Gateway (IGW) 連接到 Internet。

<br>

## VPC 元件

從上面可以簡單知道 VPC 架構，那我們來詳細看一下每個組成的元件是什麼，先看細圖：

<br>

{{< figure src="/aws/vpc-introduce/2.webp" width="800" caption="VPC Data Flow [Virtual Private Cloud](https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/)" >}}

<br>

我們從最大的元件開始看：

### Region & VPC

上方淺藍色外框就代表 Region，每個 Region 可以有多個 VPC，範例的 Region 是 eu-west-2。

深藍色外框就代表 VPC，VPC 是在 Region 層級底下。

<br>

### Internet Gateway (IGW) & Virtual Private Gateway (VGW)

要存取 VPC 的方式有兩種，一種是透過 IGW 可以從 Internet 存取 VPC，另一種是透過 VGW 可以從地端網路存取 VPC。

不管從 IGW 或 VGW 存取 VPC，都會先到 Route，Route 會根據 traffic 的來源跟目的地，決定要套用哪個 Route Table Rule。

接著 Route Table 會根據 Rule 決定 traffic 要往哪個方向走，並導向到不同的 Network ACL 來進行流量的控制。

<br>

### Internet Gateway (IGW)

(比較會用到，另外拉出來說)

Internet Gateway (IGW) 有自動水平擴展，以及 HA (High Availability) 的特性，會由 AWS 負責管理，我們不需要特別設定。

沒有對外的頻寬限制。

每個 default VPC 都會有一個 Internet Gateway (IGW)。

<br>

### Route Table

Route Table 是用來指引網路流量要怎麼走，以及去哪裡。Route Table 會有目的地 IP 以及下一站要去哪。

預設情況下，VPC 內部的 traffic 可以在不同的 subnet 之間自由流動，依靠的就是 `local route`。

local route 是預設存在，且無法修改的。

VPC 中可以設定多組的 Route Table，但每個 Subnet 只能指定一組 Route Table，且已經與 Subnet 關聯的 Route Table 不能刪除。

<br>

### Network ACL (NACL)

Network ACL 是網路進到 VPC 的第一道防禦，可以把它想成是一個子網級別的防火牆，用於控制進入和出去子網的流量。

Network ACL 是 stateless，這意味著進入和出去的流量都必須明確允許。如果允許進入流量 (inbound traffic)，出去的流量 (outbound traffic) 必須單獨設定規則允許。

<br>

### Security Group (SG)

是網路進入的第二道防禦，是虛擬防火牆，用於控制 EC2 實例的進入和出去流量。

但是比較不一樣的是 Security Group 是 stateful，這意味著如果允許進入流量 (inbound traffic)，相應的出去回應流量 (outbound response traffic) 也自動被允許，反之亦然。

<b>就算移除 Allow Outbound Traffic 規則，一樣還是會通</b>

<br>

### Subnet (Public & Private)

Subnet 是 VPC 的子網路，可以在不同的 Availability Zone (AZ) 建立 Subnet。

Subnet 又分成 Public Subnet 跟 Private Subnet，Public Subnet 可以連到 Internet，Private Subnet 則不能。

但如果 Private Subnet 需要連到 Internet (例如 EC2 更新套件版本等)，則需要透過 NAT Gateway 來達成 (只有單向，沒辦法讓外部網路連進來)。

Subnet 跟 Subnet 之間要溝通，則需要透過 Route Table 來指引路線。

<br>

## VPC IP 範圍

目前 AWS 支援的 IP 範圍是根據 RFC 1918 標準。以下是可用的私有 IP 範圍：

- 10.0.0.0 - 10.255.255.255（10.0.0.0/8，總共 16,777,216 個 IP 地址）
- 172.16.0.0 - 172.31.255.255（172.16.0.0/12，總共 1,048,576 個 IP 地址）
- 192.168.0.0 - 192.168.255.255（192.168.0.0/16，總共 65,536 個 IP 地址）

<b>一個 VPC 最多可以使用 5 個 Ipv4 CIDR 設定，但一般只會設定一個</b>

<br>

## Default VPC

每個 AWS 帳號都會有一個 Default VPC，這個 VPC 是 AWS 預設給你的，裡面已經設定好一些設定，方便你可以快速啟動資源。

在 Default VPC 中，會在每個 Availability Zone (AZ) 建立好 Subnet。

每個 Subnet 都會有一組可以連到 internet 的 Route Table，所以每個 Subnet 都預設具備連網的功能

每個 Subnet 都會有一個 Internet Gateway 存在以及連接，因此 Default VPC 所有 Subnet 都屬於 Public Subnet。

<br>

## VPC Peering

當不同的 VPC 想要連接時，可以透過 VPC Peering 來連接，VPC Peering 是一種私有的連接，可以在兩個 VPC 之間傳輸流量，使用 Private IP 將兩個 VPC 連接起來。

不同的 VPC 的 CIDR 不能重疊，否則無法建立 VPC Peering。

VPC Peering 不限制同一個帳號，可以跟其他帳號的 VPC 連接。

也可以跨 Region，這種稱為 `Inter-Region VPC Peering`

Peering 屬於一對一的連接，如果要連接多個 VPC，則需要建立多個 Peering。例如：A 與 B Peering，B 與 C Peering，A 與 C 之間是無法直接溝通。

可以設定整個 VPC 進行 Peering，也可以設定特定 Subnet 進行 Peering。

需要額外設定 Route Table，才可以兩個 VPC 之間可以溝通。

詳細可以看：[What is VPC peering? 什麼是 VPC 對等互連？](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

<br>

## VPC 懶人包

如果對於上面比較詳細的說明有點複雜，這邊我總結一下 VPC 的重點：

### 以網路的角度出發

下面都先用 EC2 來當作 Subnet 底下的後端服務：

- Private Subnet 沒辦法對外連網，外面也連不進來。
- 同一個 Subnet 裡面的 EC2 可以互相溝通，不需要透過其他的元件。
- 不同的 Subnet 要溝通時，則需要透過 Route Table 元件。
- Route Table 功用就是去指引網路流量要怎麼走，以及去哪裡。會有目的地 IP 以及下一站要去哪。會經過 Local 中繼站。
- Public Subnet 讓裡面的 EC2 可以連到 Internet，一樣需要 Route Table 去指引路線，目的地是 Internet，下一站則需要到 Internet Gateway (IGW)。
- Internet Gateway (IGW) 會放在 VPC 層級上面，是 Public Subnet 對外連線的重要元件。
- 如果要讓 Private Subnet 能夠連 Internet，Route Table 下一站則需要到 NAT Gateway。
- NAT Gateway 是設定在 Public Subnet 上面，透過它才能讓 Private Subnet 連到 Internet。整個流程如下：`Private Subnet > Route Table > NAT Gateway > Internet Gateway (IGW) > Internet`，這個流程是單向的，外面的網路還是連不進來 Private Subnet。

<br>

### 以安全性的角度出發

- 外部連線想要進來 Subnet 時，會需要先經過 Network Access Control List (NACL)。
- Network Access Control List (NACL) 是隸屬於 Subnet 層級，NACL 會規範，什麼請求可以進來，什麼請求可以出去 Subnet。
- 進入到 Subnet 後，要在到 EC2 時，會在經過一層保護名為 Security groups (SG)。
- Security groups (SG) 是給 EC2 instance 用的，類似防火牆規範。

Network Access Control List (NACL) 跟 Security groups (SG) 差異
| 差異 | NACL | SG |
| --- | --- | --- |
| 說明 | 是一個子網級別的防火牆，用於控制進入和出去子網的流量。| 是虛擬防火牆，用於控制 EC2 實例的進入和出去流量。|
| 狀態 | stateless，這意味著進入和出去的流量都必須明確允許。如果允許進入流量，出去的流量必須單獨設定規則允許。 | stateful，這意味著如果允許進入流量（inbound traffic），相應的出去回應流量（outbound response traffic）也自動被允許，反之亦然。|

<br>

最後如果懶得看文字，建議大家可以去看這部影片：[AWS 專家教你：打造高效 VPC 網絡（包含 Subnet、IGW、NAT 等）
](https://www.youtube.com/watch?v=0YG3vo78gSM)，6 分鐘的影片，可以快速了解 VPC 的重點。

<br>

## 參考資料

什麼是 Amazon VPC：[https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html](https://docs.aws.amazon.com/zh_tw/vpc/latest/userguide/what-is-amazon-vpc.html)

AWS CSA Associate 學習筆記 - VPC(Virtual Private Cloud) Part 1：[https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/](https://godleon.github.io/blog/AWS/AWS-CSA-associate-VPC-part1/)

Virtual Private Cloud：[https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/](https://salmankhalid85.wordpress.com/aws/virtual-private-cloud/)

【技術分享】Internet Gateway 和 NAT Gateway 的區別在哪? 功能分別是什麼?：[https://medium.com/@awseducate.cloudambassador/%E6%8A%80%E8%A1%93%E5%88%86%E4%BA%AB-internet-gateway-%E5%92%8C-nat-gateway-%E7%9A%84%E5%8D%80%E5%88%A5%E5%9C%A8%E5%93%AA-%E5%8A%9F%E8%83%BD%E5%88%86%E5%88%A5%E6%98%AF%E4%BB%80%E9%BA%BC-b676f62b1d31](https://medium.com/@awseducate.cloudambassador/%E6%8A%80%E8%A1%93%E5%88%86%E4%BA%AB-internet-gateway-%E5%92%8C-nat-gateway-%E7%9A%84%E5%8D%80%E5%88%A5%E5%9C%A8%E5%93%AA-%E5%8A%9F%E8%83%BD%E5%88%86%E5%88%A5%E6%98%AF%E4%BB%80%E9%BA%BC-b676f62b1d31)
