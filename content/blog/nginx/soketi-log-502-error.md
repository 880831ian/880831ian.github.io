---
title: "Soketi WebSocket Server LOG 不定時出現 502 error 以及 connect() failed (111: Connection refused)"
type: docs
weight: 999
---

此文章要來記錄一下 RD 同仁前陣子有反應使用 Soketi 這個 WebSocket Server 會不定時在 LOG 出現 502 error 錯誤訊息以及 connect() failed (111: Connection refused) while connecting to upstream，雖然說服務使用上不會影響很大，但還是希望我們可以協助找出 502 的原因。

<br>

{{< figure src="/nginx/soketi-log-502-error/1.png" width="1200" caption="出錯的 LOG" >}}

<br>

在開始找問題前，先簡單介紹一下 Soketi 是什麼東西好了，根據官網的說明，他是簡單、快速且有彈性的開源 WebSockets server，想要了解更多的可以到它[官網](https://docs.soketi.app/)去查看。

另外會把程式碼相關放到 GitHub >> [點我前往](https://github.com/880831ian/soketi-log-502-error)

<br>

## 解決過程

我們可以看到上方錯誤 LOG 中，發現有出現 502 error 以及 connect() failed (111: Connection refused) while connecting to upstream，這兩個錯誤都是由 Nginx 所產生的，那我們先來理解一下，Nginx 與 Soketi 之間的關係。

在使用上，RD 的程式會打 Soketi 專用的 Subdomain 來使用這個 WebSocket 服務，而在我們的架構上，這個 Subdomain 會經過用 nginx proxy server，來轉發到 Soketi WebSocket Server (走 k8s svc)，設定檔如下圖：

<br>

{{< figure src="/nginx/soketi-log-502-error/2.png" width="800" caption="入口 nginx 設定" >}}

<br>

然後會出現 `connect() failed (111: Connection refused) while connecting to upstream` 的錯誤訊息，代表我們的 Nginx 設定少了一個重要的一行設定，就是 `proxy_http_version 1.1;`，這個設定要讓 Nginx 作為 proxy 可以和 upstream 的後端服務也是用 keepalive，必須使用 http 1.1，但如果沒有設定預設是 1.0，也要記得設定 `proxy_set_header Upgrade`、`proxy_set_header Connection`。調整過後就變成：

<br>

```nginx {filename="ws.conf"}
server {
  server_name socket.XXX.com;
  listen 80 ;
  listen [::]:80 ;
  listen 443 ssl;
  listen [::]:443 ssl;
  ssl_certificate /etc/nginx/ingress.gcp.cert;
  ssl_certificate_key /etc/nginx/ingress.gcp.key;
  access_log /var/log/nginx/access.log main;
  location / {
    proxy_pass http://soketi-ws-ci:6001;
    proxy_connect_timeout 10s;
    proxy_read_timeout 1800s;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

<br>

解決完 `connect() failed (111: Connection refused)` 這個問題後，接下來就是要解決 `502 error` 這個問題，會導致 502 代表 Nginx 這個 proxy server 連不上後端的 Soketi WebSocket Server，再觀察 LOG 以及測試後發現，當 Pod 自動重啟，或是手動重啟 Deployment 的時候，就會有 502 的錯誤，代表 Nginx 在 proxy 到後面的 Soketi svc 再到 Pod 的時候，有一段時間是連不上的，所以就會出現 502 的錯誤，可以推測是流量進到正在關閉的 Pod 或是進到還沒有啟動好的 Pod 才導致的。

那我們先來看一下 Soketi WebSocket Server 的服務 yaml 檔案：

<br>

- deployment.yaml

```yaml {filename="deployment.yaml"}
    spec:
      terminationGracePeriodSeconds: 30
      securityContext: {}
      containers:
        - name: soketi
          securityContext: {}
          image: "quay.io/soketi/soketi::1.6.0-16-alpine"

          ... 省略 (可以到 github 看 code)...

          livenessProbe:
            failureThreshold: 3
            httpGet:
              httpHeaders:
                - name: X-Kube-Healthcheck
                  value: "Yes"
              path: /
              port: 6001
            initialDelaySeconds: 5
            periodSeconds: 2
            successThreshold: 1
```

<br>

可以看到原來的設定只有 `livenessProbe` 而已，因此我們為了要避免流量進到正在關閉的 Pod 或是進到還沒有啟動好的 Pod，所以我們需要加上 `readinessProbe` 以及 `preStop`，讓 Pod 確定啟動完畢，或是等待 Service 的 endpoint list 中移除 Pod，才開始接收流量，這樣就可以避免出現 502 的錯誤。

<br>

```yaml {filename="deployment.yaml"}
    spec:
      terminationGracePeriodSeconds: 30
      securityContext: {}
      containers:
        - name: soketi
          securityContext: {}
          image: "quay.io/soketi/soketi::1.6.0-16-alpine"

          ... 省略 (可以到 github 看 code)...

          livenessProbe:
            failureThreshold: 3
            httpGet:
              httpHeaders:
                - name: X-Kube-Healthcheck
                  value: "Yes"
              path: /
              port: 6001
            initialDelaySeconds: 5
            periodSeconds: 2
            successThreshold: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              httpHeaders:
                - name: X-Kube-Healthcheck
                  value: "Yes"
              path: /ready
              port: 6001
            initialDelaySeconds: 5
            periodSeconds: 2
            successThreshold: 1
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 20"]
```

<br>

{{< figure src="/nginx/soketi-log-502-error/3.png" width="800" caption="Pod 終止的過程" >}}

<br>

## 壓測

最後調整完，我們來測試看看是否在 Pod 自動重啟 or 更新 Deployment 的時候(並且有大量連線時)還會噴 502 error 或是 `connect() failed (111: Connection refused)`，我們這邊使用 k6 來做 websocket 服務的壓測，有簡單寫一個壓測程式如下：

<br>

{{< callout note >}}
k6 壓測

k6 是一個開源的壓測工具，可以用來測試 API、WebSocket、gRPC 等服務，可以到它的[官網](https://k6.io/)查看更多資訊。

MacOS 安裝方式：brew install k6
{{< /callout >}}

<br>

```js {filename="websocket.js"}
import ws from "k6/ws";
import { check } from "k6";

export const options = {
  vus: 1000,
  duration: "30s",
};

export default function () {
  const url =
    "wss://socket.XXX.com/app/hex-ws?protocol=7&client=js&version=7.4.1&flash=false";

  const res = ws.connect(url, function (socket) {
    socket.on("open", () => console.log("connected"));
    socket.on("message", (data) => console.log("Message received: ", data));
    socket.on("close", () => console.log("disconnected"));
  });

  check(res, { "status is 101": (r) => r && r.status === 101 });
}
```

<br>

簡單說明一下上面程式在寫什麼，我們在 const 設定 vus 代表有 1000 個虛擬使用者，會在 duration 30s 內完成測試，下面的 default 就是測試連線 ws 以及 message 跟 close 等動作，最後需要回傳 101 (ws 交握)

<br>

執行 `k6 run websocket.js` 後，就會開始壓測，可以看到會開始執行剛剛在上面提到 default 的動作：

<br>

{{< figure src="/nginx/soketi-log-502-error/4.png" width="800" caption="k6 壓測過程" >}}

<br>

等到跑完，就會告訴你 1000 筆裡面有多少的 http 101，這邊顯示 status is 101，就代表都是 101，代表都有連線成功，沒有出現 502 error 或是 `connect() failed (111: Connection refused)` 的錯誤，這樣就代表我們的問題解決了。

<br>

{{< figure src="/nginx/soketi-log-502-error/5.png" width="800" caption="k6 壓測結果" >}}

<br>

## 參考資料

[Nginx] 解決 connect() failed (111: Connection refused) while connecting to upstream：[https://wshs0713.github.io/posts/8c1276a7/](https://wshs0713.github.io/posts/8c1276a7/)

WebSocket proxying：[http://nginx.org/en/docs/http/websocket.html](http://nginx.org/en/docs/http/websocket.html)

day 10 Pod(3)-生命週期, 容器探測：[https://ithelp.ithome.com.tw/articles/10236314](https://ithelp.ithome.com.tw/articles/10236314)
