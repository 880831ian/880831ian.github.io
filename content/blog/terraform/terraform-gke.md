---
title: "使用 Terraform 建立 Google Kubernetes Engine"
type: docs
weight: 3
---

我們接續昨天的建立 Google Kubernetes Engine 文章，今天要來介紹的是如何用 Terraform 建立 Google Kubernetes Engine，由於使用 terraform 去建立、修改、刪除的指令大家應該都清楚了，那我今天的文章就不在多說，直接來介紹一下要怎麼撰寫 Google Kubernetes Engine tf 檔案 😏

<br>

## 撰寫 Google Kubernetes Engine tf 檔案

程式碼會同步到 Github ，需要的也可以去 Clone 來使用歐！ [Github 程式碼連結](https://github.com/880831ian/terraform-gke)，小提醒：由於程式碼較長，我將他拆開來說明 💖

由於等等程式碼較長，所以我在前面這邊先做說明，GKE 的結構是 叢集(cluster) > 節點池(node_pool) > 節點(node)，本次的介紹範例，會有一個叢集裡面有一個節點池，節點池裡面有 6 個節點數量，範例裡面會加上我比較常用到的一些設定，以及一些文件裡面的用法，大家可以依照自己的需求來使用參數：

<br>

### 限制使用的版本

在上一篇 [使用 Terraform 建立 Google Compute Engine](https://pin-yi.me/terraform-gce/)，我們知道 Terraform 其實就是對應的提供商，提供對應的 api 來讓我們可以用 terraform 去建置很多 IaC，但供應商提供的 api 會隨著版本而有所更動，可能換了一個版本，原本可以使用的 resource 參數就會有所不同，所以我們可以在一開始，先設定好這隻 tf 要使用的供應商及對應的版本，可以參考以下程式碼：

```tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "~> 4.38.0"
    }
  }
}
```

可以看到我們把 google 這個供應商裡面設定好他的 source 以及 version，這樣就算之後 goolge 有更新 terraform 的 api，我們也不需要去更換參數就可以使用了～

<br>

### 選擇供應者以及對應的專案

```tf
provider "google" {
  project = "project"
}
```

除了可以使用專案 ID 以外，當然也可以使用專案的名稱拉 🥳

<br>

### resource 設定

#### google_container_cluster

```tf
resource "google_container_cluster" "cluster" {
  name     = "test"
  location = "asia-southeast1-b"
  min_master_version = "1.22.12-gke.300"
  network = "projects/gcp-202011216-001/global/networks/XXXX"
  subnetwork = "projects/gcp-202011216-001/regions/asia-southeast1/subnetworks/XXXX"
  default_max_pods_per_node = 64
  remove_default_node_pool = true
  initial_node_count       = 1
  enable_intranode_visibility = false
  ip_allocation_policy {
    cluster_secondary_range_name = "gke-pods"
    services_secondary_range_name = "gke-service"
  }
  resource_labels = {
    "env" = "test"
  }
  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
    network_policy_config {
      disabled = false
    }
  }
  master_auth {
    client_certificate_config {
      issue_client_certificate = false
    }
  }
  private_cluster_config {
    enable_private_endpoint = false
    enable_private_nodes = true
    master_ipv4_cidr_block = "172.16.0.0/28"
  }
  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }
  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS"]
  }
  node_config {
    machine_type = "e2-medium"
    disk_size_gb = 100
    disk_type    = "pd-standard"
    image_type   = "COS_CONTAINERD"
    oauth_scopes    = [
      "https://www.googleapis.com/auth/devstorage.read_only",
      "https://www.googleapis.com/auth/logging.write",
      "https://www.googleapis.com/auth/monitoring",
      "https://www.googleapis.com/auth/servicecontrol",
      "https://www.googleapis.com/auth/service.management.readonly",
      "https://www.googleapis.com/auth/trace.append"
    ]
    metadata = {
      disable-legacy-endpoints = "true"
    }
  }
}
```

- name：叢集的名稱，在這個專案及區域唯一名稱 <font color='red'>(必填)</font>
- location：要將此叢集建立在哪一個區域 <font color='blue'>(選填)</font>
- min_master_version：master 的最低版本 <font color='blue'>(選填)</font>
- network：叢集連接到的 Google Compute Engine 網絡的名稱或 self_link <font color='blue'>(選填)</font>
- subnetwork：啟動叢集的 Google Compute Engine 子網的名稱或 self_link <font color='blue'>(選填)</font>
- default_max_pods_per_node：此叢集中每個節點的預設最大 pod 數 <font color='blue'>(選填)</font>
- remove_default_node_pool：如果設定為 `true`，則在創建叢集時會幫我們刪除預設的節點池。會使用到這個的原因是因為 terraform 沒辦法修改預設節點池的名稱，所以我的做法是，會新增要的節點池，在使用這個參數把預設的給刪掉<font color='blue'>(選填)</font>
- initial_node_count：要在此叢集的預設節點池中創建的節點數 <font color='blue'>(選填)</font>
- enable_intranode_visibility：是否為此叢集啟用了節點內可見性 <font color='blue'>(選填)</font>
- ip_allocation_policy：為 VPC 原生叢集分配叢集 IP <font color='blue'>(選填)</font>
- resource_labels：應用於叢集的 GCE 資源標籤 key/value <font color='blue'>(選填)</font>
- addons_config
  - http_load_balancing：是否要啟用 HTTP (L7) 的負載平衡 <font color='blue'>(選填)</font>
  - horizontal_pod_autoscaling：是否要啟用 HPA 水平 Pod 自動擴展 <font color='blue'>(選填)</font>
  - network_policy_config：是否要啟用網路策略 <font color='blue'>(選填)</font>
- master_auth
  - client_certificate_config：是否要啟用該叢集客戶端證書授權 <font color='blue'>(選填)</font>
- private_cluster_config
  - enable_private_endpoint：是否要啟用叢集專用的私有端點，禁止公共端點的訪問 <font color='blue'>(選填)</font>
  - enable_private_nodes：是否要啟用私有叢集功能，在叢集創建私有端點 <font color='blue'>(選填)</font>
  - master_ipv4_cidr_block：私有端點 IP 範圍 <font color='blue'>(選填)</font>
- logging_config：叢集的日誌記錄配置
  - enable_components (公開日誌的 GKE 組件) 設定，包含：`SYSTEM_COMPONENTS`、`APISERVER`、`CONTROLLER_MANAGER`、`SCHEDULER`、`WORKLOADS` <font color='red'>(必填)</font>
- monitoring_config：叢集的監控配置
  - enable_components (GKE 組件公開指標) 設定，包含：`SYSTEM_COMPONENTS`、`APISERVER`、`CONTROLLER_MANAGER`、`SCHEDULER` <font color='blue'>(選填)</font>
- node_config：創建預設節點池參數
  - machine_type：Google Compute Engine 機器類型，預設為 `e2-medium` <font color='blue'>(選填)</font>
  - disk_size_gb：每個節點的 disk 大小，以 GB 為單位。允許最小為 10 GB，預設為 100 GB <font color='blue'>(選填)</font>
  - disk_type：連接到每個節點的 disk 類型，有 `pd-standard`、`pd-balanced` 或 `pd-ssd`，預設為 `pd-standard` <font color='blue'>(選填)</font>
  - image_type：創建新節點池後 NAP 使用的預設 image 類型。該值必須是 [`COS_CONTAINERD`、`COS`、`UBUNTU_CONTAINERD`、`UBUNTU`] 之一。`COS` 和 `UBUNTU` 已於 GKE 1.24 棄用 <font color='blue'>(選填)</font>
  - oauth_scopes：在預設服務帳戶下的所有節點虛擬機上可用的一組 Google API 範圍。 <font color='blue'>(選填)</font>
  - metadata：分配給叢集中實例的 key/value <font color='blue'>(選填)</font>

<br>

#### google_container_node_pool

```tf
resource "google_container_node_pool" "aaa" {
  name       = "aaa"
  project    = "project"
  location   = google_container_cluster.cluster.location
  cluster    = google_container_cluster.cluster.name
  node_count = 6
  node_locations = [
    google_container_cluster.cluster.location
  ]
  node_config {
	# 省略 ... 與上面的 google_container_cluster 相同
  }
  management {
    auto_repair  = true
    auto_upgrade = false
  }
  upgrade_settings {
    max_surge       = 1
    max_unavailable = 0
  }
}
```

- name：節點池名稱 <font color='blue'>(選填)</font>
- project：創建節點池的項目 ID <font color='blue'>(選填)</font>
- location：叢集所在位置，可以使用資源名稱 (google_container_node_pool) +命名 (cluster) + 參數 (location) 來代表 <font color='blue'>(選填)</font>
- cluster：叢集名稱，可以使用資源名稱 (google_container_node_pool) +命名 (cluster) + 參數 (name) 來代表 <font color='blue'>(選填)</font>
- node_count：節點數量 <font color='blue'>(選填)</font>
- node_locations：節點區域 <font color='blue'>(選填)</font>
- management：節點管理配置
  - auto_repair：是否要自動修復 <font color='blue'>(選填)</font>
  - auto_upgrade：是否要自動升級 <font color='blue'>(選填)</font>
- upgrade_settings：指定節點升級設定及方式
  - max_surge：升級期間可以添加到節點池的額外節點數 <font color='blue'>(選填)</font>
  - max_unavailable：升級期間可以同時不可用的節點數 <font color='blue'>(選填)</font>

<br>

可以看到有很多設定都是選填的，所以不需要像我範例一樣，把所有的都打出來，可以參考官方文件，將自己想要的設定寫出來，並注意其他參數的預設值是多少，就可以打造屬於自己的 Terraform 建立 Google Kubernetes Engine IaC 程式囉～

<br>

## 參考資料

registry.terraform.io/providers (google_container_cluster)：[https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_cluster](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_cluster)

registry.terraform.io/providers (google_container_node_pool)：[https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_node_pool](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_node_pool)
