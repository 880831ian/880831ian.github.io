---
title: "GCS Bucket CORS 錯誤解決方法"
type: docs
weight: 9988
description: GCS Bucket CORS 錯誤解決方法
images:
  - gcp/gcs-cors/og.webp
date: 2025-03-18
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Google Cloud Platform
  - GCP
  - GCS
  - CORS
  - Bucket
---

最近公司有需求需要透過前端去打 GCS Bucket 的檔案，但會遇到 CORS 錯誤，所以寫一篇來記錄此問題的解決方法。

<br>

有先簡單寫一個前端頁面，可以透過 Axios 去打後端，詳細程式可以[點我查看](https://github.com/880831ian/gcs-cors)，我們也另外建立一個公開的 GCS Bucket，並放一個測試用的 JSON 檔案
(都有附在此專案中)。

<br>

- 公開的 GCS Bucket URL 及內容 (bucket 會用 ian-test-demo 來示範，請自行修改成自己的 bucket 名稱)

<br>

{{< figure src="/gcp/gcs-cors/1.webp" width="550" caption="curl 測試" >}}

<br>

使用測試程式需要先做以下步驟：

1. 執行 `cd code; docker build -t gcs-cors-test .`

2. 執行 `docker run -d -p 8080:80 --name gcs-cors-test gcs-cors-test`

3. 開啟瀏覽器 `127.0.0.1:8080`

<br>

開啟後，我們輸入剛剛公開 GCS Bucket URL 到輸入欄位，開啟 F12 Network，並按下測試

<br>

{{< figure src="/gcp/gcs-cors/2.webp" width="550" caption="使用程式來測試" >}}

<br>

就會發現，出現 CORS error 錯誤，我們可以查看下方的錯誤說明

<br>

{{< figure src="/gcp/gcs-cors/3.webp" width="550" caption="發現 CORS 錯誤" >}}

<br>

因為我們從 http://localhost:8080 要打到 https://storage.googleapis.com/ian-test-demo/hello.json ，觸發了瀏覽器的 CORS 限制，所以導致噴錯，CORS 的說明詳細可以直接參考：[https://www.explainthis.io/zh-hant/swe/what-is-cors](https://www.explainthis.io/zh-hant/swe/what-is-cors)

<br>

那要怎麼解決呢，[根據 Google 文件](https://cloud.google.com/storage/docs/using-cors#command-line)，我們需要為 Bucket 配置 cors 設定

<br>

 我們可以先下此指令來查看該 bucket 是否有設定 cors： `gcloud storage buckets describe gs://ian-test-demo --format="default(cors_config)"`

<br>

{{< figure src="/gcp/gcs-cors/4.webp" width="550" caption="gcloud 指令查看 cors 設定" >}}

<br>

如果還沒設定就會顯示 null

<br>

那我們先來寫一下 cors 的設定檔案

```json
[
    {
      "origin": ["*"],
      "method": ["GET", "POST", "PUT", "DELETE"],
      "responseHeader": ["Content-Type", "Authorization"],
      "maxAgeSeconds": 30
    }
]
```
(可以再根據自己需求去調整，先以這樣來 demo)

<br>

寫完後，我們需要設定到 Bucket 上面，可以用此指令將 cors.json 設定到指定的 Bucket 上：`gcloud storage buckets update gs://ian-test-demo --cors-file=cors.json`

下完指令後，我們可以在用 describe 來確認是否設定完成，正常會如下顯示：

<br>

{{< figure src="/gcp/gcs-cors/5.webp" width="550" caption="設定及檢查" >}}

<br>

接著我們就可以來測試看看是否還會碰到 CORS error 的問題了

<br>

{{< figure src="/gcp/gcs-cors/6.webp" width="550" caption="再次使用前端程式測試" >}}

<br>

另外，如果不需要設定 CORS 時，可以用以下指令移除該 Bucket 的 CORS 設定：`gcloud storage buckets update gs://ian-test-demo --clear-cors`

<br>

{{< figure src="/gcp/gcs-cors/7.webp" width="550" caption="移除 CORS 設定" >}}

<br>

## 參考

[Set up and view CORS configurations](https://cloud.google.com/storage/docs/using-cors#command-line)

[CORS configuration examples](https://cloud.google.com/storage/docs/cors-configurations)

