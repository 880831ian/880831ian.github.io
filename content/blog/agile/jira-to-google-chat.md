---
title: "串接 Jira 工單自動化通知"
type: docs
weight: 99
description: 如何串接 Jira 工單自動化通知
images:
  - agile/jira-to-google-chat/og.webp
date: 2024-08-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 介紹

我們使用 Jira 的 Ticket 來當作工單，其他單位如果需要請 SRE 協助，都需要填寫此工單，我們的流程是以下：

```
待審核 -> SRE 已簽核 -> 進行中 -> (阻塞) -> 完成
```

<br>

{{< figure src="/agile/jira-to-google-chat/1.png" width="750" caption="工單看板" >}}

<br>

我們會由資深工程師來審核，狀態會從 `待審核` 變成 `SRE 已簽核`，這時候我們希望能夠收到通知，才由當週值班的工程師來進行處理。

<br>

## 如何送通知到 Google Chat

由於公司目前使用 Google Chat 來當作通知工具，所以我們希望能夠將 Jira 工單的狀態變更時，能夠自動送通知到 Google Chat。

那 Google Chat 有提供幾種方式可以串接，我們這邊用 Webhook 來串接。

{{% steps %}}

### 步驟ㄧ

我們先建立一個聊天室

<br>

{{< figure src="/agile/jira-to-google-chat/2.png" width="450" caption="建立聊天室" >}}

<br>

### 步驟二

進入聊天室，點選聊天室名稱 (jira 串接 google chat 測試)，選擇應用程式與整合

<br>

{{< figure src="/agile/jira-to-google-chat/3.png" width="400" caption="點選應用程式與整合" >}}

<br>

{{% /steps %}}

首先，
