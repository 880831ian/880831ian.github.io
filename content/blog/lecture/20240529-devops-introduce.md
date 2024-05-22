---
title: "朝陽科技大學 - 安全前瞻：網站防護與 DevOps 技術講座 (2024/05/29)"
# type: docs
# toc: false # 暫時不顯示目錄
---

## 講座資訊

{{< callout >}}

- 時間：2024/05/29 19:00 - 21:00
- 地點：朝陽科技大學 & Google Meet [線上連結](https://meet.google.com/aiq-qcoy-wid)
- Google Developer Student Clubs 相關連結：[https://gdsc.community.dev/events/details/developer-student-clubs-chaoyang-university-of-technology-taichung-taiwan-presents-wang-zhan-fang-hu-yu-devopsji-shu-jiang-zuo-ft-zhuang-pin-yi-gong-cheng-shi/](https://gdsc.community.dev/events/details/developer-student-clubs-chaoyang-university-of-technology-taichung-taiwan-presents-wang-zhan-fang-hu-yu-devopsji-shu-jiang-zuo-ft-zhuang-pin-yi-gong-cheng-shi/)

{{< /callout >}}

<br>

## 雲端和地端的差別

首先，想問一下大家，你們知道雲端和地端的差別嗎？

<br>

### 地端

我們先來説説「地端」是什麼？地端的英文是 [On-Premises](https://en.wikipedia.org/wiki/On-premises_software)，簡稱 On-Prem，代表所有安裝、運行都在個人電腦或公司自己內部的伺服器上所執行的運行方式。

最好的例子就是你電腦裡面裝的 Windows 作業系統、Office 文書軟體、或是公司內部的 ERP 系統等等。這些軟體都是安裝在你的電腦或公司伺服器上，並且由你自己或公司的 IT 團隊負責管理，所以這也是目前最普遍的ㄧ種軟體運行方式。

<br>

#### 地端的優點是什麼？

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

{{< figure src="/lecture/20240529-devops-introduce/cloud.png" width="700" caption="如何使用雲端 [圖片來源](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/)" >}}

<br>

雲端得以實現，是因為一種稱為虛擬化的技術。虛擬化允許在一台實體的電腦上建立許多模擬「虛擬」電腦，其行為如同具有自己硬體的實體電腦，這類電腦的術語稱為[虛擬機器](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-a-virtual-machine/)。虛擬主機簡單介紹到這邊，後面的 [Docker 介紹](#docker-介紹) 會在提到。

一個資料中心有多台的伺服器，一個伺服器可以執行許多虛擬「伺服器」，能夠為許多組織或是客戶提供不同的服務，即使個別伺服器故障，雲端供應商會在多個機器及多個區域備份服務。[Google Data Center 影片連結](https://www.youtube.com/watch?v=zDAYZU4A3w0&ab_channel=GoogleCloudTech)

<br>

{{< figure src="/lecture/20240529-devops-introduce/google-data-center.webp" width="700" caption="Google Data Center [圖片來源](https://www.youtube.com/watch?v=kd33UVZhnAA&ab_channel=GoogleCloudTech)" >}}

<br>

雲端又可以分為以下幾種：

#### 公有雲（Public Cloud）

故名思義，公有雲由第三方雲端服務商提供，如 Google Cloud、AWS、Azure 等。這些服務商提供的平台允許用戶租用伺服器、資料庫、儲存空間等資源，用戶只需一組帳號和密碼即可訪問。公有雲的特點包括：

- **彈性和可擴展性**：用戶可以根據需求動態調整資源的規模。
- **成本效益**：按需付費模式，企業可以根據實際使用情況支付費用。[Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator?hl=zh_tw)
- **全球覆蓋**：可選擇部署的地區範圍廣泛。[Goolge Cloud Service Health](https://status.cloud.google.com/index.html)

除了彈性和成本效益，最重要的還有全球部署和服務商管理安全和維護及 SLA 的優勢。[Google Cloud Platform Service Level Agreements](https://cloud.google.com/terms/sla)

<br>

{{< figure src="/lecture/20240529-devops-introduce/Top-10-Cloud-Service-Providers-Globally-in-2022.jpg.webp" width="600" caption="Top 10 Cloud Service Providers Globally in 2024 [圖片來源](https://dgtlinfra.com/top-cloud-service-providers/)" >}}

<br>

#### 私有雲（Private Cloud）

私有雲是由企業自建或由第三方供應商為特定企業提供的雲端基礎設施，僅供企業內部使用。其特點包括：

- **專屬使用**：企業獨享全部資源，不與其他用戶共享。[Spot VM](https://cloud.google.com/spot-vms?hl=zh-tw)
- **高可控和安全性**：企業可以自行管理和控制數據，確保數據安全性和合規性。
- **高成本**：需要自行購買和維護硬體和基礎設施，初期投資和運營成本較高。

<br>

{{< figure src="/lecture/20240529-devops-introduce/akamai-public-private-cloud.png" width="600" caption="What Is Private Cloud? [圖片來源](https://www.akamai.com/glossary/what-is-private-cloud)" >}}

<br>

聽完公有雲和私有雲的介紹，有沒有可以結合兩種雲端的優點，來達到更好的效果呢？

#### 混合雲（Hybrid Cloud）

混合雲是一種結合了公有雲和私有雲優勢的雲端架構，允許企業在公有雲和私有雲之間靈活地配置和管理資源。這意味著企業可以將一些工作負載和應用程式部署在公有雲上，以利用其彈性和成本效益，同時將敏感資料和關鍵業務應用部署在私有雲或內部資料中心，以確保安全性和控制。

- **彈性和可擴展性**：企業可以根據需求動態調整公有雲和私有雲的使用比例。例如，當工作負載增加時，可以暫時使用公有雲資源來擴展運算能力，而不必立即擴展內部基礎設施。

- **成本效益**：通過利用公有雲的按需付費模式，企業可以降低運營成本，避免在私有雲上過度投資。而私有雲則可以用來運行穩定且持久的工作負載。

- **安全性和合規性**：敏感數據和應用可以保存在私有雲或內部資料中心，以滿足企業的安全和合規要求。同時，非敏感的工作負載可以部署在公有雲上，以利用其便利性和成本優勢。

- **高可用性和災難恢復**：混合雲架構允許企業在公有雲和私有雲之間設置冗餘和備份方案，增強系統的高可用性和災難恢復能力。

所以混合雲也是目前最多企業採用的雲端架構，可以兼顧公有雲和私有雲的優勢，並根據實際需求靈活調整資源的使用。

<br>

### 雲端還是地端好，要如何選擇？

沒有一個明確的答案，要看你的需求和預算，以及公司的規模和發展方向。

假設你是一個小型公司，可能沒有太多資金去購買伺服器和軟體，也沒有太多人力去管理、或是者你的公司業務是跨國的，需要在全球提供服務以及需求高可用性，或是長時間有大量的流量，例如：遊戲公司，那麼雲端就是一個不錯的選擇。

如果公司資料敏感性高，需要自行控制資料存取權限，或是公司有自己的 IT 團隊，有能力管理伺服器和軟體，那麼地端就是一個不錯的選擇。

<br>

我們了解了雲端和地端的差別，接下來了解一下有什麼技術可以幫助我們更好的部署我們的服務呢？

## Docker 和 Kubernetes 概念介紹

<br>

### Docker 介紹

Docker 是一種軟體平台，它可以快速建立、測試和部署應用程式。為什麼可以快速建立呢？因為 Docker 會將軟體封裝到名為『容器』的標準單位。其中會包含程式庫、系統工具、程式碼、執行軟體所需的所有項目。 剛剛有提到容器 (Container)，是一種虛擬化技術，它高效率虛擬化及易於遷移和擴展的特性，非常適合現代雲端的開發及佈署。

那 Container 與傳統的虛擬機 (VM) 有什麼差別呢？我們來看看下面這張圖

<br>

{{< figure src="/lecture/20240529-devops-introduce/container-vm.png" width="700" caption="Container 與 VM 的差異 [圖片來源](https://www.todaysoftmag.com/article/2576/container-based-micro-services-in-the-cloud)" >}}

<br>

可以看到 Container 是以應用程式為單位，而 VM 則是以作業系統為單位。雖然本質來說兩者都是運行在有實體的硬體機器上，但 Container 是一個封裝了相依性資源與應用程式的執行環境。VM 則是一個需要配置好 CPU、RAM 與 Storage 的作業環境，為了更好的做區別，我把 Container、VM 兩個差別用表格來說明：

<br>

|     比較     |                                                                                                       容器 (Container)                                                                                                        |                                                                             虛擬機(VM)                                                                              |
| :----------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|     單位     |                                                                                                           應用程式                                                                                                            |                                                                              作業系統                                                                               |
|   適用服務   |                                                                                                        多使用於微服務                                                                                                         |                                                                          使用較大型的服務                                                                           |
|   硬體資源   |                                                                                              是以程式為單位，需要的硬體資源很少                                                                                               |                                                     VM 會先佔用 CPU、RAM 等等硬體資源，不管有沒有使用都會先佔用                                                     |
|   造成衝突   |                                                                               Container 間是彼此隔離的，因此在同一台機器可以執行不同版本的服務                                                                                |                                                                     會因為版本不同造成環境衝突                                                                      |
| 系統支援數量 |                                                                                                      單機支援上千個容器                                                                                                       |                                                                           一般最多幾十個                                                                            |
|     優點     |                                              Image 較小，通常都幾 MB<br>啟動速度快，通常幾秒就可以生成一個 Container<br>更新較為容易，只需要利用新的 Image 重新啟動就會更新完了                                               |     因為硬體層以上都虛擬化，因此安全性相對較高<br>系統選擇較多，在 VM 可以選擇不同的 OS<br>不需要降低應用程式內服務的耦合性，不需要將程式內的服務個別拆開來部署     |
|     缺點     | 安全性較 VM 差，因為環境與硬體都與本機共用<br>在同一台機器中，每一個 Container 的 OS 都是相同的，無法一個為 Windows、一個為 Linux，還是依賴 Host OS<br>Container 通常會切成微服務的方式作部署，在各元件中的網路連結會比較複雜 | Image 的大小通常 GB 以上，比 Container 大很多<br>啟動速度通常要花幾分鐘，因此服務重啟速度較慢<br>資源使用較多，因為不只程式本身，還要將一部分資源分給 VM 的作業系統 |

<br>

### Docker 小總結

- **更快速的交付和部署**：對於開發和維運人員來說，最希望就是一次建立或設定，可以再任意地方正常運行。開發者可以使用一個標準的映像檔來建立一套開發容器，開發完成之後，維運人員可以直接使用這個容器來部署程式。Docker 容器很輕很快！容器的啟動時間都是幾秒中的事情，大量地節約開發、測試、部署的時間。

- **更有效率的虛擬化**：Docker 容器的執行不需要額外的虛擬化支援，它是核心層級的虛擬化，因此可以實作更高的效能和效率。

- **更輕鬆的遷移和擴展**：Docker 容器幾乎可以在任意的平台上執行，包括實體機器、虛擬機、公有雲、私有雲、個人電腦、伺服器等。 這種兼容性可以讓使用者把一個服務從一個平台直接遷移到另外一個。

- **更簡單的管理**：使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分發和更新，從而實作自動化並且有效率的管理

<br>

### Docker 三大元素

Docker 是由這三個東西所組成，了解後就可以知道整個 Docker 的生命週期。

- 映像檔 (Image)
- 容器 (Container)
- 倉庫 (Repository)

#### 映像檔 (Image)

Docker 映像檔是一個唯獨的模板。

例如：一個映像檔可以用一個 Ubuntu Linux 作業系統當作基底，裡面只安裝 Nginx 或使用者想用的套件等。

我們會使用映像檔來建立 Docker 的容器 (Container) ，Docker 也提供很簡單的機制來建立映像檔或是更新現有的映像檔，也可以去下載別人已經做好的映像檔。

<br>

#### 容器 (Container)

Docker 是利用容器來執行應用程式。

每一個容器都是由映像檔所建立的的執行程式。它可以被啟動、開始、停止、刪除。且每一個容器都是相互隔離的，不會相互影響。

<br>

#### 倉庫 (Repository)

倉庫是集中放置映像檔的所在地，倉庫分為公開倉庫 (Public) 和 私有倉庫 (Private) 兩種形式。最大的倉庫註冊伺服器當然是 [Docker hub](https://hub.docker.com/)，存放數量龐大的映像檔供使用者下載，使用者也可以在本地網路內建立一個私有倉庫。可以將倉庫的概念理解成跟 Git 相似。

<br>

這樣講還是有點抽象，我們來看一個實際的例子：

首先我們先看一下 Docker 的 Logo，你們覺得這是什麼動物呢？

<br>

{{< figure src="/lecture/20240529-devops-introduce/Docker.png" width="600" caption="Docekr Logo [圖片來源](https://logowik.com/docker-vector-logo-2767.html)" >}}

<br>

沒錯就是鯨魚，但你們有沒有注意圖片有三個大元素呢？

- 海洋
- 鯨魚
- 貨櫃

海洋代表的是 Docker 的執行環境，不論是在哪個海洋 (計算環境)，都可以執行，鯨魚代表的是 Docker 這個平台 (或可以指 Image)，而貨櫃代表的是 Docker 的容器，也就是說我們可以有很多個 Container 在同一個 Image 上執行，且互不影響，能夠快速的部署和遷移，並在不同的環境中執行。

<br>

其他 Docker 詳細介紹可以參考我之前寫的文章：[Docker 介紹 (如何使用 Docker-compose 建置 PHP+MySQl+Nginx 環境)](../../docker/docker)

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

企業如何挑選合適的雲端服務？公有雲、私有雲差異比較總整理：[https://aws.amazon.com/tw/events/taiwan/techblogs/public-cloud-private-cloud/](https://aws.amazon.com/tw/events/taiwan/techblogs/public-cloud-private-cloud/)

給資料科學家的 Docker 指南：3 種活用 Docker 的方式（上）：[https://leemeng.tw/3-ways-you-can-leverage-the-power-of-docker-in-data-science-part-1-learn-the-basic.html](https://leemeng.tw/3-ways-you-can-leverage-the-power-of-docker-in-data-science-part-1-learn-the-basic.html)
