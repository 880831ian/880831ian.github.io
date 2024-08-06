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
