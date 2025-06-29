---
title: "如何使用 MFA Token 驗證 AWS CLI"
type: docs
weight: 9996
date: 2024-08-01
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

會有這一篇文章也是跟 [Identity and Access Management (IAM) 介紹](../iam-introduce) 一樣，在測試 eksctl 建立 EKS 時，發現一直噴錯，說我沒有權限，但我去看我的 IAM Policy 也有加上對應 action 以及 resource，但還是不行，後來發現 `cloudformation:*` 這個 action 所在的 Policy 有加上 Condition MFA 驗證的條件，所以就來寫一篇如何使用 MFA Token 驗證 AWS CLI 的文章。

<br>

{{< figure src="/aws/cli-mfa/1.png" width="850" caption="沒有權限噴錯QQ" >}}

<br>

## 前情提要

我目前有 3 個 IAM Policy，分別是：

1. default-policy：放一些個人基本的 IAM 權限

<br>

{{< figure src="/aws/cli-mfa/2.png" width="650" caption="default-policy" >}}

<br>

2. poc-general-policy：放一些我在測試的 PoC 的 IAM 權限，<b>但是仔細看這邊有 Condition MFA 驗證的條件</b>。

<br>

{{< figure src="/aws/cli-mfa/3.png" width="650" caption="poc-general-policy" >}}

<br>

3. ReadOnlyAccess：預設的 ReadOnlyAccess 權限。

<br>

{{< figure src="/aws/cli-mfa/4.png" width="650" caption="ReadOnlyAccess" >}}

<br>

當你設定好 `aws configure` 後，你會拿到一個永久性的 IAM 憑證 (也就是 Access Key ID 和 Secret Access Key，設定可以看[配置 AWS CLI](../aws-cli/#%e9%85%8d%e7%bd%ae-aws-cli))，但是如果你的 Policy 有加上 MFA 驗證的條件，那你就需要使用 MFA Token 來驗證，那要怎麼做呢？

<br>

## 查詢目前有沒有設定 MFA

可以先查詢目前有沒有設定 MFA，可以透過以下指令：

```bash
aws iam list-mfa-devices --user-name {USER_NAME}
```

將 `{USER_NAME}` 換成自己的使用者名稱，如果有設定 MFA，就會顯示出來，如下：

<br>

{{< figure src="/aws/cli-mfa/5.png" width="650" caption="顯示 mfa 裝置" >}}

<br>

## 產生 MFA Token

我們確定我們有 MFA 設定後，就可以透過已經設定好的 MFA 以及剛剛的 SerialNumber 來產生 MFA Token，指令如下：

```bash
aws sts get-session-token --duration-seconds 129600 \
 --serial-number {MFA_DEVICES_NUMBER} \
 --token-code {MFA_CODE}
```

首先 `--duration-seconds` 參數是 MFA Token 的有效時間，單位是秒，預設是 12 小時，我們可以調整 900 秒 (15 分鐘) 到 129600 秒 (36 小時)，root 使用者憑證，範圍為 900 秒 (15 分鐘）到 3600 秒 (1 小時)，大家可以自行調整。

`{MFA_DEVICES_NUMBER}` 換成剛剛查詢到的 MFA 裝置的 SerialNumber，最後將 `{MFA_CODE}` 換成你的 MFA 的 Code。

<br>

一起來看一下輸出結果：

```bash
{
    "Credentials": {
        "AccessKeyId": "ASIA2HCXXXXXXXXXXXXX",
        "SecretAccessKey": "25CBN6CjRNLPukXXXXXXXXXXXXX",
        "SessionToken": "FwoGZXIvYXdzEXXXXXXXXXXXXX",
        "Expiration": "2024-08-02T19:42:36+00:00"
    }
}
```

AccessKeyId、SecretAccessKey 就跟 `aws configure` 拿到的 Access Key ID 和 Secret Access Key 是一樣的類型，只是還多一個 SessionToken，這個 SessionToken 是有經過 MFA 驗證過，所以可以用來執行需要 MFA 驗證條件的 Policy。

<br>

## 寫入設定檔

當你拿到 Token 以外，你不可能每次下指令都帶 Token，所以我們要把它寫到 `~/.aws/credentials` 檔案中，這樣就可以自動帶入 Token 了。

我們在 `aws configure` 的時候，會有一個 `profile` 的選項，這個選項就是用來設定不同的憑證，預設是 `default`，我們可以自己設定一個新的憑證，如下：

```bash
[mfa]
aws_access_key_id = ASIA2HCXXXXXXXXXXXXX
aws_secret_access_key = 25CBN6CjRNLPukXXXXXXXXXXXXX
aws_session_token = FwoGZXIvYXdzEXXXXXXXXXXXXX
```

<br>

所以當你要使用 MFA Token 的時候，只要在指令後面加上 `--profile mfa` 就可以了。

<br>

{{< figure src="/aws/cli-mfa/6.png" width="850" caption="就可以用 poc-general-policy 的 action 囉～" >}}

<br>

當然，如果每次都需要下指令去取得 Token 也是很麻煩的，所以我寫了一個 Shell Script 來幫我們取得 Token 並寫入 `~/.aws/credentials` 檔案中，這樣就可以省去每次都要下指令的麻煩了。程式也會同步到 [Github](https://github.com/880831ian/aws-cli-mfa)，大家可以自行下載使用。

```bash
#! /bin/bash

USER_NAME=$1
MFA_CODE=$2

# 顏色設定
RED="\033[1;31m"
GREEN="\033[1;32m"
YELLOW="\033[1;33m"
BLUE="\033[1;34m"
WHITE="\033[0m"

if [ -z $USER_NAME ] || [ -z $MFA_CODE ]; then
    echo -e "\n${YELLOW}提醒說明：請輸入使用者名稱 與 MFA 驗證碼\n格式如下：./aws_mfa.sh ian_zhuang 235821 << (請依照手機上的 MFA 號碼)。${WHITE}\n"
    exit 1
fi

if [[ ! "$MFA_CODE" =~ ^[0-9]{6}$ ]]; then
    echo -e "\n${YELLOW}提醒說明：MFA 驗證碼只支援 6 碼數字。${WHITE}\n"
    exit 1
fi

MFA_DEVICES_NUMBER=$(aws iam list-mfa-devices --user-name ${USER_NAME} | jq -r '.MFADevices[0].SerialNumber')

if [ -z $MFA_DEVICES_NUMBER ]; then
    echo -e "\n${RED}錯誤說明：MFA 裝置不存在，請確認使用者名稱是否輸入正確${WHITE}\n"
    exit 1
fi

MFA_INFO=$(aws sts get-session-token --duration-seconds 129600 --serial-number ${MFA_DEVICES_NUMBER} --token-code ${MFA_CODE})

if [ $? -eq 0 ]; then
    MFA_ACCESS_KEY_ID=$(echo $MFA_INFO | jq -r '.Credentials.AccessKeyId')
    MFA_SECRET_ACCESS_KEY=$(echo $MFA_INFO | jq -r '.Credentials.SecretAccessKey')
    MFA_SESSION_TOKEN=$(echo $MFA_INFO | jq -r '.Credentials.SessionToken')
    MFA_EXPIRATION=$(echo $MFA_INFO | jq -r '.Credentials.Expiration')

    # 計算 MFA 到期時間(轉換時區)
    MFA_0_TIME=$(date -j -f "%Y-%m-%dT%H:%M:%S" "$(echo "$MFA_EXPIRATION" | sed 's/+00:00//')" "+%Y-%m-%d %H:%M:%S")
    MFA_8_TIME=$(date -j -v+8H -f "%Y-%m-%d %H:%M:%S" "$MFA_0_TIME" "+%Y-%m-%d %H:%M:%S")

    MFA_CREDENTIALS="[mfa]\naws_access_key_id = $MFA_ACCESS_KEY_ID\naws_secret_access_key = $MFA_SECRET_ACCESS_KEY\naws_session_token = $MFA_SESSION_TOKEN"

    # 檢查是否已經有 [mfa] 區塊
    if grep -q "\[mfa\]" ~/.aws/credentials; then
        # 更新 [mfa] 區塊
        sed -i '' '/\[mfa\]/,/^$/d' ~/.aws/credentials
        echo -e "$MFA_CREDENTIALS" >>~/.aws/credentials
    else
        # 新增 [mfa] 區塊
        echo -e "\n\n$MFA_CREDENTIALS" >>~/.aws/credentials
    fi

    echo -e "\n${GREEN}MFA 驗證成功，將 Key 跟 Token 加入or修改到 ~/.aws/credentials 檔案中${WHITE}"
    echo -e "${BLUE}到期時間：${MFA_8_TIME}${WHITE}\n"
else
    echo -e "\n${RED}MFA 驗證失敗，請確認 MFA 驗證碼是否正確${WHITE}\n"
    exit 1
fi
```

<br>

只需要執行 `./aws_mfa.sh {USER_NAME} {MFA_CODE}` 就可以了，程式會自動幫你取得 Token 並寫入 `~/.aws/credentials` 檔案中。

<br>

{{< figure src="/aws/cli-mfa/7.png" width="700" caption="執行腳本" >}}

<br>

## 參考資料

How do I use an MFA token to authenticate access to my AWS resources through the AWS CLI?
：[https://repost.aws/knowledge-center/authenticate-mfa-cli](https://repost.aws/knowledge-center/authenticate-mfa-cli)
