---
title: "串接 Jira 工單自動化通知到 Google Chat"
type: docs
weight: 99
description: 如何串接 Jira 工單自動化通知到 Google Chat
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

由於公司目前使用 Google Chat 來當作通訊工具，所以我希望能夠在 Jira 工單的狀態變更時，能夠自動送通知到 Google Chat。

那 Google Chat 有提供幾種方式可以串接，我們這邊用 Webhook 來串接。

{{% steps %}}

### 建立聊天室

我們先建立一個聊天室，名稱跟詳細設定就請自行設定

<br>

{{< figure src="/agile/jira-to-google-chat/2.png" width="450" caption="建立聊天室" >}}

<br>

### 選擇應用程式與整合

進入聊天室，點選聊天室名稱 (例如：jira 串接 google chat 測試)，選擇應用程式與整合

<br>

{{< figure src="/agile/jira-to-google-chat/3.png" width="400" caption="點選應用程式與整合" >}}

<br>

### 新增 Webhook

選擇 Webhook，並且點擊新增 Webhook，輸入名稱以及自定義的圖片，再點擊儲存

<br>

{{< figure src="/agile/jira-to-google-chat/4.png" width="700" caption="建立 Webhook" >}}

<br>

### 複製 Webhook 網址

建立完成後，就會在底下出現剛剛建立的 Webhook，點選複製 Webhook 網址

<br>

{{< figure src="/agile/jira-to-google-chat/5.png" width="700" caption="複製 Webhook 網址" >}}

<br>

### 測試 Webhook

我們可以使用 curl 來測試一下 Webhook 是否正常，可以參考以下指令：

```
curl -X POST "<<請帶入剛剛所複製的 Google Chat Webhook 連結>>" \
     -H "Content-Type: application/json; charset=UTF-8" \
     -d '{"text": "Hello World"}'
```

在剛剛建立的聊天室裡面，有出現 `Hello World` 就代表成功了

<br>

{{< figure src="/agile/jira-to-google-chat/6.png" width="700" caption="測試 Webhook 成功" >}}

<br>

{{% /steps %}}

<br>

## 設定 Jira

接下來我們要設定 Jira，當工單狀態變更時，就會送通知到 Google Chat，我們會使用到 Jira 的工單專案的自動化功能。

自動化可以自訂觸發條件，也可以寫判斷，針對不同的情境來做不同的動作。

下面是目前我們的設定，當工單狀態變更時，從 `To Do` 變成 `Approved by SRE`，就會觸發此自動化。

<br>

{{< figure src="/agile/jira-to-google-chat/7.png" width="1200" caption="範例自動化設定" >}}

<br>

由於我們 SRE 內部有拆組別，所以需要判斷這個工單申請的 RD 單位，是哪個小組負責的，我們這邊會使用 `if` 條件，來判斷摘要是否包含對應小組的關鍵字。

最後 `Then` 如果符合條件，就會使用 『傳送網路要求』 這個動作來送通知到 Google Chat。

<br>

{{< figure src="/agile/jira-to-google-chat/8.png" width="500" caption="動作選項" >}}

<br>

打開傳送網路要求，在 Web 要求 URL，就是貼上剛剛在 Google Chat 建立的 Webhook 連結，HTTP 方法選擇 Post，下面就可以再自訂資料裡面設定想要傳送的內容。

如果要傳送的內容是變數，例如工單緊急程度、工單號碼等等，就需要使用 `{{}}` 來包住變數，例如：`{{issue.key}}`、`{{issue.fields.summary}}`、`{{issue.fields.status.name}}` 等等。

不知道有那些變數，可以點擊圖片箭頭的 `{}` 圖示，裡面會告訴你哪些變數可以使用：

<br>

{{< figure src="/agile/jira-to-google-chat/9.png" width="800" caption="變數參考" >}}

<br>

如果是使用套件，例如：Template，變數就不會出現在 `{}` 圖示裡面，可以直接帶入 `{{issue.template}}`。

## 使用 Google Chat Card

上面的教學已經完成簡單的通知，但訊息顯示的方式不夠直覺，因此我們可以使用 Google Chat Card 來顯示更多資訊。

[Google Chat Cards v2](https://developers.google.com/workspace/chat/api/reference/rest/v1/cards) 提供了更多的選項，例如：`header`、`sections`、`widgets`、`buttons` 等等，可以讓訊息更加豐富。

<br>

{{< figure src="/agile/jira-to-google-chat/10.png" width="500" caption="官方 Card 範例" >}}

<br>

目前 Card 只能使用 v2 版本，它實際上的格式就是一個 JSON，我們可以使用官方提供的 [UI Kit Builder](https://addons.gsuite.google.com/uikit/builder?hl=zh-tw) 來幫助我們建立 Card。

可以從左側，選擇想要的元件來自訂 card，右側會預覽顯示結果，最後直接複製 JSON 內容，再貼到 Jira 的自訂資料裡面。

<br>

{{< figure src="/agile/jira-to-google-chat/11.png" width="1200" caption="UI Kit Builder 工具" >}}

<br>

最後要注意的是，使用 [UI Kit Builder](https://addons.gsuite.google.com/uikit/builder?hl=zh-tw) 所產生的 JSON 格式，不是 Google Chat Card 的最終格式，需要自行修改 (很神奇吧 ┐(´д`)┌)。

需要再多以下的架構，才會正常運作：

```
{
  "cardsV2": [
    {
      "cardId": "unique-card-id",
      "card": {
        <<這邊才放入 UI Kit Builder 產生的 JSON 內容>>
      }
    }
  ]
}

```

最終成果如下：

<br>

{{< figure src="/agile/jira-to-google-chat/12.png" width="450" caption="最終成果" >}}

<br>

有太多內容要馬賽克 (´・ω・`)

## 參考資料

Google Chat Cards v2：[https://developers.google.com/workspace/chat/api/reference/rest/v1/cards](https://developers.google.com/workspace/chat/api/reference/rest/v1/cards)

UI Kit Builder：[https://addons.gsuite.google.com/uikit/builder?hl=zh-tw](https://addons.gsuite.google.com/uikit/builder?hl=zh-tw)

Google Icon：[https://fonts.google.com/icons](https://fonts.google.com/icons)
