---
title: "Terraform 如何多人共同開發 (將 tfstate 存在後端)"
type: docs
weight: 3
---

此篇是接續上一篇 [什麼是 IaC ? Terraform 又是什麼？](https://pin-yi.me/terraform/)的 Terraform 文章，我們在上一篇有提到 `terraform apply` 完後，會多一個檔案 `*.tfstate`，這個檔案是用來存放服務狀態的檔案，它包含基礎架構的狀態和資源的詳細信息。假設大家都在自己的本地去 apply 同一個服務，會導致每個人的 tfstate 檔案內容不同，有可能去覆蓋掉其他人已經調整的內容，因此我們必須將此 tfstate 檔案存放在一個地方，讓大家都去使用同一份檔案來調整資源。

<br>

我們常用的儲存方式會將 tfstate 存在 `gitlab` 或 `gcs` (gcp 架構為例)，以下會簡單說明要如何把 tfstate 存到後端以及各頁面的功能：

<br>

## gitlab

那我們要怎麼把 tfstate 存到 gitlab 呢？首先跟之前一樣，先新增 `provider.tf` 來放供應商的來源以及版本，以及 `main.tf` 來放 gce 相關設定，最後還要多一個 `backend.tf` 來放我們要儲存 tfstate 的位置設定，如下：([同步到 GitHub 需要程式碼的可以前往查看](https://github.com/880831ian/terraform-tfstate))

(這次範例會使用 gce，此項會需要 gitlab 先啟用 Infrastructure 功能以及建立自己的 gitlab token)

<br>

```tf {filename="provider.tf"}
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "~> 4.48.0"
    }
  }
}
```

<br>

```tf {filename="main.tf"}

provider "google" {
  project = "XXXXX"
  zone    = "asia-east1-b"
}

resource "google_compute_instance" "instance" {
  name         = "test"
  machine_type = "e2-small"
  zone         = "asia-east1-b"

  labels = {
    env = "11"
  }

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "projects/XXXX/global/networks/test"
    subnetwork = "projects/XXXX/regions/asia-east1/subnetworks/testtest"
  }
}
```

<br>

- backend.tf << 新的

專案 ID 要寫我們想要放 terraform state 的 GitLab Project ID，服務名稱是指顯示在 GitLab terraform state 的名稱

gitlab 個人 token 是指個人存取權杖，大家再依照自己的來做設定

```tf {filename="backend.tf"}
terraform {
  backend "http" {
    address = "[Gitlab 網址]/api/v4/projects/[專案ID]/terraform/state/[服務名稱]"
    lock_address = "[Gitlab 網址]/api/v4/projects/[專案ID]/terraform/state/[服務名稱]/lock"
    unlock_address = "[Gitlab 網址]/api/v4/projects/[專案ID]/terraform/state/[服務名稱]/lock"
    username = "[Gitlab 帳號]"
    password = "[Gitlab 個人 token]"
    lock_method = "POST"
    unlock_method = "DELETE"
    retry_wait_min = 5
  }
}
```

<br>

當我們新增好後，就跟之前步驟一樣，先 init > plan > apply 來做測試，在 init 時會發現，與之前不太一樣的是，在 Initializing the backend 的下方有多了綠色的成功設定後端字樣，代表他也會將後端的相關資訊存進 .terraform 資料夾中，所以有變更後端儲存位置，要記得重新 init 歐

<br>

{{< figure src="/terraform/terraform-tfstate/1.png" width="700" caption="init 初始化後，會將後端資訊也存到 .terraform 資料夾" >}}

<br>

當我們 plan > apply 完成後，可以觀察一下，發現原本會產生的 terraform.tfstate 檔案沒有出現在該目錄下：

<br>

{{< figure src="/terraform/terraform-tfstate/2.png" width="700" caption="apply 完，沒有在本地產生 .tfstate 檔案" >}}

<br>

這時候，我們可以到剛剛在上面設定的專案 ID 內的有一個 Infrastructure / Terraform，裡面就會存放 Terraform state 檔案，如下：

<br>

{{< figure src="/terraform/terraform-tfstate/3.png" width="500" caption="gitlab/Infrastructure/Terraform" >}}

<br>

會顯示狀態名稱、更新資訊、以及 Actions 等欄位：

<br>

{{< figure src="/terraform/terraform-tfstate/4.png" width="1000" caption="gitlab terraform tfstate 網頁" >}}

<br>

功能部分可以看後面的 Actions 欄位底下有三個點點，可以下載對應的 tfstate 檔案、Lock 讓其他人不能對此進行 apply，或是刪除此 tfstate 檔案等

<br>

{{< figure src="/terraform/terraform-tfstate/5.png" width="500" caption="gitlab terraform tfstate 功能說明" >}}

<br>

這樣我們就可以透過同一份的 tfstate 檔案來做管理，但有個前提是，之後對該資源的變更都只能使用 tf，如果還有用 WEB UI 去調整，就會遇到線上服務與 tfstate 儲存狀態不同的問題。

<br>

### Lock

那當我們已經有了共同儲存的地方，也溝通好，不會使用 WEB UI 去調整，但如果有兩個人同時去下 apply 的話，第一個人的 apply 還在執行，後面那個人的 apply 是不是就會蓋掉前一個人的設定呢？

所以 Terraform 在 0.14 版本推出了 Lock 功能，當有人在 plan or apply 的時候，我們去查看 gitlab Terraform state，會看到我們的 tfstate 檔案會被 Lock 起來

<br>

{{< figure src="/terraform/terraform-tfstate/6.png" width="1000" caption="GitLab Lock 鎖住" >}}

<br>

此時除了第一個操作者，其他人再去 plan or apply 就會出現錯誤，可以看到是誰正在使用，以及操作的動作是 plan or apply

<br>

{{< figure src="/terraform/terraform-tfstate/7.png" width="700" caption="在 Lock 下，其他人沒辦法去 plan or apply" >}}

<br>

在 CI 時，需要在 plan 時就將它給 lock，避免第一個人 plan 完，沒有及時的去執行 apply，後來有其他人比第一個人先調整了資源，第一個人再來執行 apply，就會導致第一個人 apply 的內容與自己原先看 plan 的內容會不同，也有可能會將上一個人調整的設定給覆蓋，當第一個操作者結束動作後，該 Lock 才會被解鎖。

<br>

其他 gitlab terraform state 詳細內容可以參考：[https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html](https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html)

<br>

## gcs

gcs 儲存比較簡單一點，因為他就是一個 bucket，所以頁面就跟一般的 gcs 一樣，會顯示檔案名稱、大小、類型等，如果需要查看 tfstate，可以點擊最後的下載按鈕來查看

那我們接著使用上面 gitlab 的範例檔案，只是要將 backend.tf 內容改為以下：

<br>

```tf {filename="backend.tf"}
terraform {
  backend "gcs" {
    bucket      = "pid-terraform-state"
    prefix      = "/aaa"
  }
}
```

<br>

上面的設定是指，我們將 backend 後端設定改成 gcs，並且選擇名為 pid-terraform-state 的 bucket，此 bucket 需要先手動建立(因為 bucket 名稱是全域不重複，所以不需要特別設定其 project_id，只要有權限正確都可以跨專案使用)，以及我們要將此 tfstate 存在 aaa 資料夾內。

<br>

接著我們重新刪除剛剛 gitlab 已產生的 .terraform/ 跟 .terraform.lock.hcl 檔案，重新下 init，就可以到 gcs 對應資料夾下，新增了 defaulte.tfstate 檔案。

<br>

{{< figure src="/terraform/terraform-tfstate/8.png" width="700" caption="產生 defaulte.tfstate" >}}

<br>

### Lock

我們一樣來看一下 gcs 的 lock 會長什麼樣子，gcs lock 會產生一個 default.tflock 檔案，由他去判斷現在是否是 Lock 狀態

<br>

{{< figure src="/terraform/terraform-tfstate/9.png" width="700" caption="gcs Lock 會出現 .tflock 檔案" >}}

<br>

當有其他人也執行 plan or apply 後，就會顯示以下：

<br>

{{< figure src="/terraform/terraform-tfstate/10.png" width="700" caption="gcs Lock 其他人不能操作" >}}
