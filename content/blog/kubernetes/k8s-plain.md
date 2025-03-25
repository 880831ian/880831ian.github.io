---
title: "用大型社區來介紹 Kubernetes 元件"
type: docs
weight: 9997
date: 2022-06-02
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

前陣子對於 Kubernetes 部分內容還不是很清楚，在網路上閒逛的時候發現一篇很有趣的文章，標題是 『 [給行銷跟業務的 Kubernetes 101 中翻中介紹](https://blog.pichuang.com.tw/20210111-Kubernetes-for-sales-and-marketing/#%E4%B8%AD%E7%BF%BB%E4%B8%AD%E7%9A%84-Kubernetes-%E7%B5%84%E4%BB%B6%E4%BB%8B%E7%B4%B9) 』，點進去後才發現，作者 [Phil Huang](https://blog.pichuang.com.tw/about/) 將 Kubernetes 元件的內容用大型住戶社區來介紹，讓我更清楚每一個元件的意思，以下我會用我理解的以及作者的思考邏輯來去寫這篇筆記，再次感謝作者文章 😍

<br>

## Kubernetes 元件白話文

<br>

{{< figure src="/kubernetes/k8s-plain/1.jpg" width="550" caption="大型社區示意圖  (圖片來源：[蘋果地產](https://tw.feature.appledaily.com/house/news/article/205878))" >}}

可以看到上面這張圖片，他是一個很典型的大型社區，我們這次的 Kubernetes 元件白話文，會以大型社區來當作現實中的元件，以社區的例子來說明 Kubernetes 。

<br>

當然，我們之前的文章也有介紹過 Kubernetes，有興趣可以先飛回去看看！

[Kubernetes (K8s) 介紹 - 基本](/blog/kubernetes/k8s/)

[Kubernetes (K8s) 介紹 - 進階 (Service、Ingress、StatefulSet、Deployment、ReplicaSet、ConfigMap)](/blog/kubernetes/k8s-advanced/)

[Kubernetes (K8s) 搭配 EFK 實作 (Deployment、StatefulSet、DaemonSet)](/blog/kubernetes/k8s-efk/)

我們先想像一下我們建立 Kubernetes 完整的叢集服務，就好比是建立一個大型的社區，所以以下會將名詞與社區來做連結，那．．．開始囉！

<br>

### Kubernetes 元件

- Kubernetes：建立這個大型社區的藍圖，規劃的社區內的大大小小的設計。
- 虛擬化平台、公有雲平台、實體機器：就是我們要蓋社區的地皮。(虛擬化平台：vSphere/Proxmox/VMware、公有雲平台：AWS/GCP/Azuer、實體機器：就是實體機器 😂)
- OS 作業系統：每棟大樓的骨架和地基。
- Master Node：住戶管委會所居住大樓 (真好自己有所屬的大樓ＸＤ)，為了保證他們不會吵架，所以建議最少需要三棟。
- Etcd Cluster：管委會的人，一樣為了怕一黨獨大(?，所以建議最少需要有三位管委，且互相投票選出一個頭頭。
- Worker Node：就是偶們住戶所居住的大樓。
- Pod：住戶，所以我們一棟 Worker Node 大樓，可以有很多 Pod 住戶。
- Pod IP：每個住戶的門牌，既然是門牌，代表他也不會重複。
- Container Registry：包裹集中的存放中心。
- Container Images：還沒有被打開的包裹。
- Containers：已經被打開且正在使用的包裹，那每個 Pod 住戶裡面，可以有一個或很多個以上的 Containers 包裹。
- Service：社區裡面的社團，例如：羽球社、麻將社等等，可以集合大家的地方。
- Service Mesh：社團的聯絡名冊，會記住誰是哪一個社團的成員。
- Ingress Controller：社區的大門，可以指定讓社區成員都固定走同一個或是多個門的入口管控。
- Egress Controller：社區的後門，可以指定讓社區成員都固定走同一個或是多個門的出口管控。
- Internal DNS / Service Discovery：大樓住戶的電話簿。
- External DNS：指向各大樓的路標。
- OCI (Open Container Initiative)：制定大樓鋼筋水泥或是行李箱標準的組織。
- CRI (Container Runtime Interface)：大樓鋼筋水泥的廠商。
- CNI (Container Network Interface)：大樓水電系統的廠商。
- CSI (Container Storage Interface)：大樓空間規劃的廠商。
- Bastion：專門維護社區的工程車。

<br>

### 常用套件

- Promethus：社區整體的監控中心 (Meterics)，一堆攝影機的管理室 XD
- Grafana：監控中心裡面大型的 LED 儀表板。
- Elaticsearch：社區整體的情資中心 (Logging)，這應該是一群愛八卦的大媽吧 😏
- Kibana：情資中心裡面大型的 LED 訊息版。

<br>

## 常見問題

作者也有整理了一些常見的問題，我把它整理一下挑選出幾個我自己一開始也會有疑問的問題，我們一起來看看吧！一樣我會用我所理解的意思來介紹 ~

### Q1：為什麼 Kubernetes 的最小單位是 Pod 這個要怎麼理解？

Ans ： 試想一下，難道管委會會管你的包裹裡面內容物放什麼嗎 XD

<br>

### Q2：Docker 在 Kubernetes 的角色是什麼？

Ans ： 是眾多的 CRI (Container Runtime Interface) 選擇之一，也就是大樓的鋼筋水泥廠商有很多間，有一間叫做 Docker 的廠商特別有名。

<br>

### Q3：呈上，那 CRI/CNI/CSI 是不是也可以替換成其他的廠商？

Ans ： 當然可以，現在可以看到越來越多廠商都開始支援 Kubernetes 就是因為這個原因，因為一般情況下，Kubernetes 並不會特別限定 `大樓鋼筋水泥廠商`、`大樓水電系統廠商`、`大樓空間規劃廠商`，只要有符合特定的標準即可，但要留意 Kubernetes 的版號也會受到這三個介面支援發行的版號所影響，要留意相容性的問題！

<br>

### Q4：整個住戶社區最重要的角色是什麼？

Ans ： 那三棟管委會大樓，和裡面的的三位委員，三位掛掉一位還可以維持正常運作，掛掉兩位會維持唯獨運作。

<br>

### Q5：Kubernetes、VM、Container 的差異性

Ans ： 我們可以建立好一個大樓(VM)，隨意放置一個或是多個包裹(Containers)，必須手動管理這些包裹，如果資源不足或是這棟大樓倒了，就沒有辦法自動轉移這些包裹的內容了。

但如果是 Kubernetes，我們就可以建立多個大樓(VM)，透過 Kubernetes 所規定的放置計畫 (例如：Deployment、DaemonSet)，我們將可以統一調度這些 Pod，當某棟大樓資源不夠或是這棟大樓倒時，可以根據 Kubernetes 規則來進行搬遷或是擴充大樓。

<br>

### Q6：為什麼有人會把 Container 跟 Pod 混在一起講

Ans ： 因為大部分的情況下，都是一個住戶(Pod)放一個包裹(Container)，，才會導致這樣的誤會。但實際上，一個住戶(Pod)是可以放一個或多個包裹在裡面的！

<br>

### Q7：什麼是 Node Scaling ?

Ans ： 可以把它理解成，當住戶太多時，Kubernetes 會自動或手動興建大樓，來把多出來的用戶給塞進去。

<br>

### Q8：在 Q5 有提到放置計畫是什麼意思？

Ans ： 只要你有需要將包裹放在社區內，都必須提出部署計畫(Deployment) 給管委會審核，只要通過審核，他們依照你事先聲明的計畫，盡最大可能性來放置包裹。

所以當我們有任何變更計畫時，都需要重新提交一份新的部署計畫給管委會重新審核和接受。

<br>

## 參考資料

[給行銷跟業務的 Kubernetes 101 中翻中介紹](https://blog.pichuang.com.tw/20210111-Kubernetes-for-sales-and-marketing/#%E4%B8%AD%E7%BF%BB%E4%B8%AD%E7%9A%84-Kubernetes-%E7%B5%84%E4%BB%B6%E4%BB%8B%E7%B4%B9)
