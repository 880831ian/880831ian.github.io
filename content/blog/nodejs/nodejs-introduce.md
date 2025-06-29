---
title: "Node.js 介紹"
type: docs
weight: 9999
description: Node.js 介紹
images:
  - nodejs/nodejs-introduce/og.webp
date: 2022-03-30
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 什麼是 Node.js ?

Node.js 是能夠在伺服器端運行 JavaScript 的開放原始碼、跨平台執行環境。

Node.js 的出現，讓前端網站開發人員可以使用 JaveScript 來做後端或是系統層面的工作。讓前端開發網站開發人員使用已懂的 JavaScript 語言就可以自行設定網站伺服器。

Node.js 採用 Google 開發的 Chrome V8 JavaScript 引擎和 libuv 函式庫，可以用指令去執行 JavaScript，使用非阻塞輸入輸出、非同步、事件驅動等技術來提高效能，可最佳化應用程式的傳輸量和規模。這些技術通常用於資料密集的即時應用程式。

<br>

{{< figure src="/nodejs/nodejs-introduce/main-nodejs.png" width="700" caption="主要的 Node.js 組件 ([An Intro to Node.js That You May Have Missed](https://itnext.io/an-intro-to-node-js-that-you-may-have-missed-b175ef4277f7))" >}}

<br>

其他開源、低級組件，主要用 C/C++ 編寫：
* c-ares : 用於非同步 DNS 請求的 C 函式庫，用於 Node.js 中的一些 DNS 請求。
* http-parser：一個輕量級的 HTTP 請求/回應解析器。
* OpenSSL：一個著名的通用密碼庫。用於 `tls` 和 `cryto` 模組。
* zlib：無損數據壓縮庫。

<br>

Node.js 能快速的原因是因為他對資源的調校不同，當程式收到一筆連線，

相較於 PHP 的每次連線都會新生成一個執行緒，當連線數量暴增時很快就會消耗掉系統的資源，並且容易產生阻塞 (block)，

而 Node.js 則是會通知作業系統透過 epoll、kqueue、/dev/poll、select 等將連線保留，並放入 heap 中配置，先讓連線進入休眠 (Sleep) 狀態，等系統通知才觸發連線的 callback。

這種處理方式只會佔用記憶體，並不會使用到 CPU 資源。另外因為 JavaScript 語言的特性，每一個 request 都會有一個 callback，可以避免發生阻塞的狀況發生。

<br>

以下會先從非阻塞輸入輸出、非同步，再談到事件驅動。

<br>

### 非阻塞 (non-blocking)

輸入輸出(I/O) 是程式跟系統記憶體或網路的互動，例如發送 HTTP 請求、對資料庫CRUD 操作等等。

以網站開發者角度來看，大部分的網站程式都不需要太多的 CPU 計算，反而是在等待大量的 I/O 處理完畢 (HTTP 請求、資料庫的取得資料或是更新資料等)，所以處理 I/O 的速度會是網頁程式效能的關鍵。

<br>

那要怎麼才能讓等待 I/O 的時間，不要卡住後續的程式碼呢？可以讓程式一邊等 I/O 處理，一邊繼續執行其他部分的程式碼，主要有兩種方法：

1. 多執行緒 (multi-threaded)：使用阻塞 (blocking) I/O 的設計。
2. 單執行緒 (single-threaded)：使用非阻塞 (non-blocking) I/O 的設計 + 非同步 (asynchronous) 處理。

<br>

**阻塞就是 I/O 的處理阻擋了其他後續程式碼的執行**。舉個例子：

| 阻塞 (blocking) | 非阻塞 (non-blocking) |
| :---: | :---: |
| 阻塞後續程式碼的執行，就好像是我們去附近買烤肉，交給老闆後，為了想吃到熱騰騰的食物，所以只能留在原地，不能先去其他地方 | 非阻塞不會阻擋後續程式碼的執行，就好像是我們去百貨公司美食街點餐，點完餐後會拿到一個呼叫器，就可以先離開，等到呼叫器響了再回去拿即可 |

<br>

像是 **Python、Ruby 等語言是使用多執行緒 (multi-threaded)，使用阻塞 I/O** ：

程式會等網路或是記憶體的作業結束後才會繼續往下，等待時間這個作業中的執行緒不會去做其他事情。

如果想要達到『等待 I/O 期間不要卡住其他程式碼』，的做法就是新開一個執行緒，直到任務完成，再告訴主執行緒說( 我完成囉！) 即可。

<br>

**Node.js 使用單執行緒 (single-threaded) ，非阻塞 I/O + 非同步函式**：

Node.js 使用非阻塞設計，那要怎麼去操作資料庫或是 HTTP 請求的輸入輸出呢？就要透過非同步 (asynchronous) 來處理囉！

<br>


###  非同步 (asynchronous)

非同步也可以稱為異步，它的作用就是讓程式不要被阻擋等 I/O 處理完，才可以跑下一行程式碼，是直到函式中的 callback 被呼叫的時候再執行後續要做的事情。

<br>

同步 vs. 非同步


<br>

* 非同步 (asynchronous)

可以想像一下你去咖啡廳買拿鐵跟黑咖啡，可能會發生的情況是：
1. 你點了拿鐵跟黑咖啡
2. 店員在收銀機上輸入點餐內容
3. 店員請同事 A 準備拿鐵、請同事 B 準備黑咖啡，並告知做完後，要提醒店員
4. 黑咖啡製作會比較快，B 同事會先完成，而剛好店員剛幫你結帳完沒事，所以把黑咖啡拿給你
5. 拿鐵製作包含較多步驟，花費時間較久，等 A 同事完成後，店員剛好沒事，所以把拿鐵拿給你

<br>

可以看到，櫃檯店員一次只能做一件事情，但為了節省時間，店員將工作分配給其他同事，在下完指令後，店員會繼續幫你結帳，等同事們各自完成後會告知店員，店員在依序把飲料交給你 - 最終等待時間減少，也不會浪費閒置的資源，這就是現實生活中非同步的情況。

<br>

* 同步 (synchronous)

一樣我們點了拿鐵跟黑咖啡，換成同步的話：

1. 店員在收銀機上輸入點餐內容
2.  店員請同事 A 開始準備拿鐵
3.  A 同事準備完拿鐵，店員轉交給你
4.  店員請同事 B 開始準備黑咖啡
5.  B 同事準備完黑咖啡，店員交給你
6.  店員幫你刷載具、打統編、找錢等等

<br>

同樣的餐點內容，如果是同步處理，代表要等每一件事情做完，才可以做下一步。也就是說同事 A 完成拿鐵後，店員才請 B 同事準備黑咖啡。相對於非同步來說，會花費不少時間以及浪費不少閒置資源。


<br>

從咖啡店的例子中可以發現：

1. 櫃檯店員手上一次能做的事情只有一件 (Single thread 單執行緒)，只是在非同步的例子中，將製作咖啡的事情委派給其他同事處理，讓自己可以繼續幫你結帳，來提高效率 — JavaScript 是 Single thread，一次只能做一件事情。
2. 在非同步的例子中，櫃檯店員將製作咖啡的事情委派出去，其實店員也不知道哪一個任務會先被完成，但店員還是可以繼續完成結帳的任務，不會因為同事 A、B 還在製作咖啡，就不能接下去動作 (non-blocking) — JavaScript 一次只能做一件事，但藉由 Node 提供的 API 協助，在背後處理這些事件 (同事在背後製作咖啡)，可以等待製作同時，不會被阻塞 (blocking) 到下一件事情的執行。
3. 非同步的例子中，店員委派事情的流程很簡單：就是請同事完成製作咖啡 (event) + 在收到同事通知完成後接手咖啡，並轉交給你 (callback function) — 若是採用非同步處理，會有 callback function 來指定事件完成後要接續做什麼：它不會立即被執行，而是等待委託的事情被完成後才觸發。
4. 當同事 A、B 分別通知完成後，就會依序把咖啡放在店員的旁邊排成一排 (event queue)。想向店員有一個小助手 (event loop) ，他的工作內容是確認店員結帳完了沒有：如果結帳完了，就會把隊伍中第一杯咖啡叫給店員，讓店員交給你 (觸發 callback function) ; 如果店員還在結帳，就會讓隊伍中的咖啡擺在旁邊繼續等待。


<br>

JaveScript 實現非同步的方法不斷演進著：從 callback、promise 到最新的 async-await 函式。

<br>

#### Callback

##### 什麼是 Callback

假設有 A、B、C 三件工作，其中 B 必須等待 C 做完才能執行。大部份的人幾乎都是做 A，再做 C，等待 C 做完以後最後做 B。但對於可多工的人來說，卻可能是同時做 A 與 C（多工），等待 C 完成後做 B。

Callback function 是一個被作為參數帶入另一個函式中的「函式」，這個被作為參數帶入的函式將在「未來某個時間點」被呼叫和執行 — 這是處理非同步事件的一種方式。

再次舉 A、B、C 三件工作的例子，其中 B 必須等待 C 做完才能執行，於是我們將 B 放到 C 的 callback 中，讓宿主環境在收到 C 完成的回應時後 B 放到佇列中準備執行。

<br>

```js
doA();

doC(function() {
	doB();
});
```

<br>

常見的例子：

* 使用瀏覽器所提供的 `setTimeout()`或是 `setInterval()`

```js
setTimeout(() => {
 console.log('這個訊息將在三秒後被印出來')
}, 3000)
```
提供一個匿名函式作為參數帶入 setTimeout() 函式中，目的就是請 setTimeout() 在未來某個時間點（三秒後）呼叫和執行這個匿名函式。

<br>

* DOM 的事件監聽

```js
const btn = document.querySelector('button')
btn.addEventListener('click', callbackFunctionName)
```
callbackFunctionName 做為參數被帶入 addEventListener() 中，callbackFunctionName 不會立即被執行，而是未來按鈕被點擊時才會執行。

<br>

Callback 主要有一個缺點：`回呼地獄`

<br>

##### 回呼地獄 (Callback Hell)

回呼地獄 (Callback Hell) 又稱為「毀滅金字塔」，指的是層次太深的巢狀 Callback，讓程式變的更複雜且難以預測或是追蹤。

向遠端伺服器發出請求並獲得資訊後，執行 Callback，再發出請求，獲得資訊後執行 Callback，再發出請求，獲得資訊後執行 Callback，就會不小心一層包一層，變成所謂的 Callback Hell。


```js
doA(function() {
  doB();

  doC(function() {
    doD();
  });

  doE();
});

doF();
```

缺點：
1. 可讀性低：如果程式碼出錯，要回頭慢慢找錯誤的地方
2. 可維護性低：如果要修改其中一組函式，牽一髮而動全身

<br>

那在同步執行情況下，可以使用 ```try...catch``` 來捕捉錯誤訊息，但如果是在非同步情況下，要怎麼處理錯誤或是例外訊息呢！？ 主要有兩種方式：

<br>

##### 分別回呼 (Split Callback)

分別的回呼要設定兩個 Callback，一個用於成功通知，另一個則用於錯誤通知。如下，第一個參數是用於成功的 Callback，第二個參數是用於失敗的 Callback：

```js
function success(data) {
  console.log(data);
}

function failure(err) {
  console.error(error);
}

ajax('http://sample.url', success, failure);
```

<br>

那如果在 Callback 中發生錯誤，要怎麼辦呢!?

<br>

```js
function success(data) {
  console.log(x);
}

function failure(err) {
  console.error(error);
}

ajax('http://sample.url', success, failure);

// Uncaught (in promise) ReferenceError: x is not defined
```

會直接報錯，並不會進入到 failure 這個 Callback 裡面，也就是說，如果是在 Callback 內發生錯誤，是不會被捕捉到的。

<br>

##### 錯誤優先處理 (Error-First Style)

Node.js 的 API 常見這樣的設計方式，第一個參數是 error ，第二個參數是回應的資料 (data)。檢查 error 是否有值或是 true，否則就接續處理 data。

```js
function response(err, data) {
  if(err) {
    console.error(err);
  } else {
    console.log(data);
  }
}

ajax('http://sample.url', response);
```

接下來我們來看 Promise，可以解決 callback 可讀性低的 Callback Hell 問題。

<br>

#### Promise

Promise：Callback 以外的另一種方式來處理非同步事件，且可讀性與可維護性比 Callback 好很多。
Promise 是一個物件，代表著一個尚未完成，但最終會完成的一個動作 - 在一個非同步處理流程中，它只是一個暫存的值。

我們一樣來說剛剛咖啡店的例子：
當我們點完拿鐵跟黑咖啡後，店員會給你一張印有號碼的收據，然後告訴你等等聽到號碼，就可以來領咖啡了，而這張收據就是 Promise，代表這個任務完成後，就可以接著執行接下來的動作了。

在等待過程中，其實無法百分百確定最後一定會拿到咖啡 (Promise) ; 店員可能順利做完咖啡交到你手上 (Resolved) ; 可能牛奶或是咖啡豆沒了，所以店員告訴你今天做不出來咖啡 (Rejected)。

Promise 就像上面的例子中，會處在三個任意階段中：

* Pedning：等待事情完成中，但不確定最終會順利完成或失敗
* Resolved（或稱 Fulfilled）：代表順利完成了，並轉交結果
* Rejectesd：代表失敗了，並告知失敗原因

<br>


```js
const getData = new Promise((resolve, reject) => {
    // 製作咖啡.....

    // 作業完成，並回傳錯誤訊息時
    if (error) {
        return reject('牛奶或是咖啡豆沒了')
    }
    // 作業成功完成
    resolve({
        data: '咖啡交到你手上',
    })
})
```
* resolve 函式：當非同步作業成功完成時，將結果做為參數帶入執行。
*  reject 函式：當非同步作業失敗時，將錯誤訊息作為參數帶入執行。

<br>

創建出來的 Promise 物件在實例上有兩個重要的方法：

##### .then() 方法

當成功從 `resolve()` 獲得結果時 - 狀態會由 Pending 轉為 Resolved - `then()` 方法就會被調用。

```js
getData
  // 使用 then 方法，並將成功訊息印出來
  .then(data => {console.log('成功資料', data)})
```
以剛剛咖啡店的例子來看， `then()` 方法就像你會去聽咖啡人員是否叫號，並確認咖啡是否成功準備好了，就領取咖啡來喝。


<br>

##### .catch() 方法

當從 `reject()` 獲得錯誤訊息時 - 狀態由 Pending 轉為 Rejected - `catch()` 方法就會被調用來處理錯誤。

```js
getData
  // 使用 then 方法，並將成功訊息印出來
  .then(data => {console.log('成功資料', data)})
  // 使用 catch 方法，並將錯誤訊息印出來
  .catch(error => console.log('錯誤訊息', error))
```
一樣以剛剛咖啡店的例子來看， `catch()` 方法就像你會去聽咖啡店店員是否有告知咖啡做不出來的訊息，如果有，你就會去櫃檯退錢或是改其他產品代替。

<br>

##### 串接 then() 方法

還記得上面我們示範的 callback hell ，因為我們依序要向不同的伺服器或資料庫取得資料，也要依序處理多個非同步的作業，所以會有一層一層的 callback 去達成。

從上面 `then()` 和 `catch()` 方法例子中可以發現，非同步作業執行完成後的處理步驟被獨立開來，這樣程式碼的可讀性就會高很多 - 在依序處理多個非同步作業時會更為明顯，我們可以看到下面我們用 `then()` 方法做串接：

<br>

```js
getData
  // 使用 then 方法，並將成功訊息印出來
  .then(data => {
    console.log(data.data1) // abc
    return data.data1 + 'def'
  })
  // 獲得前一個 then() 回傳的結果
  .then(data => {
    console.log(data) // abcdef
    return data + 'ghi'
  })
  // 獲得前一個 then() 回傳的結果
  .then(data => {
    console.log(data) // abcdefghi
  })
  // 使用 catch 方法，並將錯誤訊息印出來
  .catch(error => console.log('錯誤訊息', error))
```

<br>

##### Promise.all([....]) 方法

當在處理「多個非同步事件」時，Promise.all() 方法會等所有 Promise 都被順利完成 (Resolved) 後，才會執行接下去的動作 ; 一旦收到某一個 Promise 回傳錯誤 (Rejected)，就會立即執行後續的錯誤處理流程。

我們以剛剛咖啡店的例子來說，我們點完咖啡後，又因為嘴饞，多買了一個麵包來吃，這時我們手上就有兩個領餐號碼，那 `Promise.all([...])` 就是當你等到兩個號碼都被叫到後，才會去領餐。

<br>

那 `Promise.all([....])` 的好處什麼呢!? 它可以縮短等待時間：

```js
const oneSecond = new Promise((resolve, reject) => {
  setTimeout(() => {
    // 一秒後回傳資料
    resolve('one second')
  }, 1000);
})

const twoSecond = new Promise((resolve, reject) => {
  setTimeout(() => {
    // 兩秒後回傳資料
    resolve('two second')
  }, 2000);
})

const threeSecond = new Promise((resolve, reject) => {
  setTimeout(() => {
    // 三秒後回傳資料
    resolve('three second')
  }, 3000);
})

// 等到三個 Promise 都成功回傳後，才執行接下去的流程
Promise.all([oneSecond, twoSecond, threeSecond])
  .then(([oneSecond, twoSecond, threeSecond]) => {
    console.log(oneSecond, twoSecond, threeSecond)
  })
```

我們模擬一下我們要分別向三個資料庫請求資料 (我們以 `setTimeout()` 示意非同步處理事件)，可以看到我們要先等第一個資料庫成功回傳資料後，才展開第二次請求，等到成功收到回傳後才開始第三次請求。而使用 `Promise.all()` 方法，我們能先依序向資料庫發出請求，並在成功獲得全部回傳資料後，才會做後續的動作。

<br>

#### Async/Await

那我們剛剛透過 Promise 包裝和使用，的確避免了 callback hell 讓整個流程變得很清楚，提升了程式碼的易讀性與可維護性。

Async/Await 是所謂的語法糖衣，是一種新的語法撰寫方式，來處理「非同步事件」，讓非同步的程式碼讀起來更像在寫「同步程式碼」。

Async/Await 是用來簡單化和清楚化 Promise 串連 then 這種相對複雜的結構，他回傳的一樣也是 Promise 物件，只是針對 promise-based 寫法進行包裝。

<br>

Async 關鍵字

`async` 關鍵字可以放在任意函式前面，它代表「我們正宣告一個非同步的函式，且這個函式會回傳一個 Promise 物件」：

```js
// 一般函式
function getGroupInfo() {
  return data
}

// 使用 async 函式
async function getGroupInfo() {
  return  data
}
```

<br>

Await 關鍵字

在 `async` 函式中使用 `await` 關鍵字，代表「我們請 JavaScript 等待這個非同步的作業完成後，才展開後續的動作」，換句話說：await 讓 async 函式的執行動作暫停，等到它獲得回傳的 Promise 物件後 - 無論執行成功 (resolve) 或是失敗 (rejected) -才會恢復執行 async 函式。

* 優點：可以直接使用 await 後獲得的回傳值存於一個變數中做後續使用，而不是在呼叫 `then()` 方法一個一個串，讓程式碼看起來像是在處裡一般的同步程式碼，提升了易讀性與可維護性。

<br>

##### Async/Await 範例

我們在後續的範例，一樣使用 `setTimeout()` 來模擬資料庫請求和等待資料的非同步：

```js
function getFirstInfo() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('first data')
    }, 1000);
  })
}

function getSecondInfo() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('second data')
    }, 2000);
  })
}

async function getGroupInfo() {
  // 代表等到第一筆資料回傳後，才印出結果和請求第二筆資料
  const firstInfo = await getFirstInfo()
  console.log(firstInfo)
  // 代表等到第二筆資料回傳後，才印出結果
  const secondInfo = await getSecondInfo()
  console.log(secondInfo)
}

getGroupInfo()
```
在 `getGroupInfo()`  函式前加上 `async` 關鍵字，來告知值這是一個非同步函示， `getFirstInfo()` 跟 `getSecondInfo()` 是兩個要處理非同步的地方，因此分別在兩個呼叫函式前加上 `await` 關鍵字。


<br>

##### await 搭配 Promise.all([...]) 方法

```js

function getFirstInfo() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // 1秒後回傳結果
      resolve('first data')
    }, 1000);
  })
}

function getSecondInfo() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // 2秒後回傳結果
      resolve('second data')
    }, 2000);
  })
}

async function getGroupInfo() {
  try {
    const firstInfo = getFirstInfo()
    const secondInfo = getSecondInfo()
    // 同步發出非同步請求，並等到兩秒後（非三秒）成功獲得兩筆回傳後，才印出結果
    const [firstData, secondData] = await Promise.all([firstInfo, secondInfo])
    console.log(firstData, secondData)
  } catch (error) {
    // 處理錯誤回傳
    console.log(error)
  }
}

getGroupInfo()
```

透過 `await` 關鍵字搭配 `Promise.all()` 方法，同步向兩個資料庫獲取資料，並等到兩筆資料都成功回傳後，才印出結果 - 省時效率提高！

<br>

我們可以看到下面這張圖，他是 Node.js 運行時的縮圖，一旦你的 Node.js 應用程序啟動，它會先開始一個初始化階段，運行啟動腳本，包括請求模組和註冊事件 callback。然後應用程序進入事件循環（也稱為主線程、事件線程等），從概念上講，它是為通過執行適當的 JS callback 來響應傳入的客戶端請求而構建的。

<br>

{{< figure src="/nodejs/nodejs-introduce/run-nodejs.png" width="700" caption="Node.js 運行時的圖示 ([An Intro to Node.js That You May Have Missed](https://itnext.io/an-intro-to-node-js-that-you-may-have-missed-b175ef4277f7))" >}}

<br>

libuv 是非同步處理的函式庫（以 C 語言為主)，在面對非同步作業時，會開啟執行緒池 (thread pool / worker pool) ，預設共有四個執行緒 (也就是 worker threads)，運算這些 I/O 用的方式是事件迴圈 (event loop)。

所以就算 V8 處理程式碼是單執行緒，一但進入到 libuv 手上，它還是會幫你把會阻塞的 I/O 分工到執行緒池中交給不同的執行緒去處理，直到 callback 發生才會丟回應讓程式知道。


那我們來看看 libuv 如何透過事件迴圈有效的處理非同步 I/O。

<br>

### 事件驅動 (event-driven)

在講事件驅動前，我們先來了解一下什麼是事件：

<br>

#### 事件 (event)

事件是指用戶或是系統作出的動作，例如使用者點選按鈕，或是檔案讀取完成、某種錯誤產生等等，都叫做事件。

<br>

#### 事件驅動 (event-driven)

事件驅動是一種程式執行模型，表示程式的進行是依據事件的發生而定，監聽到事件就處理、處理完就執行 callback ，透過不斷的監聽跟回應事件執行程式。

而事件驅動在不同的地方有不同的實現。瀏覽器 (前端) 和 Node.js (後端) 基於不同的技術實現了各自的事件迴圈。就 Ndoe.js 來說，事件就是交給 libuv 去處理 ; 至於瀏覽器的事件迴圈在 HTML 5 的規範中有定義。

<br>

#### 事件迴圈 (event loop)

因為 Node.js 只有一個執行緒，所以當 libuv 把非同步事件處理完後，callback 要被丟回應用程式中排隊，等待主執行緒的 stack 為空的時候，才會開始執行。這個排隊的地方就是事件佇列 (event queue)。

libuv 會不斷檢查有沒有 callback 需要被執行，有的話分配到主執行緒結束手邊的程式後處理，因此這個過程稱為 『事件迴圈』。

<br>

{{< figure src="/nodejs/nodejs-introduce/event-loop.png" width="700" caption="事件迴圈(event loop) ([Node.js 101 — 單執行緒、非同步、非阻塞 I/O 與事件迴圈](https://medium.com/wenchin-rolls-around/node-js-101-%E5%96%AE%E5%9F%B7%E8%A1%8C%E7%B7%92-%E9%9D%9E%E5%90%8C%E6%AD%A5-%E9%9D%9E%E9%98%BB%E5%A1%9E-i-o-%E8%88%87%E4%BA%8B%E4%BB%B6%E8%BF%B4%E5%9C%88-ef94f8359eee))" >}}

<br>

libuv 事件迴圈有哪些階段呢？

{{< figure src="/nodejs/nodejs-introduce/libuv.png" width="700" caption="libuv 事件迴圈 ([nexocode](https://nexocode.com/blog/posts/behind-nodejs-event-loop/))" >}}

<br>

libuv 的事件迴圈共有六個階段，每個階段的作用如下：

1. Timers：等計時器 (setTimeout、setInterval) 的時間一到，會把他們的 callback 放到這裡等待執行。
2. Pending callbacks：會把作業系統層級的錯誤給callback (TCP errors、sockets 連線被拒絕)。
3. Idle, prepare：內部使用。
4. Poll：最重要的一個階段。
	1. 如果 Queue 不為空，依次取出 callback 函數執行，直到 Queue 為空或是抵達系統最大限制。
	2. 如果 Queue 為空但有設置 「setImmediate」，就進入 check 階段。
	3. 如果 Queue 為空但沒有設置 「setImmediate」，就會在 Poll 階段等到直到 Queue 有東西或是 Timers 時間抵達。

5. Check：處理 setImmediate 的 callback。
6. Close callbacks：執行 close 事件的 callback，利如 socket.destroy()。

<br>

事件迴圈就是不斷重複以上階段。每個階段都有自己的 callback 佇列，在進入某個階段時，都會從所屬的佇列中取出 callback 來執行，當佇列為空或者被執行 callback 的數量達到系統的最大數量時，就會進入下一階段。

<br>

根據以上提到事件驅動、單執行緒和非同步、非阻塞的 I/O 處理特性，Node.js 很適合拿來開發 I/O 密集型應用程式，如影音串流、即時互動、在線聊天、遊戲、協作工具、股票行情等軟體。

<br>

## NPM

NPM 是跟 Node.js 一起安裝的線上套件庫，可以下載各式各樣的 JavaScript 套件來使用，能解決 Node.js 代碼部署上的很多問題。

<br>

安裝好後，可以使用 `npm -v` 來檢查版本：

```sh
$ npm -v
8.5.0
```
<br>

跟 NPM 息息相關的是 `package.json` 這個檔案，他是掌管專案資訊的重要檔案，我們可以使用 `init` 指令來設定 package.json：

```sh
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.
.... 省略 ....

Press ^C at any time to quit.
package name: (message) demo
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author: ian <880831ian@gmail.com>
license: (ISC)
```
* name: 就是該專案的名字，它預設就是該目錄名。
* description: 專案描述。
* entry point: 專案切入點，這有點複雜，之後再說。
* test command: 專案測試指令，之後說。
* git repository: 專案原始碼的版本控管位置。
* keywoard: 專案關鍵字
* author: 專案作者，以 author-name <author@email.com> 寫之。
* license: 專案版權。

<br>

設定好後，專案資料夾就會多一個 `package.json` 的檔案，打開後可以看到，是我們剛剛所設定好的資訊：

```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "ian <880831ian@gmail.com>",
  "license": "ISC"
}
```

<br>

接下來，要如何下載網路上的模組，要使用 `install` 指令來下載，我們下載 Node.js 最小又靈活的 Web 應用程式框架 express 來做示範：

```sh
$ npm install express

added 50 packages, and audited 51 packages in 1s

2 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

<br>

下載好後，可以看到剛剛 `package.json` 檔案多了 `dependencies` 欄位，它裡面會紀錄我們安裝了哪些套件，所以未來我們想知道專案使用了哪些套件，我們可以從 `dependencies` 這個欄位知道。

```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "ian <880831ian@gmail.com>",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.3"
  }
}
```

<br>

除此之外，還會多一個 `node_modules` 資料夾，這個資料夾就會存放我們所下載的套件

```sh
$ ls node_modules/
accepts             cookie-signature    etag                inherits            mime-types          qs                  setprototypeof
array-flatten       debug               express             ipaddr.js           ms                  range-parser        statuses
body-parser         depd                finalhandler        media-typer         negotiator          raw-body            toidentifier
bytes               destroy             forwarded           merge-descriptors   on-finished         safe-buffer         type-is
content-disposition ee-first            fresh               methods             parseurl            safer-buffer        unpipe
content-type        encodeurl           http-errors         mime                path-to-regexp      send                utils-merge
cookie              escape-html         iconv-lite          mime-db             proxy-addr          serve-static        vary
```

<br>


當我們都安裝好後，express 已經包在 `node_modules` 目錄內，在專案裡面，就可以使用 require('套件名稱‘) 來使用套件囉！
```js
var express = require('express');
```

<br>

因為這些套件都可以直接在網路上下載到，所以在推送 git 專案時，可以使用 `.gitignore` 來隱藏不想被 push 的檔案，當我們想要下載套件回來時，只要使用：

```sh
$ npm install
```
就可以依照 `package.json` 裡面的 `dependencies` 來下載套件！

## REPL

Node.js REPL  (交互式解釋器)：表示一個電腦環境，類似 Windoes 系統的終端或是 Unix/Linux 的 Shell，我們可以在終端機上輸入命令，並接收系統的響應。

Node 自帶了交互式解釋器，可以執行以下任務：

* 讀取：讀取用戶輸入，解析輸入的 JavaScript 數據結構並儲存在內存中。
* 執行：執行輸入的數據結構。
* 顯示：輸出結果。
* 循環：循環以上任務直到用戶按下兩次的 ctrl+c 按鈕退出。

<br>

我們可以輸入以下命令來啟動 Node 的終端：
```sh
$ node
Welcome to Node.js v16.14.2.
Type ".help" for more information.
>
```

<br>

可以執行簡單的數學運算：

```sh
$ node

Welcome to Node.js v16.14.2.
Type ".help" for more information.
> 1+4
5
> 5/2
2.5
> 3*7
21
> 4-3
1
> 1 + (2*4) -5
4
>
```

<br>

也可以將數據存在變數中，在需要時使用它。變數宣告需要使用 `var` 關鍵字，如果沒有使用關鍵字，會直接顯示出來。也可以使用 `console.log()` 來輸出變數

```sh
$ node

Welcome to Node.js v16.14.2.
Type ".help" for more information.
> x = 19
19
> var y = 10
undefined
> x + y
29
> console.log("Hello")
Hello
undefined
>
```

<br>

Node REPL 也支持輸入多行程式，我們試著寫一個 do-while 迴圈：

```sh
Welcome to Node.js v16.14.2.
Type ".help" for more information.
> var x = 0
undefined
> do {
... x++;
... console.log("x:" + x);
... } while (x<5);
x:1
x:2
x:3
x:4
x:5
undefined
>
```

<br>

也可以使用下底線(_)來獲得上一個程式的運算結果：

```sh
$ node

Welcome to Node.js v16.14.2.
Type ".help" for more information.
> var x = 10
undefined
> var y = 20
undefined
> x + y
30
> var sum = _
undefined
> console.log(sum)
30
undefined
>
```

<br>

## Express 框架

Node.js 在實作上不會單獨使用，通常會搭配框架去使用，像是 [Express JS 後端框架](https://expressjs.com/)，可以讓開發人員在寫同一個功能時，少寫很多程式的工具。

Express 是 Node.js 環境下提供的輕量後端架構，自由度極高，透過豐富的 HTTP 工具，能快速發開後端應用程式，它提供：

* 替不同 HTTP Method、不同 URL 路徑的 requests 編寫不同的處理方法。
* 透過整合「畫面」的渲染引擎來達到插入資料到樣板產生 response。
* 設定常見的 web 應用程式，例如：連線用的 Port 和產生 response 樣板的位置。
* 在 request 的處理流程中增加而外的中間層 (Middleware) 進行處理。

<br>

第一個 Express Hello world 程式：

```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```
```sh
$ node app.js

Example app listening on port 3000
```

使用 `node` 來啟動伺服器，並使用 Port 3000 來連線。應用程式指向 URL (/) 的路由，以 "Hello World!" 回應如果是其他路徑，res 就會回應 404 找不到。

<br>

### 畫面 (view)

剛剛有提到說它可以使用整合「畫面」的渲染引擎來顯示到樣板，我們可以透過 `--view` 指令來產生樣板以及應用程式的目錄：

```sh
$ express --view=pub
   create : public/
   create : public/javascripts/
   create : public/images/
   create : public/stylesheets/
   create : public/stylesheets/style.css
   create : routes/
   create : routes/index.js
   create : routes/users.js
   create : views/
   create : app.js
   create : package.json
   create : bin/
   create : bin/www
```

<br>


### 路由 (router)

路由是判斷應用程式如何回應用戶端對特定端點的要求，而特定端點是一個 URL 或是路徑，與一個特定的 HTTP 要求方法 (GET、POST) 等，路由定義的結構如下：

```js
app.METHOD(PATH, HANDLER)
```
其中
* app 是 express 的實例。
* METHOD 是 HTTP 要求的方法。
* PATH 是伺服器上的路徑。
* HANDLER 是當路由相符時要執行的函數。

<br>

以下範例簡單說明不同 HTTP 要求的方法：

首頁中以 Hello World! 回應。
```js
app.get('/', function (req, res) {
  res.send('Hello World!');
});
```

<br>

對根路由 (/)（應用程式的首頁）發出 POST 要求時的回應：
```js
app.post('/', function (req, res) {
  res.send('Got a POST request');
});
```

<br>

對 /user 路由發出 PUT 要求時的回應：

```js
app.put('/user', function (req, res) {
  res.send('Got a PUT request at /user');
});
```
<br>

對 /user 路由發出 DELETE 要求時的回應：

```js
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user');
});
```

<br>

## 參考資料

Node.js 官網：[https://nodejs.dev/learn/introduction-to-nodejs](https://nodejs.dev/learn/introduction-to-nodejs)

Node.js 101 — 單執行緒、非同步、非阻塞 I/O 與事件迴圈：[https://medium.com/wenchin-rolls-around/node-js-101-%E5%96%AE%E5%9F%B7%E8%A1%8C%E7%B7%92-%E9%9D%9E%E5%90%8C%E6%AD%A5-%E9%9D%9E%E9%98%BB%E5%A1%9E-i-o-%E8%88%87%E4%BA%8B%E4%BB%B6%E8%BF%B4%E5%9C%88-ef94f8359eee](https://medium.com/wenchin-rolls-around/node-js-101-%E5%96%AE%E5%9F%B7%E8%A1%8C%E7%B7%92-%E9%9D%9E%E5%90%8C%E6%AD%A5-%E9%9D%9E%E9%98%BB%E5%A1%9E-i-o-%E8%88%87%E4%BA%8B%E4%BB%B6%E8%BF%B4%E5%9C%88-ef94f8359eee)

An Intro to Node.js That You May Have Missed：[https://itnext.io/an-intro-to-node-js-that-you-may-have-missed-b175ef4277f7](https://itnext.io/an-intro-to-node-js-that-you-may-have-missed-b175ef4277f7)

Sequelize：[https://sequelize.org/v6/index.html](https://sequelize.org/v6/index.html)

透過 sequelize 來達成 DB Schema Migration：[https://hackmd.io/@TSMI_E7ORNeP8YBbWm-lFA/ryCtaVW_M?print-pdf#%E4%BD%BF%E7%94%A8sequelize%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%BC%B5user-table](https://hackmd.io/@TSMI_E7ORNeP8YBbWm-lFA/ryCtaVW_M?print-pdf#%E4%BD%BF%E7%94%A8sequelize%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%BC%B5user-table)

認識同步與非同步 — Callback + Promise + Async/Await：[https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E5%BF%83%E5%BE%97-%E8%AA%8D%E8%AD%98%E5%90%8C%E6%AD%A5%E8%88%87%E9%9D%9E%E5%90%8C%E6%AD%A5-callback-promise-async-await-640ea491ea64](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E5%BF%83%E5%BE%97-%E8%AA%8D%E8%AD%98%E5%90%8C%E6%AD%A5%E8%88%87%E9%9D%9E%E5%90%8C%E6%AD%A5-callback-promise-async-await-640ea491ea64)

你懂 JavaScript 嗎？#23 Callback：[https://ithelp.ithome.com.tw/articles/10206555](https://ithelp.ithome.com.tw/articles/10206555)

How to create JOIN queries with Sequelize：[https://sebhastian.com/sequelize-join/](https://sebhastian.com/sequelize-join/)
