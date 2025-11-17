---
title: "統一機密、身份與加密的管理系統 - HashiCorp Vault 介紹與說明"
type: docs
weight: 9999
description: 統一機密、身份與加密的管理系統 - HashiCorp Vault 介紹與說明
images:
  - security/vault-introduce/og.webp
date: 2025-11-17
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
tags:
  - Security
  - Vault
  - HashiCorp
  - HashiCorp Vault
  - Cloud Provider
  - Cloud
  - AWS
  - GCP
  - Azure
  - DevOps
  - SRE
---

我們在現代的開發或是雲端管理，會有很多的 API 金鑰、Token、密碼、憑證 讓服務或是資源來去做驗證與授權，但有什麼工具能夠更好的存放這些金鑰呢？因此有了今天想要學習研究的主題 - HashiCorp Vault

<br>

學習一個東西最快的方式，就我而言就是先看影片 😆
這邊附上我一開始學習的參考影片 [HashiCorp Vault on AWS & K8s - 雲端地端通吃的私鑰管理平台 【Webinar： HashiCorp Vault】| 歐立威科技](https://www.youtube.com/watch?v=YU_rQz8emoE)

可以先透過影片了解 Vault 的功用，他是怎麼去做驗證，以及他怎麼儲存資料，或是後面需要怎麼監控以及備份等初步的介紹，那看完影片後，我們就可以來看官方的介紹

<br>

## 什麼是 Vault

Vault 官網介紹：（以下機翻）

- Vault 是一個基於身分的金鑰和加密管理系統，它集中管理金鑰，輪換舊憑證，按照需求產生憑證，稽核客戶端的交互行為，並支援監管合規性。

- 它提供加密服務，並透過身份驗證和授權方法進行控制，以確保對機密資訊的安全、可稽核和受限存取。

- 金鑰是指任何需要嚴格控制存取權限的訊息，例如令牌、API 金鑰、密碼、加密金鑰或憑證。 Vault 提供了一個統一的金鑰管理介面，同時提供嚴格的存取控制並記錄詳細的稽核日誌。

- Vault 會在向客戶端（使用者、機器、應用程式）提供對金鑰或儲存的敏感資料的存取權之前，對其進行驗證和授權。

{{< callout type="success" >}}
白話文就是：Vault 可以統一金鑰管理以及身份驗證(包含與機器、使用者、應用程式)，設計不同的路徑 Policy，並且有稽核紀錄跟監控的管理系統
{{< /callout >}}

<br>

{{< figure src="/security/vault-introduce/how-vault-works.webp" width="800" caption="Vault 的概述介紹圖" >}}

<br>

### 工作原理

Vault 的核心工作流程有四個階段：

- Authenticate (身份驗證)：客戶端提供訊息，Vault 會使用這些資訊來確認客戶端是否與其聲稱的身份一致，驗證過後，系統會產生一個 Token 並將其與 Policy 關聯 (Policy 後面會提到)

- Validation (驗證)：Vault 會根據第三方可信任來源驗證客戶端，例如：AWS、GCP、K8s、Github、LDAP 等

- Authorize (授權)：Vault 會將客戶端與 Vault 安全性原則進行比對。定義客戶端可以使用哪些 Valut Token 存取哪些 API Endpoint。提供一種聲明式的方式來授予或是禁止對 Vault 特定路徑的操作權限

- Access (存取權限)：Vault 透過與客戶端身份關聯的 Policy 頒發 Token 並授予對金鑰、機密資訊和加密功能的存取權限

<br>

{{< figure src="/security/vault-introduce/vault-workflow-diagram.webp" width="600" caption="Vault 的概述介紹圖" >}}

<br>

### 安裝 Valut

我們大概知道 Vault 它的用途，那接下來當然就是安裝起來玩玩看

可以使用以下指令安裝 Vault：

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

其他作業系統可以參考此篇文件：[https://developer.hashicorp.com/vault/install](https://developer.hashicorp.com/vault/install)

<br>

## 使用 Dev 開發模式來測試 Valut

由於我們還不熟悉 Vault 元件，所以我們先用 DEV 模式來測試以下內容：

(這邊會以維運角度來說明，如果開發者請查閱[開發者快速入門](https://developer.hashicorp.com/vault/docs/get-started/developer-qs)，來讀取相關 Vault secret)

- [啟動 DEV Vault](#%e5%95%9f%e5%8b%95-dev-vault)
- [啟用身份驗證插件](#啟用身份驗證插件)
- [啟用鍵/值密鑰插件(KV)](#%e5%95%9f%e7%94%a8%e9%8d%b5%e5%80%bc%e5%af%86%e9%91%b0%e6%8f%92%e4%bb%b6kv)
- [儲存第一個機密](#儲存第一個機密)
- [設計保護機密資訊的 Policy](#設計保護機密資訊的-policy)
- [測試及驗證 Policy](#測試及驗證-policy)

<br>

### 啟動 DEV Vault

安裝好 Vault 後，我們可以在本機啟動 Vault 來測試 (當然 Vault 之後正式環境不會在本機執行，這個我們之後再說)，啟動指令可以參考：

```shell
vault server -dev
```

或是你想要自定義 root token 的 ID 就可以多帶 `-dev-root-token-id="xxxx"`

<br>

啟動後，可以發現跑了一堆東西

<br>

{{< figure src="/security/vault-introduce/1.webp" width="700" caption="啟動 Vault" >}}

<br>


最後有一個 WARNING 提示，告訴我們現在啟用的是開發者模式，且這個模式下 Vault 都運行在記憶體中，並用單一解封金鑰且自動幫我們解封，Root Token 會在 CLI 自動完成驗證

<br>

{{< figure src="/security/vault-introduce/2.webp" width="800" caption="Vault DEV 環境提示" >}}

<br>

那這邊有提到解封還有解封金鑰，我們後面會提到，所以這邊先跳過，大家先知道有這個東西就可以了，在上面圖片裡面也有所謂的解封金鑰 `L7+kfL/Of/MW846XMFL7riPX5oyoW1SwZVk9T++i8A0=`，以及 Root Token `hvs.cCF4zQJ4bRwYEzouk621CFJr`

<br>

我們接下來要連線到該 Vault Server，先另開一個新的 Terminal，然後下 `export VAULT_ADDR='http://127.0.0.1:8200'` 指定好 Vault Server Address，再使用以下指令登入 Valut

```shell
vault login hvs.cCF4zQJ4bRwYEzouk621CFJr
```
後面就直接帶入 Root Token，如果不想要 export VAULT_ADDR，也可以在後面帶入 `-address=` 參數

<br>

{{< figure src="/security/vault-introduce/3.webp" width="650" caption="登入成功" >}}

<br>

可以發現登入後，他會幫我們寫入 Token 到本機的 `$HOME/.vault-token`，所以後續操作才不需要重複登入，也這個建議只在開發環境使用，不然還是會有資安疑慮

<br>

另外 vault 也有 UI 可以使用，可以訪問 `http://127.0.0.1:8200/ui`，選擇 Token 並帶入剛剛的 Root Token 即可登入，那我們後續說明基本上都會以 CLI 方式來說明，大家可以搭配 UI 來輔助查看設定內容

<br>

{{< figure src="/security/vault-introduce/4.webp" width="1000" caption="Vault UI 介面" >}}

<br>

<br>

### 啟用身份驗證插件

那在 Vault 裡面支援了很多種的身份驗證機制，它控制使用者 / 工作機器對 Vault 的存取權限，例如 AppRole、GCP、AWS、GitHub 等等，詳細可以看：[Auth methods](https://developer.hashicorp.com/vault/docs/auth)

那上面每一種方法都需要被啟用才可以使用，那我們這邊先以最簡單的方式來驗證，也就是 username + password (userpass)，啟動指令：

```shell
vault auth enable userpass
```

<br>

{{< figure src="/security/vault-introduce/5.webp" width="800" caption="啟動 auth methods userpass" >}}

<br>

啟動完成後，我們馬上來建立一個名為 pin-yi 的 User，密碼為 `123qwe`，使用以下指令建立：

```shell
vault write auth/userpass/users/pin-yi password=123qwe
```

<br>

也可以建立一個給機器或應用程式用戶端的 approle 驗證方法，使用以下指令啟動及建立：

```shell
vault auth enable approle

vault write auth/approle/role/demo-app-role \
    secret_id_ttl=10m \
    token_ttl=20m \
    token_max_ttl=30m
```

- secret_id_ttl：定義 SecretID 的存活時間。超過時間後該 SecretID 作廢，不能再用於換取 Token。

- token_ttl：定義以此 Role 生成的 Token 的初始有效時間。Token 在到期前若未續租即失效。

- token_max_ttl：定義 Token 能被續租的最長總生存時間。即便不斷續租，只要超過此上限，Token 必定失效。

<br>

{{< figure src="/security/vault-introduce/6.webp" width="800" caption="建立 userpass user & 啟動 approle 及建立 demo-app-role" >}}

<br>

### 啟用鍵/值密鑰插件(KV)

在 Vault 支援很多[密鑰引擎](https://developer.hashicorp.com/vault/docs/secrets)，例如：Key Vaule (KV)、PKI (Certificates)、LDAP、GCP、AWS、SSH 等，依照需求選用對應的引擎來管理機密

那我們現在使用 Key/Vaule (KV) 來測試，使用以下指令啟動該插件：

```shell
vault secrets enable -path demo -version 2 kv
```
- path 可以指定儲存的路徑(用來設計區分)
- version 在 KV 有版本 1 跟 2

<br>

{{< figure src="/security/vault-introduce/7.webp" width="800" caption="UI 顯示建立好的 secrets 使用 kv version 2 引擎" >}}

<br>

### 儲存第一個機密

我們已經有儲存機密的地方，路徑是 demo，那接下來我們就來儲存我們第一個機密吧，使用以下指令：

```shell
vault kv put \
  -mount demo \
  kv/uuid \
  id=1998c2d4-f0d8-465e-8a46-db6d3607e72e id=34d1802b-5adb-459c-bb3a-2f094084a6d1
```

儲存完後，會顯示它儲存在哪一個路徑，以及相關的資訊，結果不小心手滑，多儲存了幾次，剛好可以觀察一下他的版本控制 😗

<br>

{{< figure src="/security/vault-introduce/8.webp" width="650" caption="可以觀察 Version 數字不同" >}}

<br>

{{< figure src="/security/vault-introduce/9.webp" width="800" caption="使用 UI 到對應路徑查看儲存的 Secret" >}}

<br>

除此指外，我們再額外寫一個 api-keys 的機密，等等會使用到

```shell
vault kv put \
  -mount demo \
  kv/api-keys \
  api_key=9f3b12c7d84a4e0aab6e2d914f52c19d
```

<br>

### 設計保護機密資訊的 Policy

我們已經快完成初步的流程了，我們有能夠登入 Vault 的 Token，也用 userpass 建立了 pin-yi 以及 approle 的 demo-app-role，也新增了兩個機密，那我們現在就剩下要寫 Policy 來定義：這個路徑的權限是什麼，最後掛到對應的 auth 上面

首先，我們先用以下指令來建立，對了，目前只是為了更方便的測試跟學習，所以都使用 CLI，但在實際使用，強烈推薦使用 Terraform 等 IaC 工具來整理這些內容，並搭配 git flow 來完成更新 Vaule 的設定以及操作

```shell
vault policy write kv-access-policy - << EOF
path "demo/data/kv/uuid" {
   capabilities = ["read", "create", "update", "delete"]
}
EOF
```

並將該 Policy 設定到透過 userpass 的 pin-yi 上面

```shell
vault write auth/userpass/users/pin-yi \
    policies=kv-access-policy
```

<br>

{{< figure src="/security/vault-introduce/10.webp" width="650" caption="建立 Policy & 設定 Policy 到 userpass 的 pin-yi user" >}}

<br>

### 測試及驗證 Policy

耶，到了最後的驗證環節，我們要來測試看看，上面設定以及新增的內容是否可以正常運作

- 測試用 userpass 登入

```
vault login -method=userpass username=pin-yi
```

<br>

{{< figure src="/security/vault-introduce/11.webp" width="900" caption="測試登入" >}}

<br>

可以看到，我已經可以改用 userpass method 來進行登入，登入完成後，也會更新 `$HOME/.vault-token`，也可以改用代表 pin-yi 的 Token 來登入

<br>

接下來我們就可以使用以下指令來測試 Policy

```shell
vault kv get -mount=demo kv/uuid

vault kv get -mount=demo kv/api-keys
```

<br>

{{< figure src="/security/vault-introduce/12.webp" width="650" caption="測試 Policy" >}}

<br>

可以發現，由於我們的 kv-access-policy 允許 read 這個 `demo/data/kv/uuid` 路徑，所以可以正常訪問，但 `demo/kv/api-keys` 沒有設定，就會出現 403 沒有權限

<br>

我們額外測試一下，先建立一個新的 Policy 允許 read `demo/kv/api-keys` 並附加到 auth method approle 上面再進行一次測試，使用以下指令：

(記得要先切回原本的 Root Token 登入，不然目前的 pin-yi 是沒有權限的)

```shell
vault policy write kv-approle-access-policy - << EOF
path "demo/data/kv/api-keys" {
   capabilities = ["read"]
}
EOF

vault write auth/approle/role/demo-app-role \
    policies=kv-approle-access-policy
```

<br>

{{< figure src="/security/vault-introduce/13.webp" width="650" caption="建立好新的 Policy 跟綁到 demo-app-role 上" >}}

<br>

接下來要切成 Role 來查看，但因為 Role 沒辦法像 userpass 那樣，會需要取得 role-id 以及 secret-id，最後產生 Token，以下指令：

```shell
vault read auth/approle/role/demo-app-role/role-id

vault write -f auth/approle/role/demo-app-role/secret-id

vault write auth/approle/login role_id=c073455d-7b65-b55f-9adc-dc381feae85c secret_id=d86ca004-5ac0-96a1-9e45-7d7bde5a6f96
```

<br>

{{< figure src="/security/vault-introduce/14.webp" width="900" caption="approle 取得 Token" >}}

<br>

有拿到 Token 了，我們就可以像是最一開始用 Root Token 一樣登入，並測試一下兩個 kv 讀取狀態，以下指令：

```shell
vault login hvs.CAESIKJ8sMjV51f7vCGnB6ohYQenSJm5NLO9sAnIx3H8GDg0Gh4KHGh2cy5UUkhvWG42cVNRaUNyZWw1ZkhRZHJmQUE
```

<br>

{{< figure src="/security/vault-introduce/15.webp" width="650" caption="測試 Policy" >}}

<br>

可以發現，現在就可以正常讀取 kv/api-keys，但也不能讀取 kv/uuid，因為沒有設定權限

<br>

好啦到這邊，我們的開發環境測試流程就差不多結束啦，那這邊補一些前面有挖的坑：解封還有解封金鑰是什麼？

## 解封還有解封金鑰是什麼？

我們可以從官網文件 [Seal/Unseal](https://developer.hashicorp.com/vault/docs/concepts/seal) 中得知：

當我們初始化啟動 Vault Server 時，預設會是密封狀態，這個狀態下 Vault 能夠存取實體儲存 (Storage Backend) ，但無法解密其中的任何資料

且解封是取得解密金鑰的所需明文 Root 金鑰內容。再解封之前，Vault 唯一可以執行的操作就是解封 or 檢查 Server 狀態

{{< callout type="info" >}}
那這邊先簡單小提一下 Storage Backend 是什麼，後面會再說明，在 Vaule 啟動時，可以選擇資料的存放位置，那這個 Storage Backend 有很多可以選擇， 資料會透過加密儲存到裡面
{{< /callout >}}

當 Vault 解封後，會將加密密鑰存到記憶體，因此可以正常去加解密，而 Storage Backend 永遠都是儲存密文，因此之後要備份，其實只要備份該 Storage Backend，以及將解密金鑰帶入就可以解密了，就算 Storage Backend 被駭客拿到，如果沒有解密金鑰，也無法看到裡面的內容

這邊也來提一下 Vault 他的加密演算法，預設是使用 Shamir seals，他會產生多組密鑰，在解密時，需要輸入對應門檻的密鑰才可以解密，例如它產生 5 把，會需要輸入 3 把才可以正常解密，因此需要將這幾把密鑰分開存放

<br>

{{< figure src="/security/vault-introduce/16.webp" width="650" caption="測試 Policy" >}}

<br>

那我們上面測試是用 DEV 開發環境，故只有產生一組密鑰，且已經自動解密，正式環境在服務建立初始化時，就需要先輸入對應的密鑰，若有重啟等也會因為存在記憶體的加密密鑰不見，而需要重新輸入

<br>

最後，我們拿剛剛的 DEV 環境來測試看看密封跟解封吧 (記得要再切回去 Root Token)

使用以下指令，可以加原本解封的變成密封，並查看 Server 狀態

```shell
vault operator seal

vault status

vault secrets list
```

<br>

{{< figure src="/security/vault-introduce/17.webp" width="500" caption="測試密封，觀察 Server 狀態" >}}

<br>

可以看到密封后，Server 狀態的 Sealed 會從 false 變成 true，且無法進行其他的操作了

<br>

我們使用以下指令，帶入解封密鑰來解封看看(DEV 只有一把)，並測試是否能夠 list secret

```shell
vault operator unseal L7+kfL/Of/MW846XMFL7riPX5oyoW1SwZVk9T++i8A0=

vault secrets list
```

<br>

{{< figure src="/security/vault-introduce/18.webp" width="800" caption="解封並檢查功能" >}}

<br>

可以發現就正常囉～

<br>

## Storage Backend

Vault 可以選擇不同的 Storage Backend，用來放置加密後的內容

如果需要 HA 記得 Storage Backend 也需要確認是否支援

詳細的說明可以參考[官方文件](https://developer.hashicorp.com/vault/docs/configuration/storage) or 之後有時間再來更新內容 xD

<br>

## 注意事項

Vault HA 其實讀寫還是都透過 primary 去處理

當大量的服務請求，可能會把 Server 打爆

所以需要啟用 rate limit 機制來保護 Vault，或是 app 取得 vault secret 後，在本地 cache，不要每次都去跟 vault 請求

<br>

## 參考資料

Vault Documentation：[https://developer.hashicorp.com/vault/docs](https://developer.hashicorp.com/vault/docs)

HashiCorp Vault on AWS & K8s - 雲端地端通吃的私鑰管理平台 【Webinar： HashiCorp Vault】| 歐立威科技：[https://www.youtube.com/watch?v=YU_rQz8emoE](https://www.youtube.com/watch?v=YU_rQz8emoE)

從零開始的 30 天 Hashicorp Vault Workshop：[https://ithelp.ithome.com.tw/users/20120327/ironman/6764](https://ithelp.ithome.com.tw/users/20120327/ironman/6764)