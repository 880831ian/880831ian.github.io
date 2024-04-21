---
title: "什麼是 Opentelemetry？可觀測性 (Observability) 又是什麼？"
type: docs
weight: 3
---

在介紹 Opentelemetry 之前，我們要先了解一下目前軟體架構以及基礎設施的演進：

<br>

![](/opentelemetry/opentelemetry-observability/1.png "軟體架構以及基礎設施的演進")

<br>

第一階段在軟體架構設計上較為簡單，不會有什麼特別需要拆分出來的程式，所以都是一整包的程式，再測試以及除錯也比較不會有什麼問題。基礎設施都是使用 VM 或是使用放在 IDC 的機房來當 Server。

第二階段隨著雲端技術的推出，會開始將服務搬上雲供應商提供的 IaaS 服務，或是使用私有雲給企業放置較機密的內容，其他則放置公有雲上，達成混合雲的模式。

第三階段隨著雲端技術越來越成熟，有更多的雲端 IaC 以及功能推出，會開始考慮使用分散式的系統架構，將 DB 等服務也改用 Cloud SQL 的方式。在基礎設施上也隨著容器化的技術成熟而進入新的時代。

第四階段已經使用 docker 來管理好一陣子，但發現虛擬容器技術在管理上十分不方便，因此 K8s 逐漸盛行，將架構從分散式改成微服務的方式進行，在讓開發團隊使用上可以更靈活且容易。

<br>

雖然使用 K8s 可以讓我們的服務更靈活方便，但也會將服務切的越來越細，這時會讓開發變的十分複雜，我們在架構上從一開始的單體式架構，變成分散式架構，再到最後的微服務，讓開發人員需要處理的事情會越來越多。服務要如何連線？Log 要如何記錄？以及當一個請求會經過多個服務時，相對的延遲也會增加，這時要怎麼去處理等。在監控上，因為服務切分得很細，當線上有一個服務有問題時，要如何快速的找到問題點也是一大挑戰。

<br>

當我們使用分散式系統或是微服務時發生故障時，會很難快速的恢復服務，因為每個服務都互相依賴，在以往都是透過經驗以及對系統的了解來得以解決。那有什麼其他的方式，能夠讓我們更快掌握每個服務呢？我們先來了解一個名詞：可觀測性(Observability)

<br>

## 可觀測性(Observability)

可觀測性有三個重要的特性，分別是：

Metrics 負責監控系統有什麼狀況，當要發生服務故障前可以透過設定閥值搭配告警提早得知。

Logs 當問題發生時，可以用來查看故障時正在執行哪些服務，以及產生的錯誤資訊。

Traces （後面詳細介紹）

<br>

![](/opentelemetry/opentelemetry-observability/2.png "可觀測性三大支柱")

<br>

我們對於 Metrics 跟 Logs 有基本的了解，所以我這邊會注重在 Traces 的部分：

當有多個微服務的複雜分散式系統，用戶的請求會由系統中的多個微服務進行處理。Metrics 跟 Logs 可以提供我們有關系統如何去處理這些請求的一些資訊，但沒有辦法提供微服務的詳細訊息以及他是如何影響客戶端的請求。這時候就需要透過 Trace 來協助我們追蹤。

<br>

Trace 可以在連續的時間維度上，透過 Trace 以及 Span 關聯，把空間給排列展示出來，並且有 Trace-Context 規範，能夠直觀的看到請求在分散式系統中經過所有服務的紀錄。

什麼是 Trace、Span 、Trace-Context 呢？

我們先說 Span，Span 又可以叫跨度，是系統中最小的單位，可以看下方圖片，SpanA 的資料是來源 SpanB，SpanB 來源是 SpanC 等等，每一個 Span 可以把它想成一個請求後面所有經過服務的工作流程，例如：nginx_module、db、redis 等等。

請求的整個過程叫做 Trace，那他要怎麼知道 SpanA ~ SpanE 是同一個請求呢？

<br>

就需要透過 TraceID 以及 SpanID 來記錄：

<br>

TraceID：是唯一的 ID，用於識別整個分散式追蹤的一條請求路徑。在下方圖片中，當請求進入時，就會被賦予一個 TraceID，所有有經過的 Span 都會記錄此 TraceID，這樣才可以把不同服務依據 TraceID 關聯成同一個請求。

SpanID：是一條請求路徑中單個操作唯一的 ID。追蹤路徑是由多個 Span 組成，每個 Span 都代表一個操作或特定的時間段。當請求進入時，每個服務就會產生一個 Span 來代表它處理請求的的時間。這些 Span 使用 TraceID 來連接再一起，形成完整的請求追蹤。

<br>

![](/opentelemetry/opentelemetry-observability/3.png "Trace 示意圖")

<br>

那要怎麼查看每個 Span 的紀錄內容呢，就需要 Trace-Context：

會放置一些用於追蹤和識別請求的上下文信息，例如 Trace ID、Span ID 和其他相關的數據。這些上下文信息可以是一些關鍵的數據，可以幫助我們在整個分佈式系統中追蹤請求的路徑，並將相關請求和操作關聯起來。

<br>

![](/opentelemetry/opentelemetry-observability/4.png "ECK Trace 示意圖")

上面的圖片中，可以看到 call /product/XXXX 後，會經過需多的 Span，隨便點擊一個 Span 可以看到它記錄的 Trace-Context，以及都會包含 TraceID 及 SpanID

<br>

![](/opentelemetry/opentelemetry-observability/5.png "ECK Trace 示意圖")

<br>

Trace 優點可以看到跨維度看到中間的資訊，對於找到問題以及瓶頸十分方便，但缺點就是因為需要在 Span 中產生 ID 以及內容，需要在程式裡面加入一定的套件以及調整程式碼。

<br>

所以我們在可觀測性(Observability)最終的目的是希望可以透過可觀測性工具讓我們知道：

1. 請求通過哪些服務
2. 每個服務在處理請求時做了些什麼
3. 如果請求很慢，瓶頸在哪邊
4. 如果請求失敗，錯誤點在哪
5. 請求的路徑是什麼
6. 為什麼花這麼長的時間

<br>

## Opentelemetry

那我們這次要介紹的可觀測性(Observability)工具就是 Opentelemetry，縮寫 OTel，它是由 CNCF (Cloud Native Computing Foundation) 組織孵化的開源專案，在 2021 年 5 月由 OpenTracing 與 OpenCensus 兩個框架合併，結合兩項分散式追蹤框架最重要的特性成為下一代收集遙測數據的新標準。

<br>

- telemetry

又叫做遙測，是指能夠跨越不同系統來收集資料 (包含 LOG、Metric、trace) 的能力。

<br>

我們可以看一下官網的說明：

Opentelemetry 是雲原生的可觀測性(Observability)框架，提供標準化的 API、SDK 與協議自動檢測、蒐集、導出遙測數據資料 (Metrics、Log、Trace)，並支援 W3C 定義的 Http trace-context 規範，降低開發者在搜集遙測數據上的困難度，以及方便進行後續分析以及性能的優化。

<br>

![](/opentelemetry/opentelemetry-observability/6.jpg "Opentelemetry")

<br>

在 OpenTelemetry 核心元件如下：

- API：開發人員可以透過 OpenTelemetry API 自動生成、蒐集應用程式(Application)的遙測數據資料(Metrics, Log, Trace)，每個程式語言都需實作 OpenTelemetry 規範所定義的 API 方法簽章。

- SDK：是 OpenTelemetry API 的實現。

- OTLP：規範定義了遙測數據的編碼與客戶端及服務器之間如何交換的協議 (gRPC、HTTP)。

- Collector：OpenTelemetry 中儲存庫，用於接收、處理、導出遙測數據到各種後端平台。

<br>

## 參考資料

Observability：[https://linkedin.github.io/school-of-sre/level101/metrics_and_monitoring/observability/](https://linkedin.github.io/school-of-sre/level101/metrics_and_monitoring/observability/)

[OpenTelemetry] 現代化監控使用 OpenTelemetry 實現 : 可觀測性(Observability)：[https://marcus116.blogspot.com/2022/01/modern-monitoring-using-openTelemetry-with-Observability.html](https://marcus116.blogspot.com/2022/01/modern-monitoring-using-openTelemetry-with-Observability.html)

[OpenTelemetry] 現代化監控使用 OpenTelemetry 實現 : OpenTelemetry 開放遙測：[https://marcus116.blogspot.com/2022/01/opentelemetry-opentelemetry.html](https://marcus116.blogspot.com/2022/01/opentelemetry-opentelemetry.html)

淺談 Observability(下)：[https://ithelp.ithome.com.tw/m/articles/10287598](https://ithelp.ithome.com.tw/m/articles/10287598)

Manage services, spans, and traces in Splunk APM：[https://docs.splunk.com/Observability/apm/apm-spans-traces/traces-spans.html](https://docs.splunk.com/Observability/apm/apm-spans-traces/traces-spans.html)
