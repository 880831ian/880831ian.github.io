# 使用 Terraform 建立 Google Compute Engine
嗨嗨大家好，距離上一篇筆記又隔了 3 個月，最近公司有專案在忙，沒時間把上次提到的 Terraform 應用筆記寫完，現在他來拉～～～ 😂 我們這次的主題是使用 Terraform 來建立 Google Compute Engine 的機器，想知道要怎麼用一段程式碼就可以建立、修改、刪除 Google Compute Engine 的機器一定要來看這一篇～我們開始囉 🧑‍💻

<br>

## 撰寫 Google Compute Engine tf 檔案

相信大家有先看完上一篇 [什麼是 IaC ? Terraform 又是什麼？](../terraform/) 才來看這一篇的對吧 😎，對於 Terraform 的程式架構及指令，我們這邊就不多做介紹，我們直接來看程式要怎麼寫～(程式碼主要是參考[官方文件](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)，加上一些其他的設定來做介紹，程式碼也會同步到 Github ，需要的也可以去 Clone 來使用歐！ [Github 程式碼連結](https://github.com/880831ian/terraform-gce) )

<br>

小提醒：由於程式碼較長，我將他拆開來說明 💖

<br>

### 選擇供應者以及對應的專案

```tf
provider "google" {
  project = "gcp-20210526-001"
}
```

由於我們要建立的 Google Compute Engine 是由 Google 所提供的 api 來建立的，所以一開始要先設定好提供者的名稱 google 以及我們要在哪一個 GCP 的專案 ID

<br>

### resource 設定

接下來的設定都會放在以下的 google_compute_instance resource 內，為了方便介紹，就不會標明 google_compute_instance，詳細完整程式碼請參考 GitLab [Github 程式碼連結](https://github.com/880831ian/terraform-gce)

```tf
resource "google_compute_instance" "default" {
}
```

<br>

#### 基本設定

```tf
  name        = "test"
  description = "我是 test 機器"
  machine_type = "n2-standard-8"
  zone = "asia-east1-b"
  tags = ["test"]

  labels = {
    env  = "test"
  }

  deletion_protection = "true"
```

- name：GCE 要求資源的唯一名稱。如果有更改此項會直接強制創建的新資源 <font color='red'>(必填)</font>
- description：對此資源的簡單說明 <font color='blue'>(選填)</font>
- machine_type：要創建的機器類型 <font color='red'>(必填)</font>
- zone：創建機器的所在區域，若沒有填寫，則會使用提供者的區域 <font color='blue'>(選填)</font>
- tags：附加到實體的網路標籤列表 <font color='blue'>(選填)</font>
- labels：一組分配給 disk 的 key/value 標籤 <font color='blue'>(選填)</font>
- deletion_protection：刪除保護，預設是 `false`，當我們使用 `terraform destroy` 刪除 GCE 時，必須先改成 `false`，才可以刪除，否則會無法刪除且 Terraform 運行也會失敗，算是一個保護機制，後面再刪除 Google Compute Engine 時會測試畫面 <font color='blue'>(選填)</font>

<br>

#### 啟動 disk 設定

```tf
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10-buster-v20210512"
      type = "pd-balanced"
      size = "50"
    }
  }
```

- image：初始化此 disk 的 image <font color='blue'>(選填)</font>
- type：GCE disk 類型 <font color='blue'>(選填)</font>
- size：image 大小，已 `GB` 為單位，如果未指定，將會繼承其初始化 disk 的 image 大小 <font color='blue'>(選填)</font>

<br>

#### 網路設定

```tf
  network_interface {
    network = "projects/rd-gateway/global/networks/rd-common"
    subnetwork = "projects/rd-gateway/regions/asia-east1/subnetworks/rd-common-asia-east1-pid-cicd"

    access_config {
      nat_ip = ""
    }

  }
```

- network：設定附加到的網路名稱或是 self_link <font color='blue'>(選填)</font>
- subnetwork：設定附加到的子網路名稱或是 self_link <font color='blue'>(選填)</font>
- nat_ip：如果想要有外網的 ip，必須加上此參數，才會產生一組外網 ip <font color='blue'>(選填)</font>

<br>

#### 權限設定

```tf
  service_account {
    email  = "676962704505-compute@developer.gserviceaccount.com"
    scopes = ["storage-rw", "logging-write", "monitoring-write", "service-control", "service-management", "trace"]
  }
```

- email：服務帳戶電子郵件地址。如果未提供，則使用預設的 Google Compute Engine 服務帳戶 <font color='blue'>(選填)</font>
- scopes：服務範圍列表，可以[點我查看範圍](https://cloud.google.com/sdk/gcloud/reference/alpha/compute/instances/set-scopes#--scopes)的完整列表 <font color='red'>(必填)</font>

<br>

以上只是我在建立 Google Compute Engine 最簡單的設定，當然還有很多其他的設定，可以參考 [registry.terraform.io/providers (google_compute_instance)](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance) 裡面有更多的 resource 設定，有需要的就自己來看看吧 🧐

<br>

## 建立 Google Compute Engine

當我們寫好 Google Compute Engine tf 檔案後，我們接著把他建立，建立前要先使用 `terraform init` 來做初始化

<br>

{{< figure src="/terraform/terraform-gce/1.webp" width="700" caption="terraform init" >}}

<br>

接著可以先使用 `terraform plan` 來查看我們的設定是否是我們想要的，或是直接用 `terraform apply` 來建立 Google Compute Engine

<br>

{{< figure src="/terraform/terraform-gce/2.webp" width="700" caption="terraform apply" >}}

<br>

可以看到成功建立我們的 test Google Compute Engine（也可以看到因為我們有開 nat_ip 所以有外部 IP）

<br>

{{< figure src="/terraform/terraform-gce/3.webp" width="700" caption="GCP Google Compute Engine" >}}

<br>

## 修改 Google Compute Engine

當我們發現我們建立的 Google Compute Engine 參數有錯，想要修改時，我們只需要修改程式碼部分，並重新下一次 `terraform apply` 來修改 Google Compute Engine，就會看到以下畫面 (有些設定檔是不能修改的，若修改他會重新創建一個新的機器，像是 name 之類的，使用時要小心一點 😉)

我們拿剛剛提到的 nat_ip，我們先把它註解掉，再下 `terraform apply` 看看機器有什麼變化～

<br>

{{< figure src="/terraform/terraform-gce/4.webp" width="700" caption="terraform changed" >}}

<br>

可以看到他會提示說，他會改變 network_interface，也移除 access_config 的設定，執行後的 Resources 也會從剛剛的 added 變成 changed，我們看一下 GCP 有沒有改變：

<br>

{{< figure src="/terraform/terraform-gce/5.webp" width="700" caption="terraform changed" >}}

<br>

可以看到原本的外部 IP 位置被改掉了～ <font color='red'>最後提醒</font>：如果有使用 terraform 來修改資源設定，不能動到特定的項目，不然他的流程是先把原本的給刪掉，再重新建立一個新的，原本的機器沒有備份，東西就會不見歐～

<br>

## 刪除 Google Compute Engine

最後假如我們要刪除 Google Compute Engine，也可以使用 terraform 的指令來刪除，我們順便來測試一下上面有設定的 deletion_protection 刪除保護機制是不是正常～

目前 deletion_protection 還是 `true`，我們直接下 `terraform destroy`，看看是否可以刪除 Google Compute Engine

<br>

{{< figure src="/terraform/terraform-gce/6.webp" width="900" caption="terraform destroy" >}}

<br>

可以看到他會提醒你說 Deletion Protection is enabled，必須先把他改成 `false` terraform apply 後才可以刪除～

<br>

{{< figure src="/terraform/terraform-gce/7.webp" width="900" caption="改成 false 在下 terraform apply" >}}

<br>

現在 deletion_protection 是 `false`，我們就可以下 `terraform destroy` 來刪除 Google Compute Engine 🔪

<br>

{{< figure src="/terraform/terraform-gce/8.webp" width="700" caption="terraform destroy" >}}

<br>

{{< figure src="/terraform/terraform-gce/9.webp" width="500" caption="Google Compute Engine 刪除中..." >}}

<br>

以上就是簡單的使用 Terraform 建立 Google Compute Engine 介紹囉～歡迎大家留言指教，明天的文章是介紹如何使用 Terraform 建立 GKE 💕

<br>

## 參考資料

registry.terraform.io/providers (google_compute_instance)：[https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)

