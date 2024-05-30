---
title: "朝陽科技大學 - 安全前瞻：網站防護與 DevOps 技術講座 (2024/05/29)"
type: docs
---

## 講座資訊

{{< callout >}}

- 時間：2024/05/29 19:00 - 21:00
- 地點：朝陽科技大學 & Google Meet [線上連結](https://meet.google.com/aiq-qcoy-wid)
- Google Developer Student Clubs 相關連結：[https://gdsc.community.dev/events/details/developer-student-clubs-chaoyang-university-of-technology-taichung-taiwan-presents-wang-zhan-fang-hu-yu-devopsji-shu-jiang-zuo-ft-zhuang-pin-yi-gong-cheng-shi/](https://gdsc.community.dev/events/details/developer-student-clubs-chaoyang-university-of-technology-taichung-taiwan-presents-wang-zhan-fang-hu-yu-devopsji-shu-jiang-zuo-ft-zhuang-pin-yi-gong-cheng-shi/)
- 簡報：[https://docs.google.com/presentation/d/10yH_zQikWeYz5pyv4st6ICMs6F2F9F_Vks1ngDaNROY/edit?usp=sharing](https://docs.google.com/presentation/d/10yH_zQikWeYz5pyv4st6ICMs6F2F9F_Vks1ngDaNROY/edit?usp=sharing)
- 講座錄影：[https://drive.google.com/file/d/1-TRJ3mAYehmPSr8Sg4p09dA7t4mor4CY/view?usp=drive_link](https://drive.google.com/file/d/1-TRJ3mAYehmPSr8Sg4p09dA7t4mor4CY/view?usp=drive_link)

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

雲端會有很多資料中心，一個資料中心又有很多台的伺服器，一個伺服器可以執行許多虛擬「伺服器」，能夠為許多組織或是客戶提供不同的服務，即使個別伺服器故障，雲端供應商會在多個機器及多個區域備份服務。[Google Data Center 影片連結](https://www.youtube.com/watch?v=zDAYZU4A3w0&ab_channel=GoogleCloudTech)

<br>

{{< figure src="/lecture/20240529-devops-introduce/google-data-center.webp" width="700" caption="Google Data Center [圖片來源](https://www.youtube.com/watch?v=kd33UVZhnAA&ab_channel=GoogleCloudTech)" >}}

<br>

雲端又可以分為以下幾種：

#### 公有雲（Public Cloud）

故名思義，公有雲由第三方雲端服務商提供，如 Google Cloud、AWS、Azure 等。這些服務商提供的平台允許用戶租用伺服器、資料庫、儲存空間等資源，用戶只需一組帳號和密碼即可訪問。公有雲的特點包括：

- **彈性和可擴展性**：用戶可以根據需求動態調整資源的規模。
- **成本效益**：按需付費模式，企業可以根據實際使用情況支付費用。[Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator?hl=zh_tw)
- **全球覆蓋**：可選擇部署的地區範圍廣泛。[Google Cloud 據點](https://cloud.google.com/about/locations?hl=zh-tw#lightbox-regions-map)

除了彈性和成本效益，最重要的還有服務商安全管理和維護及 SLA 的優勢。[Google Cloud Platform Service Level Agreements](https://cloud.google.com/terms/sla)

<br>

{{< figure src="/lecture/20240529-devops-introduce/Top-10-Cloud-Service-Providers-Globally-in-2022.jpg.webp" width="600" caption="Top 10 Cloud Service Providers Globally in 2024 [圖片來源](https://dgtlinfra.com/top-cloud-service-providers/)" >}}

<br>

#### 私有雲（Private Cloud）

私有雲是由企業自建或由第三方供應商為特定企業提供的雲端基礎設施，僅供企業內部使用。其特點包括：

- **專屬使用**：企業獨享全部資源，不與其他用戶共享。
- **高可控和安全性**：企業可以自行管理和控制數據，確保數據安全性和合規性。
- **高成本**：需要自行購買和維護硬體和基礎設施，初期投資和運營成本較高。

<br>

{{< figure src="/lecture/20240529-devops-introduce/akamai-public-private-cloud.png" width="600" caption="What Is Private Cloud? [圖片來源](https://www.akamai.com/glossary/what-is-private-cloud)" >}}

<br>

聽完公有雲和私有雲的介紹，有沒有可以結合兩種雲端的優點，來達到更好的效果呢？

#### 混合雲（Hybrid Cloud）

混合雲是一種結合了公有雲和私有雲優勢的雲端架構，允許企業在公有雲和私有雲之間靈活地配置和管理資源。這意味著企業可以將一些工作負載和應用程式部署在公有雲上，以利用其彈性和成本效益，同時將敏感資料和關鍵業務應用部署在私有雲或內部資料中心，以確保安全性和控制。

所以以下幾個就是混合雲的優點：

- **彈性和可擴展性**：企業可以根據需求動態調整公有雲和私有雲的使用比例。例如，當工作負載增加時，可以暫時使用公有雲資源來擴展運算能力，而不必立即擴展內部基礎設施。

- **成本效益**：通過利用公有雲的按需付費模式，企業可以降低運營成本，避免在私有雲上過度投資。而私有雲則可以用來運行穩定且持久的工作負載。

- **安全性和合規性**：敏感數據和應用可以保存在私有雲或內部資料中心，以滿足企業的安全和合規要求。同時，非敏感的工作負載可以部署在公有雲上，以利用其便利性和成本優勢。

- **高可用性和災難恢復**：混合雲架構允許企業在公有雲和私有雲之間設置備份方案，增強系統的高可用性和災難恢復能力。

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

- **更簡單的管理**：使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式去更新，從而實作自動化並且有效率的管理。

<br>

### Docker 四大元素

Docker 是由這四個東西所組成，了解後就可以知道整個 Docker 的生命週期。

- Dockerfile
- 映像檔 (Image)
- 容器 (Container)
- 倉庫 (Repository)

<br>

#### Dockerfile

開發人員在使用 Docker 時發現，大多現成的 Docker 映像檔無法滿足他們的需求，因此需要一種能夠生成映像檔的工具。Dockerfile 是一種簡易的文件檔，裡面包含了建立新映像檔所需的指令。

Dockerfile 語法主要由 Command（命令）和 Argument （參數選擇）兩大元素組成。以下是一個簡易的 Dockerfile 示意圖：

命令式語法＋選擇參數（Command + Argument）

{{< figure src="/lecture/20240529-devops-introduce/dockerfile.png" width="300" caption="Dockerfile [圖片來源](https://www.omniwaresoft.com.tw/product-news/docker-news/docker-introduction/#Repository%EF%BC%88%E5%80%89%E5%BA%AB%EF%BC%89)" >}}

<br>

#### 映像檔 (Image)

Docker 映像檔是一個唯獨的模板。

例如：一個映像檔可以用一個 Ubuntu Linux 作業系統當作基底，裡面只安裝 Nginx 或使用者想用的套件等。

我們會使用映像檔來建立 Docker 的容器 (Container) ，Docker 也提供很簡單的機制來建立映像檔或是更新現有的映像檔，也可以去下載別人已經做好的映像檔。

<br>

{{< figure src="/lecture/20240529-devops-introduce/image.png" width="450" caption="Image [圖片來源](https://ragin.medium.com/docker-what-it-is-how-images-are-structured-docker-vs-vm-and-some-tips-part-1-d9686303590f)" >}}

<br>

#### 容器 (Container)

Docker 是利用容器來執行應用程式。

每一個容器都是由映像檔所建立的的執行程式。它可以被啟動、開始、停止、刪除。且每一個容器都是相互隔離的，不會相互影響。

<br>

{{< figure src="/lecture/20240529-devops-introduce/container.png" width="600" caption="Container [圖片來源](https://www.omniwaresoft.com.tw/product-news/docker-news/docker-introduction/#Repository%EF%BC%88%E5%80%89%E5%BA%AB%EF%BC%89)" >}}

<br>

#### 倉庫 (Repository)

倉庫是集中放置映像檔的所在地，倉庫分為公開倉庫 (Public) 和 私有倉庫 (Private) 兩種形式。最大的倉庫註冊伺服器當然是 [Docker hub](https://hub.docker.com/)，存放數量龐大的映像檔供使用者下載，使用者也可以在本地網路內建立一個私有倉庫。可以將倉庫的概念理解成跟 Git 相似。

<br>

{{< figure src="/lecture/20240529-devops-introduce/repository.png" width="600" caption="Repository [圖片來源](https://logowik.com/docker-vector-logo-2767.html)" >}}

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

由於時間關係，其他 Docker 詳細介紹以及資料可以參考我之前寫的文章：[Docker 介紹 (如何使用 Docker-compose 建置 PHP+MySQl+Nginx 環境)](../../docker/docker)

<br>

### Docker 小試身手

為了大家能夠更了解 Docker 的使用，我們來實際操作一下：

1. 首先請大家先到 GitHub 下載範例程式碼：[https://github.com/880831ian/20240529-devops-introduce](https://github.com/880831ian/20240529-devops-introduce)
2. 接著先執行 `docker build -t 0529:latest .` 這個指令，這個指令是用來建立一個 Image，這個 Image 是用來執行 Nginx 的程式碼。
3. 接著執行 `docker run -d -p 8080:80 0529:latest` 這個指令，這個指令是用來執行一個 Container，並且將 Container 的 80 port 對應到本機的 8080 port。
4. 打開瀏覽器，輸入 `http://localhost:8080`，就可以看到 Nginx 的首頁了。
5. 我們先將關閉 Container，自行修改 index.html 的內容，然後再重新執行 `docker build -t 0529:v0.0.1 .` 和 `docker run -d -p 8080:80 0529:v0.0.1`，就可以看到修改後的內容了。
6. 我們下 `docker tag 0529:v0.0.1 880831ian/0529:v0.0.1` 這個指令，將 Image 改成 Docker Hub 帳號跟 Image 名稱，例如：880831ian/0529:v0.0.1。
7. 最後我們下 `docker push 880831ian/0529:v0.0.1` 這個指令，這樣就可以將 Image 上傳到 Docker Hub 上，這樣其他人就可以使用你的 Image 來部署服務了。

<br>

### Docker 運作流程

步驟一、撰寫 Dockerfile

步驟二、將 Dockerfile 建立為 Image

步驟三、將 Image 運行為容器。透過這三個簡單的步驟，就能創建自己的 Docker 容器囉！

<br>

{{< figure src="/lecture/20240529-devops-introduce/run.png" width="900" caption="Docekr 運作流程 [圖片來源](https://medium.com/swlh/understand-dockerfile-dd11746ed183)" >}}

<br>

### Kubernetes 介紹

Kubernetes 也可以叫 K8s，這個名稱來源希臘語，意思是舵手或是飛行員，所以我們可以看到它的 logo 是一個船舵的標誌，之所以叫 K8s 是因為 Kubernetes 的 k 到 s 中間有 8 的英文字母，為了方便，大家常以這個名稱來稱呼它！

<br>

{{< figure src="/lecture/20240529-devops-introduce/k8s-logo.png" width="200" caption="K8s Logo" >}}

<br>

K8s 是一種開源的容器資源調度平台，能夠將部署流程自動化、擴展並管理不同容器間的工作負載。它的構想理念是「Automated container deployment, scaling, and management」，意即透過自動化功能提升應用程式的可靠性和減輕維運負擔，讓開發人員專注於軟體開發任務。

另外 K8s 也是 Google 開發的，並且是 CNCF (Cloud Native Computing Foundation) 的一個專案，所以 K8s 也是一個非常受歡迎的容器管理系統。

<br>

我們之前在 [Docker 介紹](../../../blog/docker/docker/) 文章中，已經有介紹以往傳統虛擬機以及容器化的 Docker 差異以及優點，那當我們在管理容器時，其中一個容器出現故障，則需要啟動另一個容器，如果要用手動，會十分麻煩，所以這時就是 Kubernetes 的厲害的地方了，Kubernetes 提供：

### Kubernetes 優點

- **輕量級**：K8s 的輕量化特性讓應用程式能被輕易地部署至不同環境，例如：地端資料中心、公有雲或其他雲端混合環境。K8s 容器化的本質讓封裝在內的應用程式與其相依的資源能夠緊密結合，從而解決不同平台的兼容問題，並降低在不同基礎架構上部署的難度。
- **服務、系統部屬更方便**：由於容器可在任何容器平台運行，因此無論是同時將多個 Container 部屬到一台機器，或是多個 Container 部屬至多台機器都不是問題。
- **自動化管理，重啟、擴張皆可行**：且 K8s 可自動偵測、管理各 Container 的狀態，若有需要，可對 Container 執行自動擴展。而若偵測到有 Container 發生故障，也可自動重啟以確保服務正確且持續地運行。
- **彈性化運用**：K8s 中每個服務、系統皆可獨立部屬，因此不會因為其中一個系統出現錯誤而影響整個運作，甚至各 Container 也可依各自需求來修改，運用上擁有高度彈性化。

<br>

Kubernetes 是如何幫我們管理以及部署 Container ? 要了解 Kubernetes 如何運作，就要先了解它的元件以及架構：

### Kubernetes 元件介紹

那我們由小的往大的來做介紹：依序是 Pod、Worker Node、Master Node、Cluster

#### Pod

Kubernetes 運作中最小的單位，一個 Pod 會對應到一個應用服務 (Application)，舉例來說一個 Pod 可能會對應到一個 Nginx Server。

- 每個 Pod 都有一個定義文件，也就是屬於這個 Pod 的 yaml 檔。
- 一個 Pod 裡面可以有一個或多個 Container，但一般情況一個 Pod 最好只有一個 Container。
- 同一個 Pod 中的 Containers 共享相同的資源以及網路，彼此透過 local port number 溝通。

<br>

#### Worker Node

Kubernetes 運作的最小硬體單位，一個 Worker Node (簡稱 Node) 對應到一台機器，可以是實體例如你的筆電、或是虛擬機，例如：GCP 上的一台 Computer Engine。

<br>

#### Master Node (Control Plane)

負責各個 Worker Node 的管理，可稱作是 K8S 的發號施令的中樞。

其他更詳細介紹，可以參考我之前寫的文章：[Kubernetes (K8s) 介紹 - 基本](../../..//blog/kubernetes/k8s/#安裝-kubernetes)

<br>

#### Cluster

Cluster 也叫叢集，可以管理眾多機器的存在，在一般的系統架設中我們不會只有一台機器而已，通常都是多個機器一起運行，在沒有 Kubernetes 的時候就必須要土法煉鋼的一台一台機器去更新，但有了 Kubernetes 我們就可以透過 Cluster 進行控管，只要更新 Master 旗下的機器，也會一併將更新的內容放上去，十分方便。

<br>

{{< figure src="/lecture/20240529-devops-introduce/k8s.jpg" width="550" caption="K8s 元件 [圖片來源](https://cwhu.medium.com/kubernetes-basic-concept-tutorial-e033e3504ec0)" >}}

<br>

### Kubernetes 小試身手

為了大家能夠更了解 Kubernetes 的使用，我們來實際操作一下：
(但由於 K8s 建立以及部署需要一些時間，所以這邊會直接拿我擁有的環境做測試，其他更詳細的操作可以參考我之前寫的文章：[Kubernetes (K8s) 介紹 - 基本](../../../blog/kubernetes/k8s/)、[Kubernetes (K8s) 介紹 - 進階 (Service、Ingress、StatefulSet、Deployment、ReplicaSet、ConfigMap)](../../../blog/kubernetes/k8s-advanced/))

首先，我們接續上面的 [Docker 小試身手](#docker-小試身手) 的範例程式碼：

1. 先執行 `docker buildx build --platform=linux/amd64 -t 880831ian/0529-arm64:latest .`，將 Image 建立起來 (這邊會多 buildx 跟 platform 是因為我的電腦是 Mac M 系列處理器，系統架構是 arm64，所以要放到 GKE 上面跑，需要多指令平台)。

2. 接著執行 `docker push 880831ian/0529-arm64:latest`，將 Image 上傳到 Docker Hub 上。

3. 進入 k8s 資料夾，裡面有幾個檔案，分別是：namespace.yaml、deployment.yaml、service.yaml、ingress.yaml。

4. 先執行 `kubectl apply -f namespace.yaml`，這個指令是用來建立一個 namespace，這樣我們就可以將我們的服務放到這個 namespace 下。

5. 接著執行 `kubectl apply -f .`，將其他服務也建立到 GKE 上。

6. 最後我們打開瀏覽器，輸入 [https://myapp.pin-yi.me](https://myapp.pin-yi.me/)，就可以看到我們寫的首頁了。 (此為範例，講座結束後會關閉服務)

<br>

看完 Kubernetes 部署服務的方式，是不是覺得有點麻煩呢？還需要手動去部署，這時候就需要 CI/CD 來幫助我們自動化部署服務了！

## 什麼是 CI/CD

一樣先來了解一下為什麼要 CI/CD？

目前大多數的企業公司都是採用敏捷開發的方式，所以在追求快速開發、快速部署，以及大量的測試時會消耗很多開發人員的時間和精力，所以這時候就需要 CI/CD 來幫助我們自動化部署服務。

<br>

接著再介紹 CI/CD 之前還有一個名詞要跟大家介紹，那就是 DevOps。

### 什麼是 DevOps

DevOps 是一種結合軟體開發人員 (Development) 和 IT 運維技術人員 (Operations) 的文化，目的是縮短開發和運營之間的距離，並且提高開發和運營的效率。而 CI/CD 工具就是為了此概念產生的自動化工具，透過持續整合 (CI) 和持續部署 (CD) 的方式，在開發階段能自動協助開發人員偵測程式碼問題，並部署到 Server 上。

<br>

{{< figure src="/lecture/20240529-devops-introduce/cicd_cycle.png" width="700" caption="CI/CD cycle" >}}

<br>

### CI (Continuous Integration) 持續整合

持續整合，顧名思義，就是當開發人員完成一個階段性的程式碼後就經由自動化工具測試、驗證，協助偵測程式碼問題，並建置出即將部署的版本（Build）。

<br>

### CD (Continuous Deployment) 持續部署

持續部署可以說是 CI 的下一階段，經過 CI 測試後所構建的程式碼可以透過 CD 工具部署至伺服器，減少人工部署的時間。

<br>

### CI/CD 常用的工具

#### GitHub

GitHub 算是目前最受歡迎的程式碼管理平台，其 CI/CD 服務稱為 GitHub Action，提供了多項控制 API，能夠幫助開發者編排、掌握工作流程，在提交程式碼後自動編譯、測試並部署至伺服器，也提供了很多現成的 Action 可以使用。

#### GitLab

GitLab 算是公司企業內部比較主流的程式碼管理平台，其 CI/CD 服務稱為 GitLab CI/CD，其 CI/CD Pipeline 功能簡單又實用，使用者只需要設定於專案根目錄下的「.gitlab-ci.yml」檔，便可以開始驅動各種 Pipeline 協助您完成自動化測試及部署

<br>

#### CI/CD 小試身手

我們今天會使用 GitGub Action 來進行 CI/CD 範例

這邊也跟上面 [Kubernetes 小試身手](#kubernetes-小試身手) 一樣，由於 K8s 建立以及部署需要一些時間，所以這邊會直接拿我擁有的環境做測試。

首先，我們接續上面的 [Docker 小試身手](#docker-小試身手) 的範例程式碼：

1. 打開 [.github/workflows/google.yml](https://github.com/880831ian/20240529-devops-introduce/blob/main/.github/workflows/google.yml) 這個檔案，裡面是我已經寫好的 GitHub Action 的流程。

這邊說明一下這個檔案的內容 (下面拆開說明)：

```yaml
name: Build and Deploy to GKE

on:
  push:
    branches: ["main"]

env:
  PROJECT_ID: pin-yi-project
  GAR_LOCATION: asia-east1
  GKE_CLUSTER: cluster
  GKE_ZONE: asia-east1-b
  DEPLOYMENT_NAME: myapp
  REPOSITORY: pin-yi-image
  IMAGE: myapp
  NAMESPACE: myapp

jobs:
  setup-build-publish-deploy:
    name: Setup, Build, Publish, and Deploy
    runs-on: ubuntu-latest
    environment: production
```

這邊主要是說明這個 Action 的名稱，以及當 push 到 main branch 時，就會觸發這個 Action，還有對應的 env，env 包含了 GCP 專案 ID、Image 儲存的地點、GKE 的 Cluster 名稱、GKE 的 Zone、部署的名稱、Image 的名稱、Namespace 的名稱。

接著是 job 的設定，設定名稱、要用什麼 image 當做基底，以及環境等等。job 可以把它理解成每個要做的小事情，例如 Build Image、Push Image、Deploy Image 等等。

<br>

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v3

  - id: "auth"
    uses: "google-github-actions/auth@v2"
    with:
      credentials_json: "${{ secrets.GCP_CREDENTIALS }}"

  - name: Docker configuration
    run: |-
      echo '${{ secrets.GCP_CREDENTIALS }}' | docker login -u _json_key --password-stdin https://$GAR_LOCATION-docker.pkg.dev

  - name: Set up GKE credentials
    uses: google-github-actions/get-gke-credentials@v2
    with:
      cluster_name: ${{ env.GKE_CLUSTER }}
      location: ${{ env.GKE_ZONE }}

  - name: Build
    run: |-
      docker build \
        --tag "$GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:$(echo $GITHUB_SHA | head -c7)" \
        --build-arg GITHUB_SHA="$GITHUB_SHA" \
        --build-arg GITHUB_REF="$GITHUB_REF" .

  - name: Add Image Tag
    run: docker tag $GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:$(echo $GITHUB_SHA | head -c7) $GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:latest

  - name: Publish
    run: |-
      docker push "$GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:$(echo $GITHUB_SHA | head -c7)" && \
      docker push "$GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:latest"

  - name: Set Image
    run: |-
      kubectl set image deployment/$DEPLOYMENT_NAME \
      myapp="$GAR_LOCATION-docker.pkg.dev/$PROJECT_ID/$REPOSITORY/$IMAGE:$(echo $GITHUB_SHA | head -c7)"  -n $NAMESPACE
  - name: Deploy
    run: |-
      kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE
      kubectl get services -o wide
```

接著就是各種的 job，有先 checkout 程式碼、登入 GCP、登入 Docker、設定 GKE 的 credentials、Build Image、Push Image、Deploy Image 等等。

2. 接著我們嘗試調整 index.html 的內容，然後 push 到 GitHub 上，就可以看到 GitHub Action 會自動幫我們 Build Image、Push Image、Deploy Image 到 GKE 上。

3. 最後我們打開瀏覽器，輸入 [https://myapp.pin-yi.me](https://myapp.pin-yi.me/)，就可以看到我們調整的首頁了。(此為範例，講座結束後會關閉服務)

<br>

## 如何最快恢復網站

如何最快恢復網站，這是一個非常重要的議題，因為網站的停擺會造成業務的損失，也會讓使用者對網站的信任度下降，所以我們需要有一個很好的應急計畫，來應對這種狀況。

<br>

### 網站停擺的原因

那在討論如何恢復前，我們要先了解會導致網站停擺的原因有哪些？

<br>

#### 伺服器供應商故障

這邊的伺服器供應商，如果以雲端的供應商來說，也就是 GCP、AWS、Azure 等等，這些供應商可能會因為硬體故障、軟體錯誤、網路問題等原因而停擺。
當然地端的機器也可能會因為主機的供應商導致網站中斷，例如：Windows 自動更新 xD。

<br>

#### 網站伺服器故障

網站的伺服器當然也可能會因為硬體故障、軟體錯誤、網路問題等原因而停擺。這也是最常見的原因之一。
這個的伺服器也可以是地端的伺服器，也可以是雲端的伺服器。

我們以雲端的 K8s 來舉例，如果 K8s 配置錯誤，會是資源不足，或是 Pod 的數量不夠，都會導致網站停擺。

<br>

#### 網站程式碼錯誤

這邊的程式碼，大家可以把他理解成網站運行的 Code，例如：PHP、Python、JavaScript 等等，有可能程式碼寫了一個無窮迴圈，或是什麼特殊的 Bug，導致資源衝高，也會導致網站停擺。

<br>

#### 網站被駭客攻擊

當然網站會依照使用需求，開放給不同的使用者，如果是一般開在公網上的網站，就有可能會被駭客攻擊，例如：DDos 攻擊、SQL Injection 等等，這些攻擊都有可能導致網站停擺，還有可能導致資料外洩。

<br>

#### 網站被判定成非法網站

這邊的非法網站，是指網站內容違反了法律規定，例如：色情、賭博、毒品等等，這些內容都有可能導致網站被封鎖，或是被檢舉，進而導致網站停擺。

<br>

{{< figure src="/lecture/20240529-devops-introduce/20220830003861.jpg" width="450" caption="網站解析被擋 [圖片來源](https://www.chinatimes.com/realtimenews/20220830003849-260405?chdtv)" >}}

<br>

### 網站恢復(避免)的方法

1. 伺服器供應商故障

雲端供應商故障的發生機率在這幾個裡面來說，算是最少的，因為雲端供應商有很多的備援機制，當然為了避免意外，在有能力之餘，也可以考慮多個雲端供應商來當作備援，這樣就可以避免單一供應商故障。

我們以雲端的 GCP 來舉例，如果 GCP 的服務有問題，可以到 [Google Cloud Status Dashboard](https://status.cloud.google.com/index.html) 來查看目前服務的狀況。

分享一下最近發生的案例：[https://status.cloud.google.com/incidents/xVSEV3kVaJBmS7SZbnre](https://status.cloud.google.com/incidents/xVSEV3kVaJBmS7SZbnre)

另外，GCP 也有提供 SLA，當服務有問題並超出一開始 Goolge 所承諾的服務水準時，可以向 GCP 提出申請，來獲得賠償。

[Google Cloud Platform Service Level Agreements](https://cloud.google.com/terms/sla)

<br>

2. 網站伺服器故障

會出現錯誤的原因有很多，例如：硬體故障、軟體錯誤、網路問題等等，這邊我們可以透過對服務系統進行不同的監控來提早發現問題，並且提早解決。當然也建議多做備份，當網站出現問題時，可以快速的恢復。平常也要演練災難復原計畫。

<br>

3. 網站程式碼錯誤

程式碼的錯誤，要再上線前多做測試，例如：單元測試、整合測試、壓力測試等等，也要先做好程式碼的規範，例如：程式碼的風格、程式碼的註解等等，避免出現一些不必要的問題。

<br>

4. 網站被駭客攻擊

網路安全是很重要的，所以從入口的防火牆設計、黑白名單，再者到程式碼的安全性，例如：SQL Injection、XSS、CSRF 等等，都要做好防護，並且要定期的去更新程式碼裡面使用的套件，才可以避免被駭客攻擊。

另外在雲端的使用上有一點要特別注意，就是權限的設定劃分要明確，確保最小權限原則，避免帳號金鑰外流導致駭客取得權限。

<br>

5. 網站被判定成非法網站

這邊當然不是討論去架設相關的網站，導致封鎖要怎麼辦 (◕ܫ◕)，我們這邊討論的是，不一定真的是架設非法的網站，才會被封鎖，我們來看一下這個新聞：[Google 被刑事局認定「涉及詐騙」網傻眼！台灣大曝真相
](https://www.chinatimes.com/realtimenews/20220830003849-260405?chdtv)

這邊建議使用的網址要是可辨識，不要使用一些奇怪的網址，才可以避免被誤判，以及如果使用雲端供應商提供的 IP 以前，也先可以先去掃描一下 IP 是否有被組織或是單位確認是非法 IP。如果真的被封鎖，可以向相關單位申訴，來恢復網站。

<br>

## 網站維運日常

這邊我就以目前我在公司的經驗來跟大家分享一下，網站維運日常都會做哪些事情：

主要分為以下幾件事情：

### 監控

我們需要做各種的監控，例如：網站的流量、CPU 使用率、記憶體使用率等等，這些都是我們需要監控的項目。除此以外也需要監控 RD 使用的程式語言有對應的指標，例如：PHP 的 FPM 的進程等等指標。

監控的目的是，讓我們服務在出現問題前，就可以提早透過告警知道有可能潛在的問題，並且提早解決。避免被使用者發現問題，進而影響使用者的體驗。

<br>

{{< figure src="/lecture/20240529-devops-introduce/monitoring.png" width="1000" caption="做各種的監控" >}}

<br>

### 查 Log

我們每個服務都需要去紀錄 Log，不管是使用者的操作、系統、還是程式的錯誤，都需要去紀錄下來，這樣當網站出現問題時，我們可以透過 Log 來查找問題，並且解決問題。

當然 Log 也不是一個簡單的議題，Log 數量很多，在不影響效能以及費用的情況下，要怎麼只存精簡且重要的 Log，也是一個需要思考的問題。

<br>

{{< figure src="/lecture/20240529-devops-introduce/log.png" width="1000" caption="查 Log" >}}

<br>

### 協助 RD 調整設定以及排查問題

由於雲端使用 K8s 微服務的架構，所以會有很多 K8s 上的設定，例如：服務入口的 Nginx 倒轉、timeout 設定的問題、IP 是來源哪一個服務等等，有些時候我們還需要配合 RD 來進行排查程式碼，才能真正解決問題。

所以要成為一個優秀的 SRE 也是需要具備一定的程式能力，才能夠更好的協助 RD 來解決問題。

不要變成 Site Restart Engineer 工程師 (雖然有時候真的 restart 就好了 ( ͡° ͜ʖ ͡°))

<br>

{{< figure src="/lecture/20240529-devops-introduce/sre.jpg" width="200" caption="Site Restart Engineer XDDD" >}}

<br>

### 開發能夠在維運上更方便的工具

除了上面說的已經可以讓我們身心俱疲的幾件日常，我們也會開發一些讓我們維護上可以更方便的一些工具，盡量都變成自動化的方式，去減少我們的工作量。

例如：

- 簡單的壓測網站 HTTP 狀態碼工具：[chece_url_http_code](https://github.com/880831ian/check_url_http_code)
- 自動修復 elasticsearch 錯誤工具：[auto_repair_es](https://github.com/880831ian/auto_repair_es)
- 計算哪時候可以下班的工具：[can_get_off_work](https://github.com/880831ian/can_get_off_work) XD

<br>

## 要走軟體工程師需要具備哪些技能？

下面是我準備的小 bonus，是出自我自己的經驗，以及我偷偷觀察我合作的 RD 夥伴們，得出的小小心得，當然這只是我自己的看法，不一定是對的，希望能夠幫助大家。

當然，想成為一個軟體工程師，你一定要會程式語言，例如 PHP、Python、JavaScript 等等，假如你對於未來的發展方向還不確定，或是不知道哪個程式語言是未來的趨勢，可以看看 stackoverflow 每年的調查報告，來了解目前最流行的程式語言是哪些：
https://survey.stackoverflow.co/2023/#most-popular-technologies-language

但除了這個基本的寫 Code 技能外，我還建議能夠提早學習到以下幾項技能，會對你未來的發展有很大的幫助：

<br>

### Git 版本控制

Git 可以說是現代軟體工程師必備的技能之一，可以幫助你管理程式碼的版本，當你的程式碼出現問題時，可以很快的回復到之前的版本，也是你跟其他工程師協作的最好工具。

因為目前，你可能都是自己一個人寫 Code，所以不會碰到同時間有多個人在調整程式時，導致程式碼衝突的問題，當你沒有使用 Git 時，你會不知道這是誰改的，這個是什麼時候改的，這個改動是為了什麼，這樣會讓你的程式碼變得非常雜亂，也會讓你的程式碼變得非常難以維護。

<br>

### 寫 Blog

培養寫 Blog 的習慣，可以幫助你整理自己的知識，也可以幫助你記錄自己的學習過程，像我當初會寫 Blog 還有一個原因，是因為我記憶不好，如果我不寫下來，就會常常忘記一樣的問題 (๑•́ ₃ •̀๑)。

當然在找工作面試時，它也可以當作自己的作品集，讓面試官能夠了解你、並知道你的學習能力。

<br>

### 溝通能力

在職場上，溝通能力是非常重要的一環，因為你不可能一個人在公司裡面工作，你一定會跟其他人合作，所以你要學會如何跟其他人溝通，如何表達自己的想法，如何聆聽別人的意見，這些都是非常重要的技能。

每個行業都是，但我認為軟體工程師更需要這個技能，因為軟體工程師的工作是非常複雜的，你需要跟 PM、RD、QA、UI/UX、SRE 等等不同的部門合作，所以你要學會如何跟不同的人合作，這樣才能夠讓你的工作更順利。

<br>

## 參考資料

地端是什麼？可以吃嗎？：[https://wanchunghuang.com/what-is-on-premises/](https://wanchunghuang.com/what-is-on-premises/s)

雲端 VS 地端，數位轉型怎麼選？：[https://blog.hackmd.io/zh/blog/2022/02/22/cloud-vs-on-premises](https://blog.hackmd.io/zh/blog/2022/02/22/cloud-vs-on-premises)

什麼是雲端？ | 雲端定義： [https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/](https://www.cloudflare.com/zh-tw/learning/cloud/what-is-the-cloud/)

企業如何挑選合適的雲端服務？公有雲、私有雲差異比較總整理：[https://aws.amazon.com/tw/events/taiwan/techblogs/public-cloud-private-cloud/](https://aws.amazon.com/tw/events/taiwan/techblogs/public-cloud-private-cloud/)

給資料科學家的 Docker 指南：3 種活用 Docker 的方式（上）：[https://leemeng.tw/3-ways-you-can-leverage-the-power-of-docker-in-data-science-part-1-learn-the-basic.html](https://leemeng.tw/3-ways-you-can-leverage-the-power-of-docker-in-data-science-part-1-learn-the-basic.html)

Docker 是什麼？Docker 基本觀念介紹與容器和虛擬機的比較：[https://www.omniwaresoft.com.tw/product-news/docker-news/docker-introduction/](https://www.omniwaresoft.com.tw/product-news/docker-news/docker-introduction/)

CI/CD 是什麼？一篇認識 CI/CD 工具及優勢，將日常瑣事自動化：[https://www.wingwill.com.tw/zh-tw/%E9%83%A8%E8%90%BD%E6%A0%BC/%E9%9B%B2%E5%9C%B0%E6%B7%B7%E5%90%88%E6%87%89%E7%94%A8/cicd%E5%B7%A5%E5%85%B7/](https://www.wingwill.com.tw/zh-tw/%E9%83%A8%E8%90%BD%E6%A0%BC/%E9%9B%B2%E5%9C%B0%E6%B7%B7%E5%90%88%E6%87%89%E7%94%A8/cicd%E5%B7%A5%E5%85%B7/)

[DevOps] CI/CD 介紹 - 基礎概念與導入準備：[https://enzochang.com/cicd-introduction/](https://enzochang.com/cicd-introduction/)

Kubernetes（K8s）是什麼？基礎介紹+3 大優點解析：[https://www.metaage.com.tw/news/technology/293](https://www.metaage.com.tw/news/technology/293)

我之前寫過的所有文章：[https://pin-yi.me/blog](https://pin-yi.me/blog)
