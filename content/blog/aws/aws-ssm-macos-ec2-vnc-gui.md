---
title: "透過 AWS SSM 連線至 macOS EC2 並啟用 VNC 遠端桌面 GUI"
type: docs
weight: 9984
description: 透過 AWS SSM 連線至 macOS EC2 並啟用 VNC 遠端桌面 GUI
images:
  - aws/aws-ssm-macos-ec2-vnc-gui/og.webp
date: 2025-01-03
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - AWS
  - SSM
  - Systems Manager Agent
  - macOS
  - EC2
  - VNC
  - GUI
---

## 名詞說明

- AWS：Amazon Web Services 亞馬遜的雲端服務

- SSM：AWS Systems Manager 可以透過 AWS 直接連上內部 VPC 的機器 (不需要在 public subnet)

- EC2：AWS 的虛擬雲端計算服務

- VNC：Virtual Network Computing 遠端控制

<br>

## 步驟

1. 確定使用的 User 有 `ssm:StartSession` 權限

2. 確定要連線的 EC2 有 IAM Role 並被賦予 `AmazonSSMManagedInstanceCore` 權限

<br>

{{< figure src="/aws/aws-ssm-macos-ec2-vnc-gui/1.webp" width="1000" caption="AmazonSSMManagedInstanceCore 權限" >}}

<br>

3. 確定 EC2 上有安裝 SSM Agent，macOS 可以查看：[在 macOS 專用 EC2 執行個體使用 SSM Agent - AWS Systems Manager](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/ssm-agent-macos.html)

4. 接著我們等到 EC2 確定啟動完畢 (macOS 啟動到可以連線需要約 5 分鐘)，可以以下指令連線測試

```shell
aws ssm start-session --target <instance-id>
```

`<instance-id>` 請帶上 EC2 的 instance id

<br>

{{< figure src="/aws/aws-ssm-macos-ec2-vnc-gui/2.webp" width="900" caption="SSM 連線" >}}

<br>

確認看看是否可以正常連線，可以下 `sudo -i` 來切換成 root

<br>

5. 當我們確定可以透過 SSM 連上 EC2 後，接著我們要安裝並啟動 macOS 畫面共用，請安裝以下指令

```shell
sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist com.apple.screensharing -dict Disabled -bool false
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
```

<br>

6. 接著要透過 VNC 連線，需要指定 User，但我們沒辦法拿到 root 密碼，所以只能在建立一個帳號，或是使用預設的 ec2-user，這邊先使用預設帳號，並新增密碼

```shell
sudo /usr/bin/dscl . -passwd /Users/ec2-user
```

<br>

7. 接著開啟新的一個 Terminal 下此指令，透過 SSM 將本機與 EC2 內部的 Port 做 Port forwarding

```shell
aws ssm start-session \
    --target <instance-id> \
    --document-name "AWS-StartPortForwardingSession" \
    --parameters '{"portNumber":["5900"], "localPortNumber":["5900"]}'
```

就會看到這個類似的畫面

<br>

{{< figure src="/aws/aws-ssm-macos-ec2-vnc-gui/3.webp" width="750" caption="SSM Port Forwarding" >}}

<br>

8. 開啟瀏覽器或是開啟 macOS 內建的螢幕共享，打上

```shell
vnc://localhost:5900
```

就可以看到此畫面囉

<br>

{{< figure src="/aws/aws-ssm-macos-ec2-vnc-gui/4.webp" width="800" caption="VNC 連線到 macOS" >}}

<br>

## 備註

假設是建立其他的 User (不是用 ec2-user)，在第一次連線時，還是需要先登入 ec2-user 才可以切換其他帳號

<br>

## 參考資料

How can I access my Amazon EC2 macOS instance through a GUI?：[Access an EC2 macOS instance through a GUI](https://repost.aws/knowledge-center/ec2-mac-instance-gui-access)

AWS Systems Manager 使用 Port-Forwarding：[https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#sessions-start-port-forwarding](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#sessions-start-port-forwarding)