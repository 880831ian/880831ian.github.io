---
title: "Google Cloud Platform (GCP) - Container Registry"
type: docs
weight: 9998
date: 2022-07-01
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

跟大家介紹一下今天的主題 Container Registry，Container Registry 是儲存、管理和保護 Docker 容器映像檔的存放區，可以讓團隊透過同一項服務服務集中管理 Docker 映像檔、也可以執行安全漏洞分析，還能透過精密的存取權管理機制，來決定誰可以存取哪些內容。簡單來說他就是一個讓我們存放 Docker 映像檔的地方，他有以下幾個特點：

<br>

## Container Registry 特點

1. 安全的私人 Docker 註冊資料庫：只需要幾分鐘的時間，即可開始在 Google Cloud Platform 中使用安全的私人 Docker 映像檔儲存空間，控管能夠存取、檢視或下載映像檔的人員，並在受到 Google 安全防護機制保護的基礎架構上穩定執行。

<br>

2. 自動建立及部署映像檔：在您修訂 Cloud Source Repositories 中的程式碼時，系統會自動建立映像檔並推送至私人註冊資料庫。您可以輕鬆設定持續整合/持續推送軟體更新管道，並整合至 Cloud Build，或是直接將管道部署至 Google Kubernetes Engine、App Engine、Cloud Functions 或 Firebase。(這個就是我們在[下一篇文章](/blog/gcp/cloud-build/)會使用到的功能)

<br>

3. 深度安全漏洞掃描：在軟體部署週期的早期階段找出安全漏洞，藉此確保您可以安全地部署容器映像檔。資料庫會持續重新整理，讓您的安全漏洞掃描作業取得最新型惡意軟體的相關資訊。

<br>

4. 鎖定有風險的映像檔：運用二進位授權的原生整合功能來定義政策，避免部署與所設定政策相衝突的映像檔。您可以觸發容器映像檔的自動鎖定功能，禁止將有風險的映像檔部署至 Google Kubernetes Engine。

<br>

5. 原生支援 Docker：您不僅可以視需求定義多個註冊資料庫，也能使用標準的 Docker 指令列介面，將 Docker 映像檔推送和提取到您的私人 Container Registry。Docker 提供依據名稱和標記搜尋映像檔的功能，讓您可以順暢使用。

<br>

6. 快速的高可用性存取能力：您可使用我們遍布全球的區域性私人存放區，於全世界皆能享有最快速的回應時間。您可以就近將映像檔儲存在位於歐洲、亞洲或美國的運算執行個體中，並透過 Google 的高效能全球網路快速完成部署作業。

<br>

上面有提到 Container Registry 其中一個特點就是掃描映像檔，會檢查有沒有奇怪的內容或是錯誤，讓我們在部署前可以先做檢查，底下是整個的流程圖：

<br>

{{< figure src="/gcp/container-registry/1.webp" width="800" caption="Container Registry 掃描流程圖" >}}

<br>

我們可以延續上一篇文章 [Cloud Source Repositories](/blog/gcp/cloud-source-repositories/) 來說明上面圖片，一開始先 commit 到 GCP 上，也可以把它當成 Cloud Source Repositories，就著透過會透過[下一篇要講的 Cloud Build](/blog/gcp/cloud-build/) 來建置，並且掃描，如果沒有問題就會 Pubish 到 Container Registry 存放囉～

<br>

我們就簡單說明要怎麼查看我們的 Container Registry，點選則左側的 menu > 選擇 **Container Registry** > 點擊 **映像檔**：

<br>

{{< figure src="/gcp/container-registry/2.png" width="350" caption="Container Registry" >}}

<br>

一開始可能沒有任何的資料夾，之後當我們 Build 時，可以新增特定的資料夾來放我們的映像檔，可以參考下方圖片：

<br>

{{< figure src="/gcp/container-registry/3.png" width="750" caption="Container Registry 資料夾" >}}

<br>

進入該資料夾後，就可以看到裡面的映像檔案：

<br>

{{< figure src="/gcp/container-registry/4.png" width="1000" caption="Container Registry 映像檔" >}}
