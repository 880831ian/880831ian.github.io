---
title: "如何啟用 GitLab 的 Package Registry 以及將儲存位置從伺服器改到 GCS 上"
type: docs
weight: 9992
date: 2022-12-29
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

今天接到一個案子，RD 部門之後想要使用 GitLab 的 Package Registry 功能來發布套件，且不想把它存在 GitLab 伺服器上，希望可以直接存到 GCP 的 Google Cloud Storage 上，所以才會有了此篇筆記來記錄一下整個過程。

<br>

版本資訊

- GitLab 14.10 (有部分設定會於新版本棄用，請記得確認好自己的版本是否支援)

先說一下，我們的 GitLab 是使用 docker-compose 來建置，所以後續的實作內容都會以 docker-compose 的方式來介紹。

<br>

## 啟動 GitLab 的 Package Registry

首先，我們當然要先啟動這項 Package Registry 功能，才可以再之後使用它，我們先看一下 GitLab 啟動的 docker-compose.yml 檔案：

```yaml
version: '3'

services:
  gitlab:
    image: 'gitlab/gitlab-ee:14.10.5-ee.0'
    restart: always
    container_name: gitlab
    hostname: gitlab-pid
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "50"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url '${GITLAB_DOMAIN}'
        letsencrypt['enable'] = false
        gitlab_rails['initial_root_password'] = '${GITLAB_ROOT_PASSWORD}'
        gitlab_rails['gitlab_shell_ssh_port'] = '${GITLAB_HOST_SSH_PORT}'
        gitlab_rails['backup_keep_time'] = 79200
        gitlab_rails['omniauth_allow_single_sign_on'] = ['google_oauth2']
        gitlab_rails['omniauth_block_auto_created_users'] = false
        gitlab_rails['omniauth_sync_profile_from_provider'] = ['google_oauth2']
        gitlab_rails['omniauth_sync_profile_attributes'] = ['name', 'email']
        gitlab_rails['omniauth_providers'] = [
          {
		略過．．．
          }
        ]
    ports:
      - '${GITLAB_HOST_SSH_PORT}:22'
      - '${GITLAB_HOST_HTTP_PORT}:80'
      - '${GITLAB_HOST_HTTPS_PORT}:443'
    volumes:
    - './config:/etc/gitlab'
    - './logs:/var/log/gitlab'
    - './data:/var/opt/gitlab'
```

有些設定有略過或是省略不寫，大家就依照自己的設定來看就好～

<br>

## 新增 Package Registry 設定

我們在上方的 `gitlab_rails['omniauth_providers'] = [ ... 略 ... ] ` 之後加上新增 Package Registry 設定內容：

```yaml
gitlab_rails['packages_enabled'] = true
gitlab_rails['packages_object_store_enabled'] = true
gitlab_rails['packages_object_store_remote_directory'] = "GCS 名稱"
gitlab_rails['packages_object_store_direct_upload'] = true
gitlab_rails['packages_object_store_background_upload'] = true
gitlab_rails['packages_object_store_proxy_download'] = true
gitlab_rails['packages_object_store_connection'] = {
'provider' => 'Google',
'google_project' => '專案 ID',
'google_json_key_location' => '/etc/gitlab/google_key.json'
}
```

- packages_enabled：啟動 packages
- packages_object_store_enabled：啟動 packages 對象存儲
- packages_object_store_remote_directory：設定 packages 對象存儲位置，這邊要輸入 GCS 的名稱
- packages_object_store_direct_upload：設定是否可以直接上傳到對象存儲位置
- packages_object_store_background_upload：設定是否以後台方式上傳到對象存儲位置
- packages_object_store_proxy_download：設定是否可以透過代理伺服器進行套件下載
- packages_object_store_connection：設定連接到對象存儲，由於我們要存到 GCS 上面，需要有這三項 provider、google_project、google_json_key_location 才可以將 packages 存到 GCS 上。如果想用其他的儲存位置，例如 Amazon S3、Azure Blob storage 可以參考 [Object storage 詳細設定](https://docs.gitlab.com/ee/administration/object_storage.html) ( 其中的 google_json_key_location 是要放可以讀寫 GCS 的 SA SECRET 檔案 )

<br>

## 先查看尚未重啟的 GitLab Package

由於公司 GitLab 預設有先開啟 packages_enabled，所以我就拿同事用 Helm 寫的 CI，來做測試。當更新 value.yaml 後會自動打包 Package 放到 Package Registry 中，我們直接進入到預設 Package Registry 的儲存位置，是在 `/var/opt/gitlab/gitlab-rails/shared/packages/`，用指令發現打包的 Package 的確存放於此 ，如下：

<br>

{{< figure src="/git-or-cicd/gitlab-package-registry-to-gcs/1.webp" width="800" caption="檢查是否還有 Package 在預設儲存位置 (尚未遷移)" >}}

<br>

## 重啟設定後再次檢查 GitLab Package

當我們重啟設定後，也有建立好可供我們權限 SA 的 GCS 後，會發現原本存在預設 `/var/opt/gitlab/gitlab-rails/shared/packages/` 沒有自動跑到 GCS 上，是因為我們還需要手動下指令將他遷移過去，指令是 `gitlab-rake "gitlab:packages:migrate"`，最後等他跑完我們在檢查一下預設儲存位置就發現已經沒有 Package 了

<br>

{{< figure src="/git-or-cicd/gitlab-package-registry-to-gcs/2.webp" width="800" caption="檢查是否還有 Package 在預設儲存位置  (已遷移)" >}}

<br>

開 GCS 網站來看會發現原先在預設儲存位置的 Package 都可以跑到 GCS 上：

<br>

{{< figure src="/git-or-cicd/gitlab-package-registry-to-gcs/3.webp" width="800" caption="查看已遷移到 Google Cloud Storage 的 Package" >}}

<br>

## 參考資料

GitLab Package Registry administration：[https://docs.gitlab.com/14.10/ee/administration/packages/](https://docs.gitlab.com/14.10/ee/administration/packages/)
