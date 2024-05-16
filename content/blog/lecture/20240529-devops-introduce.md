---
title: "朝陽科技大學 - 安全前瞻：網站防護與 DevOps 技術講座 (2024/05/29)"
# type: docs
# toc: false # 暫時不顯示目錄
---

## 雲端和地端的差別

首先，想問一下大家，你們知道雲端和地端的差別嗎？

<br>

### 地端

我們先來説説「地端」是什麼？地端的英文是 [On-Premises](https://en.wikipedia.org/wiki/On-premises_software)，簡稱 On-Prem，代表所有安裝、運行都在個人電腦或公司自己內部的伺服器上所執行的運行方式。

最好的例子就是你電腦裡面裝的 Windows 作業系統、Office 文書軟體、或是公司內部的 ERP 系統等等。這些軟體都是安裝在你的電腦或公司伺服器上，並且由你自己或公司的 IT 團隊負責管理，所以這也是目前最普遍的ㄧ種軟體運行方式。

<br>

#### 地端的好處是什麼？

1. **控制權**：你可以完全控制你的伺服器和軟體，不用擔心第三方服務商的服務品質，想把機器擺在哪裡就放哪裡、用什麼型號都可以自己決定。
2. **安全性**：企業或是個人可以自行控制資料的存取權限，不用擔心第三方服務商的資料外洩問題。
3. **成本**：一次性購買伺服器和軟體，不需要每個月支付雲端服務費用。

<br>

#### 地端的缺點是什麼？

1. **成本**：一次性購買伺服器和軟體，需要花費大量的資金，且不確定是否符合需求使用。
2. **維護**：需要自行管理伺服器和軟體，包括硬體維護、軟體更新、資料備份等等。
3. **擴充性**：當使用者或是需求增加時，需要自行擴充伺服器和軟體，且需要花費大量的時間和金錢。

<br>

那相信大家已經大致了解地端的定義了，接下來我們來看看「雲端」是什麼？

### 雲端

「雲端」的英文是 Cloud，不需要安裝在個人電腦或公司伺服器上，而是透過網際網路存取的伺服器，以及在這些伺服器上執行的軟體或是資料庫。常見的例子有 Google 雲端硬碟、Dropbox、或是 Google Cloud、AWS、Azure 等雲端服務商。

<br>

{{< figure src="/lecture/20240529-devops-introduce/1.png" width="700" caption="如何使用雲端 [圖片來源](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/)" >}}

<br>

雲端得以實現，是因為一種稱為虛擬化的技術。虛擬化允許在一台實體的電腦上建立許多模擬「虛擬」電腦，其行為如同具有自己硬體的實體電腦，這類電腦的術語稱為[虛擬機器](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-a-virtual-machine/)。虛擬主機簡單介紹到這邊，後面的 [Docker 介紹](#docker-介紹) 會在提到。

一個資料中心有多台的伺服器，一個伺服器可以執行許多虛擬「伺服器」，能夠為許多組織或是客戶提供不同的服務，即使個別伺服器故障，雲端供應商會在多個機器及多個區域備份服務。[Google Data Center 影片連結](https://www.youtube.com/watch?v=zDAYZU4A3w0&ab_channel=GoogleCloudTech)

<br>

{{< figure src="/lecture/20240529-devops-introduce/1.webp" width="700" caption="Google Data Center [圖片來源](https://www.youtube.com/watch?v=kd33UVZhnAA&ab_channel=GoogleCloudTech)" >}}

<br>

### 混合雲

混合雲 (Hybrid Cloud)

<br>

### 雲端還是地端好，要如何選擇？

沒有一個明確的答案，要看你的需求和預算，以及公司的規模和發展方向。

假設你是一個小型公司，可能沒有太多資金去購買伺服器和軟體，也沒有太多人力去管理、或是者你的公司業務是跨國的，需要在全球提供服務以及需求高可用性，有臨時大量的流量，那麼雲端就是一個不錯的選擇。

<br>

## Docker 介紹和 Kubernetes 概念介紹

<br>

## 什麼是 CICD

<br>

## 如何最快恢復網站

<br>

## 網站維運日常

<br>

## 要走軟體工程師需要具備哪些技能？

當然，想成為一個軟體工程師，你一定要會程式語言，例如 PHP、Python、JavaScript 等等

也可以看看 stackoverflow 每年的調查報告，了解目前最流行的程式語言是哪些：
https://survey.stackoverflow.co/2023/#most-popular-technologies-language

但除了這些基本的技能外，還需要具備以下幾項技能：

<br>

### Git 版本控制

Git 可以說是現代軟體工程師必備的技能之一，可以幫助你管理程式碼的版本，也是你跟其他工程師協作的最好工具。

<br>

### 寫 Blog

<br>

### 溝通能力

自我學習

<br>

## 參考資料

地端是什麼？可以吃嗎？：[https://wanchunghuang.com/what-is-on-premises/](https://wanchunghuang.com/what-is-on-premises/s)

雲端 VS 地端，數位轉型怎麼選？：[https://blog.hackmd.io/zh/blog/2022/02/22/cloud-vs-on-premises](https://blog.hackmd.io/zh/blog/2022/02/22/cloud-vs-on-premises)

什麼是雲端？ | 雲端定義： [https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/)
