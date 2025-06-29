---
title: "Google Cloud Platform (GCP) - Compute Engine"
type: docs
weight: 9998
date: 2022-06-29
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

跟大家介紹一下今天的主題 Google Compute Engine(GCE)，GCE 是 Google Cloud 上的基礎架構服務 (IaaS)，該平台可以提供大規模的虛擬機器以及相關的基礎建設 (包含硬碟、網路、附載平衡器... 等等)來建置及運作您的服務，那我們可以將 GCE 服務的主要功能劃分成以下幾點：

<br>

## Google Compute Engine 特色

### 穩健的網路功能

提供使用者擁有穩健的網路功能，以運行各項應用程式及服務。

<br>

### 自訂網路與預設網路

GCE 包含內部與外部的網路連線能力，讓使用者可以透過自訂規劃來建置屬於自己服務適用的網路，而在 GCE 服務開通當下，也提供預設的 default network，內建常用的路由與防火牆設定 (例如：SSH、RDP、ICMP... 等)，供入門使用者直接使用。

<br>

### 防火牆規則

除了預設防火牆規則外，使用者也可以透過自建防火牆來開放可以連入的 IP。

<br>

### 各區域的 HTTP(S) 的負載平衡

為 Layer 7 的負載平衡設備，透過設定可以串連多台主機或是主機群組，讓服務不再只是依賴於單點存在的伺服器。Layer 7 的負載平衡更可以識別路由規劃，進一步可以提供不同路由的重導規則設定，也可以提供 CDN 的 Cache 功能。

<br>

### 網路的負載平衡

為 Layer4 的負載平衡設備，可以透過 Protocol 與 Port 的方式來重導外部流量到 GCE 主機或主機群，並可以透過 Health Check 的方式，讓流量僅通過健康的主機，避免服務中斷。

<br>

### 子網路

透過 CIDR 的方式設定 GCE Network 中的各子網路範圍，並可以透過路由的方式串接各子網路中間的通訊，讓 GCE 網路的設計規劃可以更有彈性，也可以讓子網路的規劃來實作更安全的雲端網路架構。

<br>

## Google Compute Engine 測試

首先我們使用 cloudskillsboost 提供的 [Creating a Virtual Machine ](https://www.cloudskillsboost.google/focuses/3563?locale=zh_TW&parent=catalog) 來做練習，打開後，請先登入自己的 Google 帳號，接著點選左上角的 **Start Lab**，會跳出與下面圖片類似的內容：

<br>

{{< figure src="/gcp/compute-engine/1.png" width="250" caption="測試用的帳號密碼" >}}

<br>

### 新增新的 VM 實例

1. 點選 **Open Google Console** 按鈕來開啟 GCP 主控台
2. 登入帳號就使用上面圖片所提供的帳號密碼來進行登入
3. 登入成功會進入 GCP 主控台，點選左側的 menu > **Compute Engine** > **VM Instances**，可以參考下方圖片

<br>

{{< figure src="/gcp/compute-engine/2.png" width="600" caption="新增 VM 實例" >}}

<br>

4. 點選 **CREATE INSTANCE**，請依照下方表格來進行設定：

|     標題     |                         設定值                         |                                                          說明                                                           |
| :----------: | :----------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------: |
|     Name     |                         gcelab                         |                                                    虛擬機實例的名稱                                                     |
|    Region    |                us-central1（愛荷華州）                 | 有關區域的更多信息，請參閱 Compute Engine 指南 [Regions and zones](https://cloud.google.com/compute/docs/regions-zones) |
|     Zone     |                     us-central1-f                      |                                                                                                                         |
|    Series    |                           N1                           |                                                                                                                         |
| Machine type |                     n1-standard-2                      |                                            這是一個 2 vCPU、7.5 GB RAM 實例                                             |
|  Boot disk   | New balanced persistent disk/10 GB/Debian GNU/Linux 10 |
|   Firewall   |                   Allow HTTP taffic                    |                                                                                                                         |

<br>

{{< figure src="/gcp/compute-engine/3.png" width="800" caption="VM 實例 (Name、Region、Zone、Series)" >}}

<br>

{{< figure src="/gcp/compute-engine/4.png" width="800" caption="VM 實例 (Machine type、Boot disk、Firewall)" >}}

<br>

5. 新增完後，大約需要等待一分鐘，新的虛擬機就會列在 **VM Instances** 頁面上。

<br>

{{< figure src="/gcp/compute-engine/5.png" width="1000" caption="VM 實例" >}}

<br>

### 安裝 NGINX Web 服務器

1. 點擊 VM instances 實例最後的 SSH，會開啟 SSH 用戶端
2. 在 SSH 終端，要先獲得 `root` 訪問權限，才更方便的進行後續的動作，請先使用一下指令：

```shell
sudo su -
```

<br>

3. 利用 `root` 用戶，來更新操作系統：

```shell
apt-get update
```

<br>

{{< figure src="/gcp/compute-engine/6.png" width="1000" caption="更新操作系統" >}}

<br>

4. 安裝 NGINX：

```shell
apt-get nginx -y
```

<br>

{{< figure src="/gcp/compute-engine/7.png" width="1000" caption="安裝 NGINX" >}}

<br>

5. 最後確認 NGINX 是否運行：

```shell
ps auwx | grep nginx
```

<br>

{{< figure src="/gcp/compute-engine/8.png" width="1000" caption="確認 NGINX 是否運行" >}}

<br>

6. 可以打開瀏覽器瀏覽 `http://外部 IP/` 或是使用 `curl 外部IP`，外部 IP 會在跟剛剛 VM 實例的 External IP 欄位呦～

<br>

{{< figure src="/gcp/compute-engine/9.png" width="700" caption="curl 外部 IP" >}}

<br>

7. 完成後，記得可以點選 **Check my progress** 來檢查進度吧！

<br>

{{< figure src="/gcp/compute-engine/10.png" width="500" caption="Check my progress" >}}

<br>

### 使用 gcloud 創建一個新實例

我們剛剛是使用網頁版來新增，當然也可以使用 gcloud 指令來新增，這個工具有預先裝在 [Google Cloud Shell](https://cloud.google.com/shell#how_do_i_get_started) 中。Cloud Shell 是一個基於 Debian 的虛擬機，包含了常用的開發工具 (`gcloud`、`git` 等工具)，另外你也可以將 gcloud 下載至本機上來做使用，請閱讀[ gcloud 命令行工具指南](https://cloud.google.com/sdk/gcloud/)

<br>

1. 在 Cloud Shell 中，使用 `gcloud` 來新增新的虛擬機實例：

```shell
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
```

就會跳出以下圖片的內容，過一陣子去查看 VM Instances 也可以看到我們所新增的虛擬機實例歐～

<br>

{{< figure src="/gcp/compute-engine/11.png" width="1000" caption="Check my progress" >}}

<br>

到這邊就完成了我們 Google Compute Engine 測試囉～我們知道可以新增 GCE 將實體主機的內容，移植到雲端上囉！希望大家會喜歡今天的文章 🥰

<br>

## 參考資料

Compute Engine 基本介紹：[https://gdgcloud-taipei.gitbook.io/google-cloud-platform-in-practice/google-cloud-shang-de-yun-suan-fu-wu/compute-engine/compute-engine-ji-ben-jie-shao](https://gdgcloud-taipei.gitbook.io/google-cloud-platform-in-practice/google-cloud-shang-de-yun-suan-fu-wu/compute-engine/compute-engine-ji-ben-jie-shao)

Creating a Virtual Machine：[https://www.cloudskillsboost.google/focuses/3563?locale=zh_TW&parent=catalog](https://www.cloudskillsboost.google/focuses/3563?locale=zh_TW&parent=catalog)
