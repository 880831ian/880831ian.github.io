---
title: "在 AWS EC2 macOS 上安裝 Gitlab-Runner 注意事項"
type: docs
weight: 9983
description: 在 AWS EC2 macOS 上安裝 Gitlab-Runner 注意事項
images:
  - aws/aws-ec2-macos-gitlab-runner/og.webp
date: 2025-02-25
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - AWS
  - EC2
  - macOS
  - Gitlab
  - Runner
  - gitlab-runner
  - CI/CD
---

會寫此文章主要原因是公司會需要打包多個 IOS APP，之前是使用 github 來進行打包，但因為費用以及管理問題，所以想測試改用 AWS 的 EC2 macOS 來當作 Runner。

<br>

## 建立 macOS Runner 機器

AWS 能夠建立 macOS 的 EC2 Instances，使用的 instance_type 會是 mac2.metal 或 mac1.metal，這兩種型號的差異可以參考 [AWS 官方文件](https://aws.amazon.com/tw/ec2/instance-types/mac/)，且會需要使用專用主機(Dedicated Hosts)，這個專用主機開下去，只能 24 小時後才可以關閉。

IaC 上需要多設定 `create_dedicated_host` 或 `host_id`，下面說明兩個設定：

`create_dedicated_host`：如果是 macOS 才需打開此設定，會自動建立專用主機，自動把 host_id 帶入。

`host_id`：由於專用主機 24 小時內不能刪除，所以當刪除 macOS 的 EC2，有時候專用主機還會存在，這時候新的 EC2 可以直接把 host_id 帶入，就可以沿用，不需要額外再建立專用主機 (記得把 create_dedicated_host 關閉)。

<br>

> [!NOTE] 說明
> 上面的 IaC 是我使用 Terraform 所寫的 EC2 module，後續會出文章來說明每一個 IaC 如何使用，到時候也會更新連結到這篇文章。

<br>

另外根據 instance_type 會需要選擇對應的 ami，例如：instance_type 是 mac2.metal，mac2.metal 是 arm 架構，就需要選擇 arm 的 ami，也要記得選對 OS 版本。

<br>

{{< figure src="/aws/aws-ec2-macos-gitlab-runner/1.webp" width="1000" caption="不同的 instance_type 需要選擇對應的 AMI" >}}

<br>

## macOS EC2 注意事項

在測試的過程中，有發現一些 macOS EC2 的限制，提供給大家參加(有些上面有提到)：

1. 需要選擇 macOS 版本 (macOS Sequola、macOS Sonoma、macOS Ventura)

<br>

{{< figure src="/aws/aws-ec2-macos-gitlab-runner/2.webp" width="700" caption="有不同的 OS 版本可以選擇" >}}

<br>

2. 只有兩種機器規格可以選擇，分別是：
    - mac1.metal：12 vCPU、32 G MEM
    - mac2.metal (ARM 架構)：8 vCPU、16 G MEM

<br>

3. 建立需要用專用主機 (所以 IaC 需要有多選擇租賃類型、以及專用主機 ID)，建立專用主機後，mac 類型專用主機需要等 24小時後，才可以釋放

4. 不能使用 Spot Instance

<br>

{{< figure src="/aws/aws-ec2-macos-gitlab-runner/3.webp" width="900" caption="會顯示無法使用 Spot VM" >}}

<br>

5. 內建有 SSM，但建立完機器不能馬上連線，需要等約 5 分鐘

6. 如果要建立 AMI 後，還需要使用機器，記得把重啟機器選項關掉，macOS 機器重啟會蠻久的

7. 關機+destroy 需要約 10 分鐘

8. 測試建立 macOS 的 AMI 正常 (資料有保留)

9. macOS 預設安裝的資料會佔用 9.6G 左右的 Disk 空間 (ami-09c1170ae6e30597d，規格：macOS Sonoma、mac2.metal)

<br>

{{< figure src="/aws/aws-ec2-macos-gitlab-runner/4.webp" width="900" caption="預設的 Disk 使用情況" >}}

<br>

## 安裝 Gitlab-Runner

需要切換到 ec2-user 內，並執行此腳本，記得要輸入大寫的單位名稱 (此為公司設計的命名規則，可以依照自己需求修改)

```bash
#!/bin/bash

# 取得系統架構
ARCH=$(uname -m)

# 要求輸入符合格式的大寫單位名稱
while true; do
    read -p "請輸入大寫的英文單位名稱: " UNIT_NAME
    if [[ "$UNIT_NAME" =~ ^[A-Z]+(-[A-Z]+)?$ ]]; then
        break
    else
        echo -e "無效的單位名稱，請輸入符合格式的值\n"
    fi
done

# 基本下載 URL
BASE_URL="https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-darwin"

# 根據架構選擇檔案名稱
if [ "$ARCH" == "arm64" ]; then
    FILE_NAME="arm64"
else
    FILE_NAME="amd64"
fi

if [ ! -f /usr/local/bin/gitlab-runner ]; then
    # 下載並安裝
    echo "下載 GitLab Runner，架構：$ARCH"
    sudo curl --output /usr/local/bin/gitlab-runner "${BASE_URL}-${FILE_NAME}"

    sudo chmod +x /usr/local/bin/gitlab-runner
fi

gitlab-runner register --non-interactive \
    --name "$UNIT_NAME-MacOS-Runner" \
    --url "<GitLab Domain>" \
    --registration-token <GitLab Token> \
    --executor shell \
    --tag-list "$UNIT_NAME-MacOS-Runner"
```

<br>

完成後，請執行以下指令來執行 Runner，會將 Log 寫到 gitlab-runner.log

```shell
gitlab-runner run > gitlab-runner.log 2>&1 &
```

注意！不要使用 `gitlab-runner start`，避免遇到 `FATAL: Failed to start gitlab-runner: exit status 134` 錯誤

<br>

{{< figure src="/aws/aws-ec2-macos-gitlab-runner/5.webp" width="900" caption="使用 start 會噴 134 錯誤" >}}

<br>

這個問題官方文件也有說明，start 會需要 LaunchAgent，然後 LaunchAgent 需要先透過 GUI 登入才可以，官方文件：[Install GitLab Runner on macOS | GitLab Docs](https://docs.gitlab.com/runner/install/osx/#fatal-failed-to-start-gitlab-runner-exit-status-134-on-gitlab-runner-start-command)