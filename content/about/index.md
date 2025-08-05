---
title: 關於我
toc: false
---

<img src="/images/logo.webp" width="350" />

<br>

哈囉大家好，我叫莊品毅 (CHUANG,PIN-YI)，也可以叫我 Ian，目前是一位 System Architect (SA) 工程師，負責規劃以及測試公司的系統架構，並且協助技術團隊解決系統相關的問題。
協助公司導入災難復原 (GCP > AWS)，並且負責 AWS 服務的架構設計、導入與權限管理，確保公司基礎設施具備高擴展性與靈活性。
以及協助導入 Datadog ，來整合公司內分散的監控系統。

除此之後，也熟悉使用 Google Cloud Platform (GCP) 雲端相關服務，喜歡使用 Terraform + Terragrunt 來管理雲端大量的 IaC 資源 (GCP Module：12、AWS Module：22)，當然也會使用 Grafana、Prometheus、EFK 等監控 Log 收集工具來確保服務的穩定性以及可追蹤性。並協助 RD 建立 CICD 部署流程。

在下班空閒時間，我喜歡閱讀技術相關的文件、寫部落格，也會參加一些線下技術社群的活動，曾參加過：Google Next 2025、AWS Summit 2025、DevOpsDay 2022|2023|2024、Cloud Summit 2024|2025，希望能夠透過這些活動來學習更多的知識，並且與更多的技術人員交流。

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

(2024/10/01 - 現在)

Amazon Web Services (AWS) 服務架構設計、導入與權限管理
1. 設計公司多雲架構，規劃從 GCP 災難備援至 AWS，確保基礎設施具備高擴展性與靈活性。
2. 開發 22 組 Terraform 模組，實現 IaC（基礎設施即程式碼），確保環境一致性並加速團隊上手時間與維運成本。
3. 撰寫 40+ 篇技術文件，涵蓋 AWS IaC 指南、AWS 與 GCP 服務比較、Graviton 遷移效能及優化成本分析等、各項測試報告。
4. 負責多個專案的服務建置、CICD 自動化部署，確保系統能在 AWS 穩定且高效運行。
5. 協助 RD 研究與部署 AWS macOS GitLab Runner 進行 iOS APP 打包，負責資源建立、問題排查與最佳化，確保 CI/CD 流程順暢運行。
6. 與 AWS 代理商合作，導入 AWS Organizations 及 IAM Identity Center，統一帳戶與權限管理，簡化資源治理並確保安全與合規。

<br>

雲端服務架構重構與內網識別優化
1. 重新設計整體服務架構，導入 Internal DNS，實現服務去 IP 化，提升內部 API 識別性與一致性，同時考量 GCP、AWS 跨雲通用性，並為後續導入 Service Mesh 奠定基礎。
2. 優化網路架構，將部分需經地端處理的流量調整為透過雲端服務處理，成功減少地端設備負載與專線依賴，提升系統穩定與維運效率，並大幅降低設備與頻寬相關成本。
3. 將服務動靜態分離，搭配加速 CDN，強化前端靜態資源加載效率與用戶體驗。

<br>

Datadog 監控系統導入與實作
1. 主導技術處導入 Datadog，整合原本分散於各單位的監控系統，提升監控資料集中性與可視化，查詢效率提升達 90%，大幅加速跨單位除錯與協調流程。
2. 與 Datadog 官方技術團隊深入合作，確保公司需求符合最佳實作方案。
3. 撰寫相關技術文件，協助團隊快速導入，確保監控系統長期穩定運維。

<br>

技術趨勢追蹤與問題解決
1. 主動追蹤 GCP GKE 非註冊發布通道的強制升級公告，撰寫應對方案與檢查腳本，協助團隊避免升級造成服務中斷，並防止提前切換至 Extended Channel 而產生約 $3,000 美金開支。

2. 關注 Docker Image Pull Rate Limit 政策變更，提出解決方案並驗證其可行性，確保 CI/CD 流程穩定運作不受限制影響。
3. 將最新技術公告與因應措施整理後，於公開會議中向全技術處同仁分享，有效提升團隊對技術變動的掌握與應對效率。

### 凡谷興業有限公司 - SRE 工程師

(2022/02/07 - 2024/10/01)

導入與優化 IaC 工具鏈，建構一致且自動化的基礎設施部署流程
1. 導入 Helmfile 管理 50+ 微服務的 Helm Chart，實現 Kubernetes 應用部署流程的一致性與可重複性，提升系統可維護性與部署標準化。
2. 重構 Terragrunt 架構，統一 Terraform 模組與目錄結構，強化 IaC 管理可讀性，明顯提升維運效率與團隊協作品質。
3. 開發 12 組 Terraform 模組，推動基礎設施模組化與標準化，取代 UI 操作以避免設定偏差，並導入版本控制機制，大幅降低人為錯誤與維運風險。
4. 建立涵蓋 Helmfile 與 IaC 的 CI/CD 自動化流程，加速服務部署與基礎設施環境重建，縮短交付時間。
5. 落實最小權限原則，系統性清查移除冗餘 IAM 權限，強化基礎設施安全性並符合內部合規要求。

<br>

GCP 成本優化實踐
1. 優化 Prometheus 監控架構，調整 Samples Ingested 設定，針對指標來源進行分析與過濾，在不影響監控品質的前提下，大幅降低儲存與計算資源，每日節省約 $200 美金。
2. 與 RD 協作優化 Log 結構，剔除無效與冗餘訊息，減少不必要的儲存成本，顯著降低日誌佔用空間與長期儲存費用。

<br>

熟悉 GCP 生態系統與 SRE / DevOps 實務操作
1. 熟悉 GCP 各項核心服務，包含 GKE、GCE、GCS、Cloud Load Balancing、Cloud SQL、Memorystore、VPC、IAM 等，具備全善的建置與維運經驗。
2. 熟練使用 GitLab、GitHub、Jenkins、Ansible 等工具，規劃並實作 CI/CD 自動化部署流程，提升開發與交付效率。
3. 建置完整監控與日誌系統，使用 Fluentd、Elasticsearch、Kibana 搭配 Google Managed Prometheus，實現效能監控與問題追蹤。


{{% /steps %}}

<br>

## 證照

<br>

{{< hextra/feature-grid cols=2 >}}
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
  subtitle="證照編號: 190-008-011 / 發照日期： July 5, 2019 / 到期日：July 5, 2021"
  image="/about/rhce.webp"
  tag="Red Hat" tagType="error"
>}}
{{< card
  link=""
  title="RED HAT CERTIFIED SYSTEM ADMINISTRATOR (RHCSA)"
  subtitle="證照編號: 190-008-011 / 發照日期： Jan 11, 2019 / 到期日：Jan 11, 2021"
  image="/about/rhcsa.webp"
  tag="Red Hat" tagType="error"
>}}
{{< /hextra/feature-grid >}}

<br>

## 履歷

<br>

{{< hextra/feature-grid cols=2 >}}
{{< card
  link="https://pin-yi.me/about/resume-tw.pdf"
  title="中文版"
  subtitle="最後上傳日期：2025-08-03"
  image="/about/resume-tw.webp"
  target="_blank"
  tag="New" tagType="error"
>}}
{{< card
  link="https://pin-yi.me/about/resume-en.pdf"
  title="英文版"
  subtitle="最後上傳日期：2025-08-03"
  image="/about/resume-en.webp"
  tag="New" tagType="error"
>}}
{{< /hextra/feature-grid >}}