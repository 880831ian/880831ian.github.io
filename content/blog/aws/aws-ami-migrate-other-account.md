---
title: "AWS Amazon Machine Images (AMIs) 轉移給其他 AWS 帳號"
type: docs
weight: 9983
description: AWS Amazon Machine Images (AMIs) 轉移給其他 AWS 帳號
images:
  - aws/aws-ami-migrate-other-account/og.webp
date: 2025-02-06
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

想要研究是否可以把已經包好的 AWS AMI 轉移到其他 AWS 帳號使用，故有此篇文章來說明如何操作。

<br>

> [!NOTE] 下方會有兩個不同的 AWS 帳號來說明
>分別是：A 帳號 6517xxxxxx02，會負責產生 AMI 並將 AMI 分享給 B 帳號 ; B 帳號 1205xxxxxx13，則是接收 AMI 的帳號

<br>

1. 我們先到 A 帳號 (6517xxxxxx02) 上，選擇要的 EC2 Instances 建立 AMI，確認一下建立的 AMI 以及 Snapshots 是否都 Available 或是 Completed。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/1.webp" width="1200" caption="確認 A 帳號的 AMI 和 Snapshots" >}}

<br>

> [!IMPORTANT] 注意
這邊說明一下，AWS 的 AMI 是基於 EC2 的 Snapshots 來創建的。AMI 實際上是包含了一組快照的元數據，這些快照代表了你 EC2 實例的磁碟（包括根磁碟和附加磁碟）的完整映像。這樣，當你創建 AMI 時，它會捕獲這些磁碟的狀態和數據，並用於後續的還原或啟動新的實例。<br><br>
>請記得在一開始建立 EC2 選擇 Volumes 時，如果選擇 Encryption，則無法跨帳號分享加密 AMI (除非也把 KMS 也分享)

<br>

2. 點擊要分享的 AMI，按下右鍵，選擇 Edit AMI permissions > 找到 Shared accounts，按下 Add accout ID，輸入接收方 B 帳號 (1205xxxxxx13)，最後點儲存變更。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/2.webp" width="1200" caption="分享 AMI 的設定畫面" >}}

<br>

3. 就會顯示 `Successfully updated permissions for <ami id>`，接著我們切換到 B 帳號 (1205xxxxxx13) 的 AMI，就可以看到它囉，如果沒看到，記得切換成 Private image (預設會是 Owned by me)

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/3.webp" width="700" caption="確認 B 帳號的 AMI" >}}

<br>

4. 接著點擊 demo-shard-ami 這個從 A 帳號 (6517xxxxxx02) 來的 AMI (可以從後面 Source 看到) 右鍵，選擇 Copy AMI，就可以來複製 AMI，可以針對自己需求，調整相關名稱、描述等。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/4.webp" width="1200" caption="複製 AMI 的設定畫面" >}}

<br>

5. 但當你點擊完後，會發現出現這個錯誤，它說沒有權限來儲存這個 AMI，那是為什麼呢？還記得上面說的 AMI 它還需要 Snapshots 搭配使用，所以我們也需要再將 Snapshots 分享給 B 帳號 (1205xxxxxx13)，才可以正常複製歐。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/5.webp" width="1200" caption="複製 AMI 出現權限錯誤" >}}

<br>

6. 那我們再切回去 A 帳號 (6517xxxxxx02) 的 Snapshots，點擊右鍵選擇 Snapshot settings > 選擇 Modify permissions。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/6.webp" width="850" caption="設定 Snapshots 的分享權限" >}}

<br>

7. 就跟分享 AMI 一樣，輸入要分享的 B 帳號 (1205xxxxxx13)，按下 Modify permissions，回到 B 帳號 (1205xxxxxx13) 確認是否出現此 Snapshots。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/7.webp" width="1200" caption="設定 Snapshots 的分享權限" >}}

<br>

8. 確認有後，我們就重複上面的第四點，我 AMI 複製的名稱取為 demo-share-ami-b-accout-copy，按下儲存，就可以看到複製成功囉。當然同樣的，也會同時建立新的 Snapshots。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/8.webp" width="1200" caption="測試複製 AMI 是否正常" >}}

<br>

---

<br>

我們來測試一下，如果從 A 帳號 (6517xxxxxx02) 把分享的 AMI 移除，會發生什麼事情？一樣可以參考第四點，只是變成將分享的帳號給 Remove 掉，就可以發現 B 帳號 (1205xxxxxx13) 的 AMI 不見了

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/9.webp" width="1200" caption="B 帳號的 AMI 沒有看到 A 帳號分享的 AMI" >}}

<br>

所以這也代表，如果代理商將帳號收回，原本分享的 AMI 就會失效，後面就不能用它來建立新的 EC2，但如果是已經啟動當中的 EC2，則不會受到影響。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/10.webp" width="900" caption="已啟動的 EC2，不受影響" >}}

<br>

那如果把 A 帳號 (6517xxxxxx02) 把分享的 Snapshots 移除，會發生什麼事情？一樣可參考第七點，將分享的帳號給 Remove 掉，回到 B 帳號 (1205xxxxxx13) 查看，Snapshots 當下還會存在一段時間，過一陣子就才不見了，推測應該跟 Volume Size 有關係。

<br>

{{< figure src="/aws/aws-ami-migrate-other-account/11.webp" width="900" caption="移除 Snapshots 會過一段時間才消失" >}}

<br>