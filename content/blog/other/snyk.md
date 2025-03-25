---
title: "找出程式碼、開源套件、容器的安全漏洞工具 - Snyk"
type: docs
weight: 9999
date: 2022-09-20
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

前幾天有機會可以去參加 DevOpsDays Taipei 2022 活動，聽到了一門有關於如何將 Security 加入 DevOps - GitLab CI/CD with Snyk 的講座，覺得蠻有趣的，想說順便紀錄一下這幾天去聽的內容，文章底下有附上本次大會的共筆以及該講者於自己部落格所發的文章，歡迎大家先行查看歐 😆

<br>

{{< figure src="/other/snyk/devops.jpg" width="500" caption="DevOps 圖片 [(醫院 DevOps 如何落地(1) ─ DevOps 與醫院](https://www.cio.com.tw/how-hospital-devops-landed-1-devops-and-hospitals/))" >}}

<br>

我們平常熟知的 DevOps 文化應該是像上面這張圖一樣，可以透過自動化「軟體交付」和「架構變更」的流程，來使得構建、測試、發布軟體能夠更加地快捷、頻繁和可靠。

<br>

通常在 Development 跟 Operations 之外還有一個專門負責檢查程式有無漏洞或是安全性問題的單位，那我們試想一下，如果在 Dev 或是 Ops 中間先加上了檢查安全的步驟，是不是可以讓整體流程效率更加提升呢！？ 可以免去程式寫完才發現有安全性問題的漏洞，需要重新修改，而是在部署前，或是程式開發中就先被檢查出來。

<br>

對於資訊安全方面，開發人員進行開發時，必須關注目前的寫法是不是安全的，除了開發人員的程式碼外，還要注意使用的開源程式或是套件是不是已經有被掃出漏洞，加上現在越來越多人使用容器化，對於 Container Images 裡面所包含的套件安全也是一大問題。

<br>

所以單純的自動化整合與部署已經不能滿足需求，就像下圖一樣，開始有了 DevSecOps 這個詞產生，在 DevOps 的 CI/CD 流程中加入資安解決方案，藉由資安解決方案來減少開發人員去檢查程式漏洞的時間，也讓 DevOps 快速迭代更加的完整。

<br>

{{< figure src="/other/snyk/devsecops.png" width="450" caption="DevSecOps 圖片 ([Apps Built Better: Why DevSecOps is Your Security Team’s Silver Bullet](https://threatpost.com/apps-built-better-devsecops-security-silver-bullet/167793/))" >}}

<br>

## Snyk

那我們這次要使用的工具是 - Snyk，我先簡單介紹一下這個公司以及工具，它是位於波士頓的網絡安全公司，專門從事雲計算，提供增輕鬆集成，可以將安全掃描結合到 IDE、Repository、CI/CD 等，連續掃描以及一鍵修復等功能。他本身是一個 SaaS 的服務，所以需要到官網註冊一個帳號才能使用，也支援不同的 SSO 登入，詳細可以參考[註冊畫面](https://app.snyk.io/login?cta=sign-up&loc=nav&page=homepage)。

<br>

這個工具主要提供了 4 種掃描的功能， 分別是：

- Code
- Open Source
- Container
- IaC

<br>

那我們會依照這４種功能依序介紹，詳細的可以參考 [Snyk User Documentation # Explore Snyk products](https://docs.snyk.io/#explore-snyk-products)。

<br>

### Code

能夠直接掃描目前開發 Project 的程式碼有沒有安全性的問題，可以大幅減少開發人員去檢查程式漏洞的時間，目前支援了以下三種掃描方式：

- IDE Plugins
- Web
- CLI

<br>

#### IDE Plugins

IDE 我們這邊以 VS Code 為例來說明，如果有在用 VS Code 的大家應該知道 VS Code 有一個 Marketplace，可以安裝很多不同的套件 Plugins，之後有時間也會寫一篇好用的 Plugins 文章，大家可以在持續關注我歐 😍

1. 首先先開啟左側的 Marketplace，搜尋 `Snyk`，~~選擇下載量最高的準沒錯~~，選擇 Snyk Security - Code and Open Source Dependencies。

<br>

{{< figure src="/other/snyk/ide.png" width="1000" caption="VS Code Marketplace 安裝 Snyk" >}}

<br>

2. 安裝完後，在左側欄位應該就會看到 Snyk 的狗狗 Logo，如果沒有登入過，會先要求你登入 Snyk 並授權給 Plugins 使用。最後當你開啟專案時，會開始掃描程式碼，並且會依照開源套件安全性、程式碼安全性來分類，會顯示套件遇到什麼漏洞需要升級以及程式碼哪一段會有安全性的問題，能夠讓開發人員在第一時間就修復安全問題。

<br>

{{< figure src="/other/snyk/ide2.png" width="1000" caption="IDE Plugins 掃描" >}}

<br>

#### Web

登入 Snyk 網站，直接加入對應的 Project，會直接於網頁上掃描漏洞，並提示錯誤內容。

<br>

{{< figure src="/other/snyk/web.png" width="500" caption="選擇加入 project" >}}

{{< figure src="/other/snyk/web1.png" width="800" caption="掃描漏洞並顯示錯誤內容" >}}

<br>

#### CLI

要使用 CLI 需要先安裝 Snyk CLI，我們一樣使用 Homebrew 來安裝：

```shell
brew tap snyk/tap
brew install snyk
```

當然除了 Homebrew 以外還有其他的安裝方式，例如：curl 下載執行檔、npm、yarn、docker 執行等等，詳細可以參考以下[安裝連結](https://docs.snyk.io/snyk-cli/install-the-snyk-cli)。

<br>

可以直接使用以下指令才進行測試 `sync test`，還有其他 CLI 指令，詳細可以參考 [Snyk CLI](https://docs.snyk.io/snyk-cli)：

<br>

{{< figure src="/other/snyk/cli.png" width="800" caption="CLI 掃描 ([官網](https://docs.snyk.io/snyk-cli))" >}}

<br>

### Open Source

能夠掃描目前開發的 Project 中有沒有使用到有漏洞的 library，目前支援以下圖片語言，詳細可以參考 [Open Source - Supported languages and package managers](https://docs.snyk.io/products/snyk-open-source/language-and-package-manager-support)：

<br>

{{< figure src="/other/snyk/open.png" width="600" caption="Open Source library 掃描 ([官網](https://docs.snyk.io/products/snyk-open-source/language-and-package-manager-support))" >}}

<br>

### Container

在 Container image 中會存放著不同的 base image ，在 base image 中又會使用不同的 library。所以這些 container 的安全性也是需要被關注的。

可以直接使用 web 來選擇相對的 Container 倉庫去做掃描，如果有安裝上面說的 Snyk CLI，也可以使用該指令執行 `snyk container test <container image>`

<br>

{{< figure src="/other/snyk/container.png" width="800" caption="Container 掃描" >}}

<br>

### Iac

Snyk 的基礎設施即代碼 (IaC) 可幫助開發人員編寫安全的基礎設施配置，預防錯誤的設定產生，並且支援多種格式：

- K8s YAML
- HashiCorp Terraform
- AWS CloudFormation
- Azure Resource Manager (ARM)

<br>

## 參考資料

DevOpsDays Taipei 2022 共同筆記 - 在你的 DevOps 中加入一點 Security — GitLab CI/CD with Snyk - 鄭荃樺 (Barry.Cheng)
：[https://hackmd.io/@DevOpsDay/2022/%2F%40DevOpsDay%2FHkm1iY6xi](https://hackmd.io/@DevOpsDay/2022/%2F%40DevOpsDay%2FHkm1iY6xi)

身為 DevOps 工程師，使用 Snyk 掃描漏洞也是很正常的：[https://barry-cheng.medium.com/%E8%BA%AB%E7%82%BAdevops%E5%B7%A5%E7%A8%8B%E5%B8%AB-%E4%BD%BF%E7%94%A8snyk%E6%8E%83%E6%8F%8F%E6%BC%8F%E6%B4%9E%E4%B9%9F%E6%98%AF%E5%BE%88%E6%AD%A3%E5%B8%B8%E7%9A%84-d7d8f2ad2304](https://barry-cheng.medium.com/%E8%BA%AB%E7%82%BAdevops%E5%B7%A5%E7%A8%8B%E5%B8%AB-%E4%BD%BF%E7%94%A8snyk%E6%8E%83%E6%8F%8F%E6%BC%8F%E6%B4%9E%E4%B9%9F%E6%98%AF%E5%BE%88%E6%AD%A3%E5%B8%B8%E7%9A%84-d7d8f2ad2304)

在你的 DevOps 中加入一點 Security — GitLab CI/CD with Snyk (講師簡報)：[https://s.itho.me/ccms_slides/2022/9/26/3a2ac1ef-4143-4328-9fef-066782ac7dc6.pdf](https://s.itho.me/ccms_slides/2022/9/26/3a2ac1ef-4143-4328-9fef-066782ac7dc6.pdf)
