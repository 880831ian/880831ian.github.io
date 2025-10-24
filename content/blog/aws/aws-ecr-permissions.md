---
title: "如何共用 ECR 給同一個 AWS Organization 的其他 AWS Account EKS 使用"
type: docs
weight: 9981
description: 如何共用 ECR 給同一個 AWS Organization 的其他 AWS Account EKS 使用
images:
  - aws/aws-ecr-permissions/og.webp
date: 2025-06-25
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

這篇主要是因為公司目前的 AWS 架構，會將各單位的系統依照產品、環境給拆分，但有時候部署，總會有一些共用的 Image。
在這之前，我們需要一個一個將相同的 Image 搬到每個 AWS Account 的 ECR，當時就在想，是否可以直接共用 ECR 給同一個 AWS Organization 的其他 AWS Account 使用 (當然也不要用 Public ECR 來避免安全性問題)。

<br>

> [!NOTE] 範例 AWS Account 說明
>為了方便說明，我會使用兩個 AWS Account 來做範例，632xxxxx1762 (後面簡稱 A 帳號) 以及 722xxxxx9750 (後面簡稱 B 帳號)，這兩個帳號都屬於同一個 AWS Organization。

<br>

那我們就直接來看要如何設定，首先我先在 632xxxxx1762 (後面簡稱 A 帳號) 放了一個 ECR，名稱是：`audit/sa/cicd-runner`，Tag 是 `latest`，它會是我之後想要共享給其他 AWS Account 使用的 ECR。

<br>

{{< figure src="/aws/aws-ecr-permissions/1.webp" width="650" caption="A 帳號 ECR" >}}

<br>

那如果我們使用另一個 AWS Account 722xxxxx9750 (後面簡稱 B 帳號) 直接拉這個 ECR 的 Image，會發生什麼錯誤呢？

我們先切換到 B 帳號，然後使用 `kubectl` 來部署一個 Pod，Pod 的 Image 寫 A 帳號的 ECR Image 名稱。

- cicd-runner.yaml

```yaml
apiVersion: apps/v1
kind: Pod
metadata:
  name: cicd-runner
spec:
  containers:
    - name: cicd-runner
      image: 632xxxxx1762.dkr.ecr.ap-southeast-1.amazonaws.com/audit/sa/cicd-runner:latest
      imagePullPolicy: Always
```

<br>

觀察一下 Pod 的狀態，會發現 Pod 會出現 `ErrImagePull` 的錯誤，這是因為 B 帳號沒有權限去拉取 A 帳號的 ECR Image。

<br>

{{< figure src="/aws/aws-ecr-permissions/2.webp" width="1200" caption="Pod ErrImagePull 錯誤" >}}

<br>

那要怎麼解決呢？！
以及我們要怎麼讓 B 帳號可以拉取 A 帳號的 ECR Image 呢？

其實很簡單，我們只需要在 A 帳號的 ECR 上設定一個 IAM Policy，讓 B 帳號可以拉取這個 ECR 的 Image。

如果以 Console 來設定的話，我們可以在 A 帳號的 ECR 上，點選 `Private registry` > `Permissions`，然後選擇 `Edit policy JSON`。
(如果之後使用 Terraform 來設定的話，可以參考 [AWS ECR Repository Policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecr_repository_policy) 的相關文件。)

將以下的 JSON 給貼進去，當然相關的設定可以再自行調整：

```json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "AllowPullFromOrg",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer"
      ],
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-xxxxxxx"
        }
      }
    }
  ]
}
```

這裡的 `o-xxxxxxx` 是你的 AWS Organization ID，可以在 AWS Organizations 的 Console 中找到。

<br>

簡單說明一下這邊的設定：
- `Principal` 設定為 `*`，表示允許所有人。
- `Action` 設定為 ECR 的相關拉取權限。
- `Condition` 設定為 `aws:PrincipalOrgID`，這樣就限制了只有同一個 AWS Organization 的帳號可以拉取這個 ECR 的 Image。(當然我們也可以改指定特定的 AWS Account ID，這樣就只有指定的帳號可以拉取。)

<br>

設定完畢後，我們可以回到 B 帳號，重新部署剛剛的 Pod。

```bash
kubectl apply -f cicd-runner.yaml
```

<br>

就可以發現 Pod 成功啟動了，並且可以正常拉取 A 帳號的 ECR Image。

{{< figure src="/aws/aws-ecr-permissions/3.webp" width="1200" caption="Pod 成功啟動" >}}

<br>

這樣就完成了如何共用 ECR 給同一個 AWS Organization 的其他 AWS Account 使用的設定。