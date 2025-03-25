---
title: "Identity and Access Management (IAM) 介紹"
type: docs
weight: 9997
images:
  - aws/iam-introduce/og.webp
date: 2024-07-26
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

先來說一下為什麼會寫這篇文章，主要是在寫跟測試 [Elastic Kubernetes Service (EKS) 介紹](../eks-introduce) 文章時，想要用 eksctl 來建立 EKS，它實際上是透過 CloudFormation 來建立的，詳細可以去看 EKS 文章，總之當時遇到權限的問題，找了很久，也看不懂 AWS 的 ARN 是什麼，所以就先來寫一篇 IAM 的文章 (´≖◞౪◟≖)。

<br>

## 武功秘籍

文章主要參考：[[AWS IAM] 學習重點節錄(2) - IAM Policy](https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/)，這篇文章寫得很詳細，當然，我會加上一些自己的理解跟測試範例，如果有興趣的話，可以去看看原文。

下方文章主要都圍繞官方的 [Actions, resources, and condition keys for AWS services](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html) 文件上，如果後面沒有附連結，基本上就在講這份，請大家可以打開網頁一起看。

<br>

## 什麼是 IAM？

IAM 的全名是 Identity and Access Management，從字面上來看就可以得知，它是用來管理 AWS 資源的身份和存取權限的服務。透過 IAM，您可以控制對 AWS 資源的存取權限，以及對這些資源的操作權限。也就是 `Who (身份) 可以 Do (操作) What (資源)` 的一個功能。

<br>

## Policy Type

Policy Type 有三種：

1. AWS 受管

   由 AWS 提供的 Policy，這些 Policy 會包含一些常見的操作，例如：S3 的讀寫、EC2 的啟動、停止等等。如果 AWS 有新增服務，也會同時更新這些 policy。詳細可以看：[AWS 受管理政策
   ](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies)

<br>

2. AWS 受管 – 職務職能

   這些 Policy 會針對特定的工作角色，例如：系統管理員 (AdministratorAccess)、網路管理員任務角色 (NetworkAdministrator)、資料庫管理員任務角色 (DatabaseAdministrator) 等等，這些 Policy 會包含這些角色常見的操作。詳細可以看：[AWS 受管理的工作職能政策](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/access_policies_job-functions.html)

<br>

3. 客戶受管

   使用者自己定義的 Policy，這些 Policy 會針對自己的需求來定義，例如：只能讀取某個 S3 bucket、只能啟動某個 EC2 instance 等等。詳細可以看：[客戶受管政策](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#customer-managed-policies)

<br>

## Policy 結構

IAM Policy 會是一個 JSON 格式的文件，可以定義了一個或多個 Action，這個 Policy 會告訴 AWS 這個身份可以對哪些資源 (Resource) 進行哪些操作 (Action)。而這個 Policy 會被附加到一個身份上，這個身份可以是一個 IAM User、IAM Group 或 IAM Role。然後一個 User、Group 或 Role 可以有多個 Policy。

<br>

{{< figure src="/aws/iam-introduce/1.png" width="650" caption="我目前公司帳號的 Policy" >}}

<br>

上面的圖片就是一個典型的 IAM 的 Policy 結構，可以看到這個 Policy 會有 Version、Statement、Effect、Action、Resource、Condition (上面沒有 xD) 等等主要元素。

<br>

下面就針對 Version、Statement、Effect、Action、Resource、Condition 這些元素來做一些介紹。

重點提醒：

- 所有的元素都是有區分大小寫的，寫錯大小寫會導致 Policy 無法正確解析。

<br>

### Version

Version：用來告知 AWS 這個 Policy 是使用哪個版本的 IAM Policy 語法，目前最新的版本是 2012-10-17，舊版本是 2008-10-17。詳細請參考：[IAM JSON 政策元素：Version](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_version.html)

<br>

### Statement

Statement：是 Policy 的主要元素。此元素為必填。Statement 元素可包含單一陳述式，或是個別陳述式的陣列。每個個別的陳述式區塊都必須用大括號 { } 括起。針對多個陳述式，陣列必須用方括號 [ ] 括起。詳細請參考：[IAM JSON 政策元素：Statement](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_statement.html)

例如：

```
"Statement": [{...},{...},{...}]
```

<br>

### Effect

Effect：元素是必填的，指定陳述式是否允許或拒絕。

- Effect 的有效值為 Allow 和 Deny。Effect 值會區分大小寫。
- AWS 預設是 Deny，所以如果沒有設定 Effect，則預設是 Deny，需要明確指定 Allow。
- Policy 每個 Statement 都必須有一個 Effect 元素。
- 如果同時有 Allow 和 Deny 套用同一個資源，則 Deny 會優先於 Allow。

詳細請參考：[IAM JSON 政策元素：Effect](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_effect.html)

<br>

### Action

Action：描述哪一個服務可以做哪些動作。

- 每個 AWS 服務都有自己的一組動作，描述您可以使用該服務執行的哪些任務。
- 使用服務命名做為動作開頭 (iam、ec2、sqs、sns、s3 等) 來指定值，後面則加上對應的動作名稱。
- Action 服務和動作名稱不區分大小寫。例如，`iam:ListAccessKeys` 與 `IAM:listaccesskeys` 相同

下面舉例幾個 Action：

1. Amazon S3 取得物件：

```
"Action": "s3:GetObject"
```

2. IAM 修改密碼：

```
"Action": "iam:ChangePassword"
```

3. EC2 啟動實例：

```
"Action": "ec2:StartInstances"
```

可以從上面得知 s3、iam、ec2 是 AWS 提供的服務，而 GetObject、ChangePassword、StartInstances 是這些服務提供的動作。

<br>

當然，服務很多，再加上每個服務的動作也不一樣，那我們可以依照 [武功秘籍](#武功秘籍) 提到的 [Actions, resources, and condition keys for AWS services](https://docs.aws.amazon.com/zh_tw/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html) 去查詢對應的動作列表。

<br>

#### 如何方便的撰寫 Action？

如果，我們需要新增服務很多的 Action，我們只能一個一個輸入嗎？像下面這張圖片，我們可以看到 Elastic Kubernetes Service (EKS) 的 Action 部分清單，假如我們要這些全部的動作，需要把每個都寫出來嗎？

<br>

{{< figure src="/aws/iam-introduce/2.png" width="650" caption="Elastic Kubernetes Service Action 部分清單 [Amazon Elastic Kubernetes Service 定義的動作](https://docs.aws.amazon.com/zh_tw/service-authorization/latest/reference/list_amazonelastickubernetesservice.html)" >}}

<br>

當然不用！我們可以使用 `*` 來代表所有的動作，例如：

```
"Action": "eks:*"
```

這樣就可以代表所有的 EKS 的動作。

<br>

或是只想要 eks 的 Create 相關動作，可以這樣寫：

```
"Action": "eks:Create*"
```

這樣就包含了 `CreateAccessEntry`、`CreateAddon`、`CreateCluster`、`CreateFargateProfile`、`CreateNodegroup` 等等動作。

<br>

或是我們只想要有 eks 的 cluster 相關動作(不要 Addon、Nodegroup 等等)，可以這樣寫：

```
"Action": "eks:*Cluster*"
```

這樣就只有 `CreateCluster`、`DeleteCluster`、`DescribeCluster`、`ListClusters` 等等的動作，沒有 `CreateAddon`、`CreateNodegroup` 動作。

詳細請參考：[IAM JSON 政策元素：Action](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_action.html)

<br>

### Resource

Resource：描述哪一個服務的哪一個資源。

這邊以 Amazon S3 為例，我們建立了一個 bucket，它就是一個 Resource，我們上傳了一個檔案到這個 bucket，這個檔案也是一個 Resource。

但是，我們要如何去定義我們可以對哪些的 AWS Resource 做哪些的操作呢？這時候就可以使用 ARN (Amazon Resource Name) 來定義。

<br>

ARN 它的命名會將不同服務、不同 region、不同使用者帳號都考慮進去，產生一個唯一識別 AWS 資源的字串，下面是三種 ARN 格式：

```
arn:<partition>:<service>:<region>:<account-id>:<resource-id>
arn:<partition>:<service>:<region>:<account-id>:<resource-type>/<resource-id>
arn:<partition>:<service>:<region>:<account-id>:<resource-type>:<resource-id>
```

詳細可以參考：[ARN 格式](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference-arns.html)

<br>

那我們一樣，將 partition、service、region、account-id、resource-type、resource-id 都個別簡單介紹一下：

#### partition (分區)

它是 AWS 的分區，目前有 `aws` 和 `aws-cn` 以及 `aws-us-gov`，分別代表 AWS 區域和中國區域以及 AWS GovCloud (US) 區域，所以基本上我們都會使用 `aws`。

<br>

#### service (服務)

它是 AWS 的服務名稱字首，例如：s3、iam、ec2、eks 等等。不清楚每個服務的字首可以參考 [AWS 服務的動作、資源和條件索引鍵](https://docs.aws.amazon.com/zh_tw/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html)，假設我們點進去看到 Amazon S3 的服務名稱字首是 `s3`。

<br>

{{< figure src="/aws/iam-introduce/3.png" width="650" caption="[Amazon S3 的動作、資源和條件索引鍵](https://docs.aws.amazon.com/zh_tw/service-authorization/latest/reference/list_amazons3.html)" >}}

<br>

#### region (區域)

它是 AWS 的區域，例如：us-east-1、us-west-2、ap-northeast-1 等等。如果需區域代碼清單，可以查看[區域端點](https://docs.aws.amazon.com/zh_tw/general/latest/gr/rande.html#regional-endpoints)

<br>

#### account-id (帳號識別碼)

它是擁有資源的 AWS 帳號識別碼 (不含連字號)，例如：123456789012。

<br>

#### resource-type (資源類型)

它是資源的類型，例如：vpc (虛擬私有雲)，bucket (S3 bucket)，instance (EC2 instance) 等等。

<br>

#### resource-id (資源識別碼)

它是資源名稱、資源 ID 或資源路徑。某些資源識別碼包括父項資源 (sub-resource-type/父資源/子資源) 或限定元，

例如：

1. IAM User

```
arn:aws:iam::123456789012:user/ian_zhuang
```

這個在最上面的 Policy 結構中的圖片可以看到我的 User ARN。

2. VPC

```
arn:aws:ec2:us-east-1:123456789012:vpc/vpc-0e9801d129EXAMPLE
```

3. S3 Bucket

```
arn:aws:s3:::*
arn:aws:s3:::demo-bucket-a/*
arn:aws:s3:::demo-bucket-b/downloads/*
```

這邊 S3 Bucket 的 ARN 有三個：

第一個代表的是所有的 S3 bucket & object。

第二個代表的 demo-bucket-a 對於此 bucket 的所有 object。

第三個代表的 demo-bucket-b 的 downloads 目錄下的所有 object。

<b>另外，還會發現，因為 S3 服務它沒有 `<region>`、`<account-id>` 所以中間兩個就留空即可。</b>

詳細可以參考：[IAM JSON 政策元素：Resource](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_resource.html)

<br>

### Condition

在上面介紹的 Effect 我們可以設定 Allow 或 Deny 來限制 User、Group、Role 對資源操作的權限，但是有時候我們可能會需要更多的條件來限制這個 Policy，這時候就可以使用 Condition 來設定。

Condition：它可用於 Policy 生效時指定條件，Condition 元素是選填的。

這個功能等於是加了一個 `if` 的判斷，以及 Statement 可以有很多個 Sub-Statement，每個 Sub-Statement 都可以設定 Condition，就可以設定很多種可能。

它的格式是：

```
"Condition": {
   "<condition-operator>": {
      "<condition-key>": "<condition-value>"
   }
}
```

會有三個元素組成：分別是 condition-operator、condition-key、condition-value。(condition-value 就是一般要判斷的資料，所以這邊就不多做介紹)

<br>

#### condition-operator

condition-operator：是一個比較運算子，例如：

- 字串的有：

| 運算子                    | 說明                                                               |
| ------------------------- | ------------------------------------------------------------------ |
| StringEquals              | 檢查 condition-key 跟 condition-value 字串是否相等                 |
| StringNotEquals           | 檢查 condition-key 跟 condition-value 字串是否不相等               |
| StringEqualsIgnoreCase    | 檢查 condition-key 跟 condition-value 字串是否相等，不區分大小寫   |
| StringNotEqualsIgnoreCase | 檢查 condition-key 跟 condition-value 字串是否不相等，不區分大小寫 |

由於有些 condition-key 沒有區分大小寫，所以運算子會有 `IgnoreCase` 的不區分大小寫的運算子，在設計的時候要注意。

<br>

- 數值的有：

| 運算子                | 說明                                                 |
| --------------------- | ---------------------------------------------------- |
| NumericEquals         | 檢查 condition-key 跟 condition-value 數值是否相等   |
| NumericNotEquals      | 檢查 condition-key 跟 condition-value 數值是否不相等 |
| NumericLessThan       | 檢查 condition-key 是否小於 condition-value 數值     |
| NumericLessThanEquals | 檢查 condition-key 是否小於等於 condition-value 數值 |
| NumericGreaterThan    | 檢查 condition-key 是否大於 condition-value 數值     |

<br>

還有像是日期條件運算子、布林值條件運算子、二進位條件運算子、IP 地址條件運算子等等，由於運算子蠻多的，可以參考：[IAM JSON 原則元素:條件運算子](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)。

<br>

#### condition-key

那 condition-key 是什麼呢？它是一個 key，用來指定要檢查的條件，例如：`aws:username`、`aws:MultiFactorAuthAge`、`s3:authType`、`eks:namespaces` 等等。

在 condition-key 又分為兩種類型：分別是：

1. Global Condition Key

   這些是 AWS 提供的全域條件，可以在所有的服務或是 Action 中使用，例如：`aws:username`、`aws:userid` 等等。但是有些特定狀況下才可以使用，例如：`aws:MultiFactorAuthAge`，詳細可以參考：[AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html)

<br>

2. Service Condition Key

   這些是服務提供的條件，不同服務的 Service 也會有所不同，例如：`s3:authType`、`eks:namespaces` 等等。

   想要知道哪些服務有什麼 Condition Key，可以再次開啟 [武功秘籍](#武功秘籍) 提到的 [Actions, resources, and condition keys for AWS services](https://docs.aws.amazon.com/zh_tw/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html)，我們假設選擇 Amazon Elastic Kubernetes Service (EKS) 服務：

   <br>

   因為每個 Action 都有對應的 Condition Key，不是任何一個 Action 都與任一個 Condition Key 有搭配，如下圖：

<br>

{{< figure src="/aws/iam-introduce/4.png" width="950" caption="[Actions defined by Amazon Elastic Kubernetes Service](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonelastickubernetesservice.html#amazonelastickubernetesservice-actions-as-permissions)" >}}

<br>

所以需要查看文件裡面的清單，例如像是 AssociateAccessPolicy Action 就會有 `eks:policyArn`、`eks:namespaces`、`eks:accessScope` 的 Condition Key。

<br>

另外，想要知道 Condition Key 對應到哪個資料類型，也可以參考 Condition keys for Amazon Elastic Kubernetes Service，後面會告訴你哪個 Condition Key 的資料類型，方便找到對應的 Condition Operator。

<br>

{{< figure src="/aws/iam-introduce/5.png" width="950" caption="[Condition keys for Amazon Elastic Kubernetes Service](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonelastickubernetesservice.html#amazonelastickubernetesservice-policy-keys)" >}}

<br>

#### Condition 規範

<br>

1. Condition 可以在 (condition-operator) 有多個條件 (condition-key)，每個條件都是 `and` 來相互關聯，例如：

```json
{
   (省略其他元素)

  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/department": "finance",
      "aws:PrincipalTag/role": "audit"
    }
  }
}
```

代表的是 `aws:PrincipalTag/department` 跟 `aws:PrincipalTag/role` 都要符合才會生效。

<br>

2. 如果同一個條件有多個值，每個值都是 `or` 來相互關聯，例如：

```json
{
   (省略其他元素)

  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/department": [
        "finance",
        "hr",
        "legal"
      ],
      "aws:PrincipalTag/role": [
        "audit",
        "security"
      ]
    }
  }
}
```

<br>

官方的圖片解釋的更清楚：

{{< figure src="/aws/iam-introduce/6.png" width="400" caption="" >}}

<br>

3. Condition 的 condition-operator 不能重複，例如：

```json
{
   (省略其他元素)

  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/department": "finance"
    },
    "StringEquals": {
      "aws:PrincipalTag/role": "audit"
    }
  }
}
```

這樣是錯誤的寫法，因為 `StringEquals` 出現兩次，要將兩個條件合併成一個條件，例如：

```json
{
   (省略其他元素)

  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/department": "finance",
      "aws:PrincipalTag/role": "audit"
    }
  }
}
```

詳細可以參考：[IAMJSON 政策元素：Condition](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_elements_condition.html)

<br>

## IAM Policy 範例

那了解了 IAM Policy 的每個結構以及功能，我下面就舉例兩個 IAM Policy 來讓大家複習看看。

<br>

### 範例一

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:CreateVolume",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "ec2:CreateTags",
      "Resource": "arn:aws:ec2:::volume/*",
      "Condition": {
        "StringLike": {
          "aws:RequestTag/project": "*"
        }
      }
    },
    {
      "Effect": "Deny",
      "Action": "ec2:CreateTags",
      "Resource": "arn:aws:ec2:::volume/*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/environment": "production"
        }
      }
    }
  ]
}
```

<br>

這個 Policy 有 3 個 Sub-Statement，分別是：

1. 允許對所有的 EC2 instance 創建 Volume。
2. 允許有 `aws:RequestTag/project` 的 Tag 對所有的 EC2 Volume 創建 Tag。
3. 拒絕有 `aws:ResourceTag/environment` 是 production 的 Tag 對所有的 EC2 Volume 創建 Tag。

<br>

### 範例二

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:RunTask",
        "ecs:StartTask"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/environment": [
            "production",
            "prod-backup"
          ]
        },
        "ArnEquals": {
          "ecs:cluster": [
            "arn:aws:ecs:us-east-1:111122223333:cluster/default1",
            "arn:aws:ecs:us-east-1:111122223333:cluster/default2"
          ]
        }
      }
    }
  ]
}
```

<br>

這個 Policy 有 1 個 Sub-Statement：

允許在 `arn:aws:ecs:us-east-1:111122223333:cluster/default1` 和 `arn:aws:ecs:us-east-1:111122223333:cluster/default2` 這兩個 ECS 集群上，對 `aws:RequestTag/environment` 標籤為 `production` 或 `prod-backup` 的任務執行 RunTask 和 StartTask 操作。

<br>

範例參考：[單值內容索引鍵政策範例](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/reference_policies_condition_examples-single-valued-context-keys.html)

<br>

## 參考資料

[AWS IAM] 學習重點節錄(2) - IAM Policy：[https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/](https://godleon.github.io/blog/AWS/learn-AWS-IAM-2-policy/)

Actions, resources, and condition keys for AWS services：[https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html)

其餘對應的參考資料都直接附在文章中。
