---
title: "Git 介紹"
type: docs
weight: 9999
description: Git 介紹
date: 2022-04-15
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 什麼是 Git ?

不管是不是工程師，只要常常需要使用電腦工作，每天一定都會新增、修改、刪除許多檔案，我們看到這張圖：

<br>

{{< figure src="/git-or-cicd/git-introduce/sample.png" width="450" caption="很多人的電腦裡面都有這樣的內容" >}}

<br>

這張圖是一個菜鳥工程師在整理檔案時的方法，因為每一天都會對這份檔案做不同的處理，但為了保留以前的版本，所以也不會刪除舊的檔案，只好用日期或是版本來做分類，時間越久，檔案就累積越多，假如不小心刪除，也找不回來紀錄，也不清楚不同檔案的差異，所以有了 Git 這項工具。

<br>

* Git 為分散式版本控制系統，是為了更好管理Linux內核而開發的。
* Git 的優點：免費開源、速度快、檔案體積小、分散式系統。
* Git 的缺點：指令繁雜，但可以透過 GUI 工具解決。
* Git 會紀錄哪些資料：更動前 vs 更動後的程式碼、修改者、修改時間、修改原因（修改者需要自行撰寫 commit message）。

<br>

我們也常常聽到 GitHub or GitLab 那跟 Git 是一樣的東西嗎？

Ans：GitHub(GitLab) 是基於 Web 的平台，結合了 Git 的版本控制功能，為開發團隊提供了儲存、分享、發布和合作開發項目的中心化雲存儲的場所。

<br>

## Git 操作指令

Git 的操作指令繁多，包含環境類、查看類、提交類、分支類、遠端類、合併類、還原類等等，所以才有了 Git GUI 工具，筆者很推薦 [Gitkraken](https://www.gitkraken.com/)，雖然需要付費，但真的很方便，畫面也很乾淨簡潔。如果是學生的話，還可以使用 [GitHub Student Developer Pack](https://education.github.com/pack) 免費使用歐！

<br>

{{< figure src="/git-or-cicd/git-introduce/gitkraken.png" width="600" caption="Git GUI 工具 (Gitkraken)" >}}

<br>

接下來我們會依照環境類、查看類、提交類、分支類、遠端類、合併類、還原類依序下去介紹～

### 環境類

使用每一個程式或工具，必須先把它安裝到自己電腦上對吧！但因為大家使用的系統都不一樣，所以這邊就不列出要怎麼進行安裝，可以參考 [Git 安裝教學](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-Git-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8)

當我們安裝好後，我們就可以一起進入 Git 的世界囉！

<br>

#### init

首先，找一個你要開始進行版本控制 (Git) 的資料夾， 使用：

```sh
$ git init
```
要記得要到版本控制的資料夾目錄下才使用這個指令歐！

<br>

使用完後，會看到跑出下面這些文字：

<br>

{{< figure src="/git-or-cicd/git-introduce/init.png" width="600" caption="Git init" >}}

<br>

此外資料夾內也會多一個隱藏檔案 `.git` ，他是用來存放 git 的紀錄，所以不要亂刪除歐：

```sh
$ tree -a -d
.
└── .git
    ├── hooks
    ├── info
    ├── objects
    │   ├── info
    │   └── pack
    └── refs
        ├── heads
        └── tags
```
<br>

回到剛剛圖片，它說明默認會使用 master 這個來作為初始分支，並記得要使用 `git config` 來做設定，那 `git config` 是要做什麼用的呢！？

#### config

在推送 Commit 的時候，會顯示使用者名稱以及電子郵件，所以要先在推送前設定好，這時就使用：

```
$ git config --global user.name "your name"
$ git config --global user.email "your email"
```
分別設定使用者名稱以及電子郵件，這樣共同使用版本控制的人，才分的出來誰是誰！ (若只要在單個專案下設定使用者名稱及電子郵件，就不需要設定 --global 參數)

<br>

### 查看類

#### status

我們剛剛已經設定好 config ，如果想查看檔案的 git 狀態，就使用：

```sh
$ git status
```
可以查看現在資料夾內有哪些檔案還沒加入版本控制，或是已經加入但還沒 Commit 變成新版本。

<br>

{{< figure src="/git-or-cicd/git-introduce/status.png" width="600" caption="Git status" >}}

<br>

綠色代表已經加入版本控制但還沒有 Commit ，紅色代表尚未加入追蹤。

<br>

#### log

當我們想要查看 Commit 的幾個版本，或是是誰 Commit 的等等資訊，可以使用：

```sh
$ git log
```

<br>

{{< figure src="/git-or-cicd/git-introduce/log.png" width="600" caption="Git log" >}}

<br>

可以看到一串英文加數字，它是SHA-1 校驗碼也代表你推的這一個版本的識別 ID，也可以看到是由誰推送跟時間與推送的文字說明。

<br>

#### diff

當我們假設已經 Commit 後，想要比較不同版本內容的差異，就使用：

```sh
$ git diff 116e 442c
```
116e 代表最新的版本，442c 代表上一個版本，可以看上面 `log` 的辨識 ID，因為校驗碼很長，最少需要前4個數字跟英文，才可以知道是哪一個版本。

<br>

{{< figure src="/git-or-cicd/git-introduce/diff.png" width="600" caption="Git diff" >}}

<br>

紅色代表最新版本因為我們是用 116e 最新版本來跟綠色上一個版本 442c 來做比較。

<br>

#### reflog

如果我們在操作 Git 的時候執行錯誤，需要回復到前幾個版本，但還想要查看歷史紀錄，如果我們使用前面說的 `git log` 是看不到已經舊的紀錄，這時要使用：

```sh
$ git reflog
```

<br>

{{< figure src="/git-or-cicd/git-introduce/reflog.png" width="600" caption="Git reflog (圖一)" >}}

{{< figure src="/git-or-cicd/git-introduce/log-1.png" width="600" caption="Git log (圖二)" >}}

<br>

可以看到圖一是使用 `reflog` 就可以知道我們還原的紀錄，但使用 `log` 查看，會發現沒有辦法看到 test-2 的紀錄。

<br>

### 提交類

#### add

當我們上面使用 `git init` 初始化資料夾後，還沒有開始進行版本控制，需要使用：

```sh
$ git add .
$ git add test.txt
```

<br>

將檔案加入版本控制的暫存區。它的格式是 `git add [檔案名稱]` ，如果想要把資料夾全部檔案都加入版本控制，可以使用 `.` 來加入。

<br>

#### commit

當我們新增完後，要將它提交出去，就要使用：

```sh
$ git commit -m "內容打這"
```

<br>

{{< figure src="/git-or-cicd/git-introduce/commit.png" width="600" caption="Git commit" >}}

<br>

我們可以在雙引號內輸入我們修改的內容，方便其他人了解該版本的差異。

<br>

### 分支類

#### branch

我們專案初始化後，一開始都只會有一個 master 分支，如果想要新增分支，可以使用：

```sh
$ git branch "分支名稱"
```

會將所在分支的檔案狀態複製到新增的分支上，當該分支改動時，不會影響到原本的分支。

<br>

{{< figure src="/git-or-cicd/git-introduce/branch.png" width="600" caption="Git branch" >}}

<br>

#### checkout

如果想要切換不同版本或是分支，就可以使用：

```sh
$ git checkout "分支名稱/分支ID"
```
來切換不同的分支或是以前的版本。

<br>

{{< figure src="/git-or-cicd/git-introduce/checkout_1.png" width="600" caption="Git checkout" >}}

<br>

如果想要同時建立分支並切換，可以使用：

```sh
$ git chechout -b "分支名稱"
```
`-b` 這個參數就等於是 `git branch "分支" & git checkout "分支"`。

<br>

{{< figure src="/git-or-cicd/git-introduce/checkout_2.png" width="600" caption="Git checkout -b" >}}

<br>

### 遠端類

#### clone

如果我們遠端上已經有版本庫，想要下載到本地端，就可以使用：

```sh
$ git clone [遠端網址]
```
會在下指令的當前路徑下，下載整個遠端的專案。

<br>

#### remote

如果要新增遠端版本庫，就可以使用：

```sh
$ git remote add [簡稱] [遠端網址]
```
取一個可以代表要新增的遠端 Git 版本庫簡稱。

<br>

{{< figure src="/git-or-cicd/git-introduce/remote_1.png" width="600" caption="Git remote" >}}

<br>

如果想要檢視已經設定好的遠端版本庫，就可以使用：

```sh
$ git remote
```
他會列出每個遠端本版本庫的簡稱。

也可以使用 `git remote -v` 指令，會顯示 Git 用來讀寫遠端簡稱時所用的網址：

<br>

{{< figure src="/git-or-cicd/git-introduce/remote_2.png" width="600" caption="Git remote -v" >}}

<br>

#### push

當我們已經設定好遠端版本庫的位址，我們如果想要將本地端的專案版本庫放到遠端，就可以使用：

```sh
$ git push [簡稱] [分支名稱]
```
將本地端版本庫推到遠端的版本庫。

<br>

{{< figure src="/git-or-cicd/git-introduce/push.png" width="600" caption="Git push" >}}

<br>

上面這張圖片，就是把本地端的 `master` 分支內容，推一份到 `origin` 這個地方 (可能是 GitHub 或公司內部 Git 伺服器)，並且在 `origin` 這個地方形成同名的 `master` 分支。

<br>

但很多人不知道的是，其實 push 指令的完整型態長這樣：

```sh
$ git push origin master:master
```
意思就是「把本地的 `master` 分支內容，推一份到 `origin` 上，並且在 `origin` 上建立一個 `master` 分支」

<br>

如果我們把指令改為：

```sh
$ git push origin master:dog
```
意思就會變成「把本地的 `master` 分支內容，推一份到 `origin` 上，並且在 `origin` 上建立一個 `dog` 分支」

<br>

#### pull

如果我們想要將遠端的專案 **下載並合併** 至本地端，就可以使用：

```sh
$ git pull [簡稱] [分支名稱]
```
將遠端專案資料下載並合併到本地端。

<br>

{{< figure src="/git-or-cicd/git-introduce/pull.png" width="600" caption="Git pull" >}}

<br>

#### fetch

如果我們單純只想要將遠端的專案 **下載** 至本地端，就可以使用：

```sh
$ git fetch [簡稱] [分支名稱]
```
將遠端專案資料下載到本地端。

<br>

{{< figure src="/git-or-cicd/git-introduce/fetch.png" width="600" caption="Git fetch" >}}

<br>

#### clone、pull、fetch 差異

| 差異 | clone | fetch | pull |
| :---: | :---: | :---: | :---: |
| 功能 | 會把遠端整份專案都下載到本地端 | 只會下載，並不會合併 | 會下載且合併檔案 |
| 補充說明 | 適用於專案一開始時使用，如果 clone 之後要再更新，通常是執行 `git fetch` or `git pull` | 假設我遠端叫 `orgin`，當執行時，Git 會比對本地端與遠端，會「下載 `origin` 上有但本地端沒有的檔案下來」| pull 與 fetch 做的事情差不多，多了一個進行合併的功能 |

<br>

### 合併類

#### merge

如果想要將不同分支內容合併，就可以使用：

```sh
$ git merge [分支名稱]
```
像下面這張圖，我們將分支 123 合併到分支 master。(要記得先切換到要合併的主分支，才可以合併其他的分支進來)

<br>

{{< figure src="/git-or-cicd/git-introduce/merge.png" width="600" caption="Git merge" >}}

<br>

#### merge (fast-fastward)

merge 有一個參數叫做 fast-fastward，我們先看圖片再來說明：

<br>

{{< figure src="/git-or-cicd/git-introduce/merge_1.gif" width="700" caption="Git merge fast-fastward 圖一 ([CS Visualized: Useful Git Commands](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1#merge))" >}}

<br>

{{< figure src="/git-or-cicd/git-introduce/merge_2.gif" width="700" caption="Git merge no fast-fastward 圖二 ([CS Visualized: Useful Git Commands](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1#merge))" >}}

<br>

圖一是我們的 `fast-fastward`，也是 Git 預設的合併方式，當我們要將 dev 合併到 master，`fast-fastward` 會將 dev 分支的 commit 紀錄合併到 master 上，然而圖二是不使用 `fast-fastward` 方式，會保留 dev 分支上的 commit 紀錄，並在 master 上新增一個。

<br>

`no fast-fastward` 好處是可以完整保留每個分支的 commit 紀錄，壞處是假如 commit 紀錄只有一個，合併多次就會出現很多小叉路。要怎麽使用 `no fast-fastward`：

```sh
$ git merge --no-ff [分支名稱]
```

<br>

#### rebase (合併)

如果想要重新修改特定分支的「 基礎版本 」，要把另一個分支的變更，當成我這隻分支的基礎，就可以使用：

```sh
$ git rebase [分支名稱]
```

<br>

{{< figure src="/git-or-cicd/git-introduce/rebase_1.gif" width="700" caption="Git rebase (合併) ([CS Visualized: Useful Git Commands](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1#Rebasing))" >}}

<br>

#### rebase (修改)

如果想要修改特定分支上任何一個版本資訊，就可以使用：

```sh
$ git rebase -i [HEAD~?]
```
但要記得，如果再使用前，要先詢問是否有人正在使用此分支，因為 rebase 會改變歷史紀錄。

<br>

{{< figure src="/git-or-cicd/git-introduce/rebase_2.gif" width="700" caption="Git rebase (修改) ([CS Visualized: Useful Git Commands](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1#Rebasing))" >}}

<br>

#### cherry-pick

如果想要從某個分支，拉幾個 Commit 進來該分支，就可以使用：

```sh
$ git cherry-pick [分支ID]
```
但做此動作，需要解決修改後的版本衝突。

<br>

### 還原類

#### reset

如果想要還原任意 Commit，就可以使用：

```sh
$ git reset [HEAD~?]
```
會還原選擇的 Commit，且檔案還是維持最新版本。

<br>

`reset` 指令可以搭配參數使用，常見到的三種參數，方別是 `--mixed`、`--soft`、`--hard`，不同的參數執行之後會有稍微不太一樣的結果。
* mixed 模式：`--mixed` 是預設的參數，如果沒有特別加其他參數，`got reset` 會使用 `--mix` 模式。這個模式會把暫存區的檔案丟掉，但不會影響到工作目錄的檔案，也就是說 Commit 拆出來的檔案會留在工作目錄(實體的檔案)，但不會留在暫存區。
* soft 模式：這個模式下的 reset，工作目錄跟暫存區檔案都不會被丟掉，所以看起來只有 HEAD 的移動而已。也因此，Commit 拆出來的檔案會直接放在暫存區。
* hard 模式：在這個模式下，不管是工作目錄以及暫存區的檔案都會丟掉。

以下用表格在整理一次：

| 模式 | mixed 模式 | soft 模式 | hard 模式 |
| :---: | :---: | :---: | :---: |
| 工作目錄(實體的檔案) | 不變 | 不變 | 丟掉 |
| 暫存區 | 丟掉 | 不變 | 丟掉 |

<br>

文字說明也不太懂對吧！沒錯我也是 😂，所以我整理了三種不同的範例，我們一起做看看吧！

我們先開一個新專案，在 master 上面 commit 2 次，可以參考下方圖片：

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_1.png" width="700" caption="Git reset 示範" >}}

<br>

我們用 `git log` 來看一下記錄：

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_2.png" width="700" caption="Git reset 示範 log 紀錄" >}}

<br>

都設定好後，我們要來測試每個參數的不同之處，先以預設的 `--mixed` 來測試：

##### reset - mixed

我們下 `git reset --minxed` 按 Tab 可以看要還原的 commit，我們之後的測試都是還原到 `24aeb0d  -- [HEAD^]   add a.txt` 這個，來觀察 `add b.txt` 這個 commit 的變化。

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_3.png" width="800" caption="Git reset mixed 模式" >}}

<br>

所以我們的指令是 `git reset --mixed 24aebo4`，我們再來觀看看看，檔案狀態也就是 b.txt 以及暫存區狀態。

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_4.png" width="800" caption="Git reset mixed 模式" >}}

可以看到使用 `--mixed` 模式，檔案 b.txt 還會存在，只是移除暫存區。

<br>

##### reset - soft

我們指令是 `git reset --soft 24aebo4`：

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_5.png" width="800" caption="Git reset soft 模式" >}}

可以看到使用 `--soft` 模式，檔案 b.txt 還會存在，且會在暫存區。

<br>

##### reset - hard

我們指令是 `git reset --hard 24aebo4`：

<br>

{{< figure src="/git-or-cicd/git-introduce/reset_6.png" width="800" caption="Git reset hard 模式" >}}

可以看到使用 `--hard` 模式，檔案 b.txt 不見了，所以也不會在暫存區。

<br>

#### revert

如果想要還原任意 Commit，但又想保留在歷史紀錄，就可以使用：

```sh
$ git revert [HEAD~?]
```
會還原選擇的 Commit，檔案也會還原到舊的版本

<br>

#### reset 與 revert 差異

| 指令 | reset | revert |
| :---: | :---: | :---: |
| 改變歷史狀態 | 是 | 否 |
| 說明 | 把目前狀態設定成某個指定的 Commit 狀態，通常適用於尚未推到遠端的 Commit | 新增一個 Commit 來取消另一個 Commit 的內容，原本的 Commit 依舊會保留在歷史紀錄中。通常適用於已經推到遠端的 Commit |

<br>

### 其他

#### tag

標籤是用於標記特定的點或是提交的歷史，通常會用來標記發佈版本的名稱或是編號，例如：`v1.0`。標籤看起來有點像是分支，但打上標籤的提交是固定的，不能隨意的變更位置。

Git 中有兩種標籤類型：輕量標籤(lightweight tag)和標示標籤(annotated tag)，他們有什麼區別呢？我們分別列出他們不同之處。

<br>

* 輕量標籤(lightweight tag)
	* 不可以變更的暫時標籤
	* 可以添加名稱

* 標示標籤(annotated tag)
	* 可以添加打標籤者的名稱、email、日期
	* 可以添加名稱
	* 可以添加註解
	* 可以添加簽名

<br>

一般情況下，標示標籤都會用在較為重要的提交上，如發布提交可以使用標示標籤來新增註解或簽名，另一方面，輕量標籤通常使用在本機端最為暫時性的使用或是一次性使用。

我們分別來看一下要如何新增輕量標籤(lightweight tag)以及標示標籤(annotated tag)吧！

<br>

我們先隨意在分支上推一次 commit ，如下圖，讓我們等等有 commit 可以來新增標籤：

<br>

{{< figure src="/git-or-cicd/git-introduce/tag.png" width="800" caption="在隨意的分支推一個 commit" >}}

<br>

##### 輕量標籤

使用 `tag` 且不帶其他的參數來下指令：

```sh
$ git tag lightweight bc4c597
```
lightweight 是我們 tag 名稱，bc4c597 是剛剛 commit 的 SHA-1

<br>

接著我們使用 `git show lightweight` 來查看標籤：

<br>

{{< figure src="/git-or-cicd/git-introduce/tag_1.png" width="800" caption="輕量標籤 lightweight" >}}

可以發現因為我們使用「輕量標籤」，所以沒有存任何資訊，但可以在圖片第一行最後面看到我們使用的 tag。

<br>

##### 標示標籤

我們一樣使用 `tag`，但後面可以加上 -a -m 參數：

```sh
$ git tag annotated bc4c597 -a -m "可以備註"
```
`-a` 參數是請 Git 幫我們建立有附註的標籤，後面的 `-m` 則是跟我們 commit 一樣可以來輸入訊息

<br>

接著我們使用 `git show annotated` 來查看標籤：

<br>

{{< figure src="/git-or-cicd/git-introduce/tag_2.png" width="800" caption="標示標籤 annotated" >}}

<br>

可以看到我們使用標示標籤，所以可以查看標籤是誰填寫、他的信箱、填寫時間以及他的備註內容。

<br>

官方文件對於這兩種標籤的說明：

有標示標籤主要用來做像是軟體版號之類的用途，而輕量標籤則是來於個人使用或暫時的標記用途。簡單來說，有標示標籤的好處是有更多關於這張標籤的資訊，假設不是很在乎這些資訊，使用一般的輕量標籤也是沒有問題的！

<br>

## Git 常見問題

- Git 裡的 HEAD 是什麼？

HEAD 本身是一個指標，通常會指向某個本地端分支或是其他 Commit，所以也可以把 HEAD 當作目前所在的分支。

<br>

- 刪除合併後的分支會發生什麼事情嗎？

分支本身就像指標或是貼紙一樣，貼在某個 Commit 上面，分支並不是目錄或是檔案，所以當我們刪除已經合併過的分支，不會造成檔案或目錄跟著被刪除。

<br>

## 參考資料

什麼是 Git？為什麼要學習它？：[https://gitbook.tw/chapters/introduction/what-is-git](https://gitbook.tw/chapters/introduction/what-is-git)

Git：基本概念介紹與指令：[https://medium.com/@tina2793778/git-%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%B4%B9%E8%88%87%E6%8C%87%E4%BB%A4-d5d85607cd7d](https://medium.com/@tina2793778/git-%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%B4%B9%E8%88%87%E6%8C%87%E4%BB%A4-d5d85607cd7d)

[Git] 初始設定：[https://ithelp.ithome.com.tw/articles/10240965](https://ithelp.ithome.com.tw/articles/10240965)

連猴子都能懂的Git入門指南：[https://backlog.com/git-tutorial/tw/](https://backlog.com/git-tutorial/tw/)

Git教學】分支合併: merge 與 rebase 差異：[https://www.maxlist.xyz/2020/05/02/git-merge-rebase/](https://www.maxlist.xyz/2020/05/02/git-merge-rebase/)

Git 面試題：[https://gitbook.tw/interview](https://gitbook.tw/interview)

Reset、Revert 跟 Rebase 指令有什麼差別？：[https://gitbook.tw/chapters/rewrite-history/reset-revert-and-rebase](https://gitbook.tw/chapters/rewrite-history/reset-revert-and-rebase)