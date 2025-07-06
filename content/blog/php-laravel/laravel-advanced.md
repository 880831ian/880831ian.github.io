---
title: "Laravel 進階 (內建會員系統、驗證 RESTful API 是否登入、使用 Repository 設計模式)"
type: docs
weight: 9998
description: Laravel 進階 (內建會員系統、驗證 RESTful API 是否登入、使用 Repository 設計模式)
images:
  - php-laravel/laravel-advanced/og.webp
date: 2022-03-08
authors:
  - name: Ian_zhuang
    link: https://pin-yi.me/about/
---

本篇是 [Laravel 介紹](../laravel/) 的進階篇，由於有些說明介紹會沿用上一篇的內容，建議要先瀏覽過上一篇呦～ ([可以從這裡下載最後程式碼！](https://github.com/880831ian/laravel-restful-api-messageboard))

## Laravel 內建會員系統

Laravel 這個框架，很方便的地方就是，他可以將我們常用的帳號登入註冊等功能內建在 Laravel 框架內，那我們來看一下要怎麼使用吧。(本篇 Laravel Framework：5.4.36)

它的配置文件在 `config/auth.php` ，其中包含了用於調整認證服務行為或是標注選項配置等。

<br>

要怎麼開始呢!? 先使用 `artisan` 指令產生我們要的檔案以及路由吧！

```sh
$ php artisan make:auth
```

<br>

主要會產生

```sh
	new file:   app/Http/Controllers/HomeController.php
	new file:   resources/views/auth/login.blade.php
	new file:   resources/views/auth/passwords/email.blade.php
	new file:   resources/views/auth/passwords/reset.blade.php
	new file:   resources/views/auth/register.blade.php
	new file:   resources/views/home.blade.php
	new file:   resources/views/layouts/app.blade.php
	modified:   routes/web.php
```

<br>

其中 `routes/web.php` 多了：

```php
Auth::routes();

Route::get('/home', 'HomeController@index')->name('home');
```
<br>

views 底下放的就是顯示的畫面，所以現在可以瀏覽

登入頁面：`http://127.0.0.1:8000/login`

註冊頁面：`http://127.0.0.1:8000/register`

其他像是 `controller` 或是 `migration` 都已經內建在裡面了，稍後實作會講到！

<br>

### 實作

#### Migration
我們照上一篇的流程，由於 `route` 已經設定好，所以我們先來設定 `migration` ， 檔案就放在 `database/migrations` 底下的 `{日期時間}_create_users_table.php` ，我們來看一下預設的資料表有哪些欄位吧!

```php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('users');
    }
```
簡單看一下，他會建立一個名叫 `users` 的資料表，欄位分別有自動累加(id)、字串(name)、唯一字串(email)、字串(密碼)、rememberToken()、timestamps()。

<br>

那我們這次想要做的事，不使用 email 做登入，改成使用 username來當作登入驗證，所以我們把 name、email的欄位改成
```php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('username')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('users');
    }
```
設定 username 欄位為唯一值，移除掉 email ，再加入一個 api_token 用於後續的 RESTful API 驗證。

<br>

使用 `php artisan migrate` 將 `users` 這個 table 給建起來。

 因為我們修改 name、email 改成 username 這個欄位，所以我們也要修改一下 views 顯示的畫面，由於是簡單的 HTML 這邊就不再多描述，直接放上修改後的程式碼。

 #### View

 ##### Login.blade

 ```html
<div class="form-group{{ $errors->has('username') ? ' has-error' : '' }}">
 <label for="username" class="col-md-4 control-label">Username</label>

   <div class="col-md-6">
     <input id="username" type="text" class="form-control" name="username" value="{{ old('username') }}" required autofocus>

     @if ($errors->has('username'))
            <span class="help-block">
                    <strong>{{ $errors->first('username') }}</strong>
            </span>
     @endif
  </div>
</div>
```
<br>

##### Register.blade

```html
<div class="form-group{{ $errors->has('username') ? ' has-error' : '' }}">
 <label for="username" class="col-md-4 control-label">Username</label>

 <div class="col-md-6">
  	<input id="username" type="text" class="form-control" name="username" value="{{ old('username') }}" required>

     @if ($errors->has('username'))
            <span class="help-block">
                    <strong>{{ $errors->first('username') }}</strong>
            </span>
     @endif
  </div>
</div>
```

 <br>

 #### Model

接下來到 `app` 底下找到 `User.php` 檔案，由於筆者習慣將 `model` 放到專屬的資料夾，不要讓他在 `app` 裡面流浪，所以會建立一個 `Models`  的資料夾，來存放所有的 `models` ，那移動原本的 `model` 有些有使用到它的路徑都要做修改，這邊以 `User` 檔案為示範。(因為後續會說到怎麼驗證登入 API ，所以上一篇的 Message model 也要記得修改歐！)

 <br>

 因為我們移動後，原本的路徑是 `App` 要改成 `App\Models`，會影響到的程式有以下幾個 (附上片段程式碼)

 * `config/auth.php` 約在70行左右

```php
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class, //修改片段
        ],
```
<br>

  * `config/services.php` 約在33行左右

```php
    'stripe' => [
        'model' => App\Models\User::class, //修改片段
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],
```
<br>

   * `database/factories/UserFactory.php` 約在15行左右

```php
$factory->define(App\Models\User::class, function (Faker\Generator $faker) { //修改片段
    static $password;

    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'password' => $password ?: $password = bcrypt('secret'),
        'remember_token' => str_random(10),
    ];
});
```
<br>

   * `app/Http/Controllers/Auth/RegisterController.php` 約在5行左右

```php
use App\Models\User;
```
<br>

 都修改好了，我們就繼續來修改 `User` 這個 model ，將原本的 name、email ，修改成以下
 ```php
 <?php

namespace App\Models;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'username',  'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password',  'remember_token',
    ];
}
 ```

 <br>

 裡面有一個是 `fillable` 跟 `hiddem` ，順便解釋一下這兩個是在做什麼

 * fillable：當使用者輸入這些 attribute 以外的參數(資料表的欄位)，就會發生錯誤。
 * hidden：想要限制能出現在陣列或是JSON 格式的屬性資料，比如密碼欄位等等，不想要顯示，只需要把該欄位加入 `hidden`。

 再加碼一個

  * guarded：這個屬性與 `fillable` 相反，當使用者輸入該參數的值，就會被擋掉。以下這個例子是允許任何的 input 資料，但十分建議不要這樣。
 ```php
     protected $guarded = [];
 ```

<br>

#### Controller

接下來是要修改我們的 controller，主要需要修改的有兩個，檔案在 `app\Http\Auth` 底下的 `registerController` 、 `loginController` 兩個檔案，我們先來修改 `registerController` 註冊功能，看一下原本的程式碼在做什麼事情。

##### RegisterController

<br>
原本程式碼

```sh
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:6|confirmed',
        ]);
    }

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\User
     */
    protected function create(array $data)
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);
    }
```
可以看到在 `validator()` 就是在驗證註冊時的欄位，以 email 來說明，他需要符合必填(required)、字串(string)、email格式(email)、最大長度(max:255)、在 users 資料表中唯一(unique:users) 才可以註冊，另外兩個也是如此。

那 `create()` 就是會建立註冊的資料到資料表中，就是這麼簡單XD，詳細可以參考 [Laravel 官網 Form Request Validation](https://laravel.com/docs/5.4/validation#form-request-validation)

<br>

那還記得我們把 name、email 給刪掉，改成 username 來做登入嗎，我們來看一下我們要修改哪些東西！

```sh
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'username' => 'required|string|max:255|unique:users',
            'password' => 'required|string|min:6|confirmed',
        ]);
    }

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\Models\User
     */
    protected function create(array $data)
    {
        return User::create([
            'username' => $data['username'],
            'password' => bcrypt($data['password']),
        ]);
    }
```
由於我們刪除 email ，所以要把 unique:users 設定在 username。

<br>

##### LoginController

註冊頁面都已經修改好了，理論上連線到 `http://127.0.0.1:8000/register` 頁面，輸入 username 以及 密碼，會可以註冊成功囉，那接下來我們來修改登入的 controller。

```php
    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = '/home';

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }
```
因為 Laravel 框架內建了驗證登入功能，所以只需要使用 `use AuthenticatesUsers;` 就可以達到驗證效果，登入成功就會使用 `redirectTo` 導向到 `/home`，那問題來了，因為 `AuthenticatesUsers` 預設是使用 email 來做登入驗證，但我們改成 username，我們要怎麼修改呢！？ 一起來看看吧

```php
    //由於判斷是用 AuthenticatesUsers 這個內建的 trait 來實作，其中 username() 這個方法就是來指定登入要用的欄位
    public function username()
    {
        return 'username';
    }
```
上面有說因為使用了 `AuthenticatesUsers`  來做驗證，我們盡量不去修改框架內的檔案，當然 Laravel 也有配套措施，只要使用 `username()` 這個方法就可以指定登入要驗證的欄位，因為筆者在學習這段時也找了很久，這邊附上[ Laravel API](https://laravel.com/api/6.x/Illuminate/Foundation/Auth/AuthenticatesUsers.html) 可以去查看一下。

我們這時就可以使用 `http://127.0.0.1:8000/login` 來登入囉！試試看吧～


### 驗證 RESTful API 是否登入

本篇會使用到上一篇的 RESTful API 留言板來進行修改，所以有興趣的朋友，可以先去看玩上一篇 [Laravel 介紹](https://pin-yi.me/laravel) 歐～

接下來也會同時使用 Repository 設計模式來修改程式碼，那 Repository 設計模式是什麼呢，讓我先來介紹。

#### Repository 設計模式

還記得我們上一次，把所有的邏輯以及資料庫的處理的都放在 `Controller` 裡面嗎！如果只是單一個小專案，還可以這樣做沒關係，但專案越來越大，會使用的功能也越來越多，會造成 `Controller` 檔案肥大且難以維護，基於 SOLID 原則，我們應該要使用 Repository 設計模式來補助  `Controller`，將相關的資料庫邏輯放在不同的 `Repository`，方便中大型的專案做維護。

<br>

> [!NOTE] 小知識
> SOLID：簡單來說就是物件導向設定上為了讓軟體維護、開發變得更容易的五個準則的縮寫。
> * Single Responsibility Principle (SRP) 單一職責原則
> * Open-Closed Principle (OCP) 開放封閉原則
> * Liskov Substitution Principle (LSP) 里氏替換原則
> * Interface Segregation Principle (ISP) 介面隔離原則
> * Dependency Inversion Principle(DIP) 依賴反轉原則
>SOLID目的也就是讓你程式碼達成低耦合、高內聚、降低程式碼壞味道，透過分離與clean code來提高可讀性會讓你的程式碼等同於設計文件，所以在修改或新增過程中降低產生Bug的機率，也可以較快的找到與解決出問題的地方，可以有效的減少技術債。

(資料來源：[我該學會SOLID嗎?](https://medium.com/@f40507777/%E6%88%91%E8%A9%B2%E5%AD%B8%E6%9C%83solid%E5%97%8E-4e73887c9156))

<br>

不太清楚沒關係，我們會慢慢介紹到！那我們先來看看這次要修改什麼呢！？ 我想要在使用 API 時，去檢查有沒有登入，才會進行動作，舉個例子，我們希望在新增留言、修改留言以及刪除留言都需要登入，且是由本人操作才算成功。

<br>

#### Migration

在此之前，我們先來修改一下上次的 `migration` {日期時間}_create_message_table.php 檔案吧

```php
    public function up()
    {
        Schema::create('message', function (Blueprint $table) {
            $table->increments('id'); //留言板編號
            $table->integer('user_id')->unsigned(); //留言者ID
            $table->foreign('user_id')->references('id')->on('users');
            $table->string('content', 20); //留言板內容
            $table->integer('version')->default(0);
            $table->timestamps(); //留言板建立以及編輯的時間
            $table->softDeletes(); //軟刪除時間
        });
    }
```
可以看到我們將資料庫的名稱從 `messsages` 改為 `message` ，後續程式部分也都會修改，大家要在注意一下 ～

我們這次加入了留言者 ID (使用外鍵連接 `users` 的 `id`)、按讚者 ID (使用外鍵連接 `users` 的 `id`)、留言板樂觀鎖、softDeletes軟刪除的欄位(軟刪除後續會提到)，並且因為我們同樣的資料不要重複儲存，所以刪除 `name` 要查詢就使用 join 來做查詢。

我們還希望可以多一個來存放是誰按讚的的資料表。所以一樣使用 `migration`  新增一個 {日期時間}_create_like_table.php 檔案

```php
    public function up()
    {
        Schema::create('like', function (Blueprint $table) {
            $table->bigIncrements('id'); //按讚紀錄編號
            $table->integer('message_id')->unsigned()->nullable(); //文章編號
            $table->foreign('message_id')->references('id')->on('message');
            $table->integer('user_id')->unsigned()->nullable(); //帳號編號
            $table->foreign('user_id')->references('id')->on('users');
            $table->dateTime('created_at'); //按讚紀錄建立時間
            $table->softDeletes(); //軟刪除時間
        });
    }
```
會存放文章的編號並且使用外鍵連接 `message ` 的 `id`，以及按讚者的 ID 也使用外鍵連接 `users ` 的 `id`。


<br>

列出本次會使用的功能以及對應的方法、是否需要登入、登入後其他人是否可以操作
<br>
| 功能 | 方法 | 是否需要登入 | 登入後其他人是否可以操作 |
| :---: | :---: | :---: | :---: |
| 查詢全部留言 | getAllMessage | 否 | 不需登入 |
| 查詢{id}留言 | getMessage | 否 | 不需登入 |
| 新增留言 | createMessage | 是 | 否 |
| 修改{id}留言 | updateMessage | 是 | 否 |
| 按讚{id}留言 | likeMessage | 是 | 可以 |
| 刪除{id}留言 | deleteMessage | 是 | 否 |

<br>


#### 登入 API

我們上面介紹有使用到 Laravel 內建的登入 LoginController 來進行登入，但通常我們在使用時，都會另外再多一個登入用的 API ，那我們來看一下要怎麼設計吧！

我們先使用 `php artisan make:controller LoginController` 新增一個登入的 API，他會產生在 `app/Http/Controllers/` 目錄下

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function login(Request $request)
    {
        $rules = [
            'username' => 'required',
            'password' => 'required'
        ];
        $validator = Validator::make($request->all(), $rules);
        if ($validator->fails()) {
            return response()->json(["message" => "格式錯誤"], 400);
        }

        if (!Auth::attempt([
            'username' => $request->username,
            'password' => $request->password
        ])) {
            return response()->json(["message" => "登入失敗"], 401);
        }
        return response()->json(["message" => "登入成功"], 200);
    }

    public function logout()
    {
        Auth::logout();
        return response()->json(["message" => "登出成功"], 200);
    }
}
```
這邊的 Login 會先驗證格式是否正確，在使用 `Auth:attempt` 來檢查是否有註冊過，並且回傳相對應的訊息， Logout 就使用 `Auth::logout` 即可。

好了後我們先到 routes/api.php 新增登入跟登出 API 的路徑

```php
Route::post('login', 'LoginController@login');
Route::post('logout', 'LoginController@logout');
```


<br>

我們接下來設定 Middleware ，什麼是 Middleware 呢！？

#### Middleware
Middleware 中文翻譯是中介軟體，是指從發出請求 (Request)之後，到接收回應(Response)這段來回的途徑上，

用來處理特定用途的程式，比較常用的 **Middleware** 有身份認證 (Identity) 、路由(Routing) 等，再舉個例子

```
某天早上你去圖書館看書，
下午去公園畫畫，
晚上去KTV 唱歌，
等到要準備回家的時候發現學生證不見了，
你會去哪裡找? (假設學生證就掉在這3個地方)
```

對於記憶不好的人來說，會按照 KTV > 公園 > 圖書館的路線去尋找。

假設在公園找到學生證，就不會再去圖書館了，由於這條路是死巷，所以只能返回走去KTV的路，這個就是 **Middleware** 的運作原理。

所以我們需要再請求時，先檢查是否有登入，才可以去執行需要權限的功能。

我們可以使用內建的 `Auth::check` 來檢查是否有登入，我們接著看要怎麼做吧！

<br>

先下指令生成一個放置登入驗證權限的 `Middleware` ，我把它取名為 `ApiAuth`
```sh
$  php artisan make:middleware ApiAuth
Middleware created successfully.
```

<br>

接著要把剛剛生成的 `ApiAuth` 檔案放置在 `app/Http/Kernel.php` 檔案中

```php
    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'api.auth' => \App\Http\Middleware\ApiAuth::class // 這邊
    ];
```

接下來我們就可以開始撰寫 `ApiAuth` 檔案的內容了

```php
    public function handle($request, Closure $next)
    {
        if (!Auth::check()) {
            return response(['message' => '用戶需要認證'], 401);
        }
        return $next($request);
    }
```
這邊的意思是指，在 request 的時候，我們使用內建的 `Auth::check` 來檢查，如果登入就可以繼續使用，如果沒有登入會回傳用戶需要認證以及 401 的 status code。

<br>

#### Route

接下來，我們要把我們設定好的 `ApiAuth` 設定在 `route\api.php` 的路由中，
```php
Route::get('message', 'MessageController@getAll');
Route::get('message/{id}', 'MessageController@get');
Route::post('message', 'MessageController@create')->middleware('api.auth');
Route::put('message/{id}', 'MessageController@update')->middleware('api.auth');
Route::patch('message/{id}', 'MessageController@like')->middleware('api.auth');
Route::delete('message/{id}', 'MessageController@delete')->middleware('api.auth');
```
可以看到跟我們上一篇的 route 設定的差不多，只是將 `MessageController` 後面的方法做了一點變化，簡化名稱(這與我們後面講到的 Repository 設計模式有關)，以及加上 `like` 這個方法來當作我們的按讚功能，後面將我們需要登入驗證才可以使用的功能，加入 `middleware('api.auth');`

<br>

#### Model

因為我們在取得資料時，不希望顯示 `deleted_at` 給使用者，所以在 `app\Models\Message.php` 這個 model 裡面用 `hidden` 來做設定。

**message**

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Message extends Model
{
    use SoftDeletes;

    protected $table = 'Message';
    protected $fillable = [
        'user_id', 'name', 'content'
    ];
    protected $hidden = [
        'deleted_at'
    ];
}
```
可以看到我們還多引用 SoftDeletes ，它叫軟刪除，一般來說我們在設計資料庫時，不會真正意義上的把資料刪掉，還記得我們再新增 message 跟 like 資料表時，有多了一個  `softDeletes()` 欄位嗎，這個欄位就是當我們刪除時，他會記錄刪除時間，但在查詢時就不會顯示這筆資料，讓使用者覺得資料已經真正刪除了，但實際上資料還是存在在資料庫中。


<br>

**like**

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Like extends Model
{
    use SoftDeletes;

    protected $table = 'like';
    protected $fillable = [
        'message_id', 'user_id', 'created_at'
    ];
    public $timestamps = false;
}
```

可以看到我們還多引用 SoftDeletes ，它叫軟刪除，一般來說我們在設計資料庫時，不會真正意義上的把資料刪掉，還記得我們再新增 message 跟 like 資料表時，有多了一個  `softDeletes()` 欄位嗎，這個欄位就是當我們刪除時，他會記錄刪除時間，但在查詢時就不會顯示這筆資料，讓使用者覺得資料已經真正刪除了，但實際上資料還是存在在資料庫中。


<br>

#### Repository

還記得我們上次把 RESTful API 要處理的邏輯都寫在 Controller 裡面嗎，我們光是一個小功能就讓整個 Controller 變得肥大，在後續維護時或是新增功能時，會導致十分不便利，因此我們要將 Repository 設計模式 給導入，那要怎麼實作呢～

<br>

我們要先在 `app` 底下新增一個 `Repositories` 目錄，在目錄底下再新增一個 `MessageRepository.php` 檔案來專門放我們與資料庫拿資料的程式，讓 Controller 單純處理商業邏輯我們整個檔案分成幾段來看

<br>

先新增這個檔案的命名空間，並將我們原先放在 `MessageController` 的引用給拿進來，我們 Repositories 單純處理與資料庫的交握，或是引用 Message 跟 like 的 Models。
```php
namespace App\Repositories;

use App\Models\Message;
use App\Models\Like;
```

<br>

**查詢留言資料讀取**
```php
    public static function getAllMessage()
    {
        return Message::select(
            'message.id',
            'message.user_id',
            "users.username as name",
            'message.version',
            'message.created_at',
            'message.updated_at'
        )
            ->leftjoin('like', 'message.id', '=', 'like.message_id')
            ->leftjoin('users', 'message.user_id', '=', 'users.id')
            ->selectRaw('count(like.id) as like_count')
            ->groupBy('id')
            ->get()
            ->toArray();
    }

    public static function getMessage($id)
    {
        return Message::select(
            'message.id',
            'message.user_id',
            "users.username as name",
            'message.version',
            'message.created_at',
            'message.updated_at'
        )
            ->leftjoin('like', 'message.id', '=', 'like.message_id')
            ->leftjoin('users', 'message.user_id', '=', 'users.id')
            ->selectRaw('count(like.id) as like_count')
            ->groupBy('id')
            ->get()
            ->where('id', $id)
            ->first();
    }
```
回傳全部的留言資料 `getAllMessage()`，由於我們想要顯示留言者 id，只能一個一個把我們想要的 select 出來，不能透過 model 來顯示，使用 leftjoin 來查詢，最後多一個來顯示各個文章的總數。

回傳{id}留言資料 `getMessage($id)`，一樣跟回傳全部的留言資料一樣，只是多一個 where 來顯示輸出的 id 留言資料。

<br>

**新增留言資料讀取**

```php
    public static function createMessage($id, $content)
    {
        Message::create([
            'user_id' => $id,
            'content' => $content
        ]);
    }
```
使用 create 來新增資料，將 user_id 帶入傳值進來的 `$id`，content 帶入 `$content`。

<br>

**修改留言資料讀取**

```php
    public static function updateMessage($id, $user_id, $content, $version)
    {
        return Message::where('version', $version)
            ->where('id', $id)
            ->where('user_id', $user_id)
            ->update([
                'content' => $content,
                'version' => $version + 1
            ]);
    }
```
先使用 where 來檢查樂觀鎖 version ，在查詢此 id 是否存在，以及編輯者是否為發文者，最後用 update 來更新資料表，分別更新 user_id、content、version(樂觀鎖每次加1) 等欄位。

<br>

**按讚留言資料讀取**

```php
    public static function likeMessage($id, $user_id)
    {
        Like::create([
            'message_id' => $id,
            'user_id' => $user_id,
            'created_at' => \Carbon\Carbon::now()
        ]);
    }
```
按讚我們會在 like 的資料表中來新增紀錄，所以也是使用 create 來新增，新增 message_id、user_id、created_at 。

<br>

**刪除留言資料讀取**

```php
    public static function deleteMessage($id, $user_id)
    {
        return Message::where('id', $id)
            ->where('user_id', $user_id)
            ->delete();
    }
```
刪除功能因為我們使用軟刪除，所以不用顧慮 FK 外鍵，所以可以直接刪除 message。

<br>


#### Controller

到這裡我們講完 `MessageRepository.php` 的內容了，那原本的 Controller 剩下什麼呢 !?

```php
namespace App\Http\Controllers;

use App\Repositories\MessageRepository;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Validator;
```
我們在商業邏輯上會處理 Request 內容，以及使用 MessageRepository 來對資料庫做存取、Auth 取的登入者的資訊等等功能，所以要記得先把他引用進來歐。

<br>

**查詢留言功能**


```php
    // 查詢全部的留言
    public function getAll()
    {
        return MessageRepository::getAllMessage();
    }

    // 查詢id留言
    public function get($id)
    {
        if (!$message = MessageRepository::getMessage($id)) {
            return response()->json(["message" => "找不到留言"], 404);
        }
        return $message;
    }
```

<font color='red'>由於我們 MessageRepository 都只有單純與資料庫進行交握，所有的判斷以及回傳都會在 controller 來做處理</font>，`getAll()` 會使用到 `MessageRepository::getAllMessage()` 的查詢並回傳顯示查詢的資料。

`get(id)` 會先用 MessageRepository::getMessage($id) 來檢查 id 是否存在，如果不存在就會回傳找不到留言 404 的 status code，如果存在就回傳存在變數 `message` 的 `MessageRepository::getMessage($id)` 資料。

<br>


**新增留言功能**

```php
    // 新增留言
    public function create(Request $request)
    {
        $user = Auth::user();

        $rules = ['content' => 'required|max:20'];
        $validator = Validator::make($request->all(), $rules);
        if ($validator->fails()) {
            return response()->json(["message" => "沒有輸入內容或長度超過20個字元"], 400);
        }

        MessageRepository::createMessage($user->id, $request->content);
        return response()->json(["message" => "新增紀錄成功"], 201);
    }
```

我們先使用 `Auth::user()` 將登入的使用者資料存在 `$user` 中，在檢查輸入的 request 內容是否有超過 20 的字元，如果有就回傳內容長度超過20個字元 400。接著就用 MessageRepository::createMessage 將要新增的資料帶入，最後回傳新增紀錄成功 201 。

<br>

**修改留言功能**

```php
    // 更新id留言
    public function update(Request $request, $id)
    {
        $user = Auth::user();

        if (!$message = MessageRepository::getMessage($id)) {
            return response()->json(["message" => "找不到留言"], 404);
        }

        $rules = ['content' => 'required|max:20'];
        $validator = Validator::make($request->all(), $rules);
        if ($validator->fails()) {
            return response()->json(["message" => "沒有輸入內容或長度超過20個字元"], 400);
        }

        if (!MessageRepository::updateMessage($id, $user->id, $request->content, $message['version'])) {
            return response()->json(["message" => "更新留言失敗"], 400);
        }
        return response()->json(["message" => "修改成功"], 200);
    }
```

一樣先把登入的使用者存入 `$user`，檢查是否有這個 id ，沒有就回傳找不到留言 404，接下來檢查輸入的內容長度，如果超過，就回傳內容長度超過20個字元 400，再檢查要修改留言的與留言者是不是同一個使用者，如果不是就回傳權限不正確 403，最後就將資料透過 `MessageRepository::updateMessage` 來做更新，並回傳修改成功 200。

<br>

**按讚留言功能**

```php
    // 按讚id留言
    public function like($id)
    {
        $user = Auth::user();

        if (!MessageRepository::getMessage($id)) {
            return response()->json(["message" => "找不到留言"], 404);
        }

        MessageRepository::likeMessage($id, $user->id);
        return response()->json(["message" => "按讚成功"], 200);
    }
```
一樣先把登入的使用者存入 `$user`，先檢查是否有這個留言，沒有就回傳找不到留言 404，接著就使用 `MessageRepository::likeMessage` 來記錄按讚留言，並回傳按讚成功 404。

<br>

**刪除留言功能**

```php
    // 刪除id留言
    public function delete($id)
    {
        $user = Auth::user();

        if (!MessageRepository::deleteMessage($id, $user->id)) {
            return response()->json(["message" => "找不到留言"], 404);
        }
        return response()->json(["message" => "刪除成功,沒有返回任何內容"], 204);
    }
```

一樣先把登入的使用者存入 `$user`，先檢查是否有這個留言，沒有就回傳找不到留言 404，在檢查文章權限，錯誤就回傳權限不正確 403，最後檢查是否有被按讚，有的話要先刪除 like 裡面的按讚紀錄，最後再刪除文章(所有的刪除，因為我們在 model 裡面有使用 softdelete 軟刪除，所以不會真的刪除，而是在欄位的 deleted_at 加入刪除時間，來讓查詢時以為他被刪除了！)

<br>

### Postman 測試

那我們一樣來看一下 Postman 的測試，這邊只顯示需要登入才能使用的 API。

#### 登入

<br>

{{< figure src="/php-laravel/laravel-advanced/login.webp" width="600" caption="新增留言 成功" >}}

<br>

我們把帳號密碼放到 Body 來傳送，如果帳號密碼正確，就會顯示登入成功，並且在 Cookie 裡面的 laravel_session，可以用來判斷是否登入，以及登入的人是誰。

<br>

#### 新增留言 - 成功

<br>

{{< figure src="/php-laravel/laravel-advanced/post-api-success.webp" width="600" caption="新增留言 成功" >}}

<br>

新增留言成功，因為 **<font color='blue'>有登入，所以可以從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** 會顯示新增紀錄成功以及回應 201 Created

<br>

#### 新增留言 - 失敗

<br>

{{< figure src="/php-laravel/laravel-advanced/post-api-error.webp" width="600" caption="新增留言 失敗" >}}

<br>

新增留言失敗，因為 **<font color='red'>沒有登入，無法從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，所以會顯示用戶需要認證以及回應 401 Unauthorized

<br>

#### 修改留言 - 成功

{{< figure src="/php-laravel/laravel-advanced/put-api-success.webp" width="600" caption="修改留言 成功" >}}

<br>

修改留言成功，因為 **<font color='blue'>有登入，所以可以從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，會顯示修改成功以及回應 200 OK

<br>

#### 修改留言 - 失敗 - 沒有登入

{{< figure src="/php-laravel/laravel-advanced/put-api-error-1.webp" width="600" caption="修改留言 失敗" >}}

<br>

修改留言失敗，因為 **<font color='red'>沒有登入，無法從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，會顯示用戶需要認證以及回應 401 Unauthorized

<br>

#### 修改留言 - 失敗 - 權限不足

{{< figure src="/php-laravel/laravel-advanced/put-api-error-2.webp" width="600" caption="修改留言 失敗" >}}

<br>

修改留言失敗，雖然 **<font color='red'>有登入，但存在 Cookie 裡面的 laravel_session 不是當初的留言者</font>** ，會顯示權限不正確以及回應 403 Forbidden

<br>

#### 按讚留言 - 成功

{{< figure src="/php-laravel/laravel-advanced/patch-api-success.webp" width="600" caption="修改留言 成功" >}}

<br>

按讚留言成功，因為 **<font color='blue'>有登入，所以可以從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，會顯示按讚成功以及回應 200 OK

<br>

#### 按讚留言 - 失敗 - 沒有登入

{{< figure src="/php-laravel/laravel-advanced/patch-api-error.webp" width="600" caption="修改留言 失敗" >}}

<br>

按讚留言失敗，因為 **<font color='red'>沒有登入，無法從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，會顯示用戶需要認證以及回應 401 Unauthorized

<br>

#### 刪除留言 - 成功

{{< figure src="/php-laravel/laravel-advanced/delete-api-success.webp" width="600" caption="刪除留言 成功" >}}

<br>

刪除留言成功，因為 **<font color='blue'>有登入，所以可以從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，不會顯示訊息但會回應 204 No Content

<br>

#### 刪除留言 - 失敗 - 沒有登入

<br>

{{< figure src="/php-laravel/laravel-advanced/delete-api-error-1.webp" width="600" caption="刪除留言 失敗" >}}

<br>

刪除留言失敗，因為 **<font color='red'>沒有登入，無法從 Cookie 裡面的 laravel_session 來驗證是否登入</font>** ，會顯示用戶需要認證以及回應 401 Unauthorized

<br>

#### 刪除留言 - 失敗 - 權限不足

<br>

{{< figure src="/php-laravel/laravel-advanced/delete-api-error-2.webp" width="600" caption="刪除留言 失敗" >}}

<br>

刪除留言失敗，雖然 **<font color='red'>有輸入正確的 token ，但不是當初的留言者</font>** ，會顯示權限不正確以及回應 403 Forbidden



<br>

## 參考資料

Laravel Auth 自定義user 模型目錄結構：[https://www.itread01.com/content/1545012913.html](https://www.itread01.com/content/1545012913.html)

user ( Model )：[https://ithelp.ithome.com.tw/articles/10220381](https://ithelp.ithome.com.tw/articles/10220381)

Laravel Form Request Validation：[https://laravel.com/docs/5.4/validation#form-request-validation](https://laravel.com/docs/5.4/validation#form-request-validation)

laravel Validation 驗證格式：[https://ithelp.ithome.com.tw/articles/10250237](https://ithelp.ithome.com.tw/articles/10250237)