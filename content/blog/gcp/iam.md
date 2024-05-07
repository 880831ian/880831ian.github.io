---
title: "Google Cloud Platform (GCP) - IAM 與管理"
type: docs
weight: 56
---

IAM 的全名是 Identity and Access Management，當我們藉由 IAM，可以授與特定 Google Cloud 資源的 **精細** 訪問權限，並防止對其他資源的訪問。疑！？為什麼是精細？我們接著看下去，我們可以採用最小權限安全原則，該原則要求任何人都不應擁有超出實際所需的權限。

<br>

## IAM 的工作原理

首先我們先來了解一下 IAM 的工作原理，藉由 IAM，我們可以定義誰 (哪一個身份) 對哪些資源有哪種的訪問權限 (角色) 來管理訪問權限控制。什麼是資源？例如， Compute Engine 虛擬機 (GCE)、Google Kubernetes Engine (GKE) 集群和 Cloud Storage 存儲分區都是 Google Cloud 資源，我們用於整理資源的組織或資料夾、項目等也都是資源

<br>

{{< figure src="/gcp/iam/logo.png" width="150" caption="GCP IAM Logo" >}}

<br>

我們可以把它理解成

{{< callout >}}
什麼 『 人 』，可以對什麼『 資源 』，做什麼『 事情 』
{{< /callout >}}

<br>

IAM 不會直接向用戶授與資源的訪問權限，而是將權限分成多個角色，然後將這些角色授與經過身份驗證的主帳號。(以前 IAM 會將主帳號稱為成員，目前部分 API 仍然使用此術語。)

<br>

{{< figure src="/gcp/iam/iam-overview-basics.png" width="800" caption="IAM 中的權限管理" >}}

<br>

可以看到這張圖片，訪問權限管理主要包含三個部分：

- 主帳號 (Principal)：主帳號可以是 Google 帳號 (針對用戶)、服務帳號 (針對應用和計算工作負載)、Google 群組或 Workspace 帳號或可以訪問資源的 Cloud Identity 網域等等
- 角色 (Role)：一個角色對應一組權限，權限決定了可以對資源執行的操作。向主帳號授與某個腳色， 代表授與該角色包含的所有權限給主帳號
- 政策 (Policy)：允許政策 (Allow Policy) 是將一個或多個主體綁定在各個角色，當想要定義誰 (主體) 對資源擁有何種類型的訪問 (角色) 時，可以創建允許政策並將其附加到資源。

<br>

## IAM 測試

接下來，我們來測試看看 IAM 實際設定以及用途吧！ 我們會使用 GCP 所以提供的 [[Qwiklabs] Cloud IAM：Qwik Start](https://www.cloudskillsboost.google/focuses/551?parent=catalog) 來進行測試，之後的步驟會跟 Cloud IAM：Qwik Start 內容一樣，所以大家可以邊操作邊參考呦！那我們開始囉 🙃

進入網頁後，請先登入自己的 Google 帳號，接著點選左上角的 **Start Lab**，會跳出與下面圖片類似的內容：

<br>

{{< figure src="/gcp/iam/1.png" width="250" caption="測試用的帳號密碼" >}}

這邊會提供兩組的帳號及密碼，分別是 Username1 以及 Username2 (後面會以 Username 1 跟 2 來說明)，密碼會用共用，且會在同一個專案下。

<br>

### 用第一個 Username1 登入 GCP

1. 點選 **Open Google Console** 按鈕來開啟 GCP 主控台
2. 登入的帳號就輸入第一個 Username1 <student-03-XXXXXX>，密碼輸入共用密碼 <lsLrXXXXX>，最後登入
3. 登入成功會進入 GCP 主控台，會跳出下方圖片內容，國家選擇台灣，點選同意 **Terms of Service**，最後按 **AGREE AND CONTINUE**

<br>

{{< figure src="/gcp/iam/2.png" width="1000" caption="登入 Username 1 GCP 主控台" >}}

<br>

### 用第二個 Username2 登入 GCP

- 步驟與第一個相同，這邊就不在重複，但建議使用無痕，避免 Username1 跟 Username2 搶來搶去，以及登入帳號要選 Username2，應該不會選錯吧 🤣

<br>

### Username1 IAM 主控台

1. 到 Username1 的 GCP 主控台首頁
2. 點選左側的 menu > **IAM 與管理**
3. 點擊頁面上方的 **+ADD** 按鈕，可以從下拉選單去查看各式各項的專案相關角色，可以看到超級多的角色設定，所以我們一開始才會說他是可以設定 ** 精細** 的訪問權限
4. 我們選擇 **Base**，右側有 4 種角色，分別是瀏覽者 (Browser)、編輯者 (Editor)、所有者 (Owner)、檢視者 (Viewer)，詳細區別請看下面 👇👇

<br>

{{< figure src="/gcp/iam/3.png" width="1000" caption="ADD IAM" >}}

此表是取 Google Cloud IAM 文章[基本角色](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles)中的定義，簡單說明 4 種角色的差別：

|       角色名稱        |                                              權限                                               |
| :-------------------: | :---------------------------------------------------------------------------------------------: |
| role/viewer (檢視者)  |                不影響狀態的只讀操作權限，例如：查看 (但不修改) 現有資源或是資料                 |
| role/editor (編輯者)  |                   所有查看者權限，以及修改狀態的操作權限，例如：更改現有資源                    |
|  role/owner (擁有者)  |       以下操作的所有編輯權限： 1. 管理項目和項目內的所有資源角色及權限 2. 為項目設置帳單        |
| role/browser (瀏覽者) | 讀取權限以及瀏覽項目的層次結構，包含資料夾、組織和 IAM 政策。但此角色不包含查看項目中資源的權限 |

我們的 Username1 為 owner，Username2 為 viewer

<br>

{{< callout type=danger >}}
Google 建議：Base 角色包含所有 Google Cloud 服務的數千個權限。除非別無選擇，否則不要授與用戶 Base 角色，請設定最有限的自訂義角色或是可以滿足該需求的角色即可
{{< /callout >}}

<br>

### 切換到 Username2 IAM 主控台

我們可以在表格裡面收尋 Username1 跟 Username2 的帳號，可以看一下他們授與的角色是否與上面說的一致，大概可以參考以下圖片：

<br>

{{< figure src="/gcp/iam/4.png" width="1000" caption="Username1 跟 Username2 權限" >}}

<br>

在 Username2 因為是 Viewer 權限，所以點擊上面的 **+ ADD**，不會反應，會跳出以下照片內容：

<br>

{{< figure src="/gcp/iam/5.png" width="500" caption="Username2 權限不足" >}}

<br>

### 再切回 Username1 Cloud Storage

1. 接下來我們要建立一個 GCS 儲存空間，點選 menu > **Cloud Storage** > **Browser**
2. 點選 **Create a bucket**
3. 給予他一個獨特的名稱，以及在 Choose where to store your data 選擇 Multi-Region
4. 最後點選 **CREATE**

<br>

{{< figure src="/gcp/iam/6.png" width="1000" caption="Create a bucket" >}}

<br>

### 上傳範例檔案

1. 進入新建立的 Cloud Storage，點選 **Upload fiiles** 按鈕
2. 上傳一個 txt 檔案，可以先取名為 `sample.txt`，或是上傳後使用最後三個小點圖案內的 Rename 來修改名稱

<br>

{{< figure src="/gcp/iam/7.png" width="1200" caption="Upload sample.txt fiiles" >}}

<br>

### 驗證專案檢視者存取權限

1. 我們再切換回 Username2 的主控台，選擇 menu > **Cloud Storage** > **Browser**
2. 就可以看到跟上面一樣的儲存區

Username2 被授與 "檢視者 Viewer" 角色，這個角色有不影響狀態的只讀權限。這個範例中說明這個功能，他僅能檢視，沒有辦法上傳

<br>

### 移除專案存取權限

1. 我們再次切回 Username 主控台，選擇 menu > **IAM 與管理** ，找到 Username2 旁邊的鉛筆圖案

<br>

{{< figure src="/gcp/iam/8.png" width="1200" caption="修改 Username2 權限" >}}

<br>

2. 點擊角色名稱的垃圾桶來移除 Username2 的檢視者權限，點擊 **Save**

<br>

{{< figure src="/gcp/iam/9.png" width="550" caption="移除 Username2 權限" >}}

{{< callout type=tip title="提醒" >}}
這個動作要完成生效到所有服務上，所以會需要一點時間，詳細可以參考[點我](https://cloud.google.com/iam/docs/faq)
{{< /callout >}}

<br>

### 檢查 Username2 是否有存取權限

1. 切換到 Username2 主控台，選擇 menu > **Cloud Storage** > **Browser**
2. 會發現出現以下的錯誤訊息，代表我們移除權限成功

<br>

{{< figure src="/gcp/iam/10.png" width="800" caption="Username2 沒有權限" >}}

<br>

### 新增儲存角色

1. 我們再次切換到 Username1 主控版，選擇 menu > > **IAM 與管理**，點選上方 **+ ADD**，在 New principals 上貼上 Username2 的帳號，Role 選擇 **Storage Object Viewer**

<br>

{{< figure src="/gcp/iam/11.png" width="600" caption="新增 Username2 角色" >}}

<br>

{{< figure src="/gcp/iam/12.png" width="1000" caption="查看 Username2 權限" >}}

<br>

### 檢查 Username2 是否有存取權限

1. 切換到 Username2 主控台，因為 Username2 沒有專案檢視者的角色，所以看不到專案以及任何的資源，但這個使用者對我們剛剛設定的 Cloud Storage 有特別的存取權
2. 打開右上角的 Activate Cloud Shell 命令列工具 ，如下圖：

<br>

{{< figure src="/gcp/iam/13.png" width="1200" caption="開啟 Activate Cloud Shell 命令列工具" >}}

<br>

3. 輸入以下指令

```shell
gsutil ls gs://<剛剛建的 Cloud Storage 名稱>
```

<br>

如果出現跟下方圖片一樣，就代表我們設定成功囉！

<br>

{{< figure src="/gcp/iam/14.png" width="600" caption="Username2 Storage Object Viewer 權限" >}}

<br>

最後的最後，如果有跟我們一步一步來的朋友，在 [[Qwiklabs] Cloud IAM：Qwik Start](https://www.cloudskillsboost.google/focuses/551?parent=catalog) 頁面中，應該會看到一個叫 Check my progress 的按鈕，做完每一步驟，都可以點一下，他會自動去判斷你是否有完成這項動作歐！

<br>

{{< figure src="/gcp/iam/15.png" width="600" caption="Check my progress" >}}

到這邊就完成了我們在 IAM 的測試囉～我們知道 IAM 可以設定很多的角色，以及測試了查看、新增、修改、刪除角色的功能，希望大家會喜歡今天的文章 🥰

<br>

## 參考資料

IAM 概覽：[https://cloud.google.com/iam/docs/overview](https://cloud.google.com/iam/docs/overview)

什麼是 Cloud IAM？GCP 權限管理服務介紹：[https://blog.cloud-ace.tw/identity-security/what-is-cloud-iam/](https://blog.cloud-ace.tw/identity-security/what-is-cloud-iam/)
