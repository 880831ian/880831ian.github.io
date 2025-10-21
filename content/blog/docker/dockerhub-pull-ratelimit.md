---
title: "Docker Image Pull Rate Limit 說明 + 掃描腳本"
type: docs
weight: 9997
description: Docker Image Pull Rate Limit 說明 + 掃描腳本
images:
  - docker/dockerhub-pull-ratelimit/og.webp
date: 2025-02-24
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Docker
  - Docker Hub
  - Pull
  - Rate Limit
---

### 背景說明

因 Docker 官方最近宣布從 2025 年 04 月 01 日開始，未經過驗證的帳戶 (一般來說，GKE 拉 Docker Hub 的 Image 都屬於這類)，會有對應的 Rate Limit 限制，需要盤點及建議不要直接使用 Docker Hub 的 Image，避免 Node 擴縮 (因為沒有 Image 暫存)、或是 ImagePullPolicy 是 IfNotPresent 情況，讓 Container 需要重新 Pull 大量的 Image 出現 429 錯誤，導致 Pod 啟動不了，造成線上問題。

另外 GKE Node 有設定 Mirror Registry，可以減少直接從 Docker Hub 拉取 Image 的次數，但還是建議大家把 Image 搬到 GAR (Artifact Registry) 或是其他的 Registry 上。

<br>

{{< figure src="/docker/dockerhub-pull-ratelimit/5.webp" width="700" caption="GKE Node Mirror Registry" >}}

<br>

> [!NOTE] 提醒
> 因近期官方有再次調整時間，原本是公告 03/01，有往後延長，如後續又有延後，會在同步更新在此文件中。<br>
03/01 延長到 04/01 可以參考：[https://github.com/docker/docs/commit/d9eb2332f9581b0b7eac1c90606528707380bf85#diff-7c1db5bfcc9a88aeecd94d07fa2138f5709a9ba977b795f67516106ce3a977ba](https://github.com/docker/docs/commit/d9eb2332f9581b0b7eac1c90606528707380bf85#diff-7c1db5bfcc9a88aeecd94d07fa2138f5709a9ba977b795f67516106ce3a977ba)

<br>

{{< figure src="/docker/dockerhub-pull-ratelimit/1.webp" width="900" caption="官方限制" >}}

<br>

目前限制是 <b>Unauthenticated users  未經驗證的用戶，每個 IPv4 or IPv6 Subnet 一小時只能 Pull 10 次</b>，如果是免費帳號(需要登入)，則是一小時 100 次。

<br>

這些請求包含：網頁、API、Image Pull 等，如果超出請求限制，會出現 429 Too Many Requests 的錯誤訊息。

<br>

### 建議處理及掃描腳本

為了避免因為 Pull 頂到限制，進而造成線上問題，需要確認目前線上使用的 Image，是否還有直接 Pull 使用 Docker Hub 上面的 Image，請大家將 Image 搬到 GAR  (Artifact Registry) 或是其他的 Registry 上。

有簡單寫一個掃描腳本，可以列出每個 Deployment、Statefulset、Damonset、CronJob 使用的 image，有初步排除 asia-docker.pkg.dev、asia-east1-docker.pkg.dev、gcr.io、quay.io、registry.k8s.io、ghcr.io (可再自行新增)

<br>

```bash
#!/bin/bash
# 列出 deployments 的 images
kubectl get deployments -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,IMAGE:.spec.template.spec.containers[*].image" | awk '
BEGIN {
    format="| %-18s | %-40s | %-70s |\n"
    line="+--------------------+------------------------------------------+------------------------------------------------------------------------+"
    print line
    printf format, "Namespace", "Deployment", "Image Name"
    print line
}
NR>1 {
    split($3, images, ",")
    for (i in images) {
        if (images[i] !~ /asia-docker.pkg.dev|asia-east1-docker.pkg.dev|gcr.io|quay.io|registry.k8s.io|ghcr.io/) {
            printf format, $1, $2, images[i]
        }
    }
}
END {
    print line
}'

echo "\n"

# 列出 statefulsets 的 images
kubectl get statefulsets -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,IMAGE:.spec.template.spec.containers[*].image" | awk '
BEGIN {
    format="| %-18s | %-40s | %-70s |\n"
    line="+--------------------+------------------------------------------+------------------------------------------------------------------------+"
    print line
    printf format, "Namespace", "Statefulset", "Image Name"
    print line
}
NR>1 {
    split($3, images, ",")
    for (i in images) {
        if (images[i] !~ /asia-docker.pkg.dev|asia-east1-docker.pkg.dev|gcr.io|quay.io|registry.k8s.io|ghcr.io/) {
            printf format, $1, $2, images[i]
        }
    }
}
END {
    print line
}'

echo "\n"

# 列出 daemonsets 的 images
kubectl get daemonsets -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,IMAGE:.spec.template.spec.containers[*].image" | awk '
BEGIN {
    format="| %-18s | %-40s | %-70s |\n"
    line="+--------------------+------------------------------------------+------------------------------------------------------------------------+"
    print line
    printf format, "Namespace", "Damonset", "Image Name"
    print line
}
NR>1 {
    split($3, images, ",")
    for (i in images) {
        if (images[i] !~ /asia-docker.pkg.dev|asia-east1-docker.pkg.dev|gcr.io|quay.io|registry.k8s.io|ghcr.io/) {
            printf format, $1, $2, images[i]
        }
    }
}
END {
    print line
}'

echo "\n"

# 列出 cronjobs 的 images
kubectl get cronjobs -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,IMAGE:.spec.jobTemplate.spec.template.spec.containers[*].image" | awk '
BEGIN {
    format="| %-18s | %-40s | %-70s |\n"
    line="+--------------------+------------------------------------------+------------------------------------------------------------------------+"
    print line
    printf format, "Namespace", "CronJob", "Image Name"
    print line
}
NR>1 {
    split($3, images, ",")
    for (i in images) {
        if (images[i] !~ /asia-docker.pkg.dev|asia-east1-docker.pkg.dev|gcr.io|quay.io|registry.k8s.io|ghcr.io/) {
            printf format, $1, $2, images[i]
        }
    }
}
END {
    print line
}'
```

<br>

### 成果

{{< figure src="/docker/dockerhub-pull-ratelimit/2.webp" width="900" caption="掃描結果範例" >}}

<br>

{{< figure src="/docker/dockerhub-pull-ratelimit/3.webp" width="900" caption="掃描結果範例" >}}

<br>

{{< figure src="/docker/dockerhub-pull-ratelimit/4.webp" width="900" caption="掃描結果範例" >}}

<br>

### 參考文件

[Docker Hub usage and limits](https://docs.docker.com/docker-hub/usage/)