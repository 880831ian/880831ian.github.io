---
title: "Go (Golang) 介紹"
type: docs
weight: 9999
description: Go (Golang) 介紹
images:
  - golang/go-introduce/og.webp
date: 2022-03-21
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 什麼是 Go ?

Go 全名是 go programming language 又被稱為 Golang，是因為 go 這個詞太廣泛，不易搜尋，所以也可以叫 Golang。

Go 是由 Google 開發並維護的編譯程式語言，支援垃圾回收與併發，由於開發人之一也是 C 語言的作者，所以 Go 也繼承了許多 C 的風格
其特點有以下幾點：

* 靜態型別：因爲靜態型別的特性，可以在編譯期間就進行完整的型別檢查，可以找出大部分的型別錯誤。
* 編譯速度：因為 Go 語言先天優勢是架構設計非常單純，並不像物件導向語言龐大，在編譯時不用相依其他的 library，因此讓他有更好的執行效率。
* 語法簡潔：Go 關鍵字不多，不到30幾個，因為其關鍵字不少也與 C 的關鍵字重複，學習更容易上手。
* 垃圾回收：Go 有自動內存回收機制，不需要由開發人員來管理。垃圾回收是一種記憶體管理機制。當程式所佔用記憶體不再被該程式給存取時，會借助垃圾回收演算法，將記憶體空間歸還給作業系統。
* 原生支援併發：Go 語言支持併發，只需要透過 `go` 關鍵字來開啟 `goroutine` 將可。`goroutine` 是 Go 語言實現併發的一種方式，在執行的過程需要少量的記憶體，來暫存自己的上下文，就可在不同的時間點來分段執行程式。並且有 `channel` 可以跟 `goroutine` 進行資料溝通。
<br>

> [!NOTE] 靜態型別/動態型別
>靜態型別的意思是指當宣告一個變數時，你必須同時宣告此變數所存放的資料型態為何
> ```
>var age int // int
>var name string // string
> ```
>動態型別指的是程式執行時，系統才可以看見的型別，什麼型別都可以
> ```
>var i interface {}
>i = 18
>i = "Golang 程式設計"
> ```

> [!NOTE] 編譯式語言/直譯式語言
>靜態型別的意思是指當宣告一個變數時，你必須同時宣告此變數所存放的資料型態為何
>* 編譯式：當我們寫完程式時，我們需要將程式 compile (編譯) 成電腦看得懂的程式，再將程式拿去執行。
>* 直譯式：當我們寫完程式時，直接使用直譯器一行一行翻譯成電腦語言並執行。
>比較：
>1. 編譯式執行效率較佳
>2. 直譯式相對容易 Debug
>3. 編譯式語言編譯完後，可以直接在各類 OS 系統中執行，因為編譯完的程式，就是電腦看得懂的。

那我們大致了解 Go 後就來安裝 Go吧！

## 安裝 Go

本次作業系統為 macOS，所以後續都以 macOS 為主，如果是使用其他的作業系統，可以直接到[官網來下載](https://go.dev/doc/install)。

macOS 可以用 brew 等工具來下載，但這次我使用官網直接下載 pkg 來安裝。

<br>

{{< figure src="/golang/go-introduce/download.png" width="700" caption="Go 官網下載位置" >}}

<br>

下載完後，使用 `go version` 來檢查是否安裝成功：

```sh
$ go version
go version go1.18 darwin/amd64
```

<br>

接著我們來看一下他的環境變數，使用 `go env`，來查看：

```sh
$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/ian_zhuang/Library/Caches/go-build"
GOENV="/Users/ian_zhuang/Library/Application Support/go/env"
GOEXE=""
GOEXPERIMENT=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
GOMODCACHE="/Users/ian_zhuang/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="darwin"
GOPATH="/Users/ian_zhuang/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
```
這邊簡單的列出來，比較重要的是

* GOPATH：他是有關管理程式碼和套件執行檔的地方，在 Go 1.8 版本以前，GOPATH 預設為空。從 1.8 以後，Go 安裝完後，都會直接給預設的路徑

說一下這個路徑的內容，我在依照預設的路徑，像我是在 Users/使用者/go，在 go 底下新增三個資料夾：

* src：主要放置專案的地方
* pkg：套件主要儲存的資料夾
* bin：存放編譯好的執行檔案

在 Go 1.11 後提供了 go modules 讓我們不一定要把專案程式碼放在 `$GOPATH/src` 中做開發，因此我們先來設定我在要放專案的資料夾，打開 `.bash_profile`：

```sh
export GOPATH=你專案的路徑/goworkspace
$ source .bash_profile
```

接著我們來實作第一隻 Go 程式吧，我們會依照官網的示範，但為了要介紹 modules 是什麼，所以會小修改內容。

<br>

## 第一隻 Go 程式

我們依照官網的示範教學，先建立一個資料夾(要放在我們剛剛的 `GOPATH` 目錄下方歐)，來放我們第一個 Go 程式 "印出 Hello world" ([程式碼可以從此處下載](https://github.com/880831ian/go))：

```sh
mkdir helloworld
cd helloworld
```

<br>

接著下指令來新增 Go module：

```sh
$ go mod init helloworld
go: creating new go.mod: module helloworld
```

<br>

如果成功，會產生一個 go.mod 檔案，我們來看看內容有什麼：

```sh
$ cat go.mod
module helloworld

go 1.18
```
go.mod 是用來定義 module 的文件，用來標示此 module 的名稱、所使用的 go 版本以及相依的 Go module。


<br>

我們分別再新增兩個資料夾，以及兩個 `.go` 檔，來建立我們範例所需要的環境：

```sh
mkdir greeting cli
touch greeting/greeting.go cli/say.go
```

<br>

到目前為止結構如下：

```sh
.
├── cli
│   └── say.go
├── go.mod
└── greeting
    └── greeting.go
```

我們來修改一下 `greeting.go` 以及 `say.go` 程式碼吧。

`greeting.go` 是一個簡單的 package，用以顯示所傳入的字串 ; 而 `say.go` 則是以呼叫 `greeting.go` package 所提供的函式來顯示資料。

<br>

`greeting.go` 內容：

```go
package greeting

import "fmt"

func Say(s string) {
	fmt.Println(s)
}
```

<br>

`say.go` 內容：

```go
package main

import (
	"helloworld/greeting"
)

func main(){
	greeting.Say("Hello World")
}
```

<br>

順便來介紹一下程式裡面分別是什麼意思吧！

<br>

* Package：package 主要分成兩種，一個是可執行，另一個則是可重複使用的，而 `package main` 就是可執行的檔案，像我們上面這個有包含 package main 的檔案，在編譯時，就會產生一個 `say` 的執行檔，電腦就是依照此檔案執行的。


* Import：當我們寫程式時，一定會引入其他人寫的套件。而 Go 語言的標準函式庫為開發團隊先寫好，提供一些常用的功能，當然也可以使用其他第三方套件，還滿足內建以及標準函式庫的不足。我們在 `greeting.go` 裡面引入的 `fmt` 就是開發團隊寫好的，然而在 `say.go` 裡面引入的就是`greeting.go` ，我們就可以使用其內容的函式來做使用。


* Main Function：每個 Go 語言的專案基本上都會有一個主程式，主程式裡的程式通常都為最核心的部分。

<br>

最後使用 `go run say.go` 來將此程式運行起來：

```sh
$ go run say.go
Hello World
```

就可以看到程式成功將 Hello World 給印出來拉！

<br>

### 常見指令

接下來要簡單介紹一下常用的另外3個指令，分別是 `go build`、`go install`、`go clean`：

<br>

`go get`：來下載套件到當前的模組，並安裝他們

```sh
$ go get github.com/fatih/color
go: downloading github.com/fatih/color v1.13.0
.... 省略 ....
```

<br>

`go build`：還記得我們前面說 Go 是編譯式程式，所以我們可以將程式用 `go build` 來編譯成電腦看得懂的執行檔歐，檔案會存放在當前目錄或是指定目錄中 ~

```sh
$ ls
go.mod

$ go build cli/say.go
go.mod   say

$ ./say
Hello World
```
多的這個 say 就是編譯後的執行檔，將他執行會顯示跟我們使用 run 來運行的一樣，顯示 Hello World。

<br>

`go install`：如果編譯沒有錯誤，一樣跟 build 會產生執行檔，不同的是，會將執行檔，產生於 $GOPATH/bin 內。

```sh
$ ls /Users/ian_zhuang/go/bin
dlv          go-outline   gomodifytags goplay       gopls        gotests      impl         staticcheck

$ go install

$ ls /Users/ian_zhuang/go/bin
dlv          go-outline   gomodifytags goplay       gopls        gotests      hello        impl         staticcheck
```

<br>

`go clean`：執行後會將 build 產生的檔案都刪除 (install 的不會)

```sh
$  ls
go.mod   hello    .

$ go clean

$ ls
go.mod   .
```

<br>

### 套件相依性管理

Go modules 提供的另一個方便的功能則是套件相依性管理，接下來實際透過以下指令來安裝套件：

```sh
$ go get github.com/fatih/color
go: downloading github.com/fatih/color v1.13.0
go: downloading github.com/mattn/go-isatty v0.0.14
go: downloading github.com/mattn/go-colorable v0.1.9
go: downloading golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c
go: added github.com/fatih/color v1.13.0
go: added github.com/mattn/go-colorable v0.1.9
go: added github.com/mattn/go-isatty v0.0.14
go: added golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c
```

<br>

安裝成功，可以再查看一下 go.mod：

```go
module github.com/880831ian/go/helloworld

go 1.18

require github.com/fatih/color v1.13.0

require (
	github.com/mattn/go-colorable v0.1.12 // indirect
	github.com/mattn/go-isatty v0.0.14 // indirect
	golang.org/x/sys v0.0.0-20220319134239-a9b59b0215f8 // indirect
)
```
會多了下面這些 `require github.com/fatih/color v1.13.0` 代表目前專案使用 v1.13.0 版本的 `github.com/fatih/color`

下面的 indirect 指的是被相依的套件所使用的 package

<br>

接著我們將 `greeting.go`、`say.go` 兩個檔案修改一下，使用我們剛剛所安裝的 package：

<br>

`greeting.go`

```go
package greeting

import (
	"fmt"

	"github.com/fatih/color"
)

func Say(s string) {
	fmt.Println(s)
}

func SayWithRed (s string) {
	color.Red(s)
}

func SayWithBlue (s string) {
	color.Blue(s)
}

func SayWithYellow (s string) {
	color.Yellow(s)
}
```
我再多 import 了剛剛的 `github.com/fatih.color`，並使用該套件的函式 `color` 來分別顯示 `Red`、`Blud`、`Yellow` 三種顏色。

<br>

`say.go`

```go
package main

import (
	"github.com/880831ian/go/helloworld/greeting"
)

func main(){
	greeting.Say("Hello World")
	greeting.SayWithRed("Hello World")
	greeting.SayWithBlue("Hello World")
	greeting.SayWithYellow("Hello World")
}
```
我們將 `greeting` 三種顯示顏色的函式帶入。

<br>

一樣我們來運行一下程式，來看看結果如何，這次我們直接編譯，使用 `go build` 來編譯，最後直接執行產生的執行檔：

```sh
$ go build cli/say.go
$ ./say
Hello World
Hello World //紅色
Hello World //藍色
Hello World //黃色
```
由於 Makedown 沒辦法於程式碼區域顯示正確顏色，用註解標示一下XD

<br>


## 變數

在使用變數做宣告時，要注意以下幾個 Go 保留字，不能拿來當變數名稱，其 Go 有三種宣告的方式：

<br>

{{< figure src="/golang/go-introduce/variable.png" width="700" caption="Go 保留字，不能拿來當變數名稱" >}}

<br>

### 使用 `:=` 來宣告

表示之前沒有進行宣告過。這是 Go 中最常見的變數宣告方式，但不能用縮寫方式來定義變數 (`foo := bar`) ，因為 package scope 的變數都是以 keyword 作為開頭。且只能在 function 中使用。

```go
func main (){
	a := "bar"
	b := 4
	c := true
	// 也可以簡寫成這樣
	d,e,f := "bar",4,true

	fmt.Println(a,b,c);
	fmt.Println(d,e,f);
}
```

```sh
$ go run .
bar 4 true
bar 4 true
```

<br>

### 先宣告資料型態

當不知道變數的起始值，或是需要在 package scope 中宣告變數時可以使用。

```go
var a string
var b int

// 也可以簡寫成這樣
var (
	c string
	d float64
)
func main (){
	a = "Hello"
	b = 123
	c = "ian"
	d = 3.5
	fmt.Println(a,b,c,d)
}
```
⚠️ 不建議把變數寫在全域變數中 ⚠️

```sh
$ go run .
Hello 123 ian 3.5
```

<br>

### 直接宣告並賦值

```go
func main (){
	var (
		a string = "Hello"
		b int = 9999
	)
	fmt.Println(a,b)
}
```

```sh
$ go run .
Hello 9999
```

<br>

### 常見宣告錯誤

#### 重複宣告變數

```go
func main (){
	name := "ian"
	name := "pinyi"
}
```

```sh
$ go run .
# github.com/880831ian/go/test
./test.go:4:2: name declared but not used
./test.go:5:7: no new variables on left side of :=
```

<br>

#### 在 main 函式外賦值

```go
var a int
b := "Hello"

func main (){
	fmt.Println(a,b)
}
```

```sh
$ go run .
# github.com/880831ian/go/test
./test.go:6:1: syntax error: non-declaration statement outside function body
```
我們可以在 main 函式外宣告變數，但無法在 main 函式外賦值

<br>

#### 沒有宣告就使用變數

```go
func main (){
	a = 123
	b = true
	fmt.Println(a,b)
}
```

```sh
$ go run .
# github.com/880831ian/go/test
./test.go:6:2: undefined: a
./test.go:7:2: undefined: b
./test.go:8:14: undefined: a
./test.go:8:16: undefined: b
```

<br>

## Go 資料型態

我們在學習 Go 語言之前，要先了解一下基本的資料型態，可以簡單分為 字串、字符、整數、浮點數、布林值、映射

### 字串 String

在 Go 語言中，字串必須用雙引號給匡起來，也可以使用反引號來宣告。用雙引號刮起來的字串不能包含換行，但可以包含跳脫字元，例如 `\n` 、`\t` 。由於反引號內包含的是原始字串，可以跨越多行，所以跳脫符號在原始字串中沒有任何含義。

```go
func main() {
   var name = "ian"
   fmt.Printf("資料型態 name : %v(%T)\n", name,name)
   var address = `台中市太平區`
   fmt.Printf("資料型態 address : %v(%T)\n", address,address)
}
```

```sh
$ go run .
資料型態 name : ian(string)
資料型態 address : 台中市太平區(string)
```

<br>

### 字符 Character

字串中的每一個元素叫做字符，字符會使用單引號匡起來，像是 `"abc"` 這個字串，其中 `'a'`、`'b'`、`'c'` 就是字符，可以從字串元素中來獲得字符。

Go 語言的字符有以下兩種：
* 一種叫 byte 類型，可以叫做 uint8 類型，代表 ASCll 碼的一個字符。
* 另一種是 rune 類型，代表一個 UTF-8 字符，當需要處理中文、日文或是其他複合字符時，就會使用到 rune 類型。rune 類型等於 int32 類型。

```go
func main (){
	var a byte = 'A'
	var b rune = '嗨'
	fmt.Printf("%c %c\n",a,b) //%c 所表示的字符
	fmt.Printf("%d(%T) %d(%T)\n",a,a,b,b) //%d 十進制表示,%T 輸出型態
	fmt.Printf("%x %x\n",a,b) //%x 十六進制表示
	fmt.Printf("%U %U\n",a,b) //%U 輸出格式為 Unicode 格式:U+hhhh的字串
}
```

```sh
$ go run .
A 嗨
65(uint8) 21992(int32)
41 55e8
U+0041 U+55E8
```

<br>

### 整數 Integer

整數用於儲存整數。 Go 具有多種大小不一的內建整數型別，用來儲存有號數和無號數。

* 有號數

| 型別 | 大小 | 範圍 |
| :---: | :---: | :---: |
| int | 取決平台 | 取決平台 |
| int 8 | 8 bits | -128 到 127 |
| int 16 | 16 bits | -2^15 到 2^15-1 |
| int 32 | 32 bits | -2^31 到 2^31-1 |
| int 64 | 64 bits | -2^63 到 -2^63-1 |

* 無號數

| 型別 | 大小 | 範圍 |
| :---: | :---: | :---: |
| uint | 取決平台 | 取決平台 |
| uint 8 | 8 bits | 0 到 255 |
| uint 16 | 16 bits | 0 到 2^16-1 |
| uint 32 | 32 bits | 0 到 2^32-1 |
| uint 64 | 64 bits | 0 到 -2^64-1 |

`int` 、 `uint` 的型別大小取決於平台。在 32 位元系統上，它大小為 32 位元，64 位元系統上，它的大小為 64 位元。

使用整數值時，除非確定會用到的大小跟範圍才使用有號數及無號數，否則應該都使用 `int` 資料型別。

```go
func main() {
	var myInt8 int8 = 97
	var myInt = 1200
	var myUint uint = 500
	var myOctalNumber = 034
	var myHexNumber = 0xFF
	fmt.Printf("%d, %d, %d, %#o, %#x\n", myInt8, myInt, myUint, myOctalNumber, myHexNumber)
}
```

```sh
$ go run .
97, 1200, 500, 034, 0xff
```

在 Go 中，也可以使用前綴 `0` 來宣告八進制數字，或是使用 `0x` 來宣告十六進制數字。

<br>

### 浮點數 Float

浮點數型別用於儲存小數部分的數字。Go 有兩種浮點數型別：

* float32：在記憶體中佔用32位元，並以單精度浮點數格式儲存。
* float64：在記憶體中佔用64位元，並以雙精度浮點數格式儲存。

浮點數的預設是 `float64`，除非初始化有為浮點數變數指定型別，否則編譯器將判定為 `float64`。

```go
func main() {
	var a = 245.4664
	var b float32 = 1452.34
	fmt.Printf("%f(%T)\n%f(%T)\n", a,a,b,b)
}
```

```sh
$ go run .
245.466400(float64)
1452.339966(float32)
```

<br>

### 布林值 Bool

Go 提供了一種稱為 `bool` 的資料型別來儲存布林值。它有兩個可能的值：`true` 和 `false`。

```go
func main() {
	var a = true
	var b bool = false
	fmt.Printf("%v(%T)\n%v(%T)\n", a,a,b,b)
}
```

```sh
$ go run .
true(bool)
false(bool)
```

<br>

布林型別也可以使用運算子 `&&` (與,and)、`||` (和,or)、`!` (否定)

```go
func main() {
	var a = 4 <= 7
	var b = 10 != 10
	var c = 10 > 20 && 5 == 5
	var d = 2 * 2 == 4 || 10 / 3 == 3
	fmt.Printf("%v\n%v\n%v\n%v\n", a,b,c,d)
}
```

```sh
$ go run .
true
false
false
true
```

<br>

### 映射 Map

映射 (Map) 是 Go 內建的類型，是一種鍵值(key-value)的集合，可以透過 key 快速查詢並找到數據。

```go
func main (){
	var map3 = map[int]string{99 : "Go", 87 : "Python", 79 : "Java", 93: "Html"}
	fmt.Println(map3)
	fmt.Println("map3[99] =",map3[99],"map3[79] =",map3[79])
	map3[79] = "PHP"
	fmt.Println("修改數據後，map3[99] =",map3[99],"map3[79] =",map3[79])
}
```

```sh
map[79:Java 87:Python 93:Html 99:Go]
map3[99] = Go map3[79] = Java
修改數據後，map3[99] = Go map3[79] = PHP
```

<br>


### 數字型別的運算

Go 提供了多種用於數字、浮點數型別執行運算的運算子

* 算術運算子：`+`、`-`、`*`、`/`、`%`
* 比較運算子：`==`、`!=`、`<`、`>`、`<=`、`>=`
* 位元運算子：`&`、 `|`、 `^`、 `<<`、`>>`
* 遞增和遞減運算子：`++`、`--`
* 賦值運算子：`+=`、`-=`、`*=`、`/=`、`%=`、`<<=`、`>>=`、`&=`、`|=`、`^=`

```go
import (
	"fmt"
	"math"
)

func main() {
	var a, b = 4, 5
	var res1 = (a + b) * (a + b) / 2
	a++
	b += 10
	var res2 = a ^ b
	var r = 3.5
	var res3 = math.Pi * r * r
	fmt.Printf("res1 : %v, res2 : %v, res3 : %v\n", res1, res2, res3)
}
```

```sh
res1 : 40, res2 : 10, res3 : 38.48451000647496
```

<br>

### 型別轉換

Go 有一個強型別系統，它不允許混合型別。舉例來說：不能把 `int` 變數加到 `float64` 變數中，也不能將 `int` 變數加到 `int64` 變數中。

```go
func main() {
	var a int64 = 4
	var b int = int(a)
	var c int = 500
	fmt.Println(a, b,a+c)
}
```

```sh
$ go run .
# command-line-arguments
./.:11:19: invalid operation: a + c (mismatched types int64 and int)
```
就會跳出錯誤說明別不同，無法直接做運算，那要怎麼辦呢！？

<br>

使用型別轉換，將型別值轉成相同的

```go
func main() {
	var a int64 = 4
	var b int = int(a)
	var c int = 500
	fmt.Println(a, b,int(a)+c)
}
```

```sh
$ go run .
4 4 504
```
一般將值 `v` 轉換為型別 `T` 的語法是 `T(V)` 。

<br>

## Go 資料結構

### 指標 Pointer

指標是程式語言中的資料結構及其物件或變數，用來表示或儲存記憶體位址，這個位址的值直接指向存在該位址物件的值。

Go 支持指標，指標的聲明方式為 *T，可以藉由變數名稱前面加 `&` 來獲得變數的位址，由於 Go 支持 GC ，在 Go 語言中不支持指標的運算。

表示法
* 使用 `&` 來獲得指標位址
* 使用 `*` 來獲得指標所指向的值

```go
func main (){
	var a int = 2
	fmt.Println("a 位址 = ",&a)
	fmt.Printf("a 的值 = %v\n",a)
 	var pInt *int = &a
	fmt.Printf("pInt = %v\n", pInt);
	fmt.Printf("pInt 位址 = %v\n",&pInt);
	fmt.Printf("pInt 指向的值 = %v\n",*pInt);
	}
```

```sh
$ go run .
a 位址 =  0xc0000b2008
a 的值 = 2
pInt = 0xc0000b2008
pInt 位址 = 0xc0000ac020
pInt 指向的值 = 2
```

<br>

### 陣列 Array

我們已經學會要怎麼宣告變數，以及如何使用變數來儲存值。但當我們今天想要儲存多個數值，使用原本的方式，需要創建許多變數才能儲存，因此有了陣列可以來儲存大量資料。

陣列 (Array) 是由同類型的元素集合所組成的資料結構，分配一塊連續的記憶體來儲存。利用元素的索引可以計算出該元素所對應的儲存位址。

```go
func main (){
	var a [2] float32
	a [0] = 1.4
	a [1] = 3.14

	// 也可以寫成這樣
	var b = [] int{10,20,99,333}

	fmt.Println(a,b)
	fmt.Println(len(a),len(b))
}
```

```sh
$ go run .
[1.4 3.14] [10 20 99 333]
2 4
```

<br>
上面有提到他會分配連續記憶體來儲存，我們來看看他是怎麼存的！

```go
func main (){
	a := [...]int{1, 2, 3}
	fmt.Printf("a 的記憶體分配位置 %p \n", &a)
	fmt.Printf("陣列 a 的索引 0 記憶體分配位置 %p \n", &a[0])
	fmt.Printf("陣列 a 的索引 1 記憶體分配位置 %p \n", &a[1])
	fmt.Printf("陣列 a 的索引 2 記憶體分配位置 %p \n", &a[2])
}
```

```sh
$ go run .
a 的記憶體分配位置 0xc0000180f0
陣列 a 的索引 0 記憶體分配位置 0xc0000180f0
陣列 a 的索引 1 記憶體分配位置 0xc0000180f8
陣列 a 的索引 2 記憶體分配位置 0xc000018100
```

<br>

### 切片 Slice

前面提到陣列的使用，陣列使用上是實值類型以及陣列長度不可變的情況下，間接限制了使用場景。

切片 (slice) 是 Go 對陣列在進行一層的封裝，是一個擁有相同類型元素的可變長度序列，可以非常靈活運用，自動擴容，可以快速且方便的操作數據集合。

```go
func main (){
	a := [5]int{55, 75, 58, 60, 66}
	b := a[1:4] //基於 a 陣列創建切片，等於 b 包含 a[1],a[2],a[3]
	fmt.Printf("%v(%T)\n",b,b)
	fmt.Printf("len = %v\n",len(b))
	fmt.Printf("cap = %v\n",cap(b))
}
```

```sh
[75 58 60]([]int)
len = 3
cap = 4
```
len(b) 表示可見元素有幾個(直接打印元素看到的元素個數)，而 cap(b) 表示所有元素有幾個。
[1:4] 代表從第二個元素開始 (0為第一個元素，1位第二的元素)，取到第4個元素 (下標為 4-1=3，下標3代表第四個元素)

<br>

使用 `make` 創建切片

```go
func main (){
	a := make([]int,5,10) //創建長度 5,容量 10 的切片
	fmt.Printf("a = %v, len(a) = %v, cap(a) = %v\n",a,len(a),cap(a))
}
```

```sh
a = [0 0 0 0 0], len(a) = 5, cap(a) = 10
```
使用 make 創建長度5 ,容量 10 的切片

<br>

使用 `append` 來達成新增元素

```go
func main (){
	a := make([]int,2,2) //創建長度 5,容量 10 的切片
	fmt.Printf("a = %v, len(a) = %v, cap(a) = %v\n",a,len(a),cap(a))
	fmt.Printf("指標為 %p\n", a)
	a = append(a, 10)
	fmt.Printf("a = %v, len(a) = %v, cap(a) = %v\n",a,len(a),cap(a))
	fmt.Printf("擴容後指標 = %p 改變\n", a)
}
```

```sh
a = [0 0], len(a) = 2, cap(a) = 2
指標為 0xc000014080
a = [0 0 10], len(a) = 3, cap(a) = 4
擴容後指標 = 0xc000022080 改變
```

<br>

### 結構 Struct

我們介紹的變數都是儲存單一的值或是多個相同型態的值，那如果要用變數表示較複雜的概念，像是紀錄一個人的名字、年齡或是身高時，由於這些是不同的**資料型態**，所以要記錄下來時，就必須使用不同的容器，這裡會介紹 Go 語言中的結構 Struct：


**建立結構**

```go
type Student struct {
    Id int
    Name string
    Score float64
}

func main() {
    student := Student{1, "ian", 89.4}
    fmt.Println(student)
}

```

```sh
$ go run .
{1 ian 89.4}
```

我們先宣告一個名為 Student 的 `struct` 結構，裡面屬性有 Id 和 Name 以及 Score，再以此結構宣告一個變數 student 並填入屬性，以顯示不同資料型態的值。

<br>

## Go 控制流程

### if

結構裡會有一個條件，這個條件是個布林值，如果為 true，則會執行括號裡的程式碼，相反的，如果為 false，則會直接跳過：

```go
func main (){
	var x = 2
	var y = 1

	fmt.Println("x = ",x,",y = ",y)

	if x==y {
		fmt.Printf("%v 等於 %v\n",x,y)
	}
	fmt.Printf("%v 不等於 %v\n",x,y)
}
```

```sh
$ go run .
x =  2 ,y =  1
2 不等於 1
```

<br>

### switch

switch 其目的是簡化 if 的條件，swtich 會檢查符合的條件，並且執行條件內的程式碼，如果都沒有符合，則會執行 default 內的程式碼：

```go
func main (){
	x := 12
	switch {
	case x < 10:
		fmt.Printf("%v 小於 10\n",x)
	case x < 20:
		fmt.Printf("%v 大於等於10, 小於 20\n",x)
	default:
		fmt.Printf("%v 大於 20\n",x)
	}
}
```

```sh
$ go run .
12 大於等於10, 小於 20
```

<br>

## Go 迴圈

迴圈是每個程式語言必備的函式，藉以迴圈來達成反覆或是循環的動作。而 Go 語言的迴圈只有使用 `for` 迴圈來表達三種不同的迴圈 (for、while、loop)：

```go
func main (){
	for i := 0 ; i <= 10 ; i++ {
		fmt.Println(i)
	}
}
```

```sh
$ go run .
0
1
2
3
4
5
6
7
8
9
10
```

<br>

**break**

也可以使用 `break`，依條件需求讓迴圈提早跳出結束：

```go
func main (){
	for i := 0 ; i <= 5 ; i++ {
		if i > 3 {
			break
		}
		fmt.Println(i)
	}
}
```

```sh
$ go run .
0
1
2
3
```

<br>

**Continue**

或是使用 `continue` 若符合條件，便會跳過到此迴圈，直接進入下個迭代：

```go
func main (){
	for i := 1; i <= 10; i++ {
		if i%2 == 0 {
			continue
		}
		fmt.Println(i)
	}
}
```

```sh
$ go run .
1
3
5
7
9
```

<br>

**GoTo**

使用 goto 可以無條件轉移到程式中指定的行

```go
func main (){
	fmt.Println("1")
	fmt.Println("2")
	goto labele1
	fmt.Println("3")
labele1:
	fmt.Println("4")
	fmt.Println("5")
	fmt.Println("6")
}
```

```sh
$ go run .
1
2
4
5
6
```

<br>

## Go 方法 Method

Go 語言不像 Python 有 class ，但還是有提供可以在某種型態上定義方法(method)，method 其實是作用在接收器 (receiver) 上的一種函式，接收器要是某一種型別的變數，所以其實 method 也算是一種特殊型別的函式。

```go
type Vertex struct {
	x,y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.x * v.x + v.y * v.y)
}

func main() {
	v := Vertex{5,12}
	fmt.Println(v.Abs())
}
```

```sh
$ go run .
13
```
一開始先宣告一個名為 Vertex 結構型態，裡面的屬性包含 ｘ、y (float64)，接著就是撰寫一個 method 了，這個 method 是以 Vertex 作為接收器， method 名稱是 Abs，最後回傳浮點數，接著 method 裡頭，即為對接收器的運算並回傳值。

<br>

我們來看看如果使用 function 要怎麼來達成

**方法 (Method) vs 函式 (Function)**

```go
type Vertex struct {
	x, y float64
}

func Abs(v Vertex) float64{
	return math.Sqrt(v.x * v.x + v.y * v.y)
}

func main () {
	v := Vertex{5,12}
	fmt.Println(Abs(v))
}
```

```sh
$ go run .
13
```

<br>

## Go 介面 Interface

Interface 的概念有點像是藍圖，先定義某個方法的名稱 (function name)、會接收到的參數以及型別 (list of argument types)、會回傳的值與型別 (list of return types)。定義好藍圖之後，並不用去管實作的細節，實作的細節會由每個型別自行定義實作。

**empty interface**

沒有定義任何方法的 interface 稱作 empty interface，由於所有的 types 都能夠實作 empty interface，因此它的值會是 any type：(因為目前沒有被賦值，所以都會回傳 `nil`)

```go
type value interface{}

func main() {
    var v value
    describe(v) // (<nil>, <nil>)

    v = 42
    describe(v) //(42, int)

    v = "hello"
    describe(v) // (hello, string)

}

func describe(v value) {
    fmt.Printf("(%v, %T)\n", v, v)
}
```

```sh
$ go run .
value is <nil>
type is <nil>
```

<br>



```go
type Person interface {
    getFullName() string
    getSalary() int
}

type Employee struct {
    firstName string
    lastName  string
    salary    int
}

func (e Employee) getFullName() string {
    return e.firstName + " " + e.lastName
}

func (e Employee) getSalary() int {
    return e.salary
}

func main() {
    var p Person = Employee{"ian", "Zhuang", 2000}
    fmt.Printf("full name : %v ,Salary : %v\n", p.getFullName(),p.getSalary())
}
```

```sh
$ go run .
full name : ian Zhuang ,Salary : 2000
```

<br>

## Goroutine

有人把 Go 比作 21 世紀的 C 語言，第一是因為 Go 語言設計簡單，第二，21 世紀最重要的就是並行程式設計，而 Go 從語言層面就支援了併發。

Goroutine 是 Go 語言實現併發的一種方式。

Goroutine 是一種非常輕量級的執行緒，它是Go語言併發設計的核心。執行 Goroutine 只需極少的記憶體，可同時執行成千上萬個併發的任務。

<br>

### 單執行緒

我們先來看看一般的單執行緒

```go
func say(s string) {
    for i := 0; i < 2; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    say("world")
    say("hello")
}
```

```sh
$ go run .
world
world
hello
hello
```
在單執行緒下，每行程式碼都會依照順序執行。

<br>

{{< figure src="/golang/go-introduce/single-thread.png" width="400" caption="單執行緒 ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

### 多執行緒

在多執行緒下，最多可以同時執行與 CPU 數相等的 Goroutine。要如何使用多執行緒，使用 goroutine 來執行多併發，只要使用 `go` 這個關鍵字來執行 func 就可以了 !

```go
func say(s string) {
    for i := 0; i < 2; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    go say("world")
    say("hello")
}
```

```sh
$ go run .
hello
world
world
hello
```
如此一來, `say("world")` 會跑在另一個執行緒(Goroutine)上，使其並行執行。

<br>

{{< figure src="/golang/go-introduce/multi-thread.png" width="300" caption="多執行緒 ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

### 等待

多執行緒下，經常需要處理的是執行緒之間的狀態管理，其中一個經常發生的事情是等待。

例如A執行緒需要等B執行緒計算並取得資料後才能繼續往下執行，在這情況下等待就變得十分重要。我們會介紹三個等待的方式，並說明其缺點。

* time.sleep：休眠指定時間
* sync.WaitGroup：等待直到指定數量的 Done() 呼叫
* Channel 阻塞： Channel 阻塞機制，使用接收時等待的特性避免執行緒繼續執行

#### time.sleep

使 Goroutine 休眠，讓其他的 Goroutine 在 main 結束前有時間執行完成。

```go
func say(s string) {
    for i := 0; i < 2; i++ {
        fmt.Println(s)
    }
}

func main() {
    go say("world")
    go say("hello")

    time.Sleep(5 * time.Second)
}
```

```sh
$ go run .
world
world
hello
hello
```
缺點：休息指定時間可能會比 Goroutine 需要執行的時間長或短，太長會耗費多餘的時間，太短會使其他 Goroutine 無法完成。

<br>

{{< figure src="/golang/go-introduce/timesleep.png" width="300" caption="time.sleep ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

#### sync.WaitGroup

```go
func say(s string, wg *sync.WaitGroup) {
    defer wg.Done()

    for i := 0; i < 2; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    wg := new(sync.WaitGroup)
    wg.Add(2)

    go say("world", wg)
    go say("hello", wg)

    wg.Wait()
}
```

```sh
$ go run .
hello
world
world
hello
```
在程式碼尾端加上 wg.Wait()，需要讓他達成一些條件，才可以往後執行，而這個條件，就是收到 wg.Done() 的呼叫次數。而這個次數，即是 wg.Add(2) 裡的數字2。

* 優點：避免時間預估的錯誤
* 缺點：需要手動配置對應的數量

<br>

{{< figure src="/golang/go-introduce/wait-group.png" width="400" caption="sync.WaitGroup ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

#### Channel 阻塞

Channel 原為 Goroutine 溝通時使用的，但因其阻塞的特性，使其可以當作等待 Goroutine 的方法。

```go
func say(s string, c chan string) {
    for i := 0; i < 2; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
    c <- "FINISH"
}

func main() {
    ch := make(chan string)

    go say("world", ch)
    go say("hello", ch)

    <-ch
    <-ch
}
```

```sh
$ go run .
hello
world
hello
world
```
兩個 Goroutine ( `say("world", ch)` 、 `say("hello", ch)` ) ，因此需要等待兩個 FINISH 推入 Channel 中才能結束 Main Goroutine。

* 優點：避免時間預估的錯誤、語法簡潔

<br>

{{< figure src="/golang/go-introduce/channel-wait.png" width="400" caption="Channel 阻塞 ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

## Channel

Channel 有兩個強大的處理能力，等待以及變數共享。Channel 可以想成一條管線，這條管線可以推入數值，並且也可以將數值拉取出來。

因為 Channel 會等待至另一端完成推入/拉出的動作後才會繼續往下處理，這樣的特性使其可以在 Goroutines 間可以同步的處理資料或是剛剛提到的等待

<br>

* Channel 基本操作

```go
	ch := make(chan int)    //創建一個Channel ch
	ch <- u                 //將值u傳送到 Channel ch裡
	v := <- ch              //從Channel ch中接收數據 ，並且將其賦值給變數v
	close(ch)               //關閉channel
```

<br>

**多執行緒下的共享變數**

在執行緒間使用同樣的變數時，最重要的是確保變數在當前的正確性，在沒有控制的情況下極有可能發生問題。

```go
func main() {
    total := 0
    for i := 0; i < 1000; i++ {
        go func() {
            total++
        }()
    }
    time.Sleep(time.Second)
    fmt.Println(total)
}
```

```sh
$ go run .
986
```

為什麼會明明是用for 跑 1000 累加，但最後只有 900 多呢？我們來看一下下面的例子

<br>

{{< figure src="/golang/go-introduce/total-error.png" width="500" caption="多執行緒下的共享變數 - 錯誤 ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

假設目前加到28，在多執行緒的情況下：

goroutine1 取值 28 做運算

goroutine2 有可能在 goroutine1 做 total++ 前就取 total 的值，因此有可能取到 28

這樣的情況下做兩次加法的結果會是 29 而非 30

在多個 goroutine 裡對同一個變數total做加法運算，在賦值時無法確保其為安全的而導致運算錯誤，此問題稱為 競爭危害 (Race condition)。

<br>

**使用 Channel 來保證變數的安全性**

```go
func main() {
    total := 0
    ch := make(chan int, 1)
    ch <- total
    for i := 0; i < 1000; i++ {
        go func() {
            ch <- <-ch + 1
        }()
    }
    time.Sleep(time.Second)
    fmt.Println(<-ch)
}
```

```go
$ go run .
1000
```
goroutine1 拉出 total 後，Channel 中沒有資料了

因為 Channel 中沒有資料，因此造成 goroutine2 等待

goroutine1 計算完成後，將 total 推入 Channel

goroutine2 等到 Channel 中有資料，拉出後結束等待，繼續做運算

<br>

{{< figure src="/golang/go-introduce/total-channel.png" width="500" caption="多執行緒下的共享變數 - Channel ([Go 的並發：Goroutine 與 Channel 介紹](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html))" >}}

<br>

## 參考資料

Go 官網：[https://go.dev/doc/tutorial/getting-started](https://go.dev/doc/tutorial/getting-started)

golang後端入門分享：[https://ithelp.ithome.com.tw/users/20137500/ironman/4184](https://ithelp.ithome.com.tw/users/20137500/ironman/4184)

從一知半解到略懂 Go modules：[https://myapollo.com.tw/zh-tw/golang-go-module-tutorial/](https://myapollo.com.tw/zh-tw/golang-go-module-tutorial/)

30天就Go(3)：操作指令及Hello World!：[https://ithelp.ithome.com.tw/articles/10186546](https://ithelp.ithome.com.tw/articles/10186546)

Golang 基本型別、運算子和型別轉換：[https://calvertyang.github.io/2019/11/05/golang-basic-types-operators-type-conversion/](https://calvertyang.github.io/2019/11/05/golang-basic-types-operators-type-conversion/)

Go語言字符類型（byte和rune）：[http://c.biancheng.net/view/18.html](http://c.biancheng.net/view/18.html)

golang初探：[https://ithelp.ithome.com.tw/users/20129671/ironman/3326](https://ithelp.ithome.com.tw/users/20129671/ironman/3326)

Go 的並發：Goroutine 與 Channel 介紹：[https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html](https://peterhpchen.github.io/2020/03/08/goroutine-and-channel.html)