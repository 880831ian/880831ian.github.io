# 如何透過 OpenTelemetry 來收集 Ingress Nginx Controller 的 Metrics 與 Traces 並送到 Datadog 上
由於最近公司想要導入 Datadog，在測試過程中順便導入 OpenTelemetry 來收集 Metrics 與 Traces 並送到 Datadog 上 ～

<br>

🔥 這個範例比較特別，因為 Datadog 有提供 Ingress Nginx Controller 的 integrations，可以透過 Datadog Agent 來收集 Metrics，不需要透過 OpenTelemetry Collector 來收集。
( Datadog Agent 請參考：[https://docs.datadoghq.com/containers/kubernetes/](https://docs.datadoghq.com/containers/kubernetes/) )

程式部分也同步上傳到 github 上，[可以點我前往](https://github.com/880831ian/opentelemetry-ingress-nginx-controller)

<br>

## 檔案說明

- otel-collector.yaml：
  OpenTelemetry Collector 的設定檔，主要是設定要收集哪些 metrics、traces，並且要送到哪個 exporter，要注意的是 exporters 的 datadog 需要設定 site、api_key，以及 image 要記得用 otel/opentelemetry-collector-contrib，才會有 datadog 的 exporter。

- ingress-nginx-values.yaml：
  Ingress Nginx Controller 的設定檔，這邊的 podAnnotations 是為了讓 Ingress Nginx Controller 的 Pod 能夠透過 Datadog agent 收集 metrics 到 Datadog 才加上的。

  config 裡面的設定有很多，主要都是 openTelemetry 的設定，要注意的是 enable-opentelemetry 要設為 true，另外 otlp-collector-host 以及 otlp-collector-port 要送到哪個 collector 等等也要記得設定。
  另外如果想要將 LOG 與 Trace 串再一起，記得要把 log-format 設為 json，並且帶入，trace_id 與 span_id ( 這邊有多帶 dd.trace_id 是為了讓 datadog 可以自動串接 LOG & Trace )。

- nginx.yaml：
  一個簡單的 Nginx 整套服務 (Deployment、Service、Ingress)，要注意的是 Ingress 需要設定 annotations kubernetes.io/ingress.class: nginx (這個是 Ingress Nginx Controller 的預設 class name)，才會被 Ingress Nginx Controller 接管 (才會有 Load Balancer 的 IP)

<br>

## 執行步驟

1. 先 clone 這個 repo (廢話 xD)
2. 先建立 OpenTelemetry Collector，執行以下指令：

   ```shell
   helm upgrade collector \
    opentelemetry-collector \
    --repo https://open-telemetry.github.io/opentelemetry-helm-charts \
    --install \
    --create-namespace \
    --namespace opentelemetry \
    -f "otel-collector.yaml"
   ```

3. 再建立 Ingress Nginx Controller，執行以下指令：

   ```shell
   helm upgrade ingress-nginx \
    ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --install \
    --create-namespace \
    --namespace ingress-nginx \
    -f "ingress-nginx-values.yaml"
   ```

4. 接著建立測試用 Nginx 服務，執行以下指令：

   ```shell
   kubectl apply -f nginx.yaml
   ```

<br>

## 測試

當你執行完上面的步驟後，你會發現有產生兩個 namespace，一個是 ingress-nginx，另一個是 opentelemetry，並且會有 OpenTelemetry Collector、Ingress Nginx Controller、Nginx 等服務，如下：

<br>

{{< figure src="/opentelemetry/opentelemetry-ingress-nginx-controller/1.webp" width="800" caption="啟動服務" >}}

<br>

我們試著打 `http://nginx.example.com/` (測試網址，需要先在 /etc/hosts 綁定 Ingress Nginx Controller 咬住的 Load Balancer IP)，查看一下 Datadog 的 LOG，看看是否有收到 Nginx 的 LOG (此收集 LOG 的方式是透過在 cluster 上安裝 Datadog 的 agent)，如下：

<br>

{{< figure src="/opentelemetry/opentelemetry-ingress-nginx-controller/2.webp" width="1200" caption="Datadog LOG" >}}

<br>

接著查看 Datadog APM 的 trace，如下：

<br>

{{< figure src="/opentelemetry/opentelemetry-ingress-nginx-controller/3.webp" width="1200" caption="Datadog APM" >}}

<br>

由於我們在後面目前沒有串其他服務，所以只有一個 span，之後還有另外兩篇文章是介紹如何串其他服務 (會增加服務以及部分設定)，可以參考看看：[opentelemetry-roadrunner](https://github.com/880831ian/opentelemetry-roadrunner)、[opentelemetry-nodejs](https://github.com/880831ian/opentelemetry-nodejs)

<br>

順便看一下透過 Datadog Agent 收集的 Ingress Nginx Controller 的 Metrics，如下：

<br>

{{< figure src="/opentelemetry/opentelemetry-ingress-nginx-controller/4.webp" width="900" caption="Datadog Ingress Nginx Controller 的 Metrics" >}}

<br>

可以用這些 Metrics 來做 Dashboard，如下：

<br>

{{< figure src="/opentelemetry/opentelemetry-ingress-nginx-controller/5.webp" width="1000" caption="Datadog Dashboard" >}}

<br>

## 結論

透過 OpenTelemetry Collector 來收集 Ingress Nginx Controller 的 Metrics 與 Traces 並送到 Datadog 上，這樣就可以透過 Ingress Nginx Controller 的 Metrics 來做監控了，對於 RD 再開發上，有 Traces 也更方便 RD 他們找到程式的瓶頸 (有可能是服務導致的)。

<br>

## 參考資料

Configure Nginx Ingress Controller to use JSON log format：[https://dev.to/bzon/send-gke-nginx-ingress-controller-logs-to-stackdriver-2ih4](https://dev.to/bzon/send-gke-nginx-ingress-controller-logs-to-stackdriver-2ih4)

淺談 OpenTelemetry - Collector Compoents：[https://ithelp.ithome.com.tw/articles/10290703](https://ithelp.ithome.com.tw/articles/10290703)

