---
title: "Redis 介紹"
type: docs
weight: 9999
date: 2022-03-08
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 什麼是 Redis ?

Redis 全名是 Remote Dictionary Server ，是快速的開源記憶體鍵值資料庫 (keys-value database)。

由於 Redis 的回應時間極短，低於一毫秒，可以讓遊戲、金融服務、醫療保健等即時應用服務每秒處理幾百萬個請求。

<br>

### Redis 的優勢

#### 效能

所有的 Redis 資料都是存放在記憶體中，進而實現低延遲和高傳輸量的資料存取。

<br>

#### 彈性的資料結構

一般的鍵值資料庫提供的資料結構有限，而 Redis 提供多樣化的資料結構來滿足服務的需求，包含字串(Strings)、哈希(Hashes)、列表(Lists)、集合(Sets)、有序集合(Zset)。(後續會有詳細介紹)

<br>

#### 簡單易用

Redis 可以用更少、更精簡的指令來取代傳統複雜的程式碼，可以存取應用程式的資料。並支援 Java、Python、PHP、C/C++、C#、JavaScript、Node.js、Ruby、R、GO。

<br>

#### 複寫和持久性

Redis 採主要-複本架構，支援非同步複寫，可以將資料複寫到多個複本伺服器。不但可以提升讀取效能(因為請求可分割到多部伺服器)，還可以再主服務器發生故障時快速恢復。至於持久性，Redis 支援時間點備份，會將資料複製到磁碟中。

<br>

## 實際操作

### 安裝 Redis


使用 Homebrew 來安裝 Redis  (Mac OS：11.6)

```sh
$ brew install redis
```

<br>

安裝好用 -v 檢查版本

```sh
$ redis-server -v
Redis server v=6.2.6 sha=00000000:0 malloc=libc bits=64 build=c6f3693d1aced7d9
$ redis-cli -v
redis-cli 6.2.6
```

<br>

一個是 Server、另一個是 Cli，所以在稍後測試時，需要開啟兩個 Terminal 來執行歐！

<br>

### 執行 Redis

#### Server

```sh
$ redis-server
39403:C 08 Mar 2022 11:13:17.500 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
39403:C 08 Mar 2022 11:13:17.500 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=39403, just started
39403:C 08 Mar 2022 11:13:17.500
39403:M 08 Mar 2022 11:13:17.501 * Increased maximum number of open files to 10032 (it was originally set to 256).
39403:M 08 Mar 2022 11:13:17.501 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 39403
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

39403:M 08 Mar 2022 11:13:17.503 # Server initialized
39403:M 08 Mar 2022 11:13:17.503 * Ready to accept connections
```

如果出現上面符號，就代表 Server 已經啟動，接下來再開另一個 Terminal 來執行 Cli。

<br>

#### Cli

開啟後，在下一個 `ping` 指令，該指令用於檢測 redis 服務是否啟動，正常會顯示 `pong`。

```sh
$ redis-cli
127.0.0.1:6379> ping
PONG
```
<br>

接下來，我們都會在 Cli 視窗做測試，會詳細介紹每一個資料型態以及其適合情境。

後續會使用 Cli 畫面來做示範，但也有不錯的圖形化工具可以用於 redis 上 [AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager) 大家可以去試用看看~

<br>

### 資料型態

#### 字串 (Strings)

##### set、get

這是最基本的型態，可以存放 binary, string, integer, float資料，一個 Strings 的欄位，最高可儲存 512 Megabytes，這裡會用到的指令是 `set`、`get` ，分別用來儲存以及讀取字串，我們來看一下範例吧。

```sh
127.0.0.1:6379> set string hello-world
OK
127.0.0.1:6379> get string
"hello-world"
```
我們先用 `set` 將 hello-world 字串存到 string 這個 key，再用 `get` 顯示 string 裡面的 value。

<br>

##### incr、decr
Redis 還有一些方便的指令，如果存入的 value 是 integer 型態，就可以使用 `incr` 、`decr` ，來累加與累減。分別代表累加，像是我們的 ++ ，以及累減，像是我們的 - -。

```sh
127.0.0.1:6379> set num 10
OK
127.0.0.1:6379> incr num
(integer) 11
127.0.0.1:6379> incr num
(integer) 12
127.0.0.1:6379> decr num
(integer) 11
127.0.0.1:6379> decr num
(integer) 10
```

<br>

##### append

如果 key 已經存在並且它是字串，可以使用 `append` 指令，會從字串最後面附加進去，如果不存在，則會直接建立一個，並把值存進去。

```sh
127.0.0.1:6379> exists str
(integer) 0
127.0.0.1:6379> append str "Hello"
(integer) 5
127.0.0.1:6379> append str " World~"
(integer) 12
127.0.0.1:6379> get str
"Hello World~"
```

<br>

##### getrange

可以輸入字串的開始位元與結束位元，會依照你輸入的去顯示字串。我把它理解成是陣列的 key 與 value 的關係。

```sh
127.0.0.1:6379> set a "This is a string"
OK
127.0.0.1:6379> getrange a 0 3
"This"
127.0.0.1:6379> getrange a -3 -1
"ing"
127.0.0.1:6379> getrange a 0 -1
"This is a string"
127.0.0.1:6379> getrange a 10 100
"string"
```

<br>

##### mset
我們也可以設定的時候，把要設定的值都一起設定，只需要使用 `mset` 就可以達成。

```sh
127.0.0.1:6379> mset 1 1 2 2
OK
127.0.0.1:6379> get 1
"1"
127.0.0.1:6379> get 2
"2"
127.0.0.1:6379> get 3
(nil)
```
如果用 `get` 顯示資料，若沒有對應的 key ，會顯示 (nil)。

<br>

##### 字串型態適合場景

**字串(strings) 型態適合用於圖片快取 （使用binary）、累計次數、觀看累計次數**

<br>

{{< figure src="/redis/redis-introduce/string.webp" width="550" caption="String 適用場景圖" >}}

<br>

#### 哈希 (Hashes)
可以把他想像成二維陣列，應該會比較好理解，我網路上找了一張圖，應該會比較清楚！

<br>

{{< figure src="/redis/redis-introduce/hash.webp" width="500" caption="Hashes 示意圖 ([Redis 基本資料形態](https://blog.judysocute.com/2020/10/04/redis-%E5%9F%BA%E6%9C%AC%E8%B3%87%E6%96%99%E5%BD%A2%E6%85%8B/))" >}}

<br>

Hashes 是用來存放一組相同性質的資料，這些資料 Hashes 或是物件的某一屬性，與 String 較為不同的是他可以取回單一個欄位資料，但 String 必須取回所有資料，單一個 Key 可以存放2^32 - 1的資料欄位，

他的資料型態有像是，一個 user001 裡面是一個 Hashes，Hashes 裡面又會存放 name、phone、gender ，我們來實際操作看看。

<br>

##### hset、hget

```sh
127.0.0.1:6379> hset student name ian phone 0980123456 gender M
(integer) 3
127.0.0.1:6379> hget student name
"ian"
127.0.0.1:6379> hget student phone
"0980123456"
127.0.0.1:6379> hget student gender
"M"
```
Hashes 的話要使用 `hset`、`hget` 來對 Hashes 做儲存以及讀取。

<br>

##### hgetall

想要一次顯示 Hashes 裡面的 key 跟 value ，可以使用 `hgetall` 來顯示全部資料。

```sh
127.0.0.1:6379> hset student name ian phone 0980123456 gender M
(integer) 3
127.0.0.1:6379> hgetall student
1) "name"
2) "ian"
3) "phone"
4) "0980123456"
5) "gender"
6) "M"
```

<br>

##### hkeys

想要單獨顯示 Hashes 裡面的 key ，可以使用 `hkeys` 來顯示。

```sh
127.0.0.1:6379> hset student name ian phone 0980123456 gender M
(integer) 3
127.0.0.1:6379> hkeys student
1) "name"
2) "phone"
3) "gender"
```

<br>

##### hvals

想要單獨顯示 Hashes 裡面的 value ，可以使用 `hvals` 來顯示。

```sh
127.0.0.1:6379> hset student name ian phone 0980123456 gender M
(integer) 3
127.0.0.1:6379> hvals student
1) "ian"
2) "0980123456"
3) "M"
```

<br>

##### hlen

想要顯示 Hashes 裡面的 key 長度，可以使用 `hlen ` 來顯示。

```sh
127.0.0.1:6379> hgetall student
1) "name"
2) "ian"
3) "gender"
4) "M"
127.0.0.1:6379> hlen student
(integer) 2
```

<br>

##### hincrby

想要增加 Hashes 裡面的 value 整數，可以使用 `hincrby ` 來新增。

```sh
127.0.0.1:6379> hset test a 10 b 20
(integer) 2
127.0.0.1:6379> hincrby test a 2
(integer) 12
```

<br>

##### hdel

想要刪除 Hashes 裡面的 key，可以使用 `hdel ` 來刪除。

```sh
127.0.0.1:6379> hset student name ian phone 0980123456 gender M
(integer) 3
127.0.0.1:6379> hdel student phone
(integer) 1
127.0.0.1:6379> hgetall student
1) "name"
2) "ian"
3) "gender"
4) "M"
```

<br>

##### 哈希型態適合場景

**哈希(Hashes) 型態適合用於每次只需要取用一部分的資料**

<br>

{{< figure src="/redis/redis-introduce/hash-template.webp" width="550" caption="Hashes 適用場景圖" >}}

<br>

#### 列表 (Lists)

##### lpush、lrange

Lists 資料型態可以想像成程式語言中的Array物件。Lists 單一個Key可以存放2^32 - 1，這邊會使用到 `lpush`、`lrange` 來對 `Lists` 做儲存以及讀取。

```sh
127.0.0.1:6379> lpush list2 a a b b c d e
(integer) 7
127.0.0.1:6379> lrange list2 0 10
1) "e"
2) "d"
3) "c"
4) "b"
5) "b"
6) "a"
7) "a"
```

<br>

##### rpush
除了從隊伍頭放入資料，也可以用 `rpush` 從隊伍尾放入資料，如果使用 `lrange` 來顯示方向也會相反歐。

```sh
127.0.0.1:6379> rpush list3 a a b b c d e
(integer) 7
127.0.0.1:6379> lrange list3 0 10
1) "a"
2) "a"
3) "b"
4) "b"
5) "c"
6) "d"
7) "e"
```

<br>

##### lpop、rpop
也可以使用 `lpop`、`rpop` 分別從隊伍頭或尾彈出一筆資料。

```sh
127.0.0.1:6379> rpush list3 a a b b c d e
(integer) 7
127.0.0.1:6379> lpop list3
"a"
127.0.0.1:6379> rpop list3
"e"
```

<br>

##### lset
可以使用 `lset` 來設定指定位置的資料。

```sh
127.0.0.1:6379> lrange list3 0 2
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> lset list3 1 w
OK
127.0.0.1:6379> lrange list3 0 2
1) "a"
2) "w"
3) "c"
```

<br>

##### 列表型態適合場景

**列表(Lists) 型態適合用於文章列表或者資料分頁展示的應用**

<br>

{{< figure src="/redis/redis-introduce/list.webp" width="600" caption="Lists 適用場景圖" >}}

<br>

#### 集合 (Sets)
其實跟 Lists 一樣，就是資料的集合，只是 `Sets` 多了一層限制，就是集合中的值不能重複，這邊會使用到 `sadd`、`smembers` 來對 `Sets` 做儲存以及讀取。

```sh
127.0.0.1:6379> sadd set1 a b c d a a b b
(integer) 4
127.0.0.1:6379> smembers set1
1) "c"
2) "a"
3) "b"
4) "d"
```
可以看到他實際寫入的只有 4 筆資料。

<br>

##### spop
 從 `set` 集合中隨機跳出一定數量的資料。

```sh
127.0.0.1:6379> sadd set1 a a b b c d
(integer) 4
127.0.0.1:6379> spop set1 2
1) "c"
2) "d"
127.0.0.1:6379> spop set1 2
1) "a"
2) "b"
```

<br>

##### sismember

可以使用 `sismember` 來檢查輸入值是否為 `Set` 集合的成員。

```sh
127.0.0.1:6379> sadd set a b b c d
(integer) 4
127.0.0.1:6379> sismember set d
(integer) 1
127.0.0.1:6379> sismember set f
(integer) 0
```

<br>

##### srem

可以使用 `srem` 來刪除 `Set` 集合中的成員。

```sh
127.0.0.1:6379> sadd set a b b c d
(integer) 4
127.0.0.1:6379> srem set a d
(integer) 2
127.0.0.1:6379> smembers set
1) "b"
2) "c"
```

<br>

##### sdiff
會顯示第一個集合與其他集合不同的值。

```sh
127.0.0.1:6379> sadd set1 a b c d
(integer) 4
127.0.0.1:6379> sadd set2 a b
(integer) 2
127.0.0.1:6379> sadd set3 b
(integer) 1
127.0.0.1:6379> sdiff set1 set2 set3
1) "c"
2) "d"
```

<br>


##### 集合型態適合場景

**集合(Sets) 型態適合用於文章中的Tag標籤、或是要排除相同資料**

<br>

{{< figure src="/redis/redis-introduce/set.webp" width="700" caption="Sets 適用場景圖" >}}

<br>

#### 有序集合 (Zset)

故名思義，有序集合就是有排序的集合，Sorted Sets 結構會 **多一個數字值去為排序的權重** 來決定先後順序，這邊會使用到 `zadd` 、 ` zrangebyscore` 來對 `Zset` 做儲存以及讀取。

```sh
127.0.0.1:6379> zadd sortset 10 '10'
(integer) 1
127.0.0.1:6379> zadd sortset 6 '6'
(integer) 1
127.0.0.1:6379> zadd sortset 2 '2'
(integer) 1
127.0.0.1:6379> zadd sortset 99 '99'
(integer) 1
127.0.0.1:6379> zrangebyscore sortset 0 100
1) "2"
2) "6"
3) "10"
4) "99"
```

就可以看出在顯示的時候並不是依照寫入順序，而是依照我們所設定的權重去排序的。

<br>

##### withscores
如果想要顯示 value 與 score ，可以使用 withscores 來顯示。

```sh
127.0.0.1:6379> zadd money 2000 tom
(integer) 1
127.0.0.1:6379> zadd money 3500 peter
(integer) 1
127.0.0.1:6379> zadd money 5000 jack
(integer) 1
127.0.0.1:6379> zrange money 0 -1 withscores
1) "tom"
2) "2000"
3) "peter"
4) "3500"
5) "jack"
6) "5000"
```

<br>

##### zremrangebyscore
移除有序集合中，指定分數內區間的所有成員。

```sh
127.0.0.1:6379> zadd money 2000 tom
(integer) 1
127.0.0.1:6379> zadd money 3500 peter
(integer) 1
127.0.0.1:6379> zadd money 5000 jack
(integer) 1
127.0.0.1:6379> zremrangebyscore money 1500 3500
(integer) 2
127.0.0.1:6379> zrange money 0 -1 withscores
1) "jack"
2) "5000"
```

<br>

##### zcard
可以使用 zcard 來計算集合中元素的數量。

```sh
127.0.0.1:6379> zadd sort 10 10
(integer) 1
127.0.0.1:6379> zadd sort 3 3
(integer) 1
127.0.0.1:6379> zadd sort 66 66
(integer) 1
127.0.0.1:6379> zadd sort 33 33
(integer) 1
127.0.0.1:6379> zcard sort
(integer) 4
```

<br>

##### 有序集合型態適合場景

**有序集合(Zset) 型態適合用於需要有序排列的資料**

<br>

{{< figure src="/redis/redis-introduce/zset.webp" width="550" caption="Zset 適用場景圖" >}}

<br>

### 發布訂閱 (PUB/SUB)

Redis 發布訂閱 (pub/sub) 是一種消息通信模式，發送者 (pub) 發送消息，訂閱者 (sub) 接收消息。

Redis 客戶端可以使用 `subscribe` 來訂閱任意數量的頻道。

下面這張圖是頻道1，以及訂閱這個頻道的三個用戶端分別是客戶端2、客戶端7、客戶端5

<br>

{{< figure src="/redis/redis-introduce/channel.webp" width="550" caption="Subscribe 示意圖" >}}

<br>

當我們有新消息通過 `publish` 指令發送給頻道1 ，這個消息就會被發送給有訂閱頻道1的三個客戶端。

<br>

{{< figure src="/redis/redis-introduce/message.webp" width="550" caption="Publish 示意圖" >}}

<br>

我們來模擬一下吧 ! 先開兩個 Terminal 來執行 `redis-cli` 一個當作發送(pub)，另一個當作接收(sub)。

我們先用第一個 Terminal 訂閱一個頻道 channel_1

```sh
127.0.0.1:6379> subscribe channel_1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel_1"
3) (integer) 1
```

<br>

開啟另一個 Terminal 發送訊息到 channel_1

```sh
127.0.0.1:6379> publish channel_1 "Hello World~"
(integer) 1
127.0.0.1:6379> publish channel_1 "ian~"
(integer) 1
127.0.0.1:6379> publish channel_1 "test~"
(integer) 1
```

<br>
這時候再切換回來第一個 Terminal ，就可以看到他接收到我們傳送的訊息

```sh
127.0.0.1:6379> subscribe channel_1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel_1"
3) (integer) 1
1) "message"
2) "channel_1"
3) "Hello World~"
1) "message"
2) "channel_1"
3) "ian~"
1) "message"
2) "channel_1"
3) "test~"
```

<br>

## 參考資料
Redis 官網：[https://redis.io](https://redis.io)

Redis：[https://aws.amazon.com/tw/redis/](https://aws.amazon.com/tw/redis/)

Redis 基本資料形態：[https://blog.judysocute.com/2020/10/04/redis-%E5%9F%BA%E6%9C%AC%E8%B3%87%E6%96%99%E5%BD%A2%E6%85%8B/](https://blog.judysocute.com/2020/10/04/redis-%E5%9F%BA%E6%9C%AC%E8%B3%87%E6%96%99%E5%BD%A2%E6%85%8B/)

Redis 發布訂閱：[https://www.runoob.com/redis/redis-pub-sub.html](https://www.runoob.com/redis/redis-pub-sub.html)