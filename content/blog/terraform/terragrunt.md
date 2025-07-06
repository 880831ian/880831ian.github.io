---
title: "如何導入 Terragrunt，Terragrunt 好處是什麼？"
type: docs
weight: 9994
date: 2023-06-21
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

我們接續上一篇的 [如何將 Terraform 改寫成 module ?](../terraform-module/) ，我們已經將 Terraform 改成 module 的方式來進行管理，但當我們要管理的資源越來越多，且有分不同的專案時，整個服務架構會長的像以下：

<br>

{{< filetree/container >}}
{{< filetree/folder name="modules" >}}
{{< filetree/folder name="google_compute_instance" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="outputs.tf" >}}
{{< filetree/file name="variables.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="projects" >}}
{{< filetree/folder name="gcp-1234" >}}
{{< filetree/folder name="aaa" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="bbb" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="ccc" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< filetree/folder name="gcp-2345" state="closed" >}}
{{< filetree/folder name="aaa" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="bbb" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="ccc" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< filetree/folder name="gcp-3456" state="closed" >}}
{{< filetree/folder name="aaa" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="bbb" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="ccc" >}}
{{< filetree/file name="backend.tf" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="provider.tf" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/container >}}

<br>

這邊的範例是以不同專案來分，再分區不同的服務，每個服務裡面都會有 `backend.tf`、`main.tf`、`provider.tf` 檔案所組成，我們可以來比較一下 gcp-1234 的 aaa 服務以及 gcp-2345 的 aaa 服務差異：

<br>

{{< figure src="/terraform/terragrunt/1.webp" width="800" caption="檔案差異" >}}

<br>

可以看到在 `backend.tf` 除了 prefix 路徑以外，其他設定也都一樣，但因為 Terraform 本身沒辦法透過帶入參數的方式來設定 `backend.tf` 後端部分，所以必須要先寫好每個服務所存放的後端位置，十分的不方便。

```tf {filename="backend.tf"}
terraform {
  backend "gcs" {
    bucket      = "terragrunt-tfstate"
    prefix      = "/gcp-1234-aaa"
  }
}
```

<br>

為了減少上述這些需要一直重複寫差不多檔案的工作內容，因此有了 Terragrunt 這個工具，Terragrunt 是 Terraform 的包裝器，可以彌補 Terraform 上的一些缺陷，並且讓我們的 IaC 更貼近 [DRY 原則](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。

<br>

{{< callout note >}}

這邊說明一下 DRY 原則

DRY 全名是 Don’t repeat yourself，也就是不要做重複的事情，能夠一次做完的就不要重複的去做。

{{< /callout >}}

<br>

## Terragrunt 安裝方式

那要怎麼使用 Terragrunt 呢！？

第一步當然是安裝它囉，我們系統是 macOS，所以我們安裝方式是 [Homebrew](https://brew.sh/) 來進行安裝：

```shell
brew install terragrunt
```

<br>

接著我們的指令會從 `terraform XXX` 變成以下：

```shell
terragrunt plan
terragrunt apply
terragrunt output
terragrunt destroy
```

Terragrunt 會將所有命令、參數和選項直接轉發到 Terraform。(所以我們也需要下載 Terraform)

Terragrunt 的預設檔案名稱是 terragrunt.hcl ，Terragrunt 的設定檔案基本上與 Module 差不多，只是有更多更方便的變數可以使用。

<br>

## Terragrunt 好處

<br>

接著介紹一下 Terragrunt 的好處：

1. 方便管理後端狀態設定

2. 將後端存儲桶納入管理

3. 使用 generate 自動生成檔案

4. 使用 include 檔案來達到 DRY 原則

5. 管理 Module 之間的依賴性

6. 產生依賴關聯圖

<br>

### 方便管理後端狀態設定

<br>

首先第一個方便管理後端狀態設定，也就是我們上面提到的 `backend.tf` 設定。在 Terraform 原生為了區別不同專案不同服務的狀態檔，就必須先寫好每個儲存的路徑，但使用 Terragrunt，可以先在該目錄下，也就是 gcp-3456 目錄下先寫一個設定檔案 (我們以 gcp-3456 專案為例)，讓底下的 aaa、bbb、ccc 服務可以去 include 它，我們就不需要每個服務都寫幾乎差不多的設定檔，接著我們在 gcp-3456 資料夾下新增 terragrunt.hcl 檔案來說明：

```hcl {filename="terragrunt.hcl"}
remote_state {
  backend = "gcs"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket  = "terragrunt-tfstate"
    prefix = "${path_relative_to_include()}"
  }
}
```

<br>

這邊的設定其實跟之前的 `backend.tf` 差不多，只是後端現在儲存的 block 改叫做 `remote_state`，可以看到 backend 設定我們一樣是存在 gcs 上，`generate` 這個 block 它會判斷 backend.tf 檔案是否存在，如果沒有它就會幫我們建立，其中設定檔案內容是將狀態檔案存在 `terragrunt-tfstate` 這個 bucket，並透過`${path_relative_to_include()}` 這個變數來自動帶入有 include 這份檔案的路徑，並在相對路徑產生 `backend.tf` 檔案。

<br>

有點抽象，所以我畫一個比較簡單的架構圖來做說明一下，假設現在有三個服務，如下：

{{< filetree/container >}}
{{< filetree/folder name="aaa" >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="bbb" >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="ccc" >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/folder >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/container >}}

<br>

我們剛剛的後端設定是寫在此根目錄的 terragrunt.hcl 檔案(第 8 行)，然後 ccc 這個服務去 include 根目錄的 terragrunt.hcl 檔案， Terragrunt 就會自動幫你產生以下的 `backend.tf` 檔案：

```tf {filename="backend.tf"}
terraform {
  backend "gcs" {
    bucket      = "terragrunt-tfstate"
    prefix      = "/ccc"
  }
}
```

這樣就可以省下我們要重複寫 `backend.tf` 的時間，在維護上也會更加的方便。

<br>

### 將後端存儲桶納入管理

接著，大家有沒有想過，我們都已經使用 Terraform 來管理 IaC ，並把狀態檔案放到 gcs 上面來保存，但一開始還沒有用 Terraform 管理 gcs 的資源，我們還需要在設定 `backend.tf` 前，先手動去新增一個 gcs ，才能來存放 tfstate 狀態檔案呢？

<br>

因此在 Terragrunt `remote_state` 的 config 時，可以多設定 gcs 的 project id 以及 location，Terragrunt 在初始化後端時，會檢查是否有該 gcs bucket，如果沒有就會自動建立，設定檔如下：

```hcl {filename="terragrunt.hcl"}
remote_state {
  backend = "gcs"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    project = "gcp-xxxxxx"
    location = "asia"
    bucket  = "terragrunt-tfstate"
    prefix = "${path_relative_to_include()}"
  }
}
```

<br>

### 使用 generate 自動生成檔案

在剛剛我們可以使用 generate 來自動 `backend.tf` 檔案，那代表我們也可以把每個 `provider.tf` 的內容，也透過 generate 來生成，如下：

```hcl {filename="terragrunt.hcl"}
generate "provider" {
  path = "provider.tf"
  if_exists = "overwrite"
  contents = <<EOF
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "~> 4.48.0"
    }
  }
}
EOF
}
```

這樣子每個 include 這份設定檔的服務除了 `backend.tf` 檔案以外，還會自動產生 `provider.tf` 檔案。

<br>

### 使用 include 檔案來達到 DRY 原則

我們搞定了 `backend.tf` 跟 `provider.tf` 後，還剩下 `main.tf`，所以我們也將它改成 Terragrunt 的格式如下：

```hcl {filename="terragrunt.hcl"}
terraform {
  source = "${get_path_to_repo_root()}/modules/google_compute_instance"
}

include "root" {
  path = find_in_parent_folders()
}

inputs = {
  instance_name = "gcp-3456-ccc"
  machine_type  = "e2-small"
  instance_zone = "asia-east1-b"
  instance_tags = []
  instance_labels = {}
  boot_disk_auto_delete = true
  boot_disk_image_name  = "debian-cloud/debian-10"
  ... 設定部分省略 ...
}
```

<br>

這邊可以看到 terraform `source` 它就是我們使用 module 的路徑，也可以用 `${get_path_to_repo_root()}` 這個變數他會自動抓該專案的根目錄，我們就不需要去特別設定。

<br>

此外也可以將 module 獨立成一個專案，或是使用其他人寫好的 module，在 `source` 的時候可以使用 `git::https://[gitlab-網址]/sre/terraform/module.git//google_compute_address` 的方式來取得 module，可以設定要使用哪個分支或是 tag，只需要在網址後面加上，`?ref=[分之 or tag 名稱]` 即可，這樣可以讓開發中的 module 不會影響到線上其他正在使用中的 module 設定。

(`module.git` 後面的 `//` 是 Terraform module source 的規則，如果不加會跳警告訊息)

<br>

接著我們可以看到 `include "root" {}` 這段，裡面有使用 `find_in_parent_folders` 這邊變數，他就是上面的提到會自動去抓放在父資料夾的 `remote_state` 跟 `generate` terragrunt.hcl 檔案。

<br>

後面的 input 就跟使用 module 時一樣，將 module 的參數帶入即可。

補充：所以我們也可以把一些通用的設定寫在根目錄的 terragrunt.hcl 檔案，例如專案的 id，可以寫以下內容來讓 include 它的檔案吃到同一個參數設定：

```hcl {filename="terragrunt.hcl"}
inputs = {
  project_id = "gcp-3456"
}
```

<br>

### 管理 Module 之間的依賴性

由於 Terragrunt 是把每個服務拆分成最小化，沒辦法把使用不同 module 的資源放在一起(單純使用 module 的話，可以一次 source 多了 module，並把他放在同一個 tf 檔案中)，那像是我們建立 k8s 會使用到 `google_container_cluster`、`google_container_node_pool` 兩種不同的 module 該怎麼辦呢？

<br>

首先我們先在 modules 資料夾放上 `google_container_cluster`、`google_container_node_pool` 兩個 module 的設定檔案，詳細程式可以[點我前往](https://github.com/880831ian/terragrunt)

{{< filetree/container >}}
{{< filetree/folder name="modules" >}}
{{< filetree/folder name="google_container_cluster" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="outputs.tf" >}}
{{< filetree/file name="variables.tf" >}}
{{< /filetree/folder >}}
{{< filetree/folder name="google_container_node_pool" >}}
{{< filetree/file name="main.tf" >}}
{{< filetree/file name="variables.tf" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/container >}}

<br>

在 projects 底下新增 gke 資料夾，新增 terragrunt.hcl 來放 `remote_state provider` 的設定，並區分兩個資料夾，分別是 cluster 資料夾來存放 cluster 資訊，以及 test 資料夾來存放 test node-pool 資訊：

{{< filetree/container >}}
{{< filetree/folder name="projects" >}}
{{< filetree/folder name="gke" >}}
{{< filetree/folder name="cluster" >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/folder >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< filetree/folder name="test" >}}
{{< filetree/file name="terragrunt.hcl" >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/folder >}}
{{< /filetree/container >}}

<br>

cluster 的 terragrunt.hcl 檔案如下：

```hcl {filename="terragrunt.hcl"}
terraform {
  source = "${get_path_to_repo_root()}/modules/google_container_cluster"
}

include {
  path = find_in_parent_folders()
}

inputs = {
  cluster_name             = "tf-test"
  cluster_location         = "asia-east1-b"
  node_locations           = []
  cluster_version          = "1.24.12-gke.500"
  network_name             = "projects/gcp-202011216-001/global/networks/bbin-testdev"
  subnetwork_name          = "projects/gcp-202011216-001/regions/asia-east1/subnetworks/bbin-testdev-dev-platform"
  node_max_pods            = 64
  remove_default_node_pool = true
  initial_node_count       = 1
  enable_shielded_nodes    = false
  resource_labels = {
    "dept" : "pid",
    "env" : "dev",
    "product" : "bbin"
  }
  dns_enabled                  = false
  cluster_dns                  = "PROVIDER_UNSPECIFIED"
  cluster_dns_scope            = "DNS_SCOPE_UNSPECIFIED"
  private_cluster_ipv4_cidr    = "172.16.0.176/28"
  binary_authorization_enabled = true
  binary_authorization         = "DISABLED"
}
```

<br>

test node-pool 檔案如下：

```hcl {filename="terragrunt.hcl"}
terraform {
  source = "${get_path_to_repo_root()}/modules/google_container_node_pool"
}

include {
  path = find_in_parent_folders()
}

dependency "cluster" {
  config_path = "../cluster"
}

inputs = {
  cluster_name      = dependency.cluster.outputs.cluster_name
  cluster_location  = dependency.cluster.outputs.cluster_location
  cluster_version   = dependency.cluster.outputs.cluster_version
  node_pool_name    = "test"
  node_count        = 1
  node_machine_type = "e2-small"
  node_disk_size    = 100
  node_disk_type    = "pd-standard"
  node_image_type   = "COS_CONTAINERD"
  node_oauth_scopes = [
    "https://www.googleapis.com/auth/devstorage.read_only",
    "https://www.googleapis.com/auth/logging.write",
    "https://www.googleapis.com/auth/monitoring",
    "https://www.googleapis.com/auth/service.management.readonly",
    "https://www.googleapis.com/auth/servicecontrol",
    "https://www.googleapis.com/auth/trace.append"
  ]
  node_tags                        = []
  node_taint_enabled               = false
  node_taint_key                   = ""
  node_taint_value                 = ""
  node_taint_effect                = ""
  auto_repair                      = true
  auto_upgrade                     = true
  upgrade_max_surge                = 1
  upgrade_max_unavailable          = 0
  upgrade_strategy                 = "SURGE"
  autoscaling_enabled              = true
  autoscaling_max_node_count       = 2
  autoscaling_min_node_count       = 1
  autoscaling_total_max_node_count = 0
  autoscaling_total_min_node_count = 0
}
```

<br>

上面兩個檔案分別是 cluster 的設定，以及 test node-pool 的設定，裡面的設定，上面基本都有提過，這邊要提的是 `dependency`，`dependency` 他是 Terragrunt 提供讓我們可以方便地去管理 IaC 之間的相依性，像是我們這邊，需要先建立好 cluster 才能建立 node_pool，此時就可以依靠 dependency block 來完成需求。

( 靠 dependency 來取得 cluster 的資訊，並帶入 node_pool 中 )

<br>

此時的執行指令是在 gke 目錄下，使用 `terragrunt run-all [參數]` 來跑整個相依的 module，我們這邊就建立一個名為 tf-test 的 cluster，並且有一個名為 test 的 node_pool ，其他設定請參考上面程式：

<br>

{{< figure src="/terraform/terragrunt/2.webp" width="700" caption="測試 terragrunt run-all" >}}

(黃色的 WARN 是因為 gcs 上還沒有存過該狀態檔案，所以會跳出提示)

<br>

等到 cluster 建立完成後，會將 cluster 的資訊帶入 node_pool，才開始建立 node_pool 的資源：

{{< figure src="/terraform/terragrunt/3.webp" width="700" caption="測試 terragrunt run-all" >}}

<br>

### 產生依賴關聯圖

當我們服務使用到很多依賴關係，想要釐清是誰依賴誰，如果單純看程式會比較麻煩，在 Terragrunt 還有一個好用的指令，可以使用以下指令，產生對應的依賴關係圖，在檢視時可以更清楚知道關係：

```shell
terragrunt graph-dependencies | dot -Tpng > graph.png
```

( 這個 dot 指令是另外的套件，會將關係圖程式碼轉成圖檔，請先安裝 `brew install graphviz` )

<br>

{{< figure src="/terraform/terragrunt/4.webp" width="100" caption="關聯圖" >}}

<br>

## 參考資料

事半功倍 — 使用 Terragrunt 搭配 Terraform 管理基礎設置：[https://medium.com/act-as-a-software-engineer/%E4%BA%8B%E5%8D%8A%E5%8A%9F%E5%80%8D-%E4%BD%BF%E7%94%A8-terragrunt-%E6%90%AD%E9%85%8D-terraform-%E7%AE%A1%E7%90%86%E5%9F%BA%E7%A4%8E%E8%A8%AD%E6%96%BD-f70c30166639](https://medium.com/act-as-a-software-engineer/%E4%BA%8B%E5%8D%8A%E5%8A%9F%E5%80%8D-%E4%BD%BF%E7%94%A8-terragrunt-%E6%90%AD%E9%85%8D-terraform-%E7%AE%A1%E7%90%86%E5%9F%BA%E7%A4%8E%E8%A8%AD%E6%96%BD-f70c30166639)
