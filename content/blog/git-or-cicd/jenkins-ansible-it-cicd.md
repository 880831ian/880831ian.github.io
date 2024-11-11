---
title: "Jenkins 及 Ansible IT 自動化 CI/CD 介紹"
type: docs
weight: 73
description: Jenkins 及 Ansible IT 自動化 CI/CD 介紹"
images:
  - git-or-cicd/jenkins-ansible-it-cicd/og.webp
date: 2022-05-16
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

在軟體開發領域中，IT 自動化 (automation) 及 持續整合 (Continuous Integration, CI) 、持續佈署 (Continuous Deployment,CD) 是 DevOps 精神中很重要的兩個部分，此文章是參考 [30 天入門 Ansible 及 Jenkins](https://ithelp.ithome.com.tw/users/20103346/ironman/1473) ，再加上自己測試後的筆記紀錄，歡迎大家可先閱讀作者原文，那我們就開始一起學習吧 👊

我會分別介紹 [Ansible](https://www.ansible.com/) 與 [Jenkins](https://www.jenkins.io/) 這兩個非常熱門的開源軟體再搭配實作來讓大家更了解他們，那在之前我們先來聊聊為什麼需要 IT 自動化以及什麼是持續整合/持續部署吧 🤠

<br>

## 為什麼需要 IT 自動化 ?

當我們在開發任何軟體產品時，除了開發本身的過程需要花相當多的心力外，在產品部署的環節也是讓大家頭痛的一個部分。其環境的搭建或是參數的設定常常會因為一些小原因導致產品無法像在開發時一樣正常運作。尤其當需要部署的主機不只一台時，重複性的工作會花費我們大量的時間。再加上還會因為伺服器提供的作業環境不同、或是其他種種限制而必須做參數上的調整等等。

這時候 ` IT 自動化` 就顯得十分重要，透過自動化，不但可以幫助開發人員有效減少部署產品所需時間外，還可以在有限度的修改下分別針對不同環境去做調整。

<br>

{{< figure src="/git-or-cicd/jenkins-ansible-it-cicd/automation.webp" width="700" caption="[Best Automation Tools for DevOps](https://www.xenonstack.com/blog/best-automation-tools-for-devops)" >}}

<br>

## 什麼是持續整合/持續部署

當環境搭建成功後，對於服務本身的維護以及監控也是開發流程中相當重要的一環。當我們從原始碼代管服務 (GitHub、GitLab)上取得原始碼後，要如何確保產品在發布前品質沒有問題，一直以來都是開發人員需要思考的一個課題。

由於現在多數開發團隊都會透過版本控制來提交並整合開發人員各自修改的程式碼，若在合併分支時沒有把合併衝突 (conflict) 處理恰當，或是合併程式碼後產生某些邏輯錯誤，往往會到產品發佈後才發現不可預期的錯誤。

所以有了持續整合/持續部署的機制下，我們可以透過高頻率的整合、測試並分析程式碼品質，在最短時間發現問題以及發生點，進而確保產品每一次的發布都是穩定且高品質的。

<br>

{{< figure src="/git-or-cicd/jenkins-ansible-it-cicd/CICD.png" width="700" caption="CI/CD 流程圖 (作者打錯是 rsync 不是 rsyne) [Day12 什麼是 CICD](https://ithelp.ithome.com.tw/articles/10219083)" >}}

<br>

上面這張圖是簡化版的 CI/CD 流程圖，當我們 Developer 將程式 Push 到原始碼代管服務 (GitHub、GitLab)上，會經過 Webhook 給 Jenkins 這種自動化可以持續整合部署到各自的伺服器上。

那 CI & CD 分別負責哪些工作呢？

### 持續整合 Continuous Integration

持續整合的英文是 Continuous Integration 我們縮寫成 CI ，後續也會使用 CI 來做說明：

- 流程：

1. 程式建置

   開發人員在每一次的 Commit & Push 後，都能夠於統一的環境自動 Build 程式，透過此步驟可以避免每個開發人員因本機的環境或是套件版本不同導致出現異常。

2. 程式測試

   當程式編譯完後，透過**單元測試**測試新寫的功能是否正確，或者確定是否會影響現有功能，透過該步驟進行測試，可以避免開發人員遺忘先在本機檢查，作為**雙重驗證**之功用。

- 目的：
  1.  降低人為疏失風險
  2.  減少人工手動的反覆動作
  3.  進行版本控制
  4.  增加系統一制性與透明化

<br>

### 持續佈署 Continuous Deployment

持續佈署的英文是 Continuous Deployment 我們縮寫成 CD ，後續也會使用 CD 來做說明：

- 流程：

1. 部署服務

   透過自動化方式，將寫好的程式碼更新到機器上並公開對外服務，另外需要確保套件版本＆資料庫資料的完整性，也會透過監控系統進行服務存活檢查，若服務異常會即時發送通知告知開發人員。

- 目的：
  1.  保持每次更新程式都可以順暢完成
  2.  確保服務存活

<br>

我們了解了自動化與 CI/CD 的重要性與功用，那我們要怎麼去實現這些呢！我們先看下面這張圖：

<br>

{{< figure src="/git-or-cicd/jenkins-ansible-it-cicd/devops.webp" width="800" caption="[Top 5 DevOps Automation Tools in 2020](https://plumlogix.com/top-5-devops-automation-tools-in-2020/)" >}}

這邊整理 2020 年適合用於 DevOps Automation 的工具，那我們本次教學會介紹最多人使用的 Jenkins 以及 Ansible 兩種：

<br>

## Jenkins

Jenkins 是使用 Java 編成語言編寫最受歡迎的開源自動化服務器。它促進了軟體開發過程中的持續整合、持續部署的自動化過程。

Jenkins 支持 1800 多個其他軟體套件，Jenkins 易於安裝和使用，它還提供方便瀏覽的項目管理儀表板，它的優點還有：

- 免費開源
- 充滿活力的用戶社群
- 多種工具和技術集成
- 插件支持
- 易於安裝、配置和升級
- 監控外部工作
- 支持各種身份驗證方法、通知、版本控制等等

<br>

## Ansible

Ansible 是一種 IT 自動化工具。它可以部署軟體、配置系統，並編排更高級的自動化任務，例如 CD (持續部署) 或 RollingUpdate 零停機的滾動更新。

自動化簡化了複雜的任務，不僅使開發人員的工作更容易管理，也讓他們的注意力可以放在對團體更有價值的其他任務上。換句話說，它可以節省時間並提高效率。Ansible 使用簡單的 YAML 語法，且 Ansible 是一種輕量級且安全的解決方案，它的優點還有：

- 使用 Ansible 不需要任何特殊的編程技能，因為使用的是 YAML 語法
- Ansible 允許建立高複雜性的 IT 自動化。
- 因為不需要安裝其他套件，所以伺服器上有更多空間來容納應用服務的支援
- Ansible 在設計上非常簡單且可靠一制性。

<br>

## Jenkins 跟 Ansible 比較

| 名稱 |                 Jenkins                  |                 Ansible                 |
| :--: | :--------------------------------------: | :-------------------------------------: |
| 套件 |        Jenkins 支持 1800 多種套件        |              支持較少套件               |
| 語言 | 支持 C、C++、Java、Perl、Python、Ruby 等 |   支持 C、Python、JavaScript、Ruby 等   |
| 費用 |             Jenkins 是免費的             | Ansible 不是免費的，但有試用版(Red hat) |
| 大小 |                  重量級                  |                 輕量級                  |
| 服務 |             基於伺服器的工具             |             基於雲上的工具              |

<br>

## Jenkins 與 Ansible 介紹實作連結

由於 Jenkins 與 Ansible 介紹與實作教學文章較長，故個別分開一篇文章來做說明，大家可以去看自己有興趣的文章歐 😎

- Jenkins：[使用 Jenkins 設定 GitHub 觸發程序並通知 Telegram Bot](../jenkins-github-tg-bot/)
- Ansible：[Ansible 介紹與實作 (Inventory、Playbooks、Module、Template、Handlers)](../ansible)

<br>

## 參考資料

[30 天入門 Ansible 及 Jenkins](https://ithelp.ithome.com.tw/users/20103346/ironman/1473)

[Day12 什麼是 CICD](https://ithelp.ithome.com.tw/articles/10219083)

[Jenkins 和 Ansible 的對比和區別](http://www.srcmini.com/21799.html)
