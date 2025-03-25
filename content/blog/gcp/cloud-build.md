---
title: "Google Cloud Platform (GCP) - Cloud Build"
type: docs
weight: 9999
date: 2022-07-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

跟大家介紹一下今天的主題 Cloud Build，Cloud Build 可以幫我們做持續建構、測試和部署，我們可以把它想成簡易版的 Jenkins，從整個映像檔案打包到部署，也就幾分鐘的事情，且內建許多指令。

<br>

我們今天文章，需要使用前幾天提到的 [Cloud Source Repositories](/blog/gcp/cloud-source-repositories/) 、[Compute Engine](/blog/gcp/compute-engine/)、[Container Registry](/blog/gcp/container-registry/)，我們需要先透過 GitLab 將程式鏡像到 Cloud Source Repositories，再透過 Cloud Build 觸發將 GitLab 上面的 Dockerfile 建置到 Container Registry 中，再部署到 Compute Engine VM 上。大家可以參考流程圖，會更清楚今天的流程！那我們就開始囉 🥸

<br>

{{< figure src="/gcp/cloud-build/1.png" width="1200" caption="流程圖" >}}

<br>

## Cloud Source Repositories 測試

前面 GitLab 鏡像設定，請先參考上上篇 [Google Cloud Platform (GCP) - Cloud Source Repositories](/blog/gcp/cloud-source-repositories/)，上上篇會帶大家從 GitLab 鏡像到 Cloud Source Repositories，所以我們就接續之前的內容，繼續往下開始學習吧～

<br>

1. 開啟 GCP，選擇左側的 menu > 點擊 **Cloud Build** > 選擇 **觸發條件**，點擊 **建立觸發條件** 按鈕。

<br>

2. 輸入觸發條件的名稱，事件可以設定我們要怎麼進行觸發，我們這邊先選擇 **推送至分支的版本** 來觸發，在來源選擇上上篇建立好的 Cloud Source Repositories 存放區，分支版本我們先使用預設的 master，也就是推程式到 master 他會就觸發 Cloud Build：

<br>

{{< figure src="/gcp/cloud-build/2.png" width="900" caption="建立觸發條件 1" >}}

<br>

3. 設定類型我們選擇 Cloud Build 設定檔，他也是 Cloud Build 專用的設定檔，後面會帶大家寫一份 Cloud Build，位置當然是使用我們 Cloud Source Repositories 存放區，以及可以依照專案來修改 cloudbuild.yaml 放在專案的哪裡，最後都沒問題，就按下建立：

<br>

{{< figure src="/gcp/cloud-build/3.png" width="900" caption="建立觸發條件 2" >}}

<br>

### 撰寫 cloudbuild.yaml 設定檔

在開始建立檔案前，先來跟大家說說檔案內有哪些設定吧：

<br>

首先 Cloud Build 建構器是裝有常用的程式語言和工具的容器映像。我們可以配置 Cloud Build，讓建構器中運行特定命令，我舉個例子讓大家了解：

以下程式碼是來自 [Docker Hub](https://hub.docker.com/search?q=&type=image) 的 `ubnutu` 映像檔中所執行的命令：

```yaml
steps:
  - name: "ubuntu"
    args: ["echo", "hello world"]
```

可以看到我們配置文件中 `steps` 參數是指我們要建構的步驟， `name` 字段指定 Docker 映像檔的位置，以及 `args` 字段中是指定映像檔運行的命令。

我們的 Cloud Build 一樣會需要用 `name` 來指定建構容器的映像檔，以及使用 `args` 來執行我們映像檔所要運行的命令。

<br>

我們的 Cloud Build 設定檔中的 `name` 常用的建構器映像檔如下：

|  Builder   |               名稱               |
| :--------: | :------------------------------: |
|   bazel    |   gcr.io/cloud-builders/bazel    |
|   docker   |   gcr.io/cloud-builders/docker   |
|    git     |    gcr.io/cloud-builders/git     |
|   gcloud   |   gcr.io/cloud-builders/gcloud   |
| gke-deploy | gcr.io/cloud-builders/gke-deploy |

<br>

接著我們來試著寫一個 cloudbuild.yaml 來建構我們的 nginx 服務，並部署到 Compute Engine 上。

<br>

我們先回到 Gitlab 該專案下的目錄，新增 cloudbuild.yaml 檔案，將複製以下內容：

```yaml
steps:
  # Docker Build
  - name: "gcr.io/cloud-builders/docker"
    args: ["build", "-t", "gcr.io/$PROJECT_ID/ian-test:ian-nginx-test", "."]

  # Docker Push
  - name: "gcr.io/cloud-builders/docker"
    args: ["push", "gcr.io/$PROJECT_ID/ian-test:ian-nginx-test"]

  # Build VM
  - name: "gcr.io/google.com/cloudsdktool/cloud-sdk"
    entrypoint: "gcloud"
    args:
      [
        "compute",
        "instances",
        "create-with-container",
        "ian-test-vm",
        "--container-image",
        "gcr.io/$PROJECT_ID/ian-test:ian-nginx-test",
      ]
    env:
      - "CLOUDSDK_COMPUTE_REGION=asia-east1"
      - "CLOUDSDK_COMPUTE_ZONE=asia-east1-b"
```

我們一個一個來說說的這個 cloudbuild.yaml 裡面的設定吧！(我以前面的註解來區分)

- Docker Build：這邊的 `name` 我們用 `gcr.io/cloud-builders/docker`，代表我們將使用 docker 建構器，`args` 這邊下的意思是要把與 cloudbuild.yaml 放在一起的 Dokcerfile 給 build 起來，並改名為 gcr.io/$PROJECT_ID/ian-test:ian-nginx-test。
- Docker Push：這邊一樣使用 `gcr.io/cloud-builders/docker`，`args` 指令部分變成我們要把他 push 到 gcr.io/$PROJECT_ID/ian-test 這個 Cloud Source Repositories，其中這個映像檔案的 tag 為 ian-nginx-test。
- Build VM：這邊我們使用 `gcr.io/google.com/cloudsdktool/cloud-sdk`，可以透過它來建立 VM，並且執行 tag 名為 ian-nginx-test 的映像檔，後面環境變數是來設定 VM 的區域等等。

<br>

{{< callout type=tip title="小提醒" >}}
`ian-test` 是 Container Registry 資料夾名稱，`ian-nginx-test` 是 Container Registry 映像檔的 tag，`ian-test-vm` 是我們建立 VM 的名字，所以要記得改成自己的命名歐！
{{< /callout >}}

<br>

### 撰寫 Dockerfile

接下來剛剛有提到 Docker Build 會將我們放在一起的 Dockerfile 給 build 起來，所以我們也要先寫好要用的 Dockerfile：

```Dockerfile
FROM nginx:latest

COPY ./index.html /usr/share/nginx/html/
```

<br>

我們的 Dockerfile 很簡單，簡單寫了要使用的映像檔，以及將我們等等要測試的 index 複製到裡面

<br>

### 撰寫測試 index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>測試測試測試</title>
  </head>
  <body>
    我是測試檔案
  </body>
</html>
```

<br>

這個測試檔案，會蓋過 nginx 的預設畫面，當我們成功將 VM 建立後，瀏覽 80 Port 時，應該會跳出這個測試的網頁。

<br>

### Commit 到 GitLab

當我們都新增好檔案後，我們就將程式推到我們前幾篇的 Cloud Source Repositories 已經鏡像的 GitLab 中，接著就是等待見證奇蹟的時候了ＸＤ

1. 當我們推送後，我們先檢查 Cloud Source Repositories 是否有成功從 GitLab 鏡像過來：

<br>

{{< figure src="/gcp/cloud-build/4.png" width="700" caption="Cloud Source Repositories" >}}

<br>

2. 檢查看看 Container Registry 是否多了名為 `ian-test` 的資料夾，且裡面有一個 tag 為 `ian-nginx-test` 的映像檔：

<br>

{{< figure src="/gcp/cloud-build/5.png" width="1000" caption="Container Registry" >}}

<br>

3. 檢查一下 Cloud Build 的觸發條件是不是在運作，最後成功可以看到類似下方圖片內容：

<br>

{{< figure src="/gcp/cloud-build/6.png" width="1200" caption="Cloud Build 的觸發條件" >}}

<br>

4. 檢查 Compute Engine 的 VM 是否有成功被建立：

<br>

{{< figure src="/gcp/cloud-build/7.png" width="1200" caption="Compute Engine" >}}

<br>

5. 最後就是測試這個映像檔案，是不是我們所 Build 的，測試方法很簡單，我們剛剛在 Dockerfile 有複製我們自己寫的 index.html 檔案，去蓋掉原本 nginx 的預設檔，所以我們可以瀏覽上面圖片的外部 IP，就可以看到我們所改的頁面囉！

<br>

{{< figure src="/gcp/cloud-build/8.png" width="700" caption="測試用 index.html" >}}

<br>

## 參考資料

Cloud Builder：[https://cloud.google.com/build/docs/cloud-builders](https://cloud.google.com/build/docs/cloud-builders)

Day27 - 用 Cloud Build 實作 CI 部分：[https://ithelp.ithome.com.tw/articles/10224727](https://ithelp.ithome.com.tw/articles/10224727)
