---
title: 關於我
toc: false
layout: wide
---

<img src="/images/logo.webp" width="350" />

<br>

哈囉大家好，我叫莊品毅 (CHUANG,PIN-YI)，也可以叫我 Ian，目前是一位 System Architect (SA) 工程師，負責規劃以及測試公司的系統架構，並且協助技術團隊解決系統相關的問題。
協助公司導入災難復原 (GCP > AWS)，並且負責 AWS 服務的架構設計、導入與權限管理，確保公司基礎設施具備高擴展性與靈活性。
以及協助導入 Datadog ，來整合公司內分散的監控系統。

除此之後，也熟悉使用 Google Cloud Platform (GCP) 雲端相關服務，喜歡使用 Terraform + Terragrunt 來管理雲端大量的 IaC 資源 (GCP Module：12、AWS Module：22)，當然也會使用 Grafana、Prometheus、EFK 等監控 Log 收集工具來確保服務的穩定性以及可追蹤性。並協助 RD 建立 CICD 部署流程。

在下班空閒時間，我喜歡閱讀技術相關的文件、寫部落格，也會參加一些線下技術社群的活動，曾參加過：Google Next 2025、AWS Summit 2025、DevOpsDay 2022|2023|2024、Cloud Summit 2024|2025、Kube Summit 2025、AWS Community Day 2025，希望能夠透過這些活動來學習更多的知識，並且與更多的技術人員交流。

歡迎大家使用下方 giscus 留言系統留言交流 d(`･∀･)b。

<br>

## 聯絡方式

- Email：[880831ian@gmail.com](880831ian@gmail.com)
- LinkedIn：[https://www.linkedin.com/in/pinyi/](https://www.linkedin.com/in/pinyi/)
- GitHub：[https://github.com/880831ian](https://github.com/880831ian)
- Telegram：[https://t.me/pinyichuchu](https://t.me/pinyichuchu)

<br>

## 工作經驗

{{% steps %}}

### 凡谷興業有限公司 - SA 架構師

(2024/10/01 - 2025/10/21)

Amazon Web Services (AWS) 服務架構設計、導入與權限管理
1. 設計並導入支援 **500 人**的 AWS Organizations 與 IAM Identity Center (SSO) 架構，建立集中化帳號與權限治理機制。規劃 **AWS 端的跨雲災難備援 (DR) 架構**，確保系統具高可用與彈性擴展能力，並具備 GCP 與 AWS 間資料備援機制的基本理解。
2. 開發 **22 組** Terraform 模組，導入基礎設施即程式碼（IaC）標準化，顯著縮短環境建置時程並降低維運成本。
3. 撰寫 **45+ 篇** 技術文件，涵蓋 AWS IaC 指南、AWS 與 GCP 服務比較、Graviton 遷移效能及優化成本分析等、各項測試報告。
4. 負責初期 AWS 專案的 CI/CD 自動化部署設計，**協助系統從 GCP 平穩遷移至 AWS**，確保運作穩定與效能最佳化。
5. 導入 AWS macOS GitLab Runner，建立 iOS App 自動化打包流程，**解決環境相容與效能問題**，確保 CI/CD 流程穩定運行。

<br>

雲端服務網路重構與 Internal DNS 導入
1. 主導跨雲 GCP 與 AWS 網路架構重構，**導入 Internal DNS 實現服務去 IP 化**，提升內部 API 識別一致性，並為後續導入 Service Mesh 奠定基礎。
2. 優化網路架構，將部分需經地端處理的流量轉由雲端服務承接，減少地端設備負載與專線依賴，**提升系統穩定性與維運效率，同時降低頻寬與設備成本**。
3. 將服務動靜態分離並導入 CDN 加速機制，**強化前端靜態資源加載效率與整體用戶體驗**。

<br>

Datadog 監控系統導入與實作
1. 主導全公司 Datadog 監控平台導入與整合，統一各單位分散的監控系統，取代 EFK 架構以解決 Elasticsearch 腦裂與高維運問題，**提升監控集中性**、穩定性與可視化。
2. 建立跨單位監控與警報標準化流程，**查詢與問題定位效率提升達 90%**，有效縮短跨部門問題排查時間。

<br>

技術趨勢追蹤避免 RCA 與問題解決
1. 主動監控雲平台版本更新與服務公告，定期評估對系統的影響，撰寫應對方案與自動化檢查 Shell Script 腳本，**曾有效避免升級導致服務中斷**，並**節省約 USD 3,000 的額外支出**。
2. 關注 Docker Image Pull Rate Limit 政策變更，提出並驗證可行解法，確保 CI/CD 流程穩定運作、**不讓正式環境受到影響**。
3. 將最新技術公告與風險因應措施文件化，並於公開會議中向全技術處分享，**提升團隊技術變動掌握與問題預防能力**。

<br>

### 凡谷興業有限公司 - SRE 工程師

(2022/02/07 - 2024/10/01)

導入與優化 IaC 工具鏈，建構一致且自動化的基礎設施部署流程
1. 主導基礎設施自動化與部署流程重構，**導入並優化 IaC 工具鏈**（Terraform、Terragrunt、Helmfile），將原本以 YAML 手動維護的部署模式全面轉換為標準化管理。
2. 透過 Helmfile **管理 60+ 微服務** 並建立涵蓋應用與基礎設施的 CI/CD 流程，實現部署一致性與全自動化交付，**提升約 70% 維運效率** 並降低人為操作風險。
3. 重構 Terragrunt 架構並**開發 12 組 Terraform 模組**，推動基礎設施模組化與標準化，同時落實最小權限原則，系統性清查並移除冗餘 IAM 權限，強化整體安全與合規性。

<br>

GCP 成本優化實踐
1. 優化 Prometheus 監控架構，調整 Samples Ingested 設定，針對指標來源進行分析與過濾，在不影響監控品質的前提下，大幅降低儲存與計算資源，**每日節省約 USD 200**。
2. 與 RD 協作優化 Log 結構，剔除無效與冗餘訊息，減少不必要的儲存成本，**顯著降低日誌佔用空間與長期儲存費用**。

<br>

熟悉 GCP 生態系統與 SRE / DevOps 實務操作
1. 熟悉 GCP 核心服務（GKE、GCE、GCS、Cloud Load Balancing、Cloud SQL、Memorystore、VPC、IAM 等），具備完整建置與維運經驗。曾**管理 30 座以上 GKE 叢集、約 500 個節點**，單叢集平均 **每分鐘 100,000 次請求流量**，具備大規模雲端環境設計與調優能力。
2. 熟練使用 GitLab、GitHub、Jenkins、Ansible 等工具，規劃並實作 CI/CD 自動化部署流程，**提升開發與交付效率**。
3. 建置完整監控與日誌系統，使用 Fluentd、Elasticsearch、Kibana 搭配 Google Managed Prometheus，實現效能監控與問題追蹤。

{{% /steps %}}

<br>

## 證照

<br>

{{< hextra/feature-grid cols=3 >}}
{{< card
  link="https://www.credly.com/badges/b277830e-b1ea-4eeb-a7f2-7fbb71ed2b8d/public_url"
  title="Google Cloud Professional Cloud Architect"
  subtitle="證照編號: ed519d0fcb984d428460841eb83419c8 <br>發照日期： Feb 21, 2025 / 到期日：Feb 21, 2027"
  image="/about/pca.webp"
  tag="GCP" tagType="info"
>}}
{{< card
  link=""
  title="RED HAT CERTIFIED ENGINEER (RHCE)"
  subtitle="證照編號: 190-008-011 <br>發照日期： July 5, 2019 / 到期日：July 5, 2021"
  image="/about/rhce.webp"
  tag="Red Hat" tagType="error"
>}}
{{< card
  link=""
  title="RED HAT CERTIFIED SYSTEM ADMINISTRATOR (RHCSA)"
  subtitle="證照編號: 190-008-011 <br>發照日期： Jan 11, 2019 / 到期日：Jan 11, 2021"
  image="/about/rhcsa.webp"
  tag="Red Hat" tagType="error"
>}}
{{< /hextra/feature-grid >}}

<br>

## 履歷 Resume

<br>

{{< hextra/feature-grid cols=2 >}}
{{< card
  link="/about/resume-tw.pdf"
  title="中文版"
  subtitle="最後上傳日期：2025-11-06"
  image="/about/resume-tw.webp"
  target="_blank"
  tag="New" tagType="error"
>}}
{{< card
  link="/about/resume-en.pdf"
  title="英文版"
  subtitle="最後上傳日期：2025-11-06"
  image="/about/resume-en.webp"
  tag="New" tagType="error"
>}}
{{< /hextra/feature-grid >}}