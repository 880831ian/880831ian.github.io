---
title: "Fluentd-Server 出現 Fluent::Plugin::Elasticsearch Error 400 - Rejected by Elasticsearch 錯誤解決"
type: docs
weight: 9998
date: 2022-12-08
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

先說結論 ... ~~EFK 真的好多雷 😆~~ 我們今天又再度踩雷拉，這次又是炸的分身碎骨，花了 4 個多小時才找到原因，也有可能是小弟我跟 EFK 不是很熟 XD。就如標題所說，我在抽 Log 的時候發現有部分的 Log 送不到 Kibana，檢查 Fluentd-Server 的 Log 發現裡面有 `Fluent::Plugin::Elasticsearch Error 400 - Rejected by Elasticsearch` 的錯誤訊息，到底這個 Error 400 是什麼咧，就跟著我一起重溫找問題的苦難吧 XD

<br>

## EFK 結構

在開始之前我先簡單說一下我現在的 EFK 結構：

{{< figure src="/rd/fluentd-server-show-fluent-plugin-elasticsearch-error400/1.png" width="900" caption="EFK 結構" >}}

現在的 EFK 結構如上圖，我在要抽 Log 的 Pod 中多塞一個 fluentd-bit container，將 fluentd-bit 與服務(例如 nginx) 的 log 掛相同路徑，讓 fluentd-bit 可以抽到服務 log，接著就透過 fluentd-server-svc 打到 fluentd-server pod，後面就是常見的 EFK 結構，就不多做說明。

<br>

## 問題的出現

我們就像平常一樣寫好整個 EFK 的 yaml，run 的時候也正常，可以收到 log，但很奇怪的是我們先送 `log_false` 的 log 可以正常收到，但再接著送 `log_true ` 就收不到，我們也嘗試反過來送，先送 `log_true`，再送 `log_false`，發現換收不到 `log_false` 的 log

<br>

- log_false

```json
{
  "result": false,
  "message": "我是馬賽克",
  "code": 11223344,
  "data": {
    "end_at": "2022-12-19 04:09:00"
  },
  "response_code": "326dgf241geh1596489",
  "job_name": "叮叮噹噹聖誕節",
  "method": "GET",
  "url": "url.com"
}
```

<br>

- log_true

```json
{
  "result": true,
  "data": true,
  "response_code": "wkCJl1dfr45671607965"
}
```

<br>

接著在 Fluentd-Server Log 中發現 Fluent::Plugin::Elasticsearch Error 400 - Rejected by Elasticsearch 錯誤

<br>

{{< figure src="/rd/fluentd-server-show-fluent-plugin-elasticsearch-error400/2.png" width="1000" caption="fluentd-server 跳出錯誤 LOG" >}}

<br>

我到網路上搜尋，看看有沒有人有遇到跟我一樣的問題，發現在 [uken/fluent-plugin-elasticsearch](https://github.com/uken/fluent-plugin-elasticsearch/issues/467) 也有其他人有遇到同樣的問題

<br>

{{< figure src="/rd/fluentd-server-show-fluent-plugin-elasticsearch-error400/3.png" width="1000" caption="uken/fluent-plugin-elasticsearch issus 詢問" >}}

有人說可能是 Elasticsearch 的 disk 滿了，也有人說是裝在 Fluentd-Server 的套件 fluent-plugin-elasticsearch 版本有問題，或是將 Elasticsearch 跟 Kibana index 刪除就可以，我有砍掉 index 再重新倒入 Log，還是會有問題。

一開始以為是 `log_false` 跟 `log_true` 的欄位不同，所以才會有這個問題發生（你看就知道我跟 EFK 不熟吧ＸＤ），所以有另外倒一些其他服務(欄位也不同)的 log 進去測試，發現是可以正常抽到，且 kibana 那邊會依照 es 提供的欄位自動新增，所以也不是欄位的問題，那最後的問題是什麼呢？

<br>

## 如何解決問題

最後在我們仔細檢查後發現，`log_false` 跟 `log_true` 的欄位其中一樣存的資料型態不太一樣，發現其中的 `data` 欄位在 `log_false` 的資料型態是 array，在 `log_true` 存的時候是字串，所以導致當其中一個 log 寫入後，另一個 log 就會因為同欄位的資料型態有所不同，而無法寫入 log，且在 Fluentd-Server 會出現標題的 Error 400 原因拉～

最後重新調整欄位的資料型態，就順利解決此次問題 ✌️✌️✌️

<br>

## 參考資料

Elasticsearch 400 error #467：[https://github.com/uken/fluent-plugin-elasticsearch/issues/467](https://github.com/uken/fluent-plugin-elasticsearch/issues/467)
