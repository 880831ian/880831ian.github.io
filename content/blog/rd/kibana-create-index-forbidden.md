---
title: "Kibana 新增 index 索引時一直轉圈圈以及顯示 HTTP 403 Forbidden"
type: docs
weight: 4
---

前幾天在工作使用 Kibana 時，想要新增一個新的索引，發現選擇索引並按下新增的按鈕，會一直轉圈圈，等了一陣子，使用開發工具 F12 查看，跳出了 HTTP 403 Forbidden，到底是什麼原因導致的呢！？我們一起看下去吧，會從問題的出現到問題原因再到如何解決問題，來仔細介紹，希望大家不要像我一樣踩到雷 🤣

<br>

## 問題的出現

~~那天是個變冷的 12 月~~，要幫 RD 同仁新增 Kibana 的索引時，照往常一樣輸入 Index pattern，按下一步，選擇 Configure settings ：

<br>

{{< figure src="/rd/kibana-create-index-forbidden/1.png" width="800" caption="輸入 Index pattern (圖片為範例，正常都是用 -* )" >}}

<br>

按下新增索引，他就開始無限的轉圈圈，有去查看 ElasticSearch 的 log，發現也沒有特別的錯誤訊息

<br>

{{< figure src="/rd/kibana-create-index-forbidden/2.png" width="800" caption="新增索引後，一直轉圈圈" >}}

<br>

接著想說打開開發者工具 F12 來看看，是卡在哪一個點，卻發現有幾個紅字寫著 HTTP 403 Forbidden

<br>

{{< figure src="/rd/kibana-create-index-forbidden/3.png" width="800" caption="開發者工具網頁內容顯示 HTTP 403 Forbidden" >}}

<br>

想說為什麼會有 HTTP 403 Forbidden，之前也沒有看過類似的錯誤訊息，於是就開始在網路上亂晃，最後在同事的幫助下找到了一個跟我們情況很相似的文章 [Kibana 创建索引 POST 403 (forbidden) on create index](https://www.cnblogs.com/caoweixiong/p/10972120.html)

<br>

## 問題原因

文章的說明是索引變成只允許讀取的狀態，其原因是因為出現這個 HTTP 403 Forbidden 前，ElasticSearch 的空間滿了，導致 Kibana 會自動的將索引改成只允許讀取的狀態，我們來看一下剛剛的 Index 狀態是不是像他說的一樣變成只允許讀取的狀態呢

<br>

{{< callout tip "怎麼查看索引的狀態呢！？" >}}
可以到 kibana 的 Dev Tools 下指令來查詢歐，輸入 `GET _settings`，就會顯示以下圖片的內容囉
{{< /callout >}}

<br>

{{< figure src="/rd/kibana-create-index-forbidden/4.png" width="900" caption="索引只允許讀取的設定變成" >}}

<br>

發現正如文章所說，是因為 read_only_allow_delete 的狀態變成了 `true`，所以才沒辦法新增索引～

<br>

## 如何解決問題

我們查看文章的解決辦法，有兩種辦法，一個是擴大 ElasticSearch 的空間，以及使用 kibana 的 Dev Tools 下指令修改，那我們兩種都有做，這邊就直接介紹下指令需要輸入什麼～

需要再 Dev Tools 輸入以下指令，來修改 Index 的狀態：

<br>

```json
PUT _settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```

<br>

{{< figure src="/rd/kibana-create-index-forbidden/5.png" width="600" caption="Dev Tools 修改 Index 的狀態" >}}

<br>

最後我們用 `GET _settings` 檢查一下索引狀態是不是已經變回原本的了：

<br>

{{< figure src="/rd/kibana-create-index-forbidden/6.png" width="900" caption="索引只允許讀取的設定變成 false" >}}

<br>

最後就可以順利新增索引拉 👍👍👍

<br>

{{< figure src="/rd/kibana-create-index-forbidden/7.png" width="900" caption="順利新增索引" >}}

<br>

## 參考資料

Kibana 创建索引 POST 403 (forbidden) on create index：[https://www.cnblogs.com/caoweixiong/p/10972120.html](https://www.cnblogs.com/caoweixiong/p/10972120.html)
