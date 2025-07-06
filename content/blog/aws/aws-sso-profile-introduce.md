---
title: "AWS SSO Profile 設定介紹"
type: docs
weight: 9983
description: AWS SSO Profile 設定介紹
images:
  - aws/aws-sso-profile-introduce/og.webp
date: 2025-07-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

之前有介紹過 [設定 AWS CLI 以及 AWS CLI 指令說明](../aws-cli/)，大家應該已經知道要怎麼安裝 AWS CLI，以及設定 AWS IAM User 的 Access Key。

而這次要介紹的是導入 AWS Organizations 以及 AWS IAM Identity Center(SSO) 後，要如何設定 AWS SSO Profile，用來存取 AWS Organizations 中的各個 AWS Account 以及對應的 Permission Set。

<br>

## aws configure sso 設定

步驟也很簡單，首先可以使用 AWS CLI 的 `aws configure sso` 指令來設定 SSO Profile，這個指令會引導你完成整個設定。

```bash
aws configure sso
```

系統會要求你輸入一些資訊，例如 SSO Session 的名稱、SSO 的起始 URL、SSO 區域等等。

會像是下方圖片一樣，出現一個連結，也會自動幫你開瀏覽器，要你登入 AWS IAM Identity Center 來進行驗證。

<br>

{{< figure src="/aws/aws-sso-profile-introduce/1.webp" width="650" caption="設定 aws configure sso" >}}

<br>

瀏覽器開啟後，如果有登入完畢，會需要授予 Applications and AWS accounts 權限：


<br>

{{< figure src="/aws/aws-sso-profile-introduce/2.webp" width="450">}}

<br>

看到這個畫面，接著回到終端機，系統會要求你選擇要存取的 AWS Account 以及對應的 Permission Set：

<br>

{{< figure src="/aws/aws-sso-profile-introduce/3.webp" width="350" >}}

<br>

會跳出一個選單，讓你選擇要存取的 AWS Account。你可以使用上下鍵來選擇，然後按下 Enter 鍵來確認。

這邊會協助建立一個新的 Profile，並且會自動填入相關的資訊。

<br>

{{< figure src="/aws/aws-sso-profile-introduce/4.webp" width="650" caption="選擇 AWS Account" >}}

<br>

接著，選擇這個 AWS Account 後，如果有不同的 Permission Set，你可以再選擇對應的 Permission Set：

<br>

{{< figure src="/aws/aws-sso-profile-introduce/5.webp" width="650" caption="設定 Permission Set" >}}

<br>

最後如果跳出這個畫面，就代表已經設定完成了！

可以使用以下指令來確認是否有設定成功：

```bash
aws sts get-caller-identity --profile <your-profile-name>
```

<br>

{{< figure src="/aws/aws-sso-profile-introduce/6.webp" width="650" caption="設定 aws configure sso 完成" >}}

<br>

### 檢視設定的 Profile

我們也可以來看一下 `~/.aws/config` 檔案，裡面會有剛剛設定的 Profile 資訊：

```ini
[profile AdministratorAccess-72XXXXXXXXX0]
sso_session = test
sso_account_id = 72XXXXXXXXX0
sso_role_name = AdministratorAccess
region = ap-southeast-1
output = json
[sso-session test]
sso_start_url = https://d-XXXXXXX.awsapps.com/start/#
sso_region = ap-southeast-1
sso_registration_scopes = sso:account:access
```

可以看到會有兩個區塊，分別是 sso-session 以及剛剛設定的 Profile。

sso-session：
會記錄 SSO 的起始 URL、SSO 區域以及 SSO 的註冊範圍。

Profile 區塊：
則是記錄使用哪一個 SSO Session、AWS Account ID、Permission Set 以及預設的區域和輸出格式。

那我們通常只有一個 SSO Session，然後有多個 AWS Account，所以有多個 Profile，如果每個 AWS Account 都有不同的 Permission Set，就會有更多的 Profile。

<br>

那假設你想要手動輸入 SSO Profile 的話，也可以直接在 `~/.aws/config` 檔案中新增一個 Profile 區塊，並填入對應的 AWS Account ID、Permission Set 以及 SSO Session 名稱，就可以囉。

<br>

## aws configure sso-session

或是你也可以使用 `aws configure sso-session` 指令來設定 SSO Session，基本上流程跟 `aws configure sso` 類似，但就不會協助設定後面的 Profile。

<br>

{{< figure src="/aws/aws-sso-profile-introduce/7.webp" width="650" caption="設定 aws configure sso-session" >}}

<br>

那我們在使用 Profile 的時候，可以直接在指令後面多加上 `--profile <your-profile-name>` 來指定要使用的 Profile。
但這個方式很麻煩，所以我們下一篇會介紹「[手動切 AWS Profile 太麻煩？試試 Granted，多帳號管理神器助你效率倍增！](../aws-assuming-roles-tool-introduce/)」這個工具，可以讓你更方便地管理以及切換多個 AWS Profile。

<br>

## 參考資料

使用 AWS CLI 設定 IAM Identity Center 驗證：[https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)