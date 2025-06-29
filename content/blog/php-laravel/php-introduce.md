---
title: "PHP 介紹"
type: docs
weight: 9999
description: PHP 介紹
images:
  - php-laravel/php-introduce/og.webp
date: 2022-02-22
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

## 1. PHP 是什麼
PHP 全名是超文本前處理器(Hypertext Preprocessor)，是一種開源的通用[電腦手稿語言](https://zh.wikipedia.org/wiki/%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%80)

根據W3Techs的報告，截至2021年9月，有78.9%網站都是使用PHP，像是著名的 FaceBook、Tesla、Slack、WordPress

[stackshare](https://stackshare.io/php) 是分享開發工具的網站，可以看到不少公司也是使用PHP來進行開發的

其優點是

* 是開源、免費的
* 是跨平台的開發語言，Linux、Windows、macOS也可以使用
* 能開發：動態網站、爬蟲程式、WordPress 外掛佈景主題
* 也可以結合多種的資料庫，例如：Mysql、Mariadb、Oracle


過去使用 PHP 開發網頁程式，就是有什麼寫什麼，程式邏輯和網頁顯示混在一起，雖然開發方便但會造成高維護成本，和難以擴充功能

* 物件導向程式設計
* 函數式編程
* 事件驅動程式開發

PHP 有各式各樣的 Web Framework：

* Laravel - 最受歡迎的PHP Web Framewrok
* Codelgniter - 最新版的Codelgniter4 只有 1.4 MB，小而強

### php-cli vs php-fpm

#### CGI

CGI 是一種協定，為了保障 web server 傳過來的資料是標準格式

* 如果請求 index.html ，web server 會先去找這個文件，在丟給瀏覽器，但這個僅限靜態文件而已

* 如果請求 index.php ，就需要去找 php 的解析器來處理，那處理中一定會傳遞一些資料，像是 post 或是 url 還有 http header等，CGI 就是規定要傳哪些資料、以及怎麼樣的格式

#### FastCGI

* 是用來提高 CGI 處理性能用的

#### PHP-cli vs PHP-fpm

* PHP-cli 可以直接在命令列使用php命令來處理或顯示 php檔案，因為他內建了一個 HTTP 伺服器，可以提供 HTTP 服務

* PHP-fpm 是一個多進程架構的 FastCGI 服務，內建了 PHP 的解析器，配合Nginx來使用


## 2. PHP 基本語法

### 標籤

PHP 程式可以放置在檔案中的任何位置，其檔案副檔名是```.php```，

* PHP的標籤是開頭 < ?php 以及 ? > 結尾

```php
<?php
[ 程式碼 ]
?>
```
* PHP 不分大小寫，例如 if、else、while、echo，函數等，但**變數有區分大小寫**


```php
$color = 'blue';
echo "My car is " . $color . "<br>";
ECHo "My car is " . $cOLOr . "<br>";
eCHo "My car is " . $COLOR . "<br>";
```

```html
My car is blue
My pen is
My dog is
```


### 註解

PHP 程式中可以使用註解功能，註解後該段程式不會被執行

* PHP 單行註解

```php
// 註解的用意是可以告訴自己或是閱讀該程式的人該行程式碼的用意或想法
```

* PHP 多行註解


```php
/*
註解的用意是可以告訴自己或是閱讀該程式的人該行程式碼的用意或想法
echo "My car is " . $color . "<br>";
ECHo "My car is " . $cOLOr . "<br>";
eCHo "My car is " . $COLOR . "<br>";
*/
```

### 變數

PHP 程式中可以使用變數功能，變數是儲存訊息的容器，變數以" ```$``` "符號開頭，後面是變數名稱

PHP 變數規則

* 變數名必須以字母或是下底線開頭，不可以**用數字開頭**
* 變數名只包含字母數字或是下底線(A-z、0-9、_)
* 變數名區分大小寫 ($age 跟 $AGE 是兩個不同的變數)


```php
$color = 'blue';
$items = 'Toy';
$num1 = '13';
$num2 = '18';
echo "I have " . $num1+$num2 . " " . $color . " " . $items;
```

```html
I have 31 blue Toy
```

### 變數作用區

PHP 變數有不同的作用區(local、global、static)

#### local

函式中的變數具有local scope，只能用於該函式中使用

```php
	<?php
	function myTest() {
	  $x = 5; // local scope
	  echo "<p>Variable x inside function is: $x</p>";
	}
	myTest();

	// using x outside the function will generate an error
	echo "<p>Variable x outside function is: $x</p>";
	?>
```

```html
Variable x inside function is: 5
Variable x outside function is:
```

#### global

函式外部的變數具有global scope，只能用於該函式外部使用

```php
	<?php
	$x = 5; // global scope

	function myTest() {
	  // using x inside this function will generate an error
	  echo "<p>Variable x inside function is: $x</p>";
	}
	myTest();

	echo "<p>Variable x outside function is: $x</p>";
	?>
```

```html
Variable x inside function is:
Variable x outside function is: 5
```


想要在函式中，用外部的變數，就在函式內用global 來定義變數

```php
	<?php
	$x = 5;
	$y = 10;

	function myTest() {
	  global $x, $y;
	  $y = $x + $y;
	}

	myTest();
	echo $y; // outputs 15
	?>
```

```hmtl
15
```

#### static

正常函式執行完，變數值都會被刪除，想要保留值可以加 static

```php
	<?php
	function myTest() {
	  static $x = 0;
	  echo $x;
	  $x++;
	}

	myTest();
	myTest();
	myTest();
	?>
```

```html
012
```

### 顯示

PHP 程式中可以用 echo 以及 print 來顯示資訊

* PHP echo / print 顯示

```php
	<?php
	$txt1 = "學習PHP";
	$txt2 = "Ian_Zhuang";
	$x = 5;
	$y = 4;

	echo "<h2>" . $txt1 . "</h2>";
	echo $txt2 . " 在 " . $txt1 . "<br>";
	echo $x + $y;

	print "<h2>" . $txt1 . "</h2>";
	print $txt2 . " 在 " . $txt1 . "<br>";
	print $x + $y;
	?>
```

```html
學習PHP
Ian_Zhuang 在 學習PHP
9

學習PHP
Ian_Zhuang 在 學習PHP
9
```

### 資料型態

變數可以儲存不同型態的資料，以下是 PHP 支持的資料型態

> [!NOTE] 小提示
>可以用var_dump() 函數來查看資料的型態以及值

* 字串 (String)
* 整數 (Integer)
* 浮點數 (Float)
* 布林 (Boolean)
* 陣列 (Array)
* 物件 (Object)
* 空值 (NULL)

#### 字串 (String)
字串可以是引號內的任何文本，可以使用單引號或雙引號來表示

```php
<?php
$x = "Hello world!";
$y = 'Hello world!';

echo $x;
echo "<br>";
echo $y;
?>
```

```html
Hello world!
Hello world!
```

#### 整數 (Integer)
整數有幾點規則：

1. 必須至少有一個數字
2. 不能有小數點
3. 可以是正數也可以是負數
4. 可以指定不同的進制表示法
5. 其範圍介於 -2,147,483,648 和 2,147,483,647 之間的非十進制數

```php
<?php
$x = 5985;
var_dump($x);
?>
```

```html
int 5985
```

#### 浮點數 (Float)

浮點數是帶小數點的數字或指數形式的數字

```php
<?php
$x = 10.365;
var_dump($x);
?>
```

```html
float 10.365
```

#### 布林 (Boolean)

布爾值表示兩種可能的狀態：TRUE 或 FALSE

```php
<?php
$x = true;
$y = false;
var_dump($x);
?>
```

```html
boolean true
```

#### 陣列 (Array)

陣列可以將多個值存在一個變數中

```php
<?php
$x = 10.365;
var_dump($x);
?>
```

```html
array (size=3)
 0 => string 'Apple' (length=5)
 1 => string '小米' (length=6)
 2 => string 'Sony' (length=4)
```

#### 物件 (Object)

在討論物件前，要先知道類別(Class)與物件(Object)的關係

* 類別 (Class) ：可以比喻為房屋的設計藍圖，是為了讓大家了解房屋的結構與形狀。
* 物件 (Object) ：可以比喻為真的房屋，物件是類別的實體化

為了要蓋好一棟房子有房子的設計藍圖(類別)還不夠，還需要蓋房子的材料，而材料就是所謂的資料(data)。

類別定義結構和行為用來產生物件，當多個物件是由同一
個類別產生出來時，每個物件都是一個獨立個體。

1. 建立類別語法很簡單，只需要使用 Class 來定義一個類別

```php
<?php
class MyClass
{
    // 在大括號裡面宣告類別的屬性與方法
}

$obj = new MyClass; //使用 new 來實體化類別並將他存入變數中
var_dump($obj); //查看類別內容
?>

```

```html
object(MyClass)[1]
```

2. 我們在替 MyClass 加入屬性，用 public 決定屬性的可視性

```php
<?php
class MyClass
{
    public $prop = "I'm a class Property";
}

$obj = new MyClass;
echo $obj->prop;
?>
```

```html
I'm a class Property
```

因為有很多物件實例化都來自同一個類別，如果沒有指定被實體化的物件，程式碼會無法判斷，所以要用 **<font color='blue'>-></font>** 在 PHP 的物件中，來存取物件的屬性和方法。


3. 定義類別(Class)的方法(Methods)，方法(Methods)是類別裡面的函式(Functions)，物件可以藉由執行這些方法來更動每個物件的行為。


```php
<?php
class MyClass
{
    public $prop = "I'm a class Property!";

    public function set($newval)
    {
        $this->prop = $newval;
    }

    public function get()
    {
        return $this->prop . "<br />";
    }
}

$obj = new MyClass;
echo $obj->get();                             //得到屬性的值。

$obj->set("I'm a new Property value!");       //設定新的屬性的值。
echo $obj->get();
?>
```


```html
I'm a class Property!
I'm a new Property value!
```

物件導向允許物件透過$this來參考自己。物件使用$this就如同直接使用物件名稱來指定物件，同等於MyClass->prop1。

#### 空值 (NULL)

Null 是一種特殊的數據類型，它只能有一個值：NULL

如果創建的變量沒有值，則會自動為其分配 NULL

```php
<?php
$x = "Hello world!";
$x = null;
var_dump($x);
?>
```

```html
null
```

### 運算符號

PHP 將運算符號分為以下幾組

* 算術運算符
* 賦值運算符
* 比較運算符
* 邏輯運算符
* 字符串運算符

#### 算術運算符

```php
<?php
$x = 3; $y = 6;
echo $x + $y;  // 輸出 x + y ，3 + 6 ，顯示 9
echo $x - $y;  // 輸出 x - y ，3 - 6 ，顯示 -3
echo $x * $y;  // 輸出 x * y ，3 * 6 ，顯示 18
echo $x / $y;  // 輸出 x / y ，3 / 6 ，顯示 0.5
echo $x % $y;  // 輸出 x % y ，3 % 6 ，顯示 3
echo $x ** $y; // 輸出 x ^ y ，3 ^ 6 ，顯示 729
?>
```


#### 賦值運算符

```php
<?php
$x = 3; $y = 6; echo $x += $y;  //輸出 x = x + y ， 9 = 3 + 6 ，輸出 9
$x = 3; $y = 6; echo $x -= $y;  //輸出 x = x - y ， -3 = 3 - 6 ，輸出 -3
$x = 3; $y = 6; echo $x *= $y;  //輸出 x = x * y ， 18 = 3 * 6 ，輸出 18
$x = 3; $y = 6; echo $x /= $y;  //輸出 x = x / y ， 0.5 = 3 / 6 ，輸出 0.5
$x = 3; $y = 6; echo $x %= $y;  //輸出 x = x % y ， 3 = 3 % 6 ，輸出 3
?>
```


#### 比較運算符

```php
<?php
$x = 3; $y = 6; $z = "6";
var_dump($x == $y);  // 判斷 x 等於 y (值)，3 不等於6 ，回傳 false
var_dump($x === $z); // 判斷 x 等於 z (型態and值)，數字不等於字串 ，回傳 false
var_dump($x != $y);  // 判斷 x 不等於 y (值)，3不等於6 ，回傳 true
var_dump($x <> $y);  // 判斷 x 不等於 y (型態)，數字不等於字串 ，回傳 true
var_dump($x > $y);   // 判斷 x 大於 y ，3沒有大於6 ，回傳 false
var_dump($x < $y);   // 判斷 x 小於 y ，3小於6 ，回傳 true
var_dump($x >= $y);  // 判斷 x 大於等於 y ，3沒有大於等於6 ，回傳 false
var_dump($x <= $y);  // 判斷 x 小於等於 y ，3小於等於6 ，回傳 true
var_dump($x <=> $y); // 判斷 x y 的關係 ，如果 x 大於 y ，回傳 1 ; 如果 x 小於 y ，就回傳 -1 ; 如果 x 等於 y ，就回傳 0
?>
```

#### 邏輯運算符

```php
<?php
$x = 3; $y = 6;
if ($x == 3 and $y == 6){  echo "And"; }  // 判斷 x 等於 3 且 y 等於 6 (兩者都要true)
if ($x == 3 or $y == 9){  echo "Or"; }    // 判斷 x 等於 3 或 y 等於 9 (擇一true)
if ($x == 3 && $y == 6){  echo "And"; }   // 判斷 x 等於 3 且 y 等於 6 (兩者都要true)
if ($x == 3 || $y == 9){  echo "Or"; }    // 判斷 x 等於 3 或 y 等於 9 (擇一true)
if ($x == 3 xor $y == 9){  echo "Xor"; }  // 判斷 x 等於 3 或 y 等於 9 (任一者為true，但只能一者為true)
if ($x != 4){  echo "Not"; }              // 判斷 x 不等於 4
?>
```

### 條件判斷

條件語句用於根據不同的條件下的操作

* if
* if...else
* if...elseif...else
* switch

#### if

如果條件為真，就執行代碼

* 案例：如果現在時間(hour)小於20，就輸出 Have a good day !

```php
<?php
$t = date("H");

if ($t < "20") {
  echo "Have a good day!";
}s
?>
```

* 輸出

```html
Have a good day!
```

#### if-else

如果條件為真，就執行該程式碼，如果條件為假，就執行另一個程式碼

* 案例：如果現在時間(hour)小於20，就輸出 Have a good day! ，超過時間，就輸出 Have a good night!

```php
<?php
$t = date("H");

if ($t < "20") {
  echo "Have a good day!";
}
else {
  echo "Have a good night!";
}
?>
```

#### if-elseif-else

針對兩個以上的條件判斷，如果條件為真，就執行該程式碼，如果條件為假，就執行另一個程式碼

* 案例：如果現在時間(hour)小於10，就輸出 Have a good morning! ，如果時間小於20，輸出Have a good day!，否則，就輸出 Have a good night!

```php
<?php
$t = date("H");

if ($t < "10") {
  echo "Have a good morning!";
}
elseif ($t < 20)  {
  echo "Have a good day!";
}
else {
  echo "Have a good night!";
}
?>
```

#### switch
根據不同的條件來執行不同的操作

```php
<?php
$weather = '下雨天';

switch($weather)
{
    case '晴天':
        echo "今天天氣是『 $weather 』";
        break;
    case '陰天':
        echo "今天天氣是『 $weather 』";
        break;
    case '下雨天':
        echo "今天天氣是『 $weather 』";
        break;
    default:
        echo "世界末日";
        break;
}
?>
```

### 迴圈

想要相同程式碼反覆執行一定的次數

* while
* do...while
* for
* foreach

#### while

只要指定條件為真，就會循環通過該程式碼

```php
<?php
$x = 1;

while($x <= 3) {
  echo "The number is: $x <br>";
  $x++;
}
?>

```

* 輸出

```html
The number is: 1
The number is: 2
The number is: 3
```
#### do...while

先執行一次do，再檢查指定條件是否為真，是的話就會循環通過該程式碼

```php
<?php
$x = 1;

do {
  echo "The number is: $x <br>";
  $x++;
} while ($x <= 5);
?>
```

* 輸出

```html
The number is: 6
```
因為do…while的條件會在執行循環程式碼後才檢查，所以do…while <font color='red'>**至少會循環一次**</font>該程式碼。

#### for

假如預先知道需要循環幾次，就可以使用for，可以設定程式碼在循環中的次數。

```php
<?php
for ($x = 0; $x <= 10; $x++) {
  echo "The number is: $x <br>";
}
?>
```

* 輸出

```html
The number is: 0
The number is: 1
The number is: 2
The number is: 3
The number is: 4
The number is: 5
The number is: 6
The number is: 7
The number is: 8
The number is: 9
The number is: 10
```

#### foreach

此循環適用於陣列，可以用來顯示陣列的key跟value

```php
<?php
$age = array("Peter"=>"35", "Ben"=>"37", "Joe"=>"43");

foreach($age as $x => $val) {
  echo "$x = $val<br>";
}
?>
```

* 輸出

```html
Peter = 35
Ben = 37
Joe = 43
```

#### break

想要在迴圈執行中跳出，可以使用break 指令，來跳出循環。

```php
<?php
for ($x = 0; $x < 10; $x++) {
  if ($x == 4) {
    break;
  }
  echo "The number is: $x <br>";
}
?>
```

* 輸出

```html
The number is: 0
The number is: 1
The number is: 2
The number is: 3
```

#### continue

continue 與 break 是相對的指令。break 中斷目前執行的迴圈，continue 則是回到迴圈的開頭，執行「下一次」迴圈。

```php
<?php
for ($x = 0; $x < 5; $x++) {
  if ($x == 2) {
    continue;
  }
  echo "The number is: $x <br>";
}
?>
```

* 輸出

```html
The number is: 0
The number is: 1
The number is: 3
The number is: 4
```

### 函式

PHP除了內建的函式外，還可以創建自己的函式
注意：函式名稱必須<font color='red'>**以字母或下底線開頭**</font>。不區分大小寫。

```php
<?php
function writeMsg() {
  echo "Hello world!";
}

writeMsg(); // call the function
?>
```

* 輸出

```html
Hello world!
```

也可以在函式裡面放入任意數量的參數，只需要用逗號分開


```php
<?php
function familyName($fname, $year) {
  echo "$fname Born in $year <br>";
}

familyName("Hege", "1975");
familyName("Stale", "1978");
familyName("Kai Jim", "1983");
?>
```

* 輸出

```html
Hege Born in 1975
Stale Born in 1978
Kai Jim Born in 1983
```

<br>

## 3. PHP 常用函式

### 日期和時間
可以透過輸入參數，顯示想要的日期

* d - 代表月份中的某天（01 到 31）
* m - 代表一個月（01 到 12）
* Y - 代表年份（四位數）
* l（小寫“L”）— 代表星期幾


```php
<?php
echo "Today is " . date("Y/m/d") . "<br>";
echo "Today is " . date("Y.m.d") . "<br>";
echo "Today is " . date("Y-m-d") . "<br>";
echo "Today is " . date("l");
?>
```

* 輸出

```html
Today is 2022/02/22
Today is 2022.02.22
Today is 2022-02-22
Today is Tuesday
```
可以透過輸入參數，顯示想要的時間

* H - 一個小時的 24 小時格式（00 到 23）
* h - 小時的 12 小時格式，前面補零（01 到 12）
* i - 前面補零的分鐘（00 到 59）
* s - 前面補零的秒數（00 到 59）
* a - 小寫的 Ante meridiem 和 Post meridiem（am 或 pm）


```php
<?php
date_default_timezone_set("Asia/Taipei");
echo "The time is " . date("h:i:sa");
?>
```

* 輸出

```html
The time is 12:12:10am
```

可以使用 strtotime()，顯示想要的日期

```php
<?php
$d=strtotime("tomorrow");
echo date("Y-m-d h:i:sa", $d) . "<br>";

$d=strtotime("next Saturday");
echo date("Y-m-d h:i:sa", $d) . "<br>";

$d=strtotime("+3 Months");
echo date("Y-m-d h:i:sa", $d) . "<br>";
?>
```

* 輸出

```html
2022-02-23 12:00:00am
2022-02-26 12:00:00am
2022-05-22 04:21:41am
```


### json

可以使用 json_encode()，將陣列存為 JSON 格式，也可以用 json_decode()，將JSON格式變成陣列

```php
<?php
$age = array("Peter"=>'35', "Ben"=>'45', "Ian"=>'22');
echo $enjson = json_encode($age);
var_dump(json_decode($enjson,true));
?>
```

* 輸出

```html
{"Peter":"35","Ben":"45","Ian":"22"}
array (size=3)
  'Peter' => string '35' (length=2)
  'Ben' => string '45' (length=2)
  'Ian' => string '22' (length=2)
```

### min/max

可以使用 min()，來找到參數列表裡面的最小值，也可以 max()，來找到參數列表裡面的最大值

```php
<?php
echo(min(0, 150, 30, 20, -8, -200));
echo '<br>';
echo(max(0, 150, 30, 20, -8, -200));
?>
```

* 輸出

```html
-200
150
```

### abs

可以使用 abs()，將參數值變成絕對值

```php
<?php
echo(abs(-6.7));
?>
```

* 輸出

```html
6.7
```

### round

可以使用 round()，將浮點數四捨五入到最接近的整數

```php
<?php
echo(round(0.60));
echo '<br>';
echo(round(0.49));
?>
```

* 輸出

```html
1
0
```

### rand

可以使用 rand()，會隨機產生一個亂數，也可以設定最大值與最小值的範圍來隨機產生亂數

```php
<?php
echo(round(0,100));
?>
```

* 輸出

```html
88
```

<br>

## 4. PHP 表單

### GET vs POST

舉個例子，如果 HTTP 代表現在我們現實生活中寄信的機制，那麼信封的撰寫格式就是 HTTP。我們可以將信封外的內容稱為 http-header，信封內的書信稱為 message-body。

假設 GET 表示信封內不得裝信件的寄送方式，像是明信片一樣，你可以把要傳遞的資訊寫在信封(http-header)上。而 POST 就是信封內有裝信件的寄送方式，不但信封可以寫東西，信封內 (message-body) 還可以放你想要寄送的資料或檔案。

因為使用 GET 的方法來傳送表單訊息對每個人都是可見的(所有的變數名稱與值都會顯示在 URL)，所以絕對不要使用 GET 來傳送密碼或是一些敏感訊息。

PHP 可以用 $_GET 和 $_POST 來收集表單數據

### GET

我們先模擬 $_GET ，先建立一個 index.html 的檔案程式碼如下

* HTML 表單 (GET)

```html
<html>
<body>

<form action="welcome_get.php" method="get">
Name: <input type="text" name="name"><br>
E-mail: <input type="text" name="email"><br>
<input type="submit">
</form>

</body>
</html>
```
再建立一個 welcome.php 的檔案程式碼如下

* PHP ($_GET)

```php
<html>
<body>

Welcome <?php echo $_GET["name"]; ?><br>
Your email address is: <?php echo $_GET["email"]; ?>

</body>
</html>

```

### POST

我們先模擬 $_POST ，先建立一個 index.html 的檔案程式碼如下

* HTML 表單 (POST)

```html
<html>
<body>

<form action="welcome_get.php" method="post">
Name: <input type="text" name="name"><br>
E-mail: <input type="text" name="email"><br>
<input type="submit">
</form>

</body>
</html>
```

再建立一個 welcome.php 的檔案程式碼如下

* PHP ($_POST)

```php
<html>
<body>

Welcome <?php echo $_POST["name"]; ?><br>
Your email address is: <?php echo $_POST["email"]; ?>

</body>
</html>

```

### 資料驗證

* 密碼：需為8碼至20碼，且並包含特殊符號、大小寫英文字母、數字至少各1碼

``` php
<?php
$password = '@aaaa5aI';
if(preg_match('/(?=.*[@#$%^&+=])(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])[@#$%^&+=a-zA-Z0-9]{8,20}$/', $password)){
    echo '密碼『',$password,'』正確。';
} else{
    echo '密碼『',$password,'』不正確(需符合8至20碼，且包含特殊符號、大小寫英文、數字各一碼)。';
}
?>
```

* 輸出

``` php
密碼『@aaaa5ai』不正確(需符合8至20碼，且包含特殊符號、大小寫英文、數字各一碼)。
密碼『@aaaa5aI』正確。
```

* 手機電話：需為10碼，且開頭為09接後8碼的數字

``` php
<?php
$phone = '0912345678';
if(preg_match('/^09[0-9]{8}$/', $password)){
    echo '手機電話『',$phone,'』正確。';
} else{
    echo '手機電話『',$phone,'』不正確(需為10碼，且符合開頭為09後接8碼的數字)。';
}
?>
```

* 輸出

``` php
手機電話『0212345678』不正確(需為10碼，且符合開頭為09後接8碼的數字)。
手機電話『0912345678』正確。
```

* 信箱

``` php
<?php
$email = '123@gmail.com';
if(filter_var("$email", FILTER_VALIDATE_EMAIL))
{
    echo '信箱『',$email,'』正確。';
} else{
    echo '信箱『',$email,'』不正確。';
}
?>
```

* 輸出

``` php
信箱『123@gmailcom』不正確。
信箱『123@gmail.com』正確。
```

<br>

## 5. PHP RESTful

REST ，指得是一組架構約束條件和原則，符合 REST 設計風格的Web API 稱為 RESTful API，主要以下面三點為定義

* 直觀簡單的資源網址URL 比如：http://example.com/resources

* 傳輸的資源：Web 服務接受與返回的類型，比如：JSON、XML

* 對資源的操作：Web 服務在該資源上所支持的請求方法，比如：POST、GET、PUT、PATCH、DELETE

> [!NOTE] 加油 還有一篇可以一起學習
>詳細可以參考另一篇文章 [如何在 Nginx 下實作第一個 PHP 留言板 RESTful API
](https://pin-yi.me/blog/php-laravel/nginx-php-restful-api) 裡面有更詳細的介紹

<br>

## 6. 資料來源

PHP 是什麼，架設網站最適合的程式語言：[https://kamadiam.com/what-is-php/](https://kamadiam.com/what-is-php/)

PHP 新手指南：3分鐘快速認識PHP：[https://www.happycoding.today/posts/23](https://www.happycoding.today/posts/23)

PHP 教程：[https://www.w3schools.com/php/](https://www.w3schools.com/php/)

PHP基礎語法(一)：Hello world與基本資料型態：[https://ithelp.ithome.com.tw/articles/10218595](https://ithelp.ithome.com.tw/articles/10218595)

PHP-物件導向(OOP)介紹：[https://ithelp.ithome.com.tw/articles/10206973](https://ithelp.ithome.com.tw/articles/10206973)

秒懂PHP的FastCGI跟PHP-FPM有什麼關係：[https://www.astralweb.com.tw/what-is-differences-between-fastcgi-php-fpm/](https://www.astralweb.com.tw/what-is-differences-between-fastcgi-php-fpm/)


