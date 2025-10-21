---
title: "如何為 CI/CD 配置 GCP Artifact Registry 的 Docker 認證"
type: docs
weight: 9989
description: 如何為 CI/CD 配置 GCP Artifact Registry 的 Docker 認證
images:
  - gcp/gcs-cors/og.webp
date: 2024-12-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - CI/CD
  - GCP
  - Google Cloud Platform
  - GAR
  - Artifact Registry
  - Docker
---

## 說明

當我們在 CI 階段，會將打包的 image 丟到 Repositories，我們選擇 GCP 的 Artifact Registry，就會需要有 GCP 相關權限，可以推到 Repositories 上。

<br>

那我們通常會有兩種做法：

1. 使用 gcloud 的 image，並在 GitLab CICD Variables 塞入對應的 SA Key (當然要給他能上傳 Artifact Registry 權限)，但這個方法在跑 CI 時，會需要拉 gcloud 的 image 會比較花時間。

2. 先產生好已經驗證過的 `$HOME/.docker/config.json`，也放到 GitLab CICD Variables，讓 CI 動作時，可以拿來驗證可以直接 push 到 Repositories 上。

3. (當然還有第三種方式，之後再來介紹)

<br>

那我們這篇就來說明第二種方式，要怎麼產生 `$HOME/.docker/config.json` 來驗證呢？

如果本機有 Docker 可以先看看自己檔案內容，應該會長得像下面這樣

<br>

```json
{
	"auths": {
		"asia-east1-docker.pkg.dev": {}
	},
	"credsStore": "desktop",
	"credHelpers": {
		"asia-docker.pkg.dev": "gcloud",
		"asia-east1-docker.pkg.dev": "gcloud",
		"asia.gcr.io": "gcloud",
		"gcr.io": "gcloud",
		"marketplace.gcr.io": "gcloud",
		"staging-k8s.gcr.io": "gcloud",
		"us.gcr.io": "gcloud"
	},
	"currentContext": "desktop-linux"
}
```

<br>

可以看到 `auths` 的 `asia-east1-docker.pkg.dev` 是空的，然後在下面有 `credHelpers` `"asia-east1-docker.pkg.dev": "gcloud",` 就代表 Docker 在嘗試連接這些 Registry 時，會通過 gcloud 命令行工具來生成和管理憑證，但這樣又會需要使用 gcloud 來驗證 QQ

<br>

因此我們需要自己來打造我們的 Config xDD，我們要先準備好有權限的 SA Key，如下：
(有將其中 Key 以及 project 亂改了，不用擔心)

```json
{
  "type": "service_account",
  "project_id": "gcp-20435107-111",
  "private_key_id": "096bd873ba49ba348946eff2202199330fd89fbb",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiZ+MUvg\n9xqkVFzKpjrEE2o/L5c7XeciSoAvAF757WFWXJvPWyG48LWNE5PGy+kXo0TBWBm7\n96pgugJVAgMBAAECggEAJTTBRiRSd2Wv6T0HLolqX1xDgU6SFL7bEtub+mtq7C2b\naInTkGe8Iol0VDbysFbAEx/wQpK5a2RPUQK/QHVCkMiV13yX75OTFl3IRMoqRjmT\nHhg88ncpY4yVmUuR0dSY9asdP4hIupSe9SYq58Uu6f8wE1UhzoF6k3SsxuePKU1K7EPG3UPtFIwvc2zS0du7h\n49WUrZkzLwf/HvfUAfmbio0JdY+wEzN4lgdYD1ApEQKBgQDx4s5Znp3YBuI1rafP\n0Hrdu/5dtM1rSSP4LA6+C97fGj3N4KC1hh+6rVmr2xC3YTnhYv1uaVsbbV676Ssr\n2fBYNNqbLfzMkip4TPhg7g/rggihj5uhtui3rgyicgbgyiy\n1oJNVzYR2Q4Ay8RAne1eqdyFfQKBgQC32mKIIDBjGk/B/4g1TXEnJYJuarM72ast\nlUnvoI42FK/RJdepDdh6b+EhvAgTQD0CuJO8FE2L6/lf+LglJAzJEC8ua3PWZrS6\n2tUGd56Om9GVAtbGhvDrCprPXtN4o0ouqfRXCiKCTF0jip6t9WoCETXjvrQYN5JN\nP7WKmL2nuQKBgQDkrS20YFaNkwRtBv2tZEWkN0SlRnclxIHy74QIe6R6e46Ogpys\nwF5i19v8syA8nfhgcntx1LzDU0TKlgewb1vfqCg7qOBkbpMkJHB1AtuE+Fzr4xgi\nAyglexLgvro9o2LPbVBUPlVE5en7YVvidvm9MIP3n6KzcfDZvfRZGHFY6QKBgQCT\n7kfxt9S3KOib8/uox9MP6IJ2TaxBr/aoCsMe6FUE9sgwxP4trFJO0c6X0i+9Labp\nlZJpdvyeZRSWQA4K9GLFNRyBgTwHe0RYRNO7DGyr2nxcJZiizNj0hefiiy4kl16N\nBXrwvdredItMmbDrz9eoKijuQvettKknNuffyN5xIQKBgCe++4G6OVW6LxrY+ggm\nxRGb5p55b7RPkj2Sr1OtqsOA1PDffuBjmowOQ9atwaMotV38vFU2tMEN1A2HCtaR\n4a01Heg1jiCHyZ7KgAxkvzovxcm8KQEtgDfRKj7/5b0BWubRb5tc+yGFOiLMShRD\nBjqH9JHeNGTn5BkrcH+mWSs7\n-----END PRIVATE KEY-----\n",
  "client_email": "aaa-test@gcp-20435107-111.iam.gserviceaccount.com",
  "client_id": "102880852862346945795",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/ian-test%40gcp-20435107-111.iam.gserviceaccount.com",
  "universe_domain": "googleapis.com"
}
```


<br>

## 使用 Token 來產生 Docker Config

1. 在本機下 `gcloud auth activate-service-account --key-file=<檔案名稱>`，先將自己電腦的 gcloud 切換成 SA 帳號

2. 接著我們把 token 印出來 (已經刪除其中 token 內容)

<br>

```shell
gcloud auth print-access-token

ya29.c.c0ASRK0GZLLHHYbVgq_fzR2cIC01O9V-gUDB8Wxmt8X71SHEwYvDbFm4Pw4Y6snMS1XG2_a4M0-bY8cIV8tNnUCRv0Yxb-a38AJWLnreGpBj0fKMAsELhMSnQE1TRkdUiaLj4GV4JDXWROfYtBzGiqxeK89ZyzHGOpc4x0n_41ZBImgR8S1BhFQATYiFID7tqtuVdogZ2Ud74uOhZVvbicm86W_8hS4kipb8yyfF-IQ4uYuvXw4RrzRd9tuc5a_U7t5mWy7sfY1ZWbh3lkjfakxenRgqlVt6Xaj2SBiyYpaOBVuVjVynXtkS2gMBcIcdpkOehbZMU_t8MVBuUqst4Fm8iBQXrhROfgcQslnwhi2wWWyZWs7-VBaMO2mzkh2rFwyF9Fgzhw8spMt_XO4jn9R5zMe-IMB7fltyjvQx319dJBBQYX3eZJhzpJ36d-Brpgkb604mx8z_QtIIg5nBerR-g_avSptzRYk4e9QVz_xJdQmMSk7ojM0VY0wUXzUxFzBVgIsinq-hIeIgaIo6ld0b5I28qzIXzuj9nVdM2J5rfro6puVvydu45YeO1hntfzB7o65fBe8beFmUwuj5jI2XjimJkxv44JyZSY_JFv-lJQlz6uYt46cBboa0VcSW3l2YxWIoF-m9i
```

<br>

3. 然後下以下指令，把上面拿到的 token 塞入，並轉成 base64 (前面要多帶 oauth2accesstoken:)

```shell
echo -n "oauth2accesstoken:<token>" | base64
```

<br>

4. 最後將這串經過 base64 的字串，帶入 Docker config 中

```json
{
  "auths": {
    "asia-east1-docker.pkg.dev": {
      "auth": "<base64 後的 token>"
    }
  }
}
```

就完成啦！只是這個 token 會有個缺點，它有一小時的有效期限，所以拿來當 CI 專用的 Cofing 不太適合。

<br>

## 使用 SA Key 來產生 Docker Config

1. 在本機下 `printf "_json_key:%s" "$(cat <key 的檔案>)" | base64`，將 SA Key 帶入，並轉成 base64 (前面改帶 `_json_key:`)

    (為什麼要用 printf 是因為如果本機是用 zsh echo 會把 Key 裡面的 \n 直接換行，就會導致憑證出現問題)

<br>

2. 接著把這串字串，帶入 Docker Config 中產生的 base64，在放到底下的 JSON 裡面

```json
{
  "auths": {
    "asia-east1-docker.pkg.dev": {
      "auth": "<base64 後的 token>"
    }
  }
}
```

就完成啦！這個方式就不會有時間限制的問題，可以直接放到 CI/CD 裡面使用。