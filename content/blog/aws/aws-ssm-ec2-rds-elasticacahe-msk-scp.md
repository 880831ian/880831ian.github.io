---
title: "AWS SSM 連線到 EC2、打通 RDS & Elasticache & MSK 到本機、使用 SCP 傳送檔案/拉取遠端檔案到本機"
type: docs
weight: 9977
description: AWS SSM 連線到 EC2、打通 RDS & Elasticache & MSK 到本機、使用 SCP 傳送檔案/拉取遠端檔案到本機
images:
  - aws/aws-ssm-ec2-rds-elasticacahe-msk-scp/og.webp
date: 2025-07-22
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - AWS
  - SSM
  - Systems Manager Agent
  - EC2
  - RDS
  - Elasticache
  - MSK
  - SCP
---

AWS Systems Manager Agent (SSM Agent) 是 Amazon 軟體，在 Amazon Elastic Compute Cloud (Amazon EC2) 執行個體、邊緣裝置、內部部署伺服器和虛擬機器上執行

<br>

SSM 就跟 GCP 的 IAP 類似，可以直接連線到 EC2 內，不需要在使用 SSH 等方式連線，那使用 SSM 有幾個前提：

1. 需要先安裝 AWS CLI，以及相關的 configure 設定，詳細可以參考：
[AWS SSO Profile 設定介紹](../aws-sso-profile-introduce)、[手動切 AWS Profile 太麻煩？試試 Granted，多帳號管理神器助你效率倍增！](../aws-assuming-roles-tool-introduce)

2. 節點必須為受管節點，這表示機器上已安裝 SSM Agent，且代理程式可以與 Systems Manager 服務進行通訊。(大多數的 AMI 都有預先安裝 SSM Agnet，詳細可以參考：[尋找預先安裝了 SSM Agent的 AMIs - AWS Systems Manager](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/ami-preinstalled-agent.html) )

3. 需要有對應的權限，這邊的權限有兩種：一個是操作的 IAM User，以及被連線的 EC2 IAM Role。操作的 IAM User 需要有 `ssm:StartSession` 相關權限，EC2 IAM Role 則是需要被賦予 `AmazonSSMManagedInstanceCore`。

4. 電腦本機有需要安裝 Session Manager plugin，詳細安裝步驟請參考：[Install the Session Manager plugin on macOS - AWS Systems Manager ](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-macos-overview.html)(如果使用下方腳本，會自動檢查是否安裝，沒有也會自動安裝)，如果沒有裝此 plugin，就會出現權限錯誤提示 (`because no identity-based policy allows the ssm:TerminateSession action`)。

<br>

SSM 除了連線到 EC2 以外，還有其他的功能，可以透過 EC2 去連線 AWS 上的其他資源，例如：RDS 跟 Elasticache。在之後會建議都會透過 EC2 當作跳板機去連線，讓 RD 自行於本地選擇工具來操作，而不會直接將這些工具建立在線上服務。

<br>

- 連線到對應的 EC2 instances

```bash
aws ssm start-session --target "${instance_id}"
```

<br>

- 連線 EC2 後，進到 root / 目錄

```bash
aws ssm start-session --target "${instance_id}" --document-name "AWS-StartInteractiveCommand" --parameters '{"command":["sudo -i;cd /"]}'
```

<br>

- 透過 EC2 連線到 RDS or Elasticache or MSK

```bash
aws ssm start-session \
  --target "i-0ad1b308da085aaa4" \
  --document-name "AWS-StartPortForwardingSessionToRemoteHost" \
  --parameters "{\"host\":[\"${host}\"], \"portNumber\":[\"${service_port}\"], \"localPortNumber\":[\"${local_port}\"]}"
```

`${host}`：帶入 Elasticache Route53 private-dns 網址

`${service_port}`：如果是 RDS 預設是 3306，如果是 Elasticache 預設是 6379

`${local_port}`：本機使用的 Port，只要不衝突就可以使用，在使用自己習慣的工具去連線 `127.0.0.1:${local_port}`

<br>

如果連線上有錯誤，可能會是對應的 RDS or Elasticache SG 沒有開給該跳板機

MSK 由於其架構較為特殊，會額外安裝 kafka-proxy 來代理連線到叢集，詳細可以參考：[GitHub - grepplabs/kafka-proxy: Proxy connections to Kafka cluster. Connect through SOCKS Proxy, HTTP Proxy or to cluster running in Kubernetes.](https://github.com/grepplabs/kafka-proxy)

(建議使用下方腳本進行操作，下方腳本會連 SSM + kafka-proxy 都ㄧ併完成)

<br>

- 將本機檔案傳送到 EC2 / 從 EC2 複製檔案到本機

這個需求在傳大量檔案的時候會很方便，但由於 SSM 沒有自己的傳送檔案相關指令，所以我們統一使用 SCP 來傳送傳送檔案。

使用 SCP 則會需要先設定 KEY 到 EC2 機器上，這樣會有點麻煩，因此參考 AWS 官方文件，我們會使用 `aws ec2-instance-connect send-ssh-public-key` 指令來將 public-key 傳送到 EC2 上，可以快速的省下先 SSM 進去機器，設定好 KEY，也因為透過 `send-ssh-public-key` 只有 60s，所以不會有 `authorized_keys` 不好管理的問題，詳細說明可以參考：[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html#ec2-instance-connect-connecting-aws-cli](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html#ec2-instance-connect-connecting-aws-cli)、[步驟 8：(選用) 透過 Session Manager 允許和控制 SSH 連線的許可 - AWS Systems Manager](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html)

<br>

1. 需要額外新增以下的 SSH 設定檔，讓 SCP 可以透過 SSM 執行

SSH 組態檔案通常位於 ~/.ssh/config

```shell
# SSH over Session Manager
Host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```
(如果使用下方腳本，會自動檢查是否設定，沒有也會自動設定)

<br>

2. 建立 SSH 公私金鑰檔案，並調整權限

```shell
ssh-keygen -f ssm
```

(ssm 為範例，會產生 ssm 跟 ssm.pub)

<br>

3. 使用 `send-ssh-public-key` 將 SSH 公鑰推到 [instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)，只會保留 60s，會需要 `ec2-instance-connect:SendSSHPublicKey` 權限

```shell
aws ec2-instance-connect send-ssh-public-key \
        --region "${region}" \
        --availability-zone "${zone}" \
        --instance-id "${instance_id}" \
        --instance-os-user root \
        --ssh-public-key "file://${ssh_pub_key}" \
        --no-cli-pager
```

`${region}`：EC2 機器的 Region

`${zone}`：EC2 機器的 zone

`${instance_id}`：傳送的 EC2 機器

`${ssh_pub_key}`：公鑰路徑，例如上面的 `/Users/$(whoami)/.ssh/ssm.pub`

<br>

有回傳 Success: true 就代表成功，接著就可以嘗試下 SCP 指令將檔案傳給 EC2，或是從 EC2 複製檔案出來，這邊以 SCP 將檔案傳給 EC2 為例：

```shell
scp -o "IdentitiesOnly=yes" -i ${ssh_priv_key} ${local_path} root@${instance_id}:${remote_path}
```

`${ssh_priv_key}`：私鑰路徑，需要跟前面傳送給 EC2 是同一把

`${local_path}`：本地要傳送的檔案路徑

`${instance_id}`：傳送的 EC2 機器

`${remote_path}`：EC2 機器的檔案路徑

<br>

## 執行腳本

為了方便大家快速執行，有將上面提到的指令寫成腳本，讓大家可以用選擇及輸入的方式來使用

腳本連結：[https://github.com/880831ian/aws-ssm-ec2-rds-elasticacahe-msk-scp](https://github.com/880831ian/aws-ssm-ec2-rds-elasticacahe-msk-scp)

<br>

{{< figure src="/aws/aws-ssm-ec2-rds-elasticacahe-msk-scp/1.webp" width="450" caption="腳本使用" >}}

<br>

MSK 比較特別，會額外安裝套件

<br>

{{< figure src="/aws/aws-ssm-ec2-rds-elasticacahe-msk-scp/2.webp" width="1200" caption="MSK 安裝詢問" >}}

<br>

輸入對應的 Broker 清單，會自動轉發

<br>

{{< figure src="/aws/aws-ssm-ec2-rds-elasticacahe-msk-scp/3.webp" width="1000" caption="轉發 Broker" >}}

<br>

會跳出能在本機連線的 IP 以及 Port

<br>

{{< figure src="/aws/aws-ssm-ec2-rds-elasticacahe-msk-scp/4.webp" width="1000" caption="本機連線 IP" >}}

<br>

## 參考資料

[Connect to a Linux instance using EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html#ec2-instance-connect-connecting-aws-cli)

[透過 Session Manager 允許和控制 SSH 連線的許可](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html)