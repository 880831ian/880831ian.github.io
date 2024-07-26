---
title: "設定 AWS CLI 以及 AWS CLI 指令說明"
type: docs
weight: 40
date: 2024-07-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 介紹

AWS CLI 是用於使用 AWS 服務的命令列工具。它也用於對 IAM 使用者或角色進行身份驗證，以便從本機電腦存取 Amazon EKS 叢集和其他 AWS 資源。若要從命令列在 AWS 中設定資源，您需要取得要在命令列中使用的 AWS 存取金鑰 ID 和金鑰。

<br>

## 如何安裝 AWS CLI

Mac 安裝指令：

```bash
brew install awscli
```

自動補齊指令：

```bash
complete -C '/usr/local/bin/aws_completer' aws
```

<br>

## 如何設定 AWS CLI

AWS 跟 GCP 比較不一樣的是 CLI 在 Google (gcloud) 可以使用 Web 登入的方式來驗證 `gcloud auto login`，但是在 AWS 需要使用 Access Key ID 和 Secret Access Key 來驗證。

因此我們在開始使用前，需要先透過 UI 產生自己帳號的 Access Key ID 和 Secret Access Key，才能夠有權限下指令。

<br>

### 建立存取密鑰

需要先確認是否有以下的 IAM 權限：

```
"Action": [
  "iam:CreateAccessKey",
  "iam:DeleteAccessKey",
  "iam:GetAccessKeyLastUsed",
  "iam:ListAccessKeys",
  "iam:UpdateAccessKey",
  "iam:TagUser"
]
```

- TagUser：是用來幫金鑰加上標籤值，方便管理。

<br>

{{< figure src="/aws/aws-cli/1.png" width="850" caption="沒有權限會噴錯" >}}

<br>

1. 先登入 AWS 的管理控制台。
2. 點擊右上角的 AWS 使用者名稱，並點選「安全憑證」。

<br>

{{< figure src="/aws/aws-cli/2.png" width="750" caption="在 AWS 的管理控制台，選擇安全憑證" >}}

<br>

3. 找到存取金鑰，並點選「建立存取金鑰」，如果有建立過，也會顯示在這邊。

<br>

{{< figure src="/aws/aws-cli/3.png" width="1100" caption="建立存取金鑰" >}}

<br>

4. 點選「命令列介面 (CLI)」，下方要勾選一個確認建議的選項，最後按下一步。

<br>

{{< figure src="/aws/aws-cli/4.png" width="900" caption="選擇命令列介面 (CLI)案例" >}}

<br>

5. 可以選擇是否要建立標籤，如果要建立，需要前面提到的 `iam:TagUser` IAM 權限。

<br>

{{< figure src="/aws/aws-cli/5.png" width="950" caption="設定描述標籤" >}}

<br>

6. 就可以拿到存取金鑰 (Access Key ID) 以及私密存取金鑰 (Secret Access Key)，<b>要記住私密存取金鑰，只有這個畫面可以拿到，如果沒有儲存，就需要重新建立一個新的金鑰</b>。

<br>

{{< figure src="/aws/aws-cli/6.png" width="1100" caption="產生完金鑰" >}}

<br>

<br>

### 配置 AWS CLI

1. 接下來我們要在本機把剛剛產生的 Access Key ID 和 Secret Access Key 設定到 AWS CLI，我們可以下這個指令。

```bash
aws configure
```

2. 下完後，就會顯示以下設定畫面，分別輸入 Access Key ID、Secret Access Key、預設區域以及輸出格式。

```bash
AWS Access Key ID [None]: AAKIA2HCM5WKG7R647HTV
AWS Secret Access Key [None]: ******************************
Default region name [None]: region-code (例如：亞太地區東京 ap-northeast-1)
Default output format [None]: json
```

就成功設定完成囉～

<br>

## AWS CLI 指令說明

| 指令                          | 描述           | 輸出                                                                                                                                                      |
| ----------------------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `aws sts get-caller-identity` | 驗證使用者身份 | <pre>{<br> \"UserId\": \"AKIAIOSFODNN7EXAMPLE\",<br> \"Account\": \"01234567890\",<br> \"Arn\": \"arn:aws:iam::01234567890:user/ClusterAdmin\"<br>}</pre> |

(之後補上...)

<br>

## 參考資料

探索 AWS：從菜鳥到熟練的完全指南(四)IAM 登入方式：[https://hackmd.io/@gdw7l5sPTOyNv76kZ_twjA/SkTPl-xP3](https://hackmd.io/@gdw7l5sPTOyNv76kZ_twjA/SkTPl-xP3)

設定 AWS CLI：[https://docs.aws.amazon.com/eks/latest/userguide/install-awscli.html](https://docs.aws.amazon.com/eks/latest/userguide/install-awscli.html)

AWS CLI 指令參考：[https://docs.aws.amazon.com/cli/latest/reference/#available-services](https://docs.aws.amazon.com/cli/latest/reference/#available-services)
