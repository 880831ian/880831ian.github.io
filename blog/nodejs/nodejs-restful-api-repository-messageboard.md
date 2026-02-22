# 用 Node.js寫一個 Repository Restful API 的留言板 (express、sequelize 套件)
本文章是使用 Node.js 來寫一個 Repository Restful API 的留言板，並且會使用 express 以及 sequelize (使用 Mysql)套件。

建議可以先觀看 [Node.js 介紹](../nodejs-introduce/) 文章來簡單學習 Node 語言。

[範例程式連結 點我 😘
](https://github.com/880831ian/node-restful-api-repository-messageboard)

版本資訊

* macOS：11.6
* node：v16.14.2
*  npm：8.5.0
* Mysql：mysql  Ver 8.0.28 for macos11.6 on x86_64 (Homebrew)
<br>

## 實作

### 檔案結構

```sh
.
├── app.js
├── config
│   └── config.json
├── controllers
│   ├── auth.js
│   ├── message.js
│   └── reply.js
├── middleware
│   └── index.js
├── migrations
│   ├── 20220331054531-create-user.js
│   ├── 20220401093019-create-message.js
│   └── 20220404041905-create-reply.js
├── models
│   ├── index.js
│   ├── message.js
│   ├── reply.js
│   └── user.js
├── node_modules(以下檔案省略)
├── package-lock.json
├── package.json
├── repositories
│   ├── auth.js
│   ├── message.js
│   └── reply.js
└── router
    └── index.js
```

<br>

我們來說明一下上面的資料夾以及檔案各別功能與作用

* app.js：程式的啟動入口，裡面會放置有關程式系統需要呼叫哪些套件等等。
* config：放置資料庫連線資料 (使用 sequelize-cli 套件自動產生)。
*  controllers：商用邏輯控制。
* middleware：用來檢查登入權限。
* migrations：放置產生不同 Model 資料表 (使用 sequelize-cli 套件自動產生)。
* models：定義資料表資料型態 (使用 sequelize-cli 套件自動產生)。
*  node_modules：放置下載使用的套件位置。
*  package.json/package-lock.json：專案資訊的重要檔案 (使用 `npm init` 自動產生)。
*  repositories：處理與資料庫進行交握。
*  router：設定網站網址路由。

<br>

以下詳細說明部分，只會說明 app.js、config、controllers、middleware、migrations、models、repositories、router (介紹會依照程式流程來介紹)。

<br>

### config

```json
{
  "development": {
    "username": "root",
    "password": "",
    "database": "node",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "dialectOptions": {
      "dateStrings": true,
      "typeCast": true
    },
    "timezone": "+08:00"
  },
  "test": {
    "username": "root",
    "password": "",
    "database": "node",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "dialectOptions": {
      "dateStrings": true,
      "typeCast": true
    },
    "timezone": "+08:00"
  },
  "production": {
    "username": "root",
    "password": "",
    "database": "node",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "dialectOptions": {
      "dateStrings": true,
      "typeCast": true
    },
    "timezone": "+08:00"
  }
}
```
由 sequelize-cli 套件自動產生，可以依照不同程式狀態(開發 development、測試 test、上線 production)來修改連線時的參數。

<br>

### migrations

檔案基本上都雷同，舉其中一個為說明。migrations 也是由 sequelize-cli 套件自動產生，相關指令會統一放在最後。

* 20220401093019-create-message.js
```yaml
"use strict";
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable("message", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      user_id: {
        allowNull: false,
        type: Sequelize.INTEGER,
        references: {
          model: "user",
          key: "id",
        },
      },
      content: {
        allowNull: false,
        type: Sequelize.STRING,
      },
      version: {
        type: Sequelize.INTEGER,
        defaultValue: 0,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: true,
        type: Sequelize.DATE,
      },
      deletedAt: {
        allowNull: true,
        type: Sequelize.DATE,
      },
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable("message");
  },
};
```
這個檔案的格式也是透過指令產生，會有 up 跟 down，up 就是執行指令後會產生什麼，主要都會是新增資料表或是修改資料表，down 則是回復的功能，可以將資料表給 dropTable 。我們主要會修改的地方是 createTable 內的資料，裡面的資料代表資料表的欄位，以下列出常用的格式：
* allowNull：是否為空值
* autoIncrement：自動累加
* primaryKey：主鍵
* type：裡面就放欄位類型，例如 INTEGER、STRING、DATE等等

### app.js

```js
"use strict";
const express = require("express");
const sessions = require("express-session");
const app = express();
const port = 8888;

// 設定 Session
const oneDay = 1000 * 60 * 60 * 1;
app.use(
  sessions({
    secret: "mySecret",
    name: "user",
    saveUninitialized: false,
    rolling: true,
    cookie: { maxAge: oneDay },
    resave: false,
  })
);

// 註冊路由
app.use("/api/v1", require("./router"));

// 檢查是否有table，沒有就建立
// const Message = require("./models").message;
// const User = require("./models").user;
// Message.sync();
// User.sync();

// 開啟監聽
app.listen(port, console.log("啟動 Server,Port:" + port));
```
1. 先引入會使用的套件 (express：node web 框架、express-session：session 套件)
2. 設定 app 為 express() 實例，port 為 8888。
3.  設定 session 失效時間、名稱等設定。
	* secret(必要)：用來簽章 sessionID 的cookie, 可以是一secret字串或是多個secret組成的一個陣列。
	* name：在response中，設定的 sessionID cookie 名字。預設是 connect.sid。
	* saveUninitialized：強制將未初始化的session存回 session store，未初始化的意思是它是新的而且未被修改。
	* rolling：強制在每一次回應時，重新設置一個sessionID cookie。
	* cookie：設定sessionID 的cookie相關選項。
	* resave：強制將session存回 session store, 即使它沒有被修改。
4.  註冊路由，連線 `http://127.0.0.1/api/v1` 後面會導向 router 檔案。
5. 註解部分為可以每次啟動後先檢查是否有table，沒有就建立。
6.  開啟 port(8888) 監聽。

<br>

### router

```js
"use strict";
const express = require("express");
const middleware = require("../middleware");
const router = express();

const { register, login, lojsut } = require("../controllers/auth");

const {
  getAllMessage,
  getMessage,
  createMessage,
  updateMessage,
  deleteMessage,
} = require("../controllers/message");

const {
  createReply,
  updateReply,
  deleteReply,
} = require("../controllers/reply");

// 註冊、登入、登出
router.post("/register", express.json(), register);
router.post("/login", express.json(), login);
router.post("/lojsut", lojsut);

// 查詢留言
router.get("/message", getAllMessage);
router.get("/message/:message_id", getMessage);

//需要驗證才可以使用（新增留言、修改留言、刪除留言、新增留言回覆、修改留言回覆、刪除留言回覆）
router.use(middleware);
router.post("/message", express.json(), createMessage);
router.patch("/message/:message_id", express.json(), updateMessage);
router.delete("/message/:message_id", deleteMessage);
router.post("/message/:message_id", express.json(), createReply);
router.patch("/message/:message_id/:reply_id", express.json(), updateReply);
router.delete("/message/:message_id/:reply_id", deleteReply);

module.exports = router;
```
設定路由，分別是註冊、登入、登出、新增留言、查詢全部留言、查詢 {id} 留言、修改 {id} 留言、刪除 {id} 留言，連接到不同的 controller function。express.json() 函數是為了要讓 body-parser 解析帶有 JSON 傳入後面的 controller，其中比較特別的是因為新增留言、修改留言、刪除留言、新增留言回覆、修改留言回覆、刪除留言回覆需要登入後才可以使用，所以多一個 middleware 來驗證是否登入。

<br>

### middleware

```js
"use strict";
module.exports = (req, res, next) => {
  if (!req.session.userid) {
    return res.status(401).json({ message: "用戶需要認證" });
  }
  next();
};
```
使用 session 來驗證是否有登入。

<br>

### models

會放置與資料表資料型態有關的資訊，其中 index.js 是由 sequelize-cli 套件自動產生，用來讀取目前寫在 config 的連線設定檔等資訊。

#### index.js

```js
"use strict";
const fs = require("fs");
const path = require("path");
const Sequelize = require("sequelize");
const basename = path.basename(__filename);
const env = process.env.NODE_ENV || "development";
const config = require(__dirname + "/../config/config.json")[env];
const db = {};

let sequelize;
if (config.use_env_variable) {
  sequelize = new Sequelize(process.env[config.use_env_variable], config);
} else {
  sequelize = new Sequelize(
    config.database,
    config.username,
    config.password,
    config
  );
}

fs.readdirSync(__dirname)
  .filter((file) => {
    return (
      file.indexOf(".") !== 0 && file !== basename && file.slice(-3) === ".js"
    );
  })
  .forEach((file) => {
    const model = require(path.join(__dirname, file))(
      sequelize,
      Sequelize.DataTypes
    );
    db[model.name] = model;
  });

Object.keys(db).forEach((modelName) => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```

<br>

#### user.js

```js
"use strict";
const { Model } = require("sequelize");
module.exports = (sequelize, DataTypes) => {
  class user extends Model {
    static associate(models) {
      user.hasMany(models.message, {
        foreignKey: "user_id",
      });
    }
  }
  user.init(
    {
      username: {
        type: DataTypes.STRING,
        unique: true,
      },
      password: {
        type: DataTypes.STRING,
      },
    },
    {
      sequelize,
      paranoid: true,
      freezeTableName: true,
      modelName: "user",
    }
  );
  return user;
};
```
此為 user 資料表的欄位資料，有預設的 init 初始化，比較特別的是如果有用到關聯性的外鍵等等要記得在 associate 做設定，`user.hasMany(models.message,{foreignKey: "user_id",})` 代表 user 這張表可以有很多個 message，其 message 外鍵是 user_id。

* paranoid：代表會執行軟刪除而不是硬刪除，但必須要多一個 `deletedAt` 來存放軟刪除時間。
* freezeTableName：因為 sequelize 會自動再產生資料表時加上複數，如果不想要就必須使用它讓 sequelize 不會自動加入 s (複數)。
* modelName：此 model 名稱。

<br>

#### message.js

```js
"use strict";
const { Model } = require("sequelize");
module.exports = (sequelize, DataTypes) => {
  class message extends Model {
    static associate(models) {
      message.hasMany(models.reply, {
        foreignKey: "message_id",
      });
      message.belongsTo(models.user, {
        foreignKey: "user_id",
      });
    }
  }
  message.init(
    {
      user_id: {
        type: DataTypes.INTEGER,
        allowNull: false,
      },
      content: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      version: {
        type: DataTypes.STRING,
      },
    },
    {
      sequelize,
      paranoid: true,
      freezeTableName: true,
      modelName: "message",
    }
  );
  return message;
};
```
此為 message 資料表的欄位資料，associate 設定有 `message.hasMany(models.reply,{foreignKey: "message_id",})` 代表 message 這張表可以有很多個 reply，其 reply 外鍵是 message_id。以及 `message.belongsTo(models.user, {foreignKey: "user_id",});` 代表 message 存在一對一的關係，外鍵是 user_id。

<br>

#### reply.js

```js
"use strict";
const { Model } = require("sequelize");
module.exports = (sequelize, DataTypes) => {
  class reply extends Model {
    static associate(models) {
      reply.belongsTo(models.message, {
        foreignKey: "message_id",
      });
      reply.belongsTo(models.user, {
        foreignKey: "user_id",
      });
    }
  }
  reply.init(
    {
      message_id: {
        type: DataTypes.INTEGER,
        allowNull: false,
      },
      user_id: {
        type: DataTypes.INTEGER,
        allowNull: false,
      },
      content: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      version: {
        type: DataTypes.STRING,
      },
    },
    {
      sequelize,
      paranoid: true,
      freezeTableName: true,
      modelName: "reply",
    }
  );
  return reply;
};
```
此為 reply 資料表的欄位資料，associate 設定有`reply.belongsTo(models.message, {foreignKey: "message_id",});` 代表 reply 存在一對一的關係，外鍵是 message_id，以及 `reply.belongsTo(models.user, {foreignKey: "user_id",});` 代表 reply 存在一對一的關係，外鍵是 user_id。

<br>

### controllers

我們遵循 MVC 的設計規範，所有的商用邏輯都會放在 controller 內，repository 只負責與資料庫進行交握。

#### auth.js

```js
const Auth = require("../repositories/auth");
const bcrypt = require("bcrypt");

// 註冊
const register = async (req, res) => {
  if (!req.body.username || !req.body.password) {
    return res.status(400).json({ user: "沒有正確輸入帳號或密碼" });
  }

  const salt = await bcrypt.genSalt(10);
  const bcrypt_password = await bcrypt.hash(req.body.password, salt);

  user = await Auth.register(req.body.username, bcrypt_password);
  if (!user[1]) {
    return res.status(400).json({ user: "username已存在" });
  }
  return res.status(201).json({ user: user });
};

// 登入
const login = async (req, res) => {
  if (!req.body.username || !req.body.password) {
    return res.status(400).json({ user: "沒有正確輸入帳號或密碼" });
  }

  user = await Auth.login(req.body.username, req.body.password);
  if (!user) {
    return res.status(400).json({ user: "帳號或密碼錯誤" });
  }
  session = req.session;
  session.userid = user.id;
  return res.status(200).json({ user: "登入成功" });
};

// 登出
const logout = async (req, res) => {
  session = req.session;
  session.destroy();
  return res.status(200).json({ user: "登出成功" });
};

module.exports = {
  register,
  login,
  logout,
};
```
主要有3 個部份，註冊、登入、登出：
* 註冊：會先驗證是否正確輸入資料(包含欄位是否錯誤等)，如果有錯，就會回應 400 以及沒有正確輸入帳號或密碼。接下來會使用到 `bcrypt` 來幫密碼進行加密處理，再把加密後的密碼以及帳號丟到 Auth.register repository 來進行資料庫的存取，如果錯誤會回應 400 以及 username 已存在，成功會回應 201 以及新增的帳號資訊。
* 登入：一樣會先檢查輸入資料，接著把資料丟到 Auth.login repository 來進行資料庫的驗證，如果錯誤會回應 400 以及帳號或密碼錯誤，成功會回應 200 以及登出成功。
* 登出：將 session 給清除，然後回應 200 以及登出成功。

<br>

#### message.js

(由於內容較多，所以依照功能拆分後說明)


**查詢留言功能**
```js
const Message = require("../repositories/message");
const Reply = require("../repositories/reply");

// 查詢所有留言
const getAllMessage = async (req, res) => {
  message = await Message.getAll();
  return res.status(200).json({ message: message });
};

//查詢{id}留言
const getMessage = async (req, res) => {
  if (!(message = await Message.get(req.params.message_id))) {
    return res.status(404).json({ message: "找不到留言" });
  }
  return res.status(200).json({ message: message });
};
```
先載入要使用的 repository ，查詢所有留言會使用 Message.getAll() repository 來進行資料庫的讀取，會回應 200 以及查詢內容，若是尚未有留言則會顯示空陣列。查詢{id}留言會將 `req.params.message_id` 丟到 Message.get repository 來進行資料庫的讀取，如果錯誤會回應 404 以及 找不到留言，成功會回應 200 以及查詢內容。

<br>

**新增留言功能**
```js
//新增留言
const createMessage = async (req, res) => {
  if (!req.body.content || req.body.content.length > 20) {
    return res.status(400).json({ message: "沒有輸入內容或長度超過20個字元" });
  }

  message = await Message.create(req.session.userid, req.body.content);
  return res.status(201).json({ message: message });
};
```
新增留言會先檢查輸入內容是否為空以及不能大於20個字元，如果錯誤就回應 400 以及沒有輸入內容或長度超過20個字元，再將 `req.session.userid`、`req.body.content` 丟到 Message.create repository 來進行資料庫的存取，成功會回應 201 以及新增的留言。

<br>

**修改留言功能**
```js
//修改留言
const updateMessage = async (req, res) => {
  if (!(message = await Message.get(req.params.message_id))) {
    return res.status(404).json({ message: "找不到留言" });
  }

  if (!req.body.content || req.body.content.length > 20) {
    return res.status(400).json({ message: "沒有輸入內容或長度超過20個字元" });
  }

  if (
    (await Message.update(
      req.params.message_id,
      req.session.userid,
      req.body.content,
      message["version"]
    )) == "0"
  ) {
    return res.status(400).json({ message: "修改留言失敗" });
  }
  return res.status(200).json({ message: "修改留言成功" });
};
```
修改留言因為需要先從資料中取得 `version` 來檢查樂觀鎖，所以會先將 `req.params.message_id` 丟到 Message.get repository 來進行資料庫的查詢，錯誤會回應 404 以及找不到留言。
先檢查輸入內容是否為空以及不能大於20個字元，如果錯誤就回應 400 以及沒有輸入內容或長度超過20個字元，再將 `req.params.message_id`、`req.session.userid`、`req.body.content`、`message["version"]` 丟到 Message.update repository 來進行資料庫的更新，錯誤會回應 400 以及修改留言失敗，成功會回應 200 以及修改留言成功。

<br>

**刪除留言功能**
```js
//刪除留言
const deleteMessage = async (req, res) => {
  if (!(await Message.delete(req.params.message_id, req.session.userid))) {
    return res.status(400).json({ message: "刪除留言失敗" });
  }
  // 刪除留言時同步刪除所有回覆
  await Reply.deleteMessage(req.params.message_id);
  return res.status(204).json({ message: "刪除留言成功" });
};
```
刪除留言會將 `req.params.message_id`、`req.session.userid` 丟到 Message.delete repository 來進行資料庫的軟刪除，錯誤會回應 400 以及刪除留言失敗，因為我們刪除留言後，不能在對該留言進行回覆的任何功能，所以一同軟刪除所有的回覆，成功後會回應 204 以及刪除留言成功。

<br>

#### reply.js

(由於內容較多，所以依照功能拆分後說明)

**新增回覆功能**

```js
const Reply = require("../repositories/reply");

//新增回覆
const createReply = async (req, res) => {
  if (!req.body.content || req.body.content.length > 20) {
    return res.status(400).json({ message: "沒有輸入回覆或長度超過20個字元" });
  }

  if (
    !(reply = await Reply.create(
      req.params.message_id,
      req.session.userid,
      req.body.content
    ))
  ) {
    return res.status(400).json({ message: "新增回覆失敗" });
  }
  return res.status(201).json({ message: reply });
};
```
先載入要使用的 repository，新增回覆先檢查輸入內容是否為空以及不能大於20個字元，如果錯誤就回應 400 以及沒有輸入內容或長度超過20個字元，再將 `req.params.message_id`、`req.session.userid`、`req.body.content` 丟到 Reply.create repository 來進行資料庫的存取，錯誤會回應 400 以及新增留言失敗，成功會回應 201 以及新增回覆內容。

<br>

**修改回覆功能**

```js
//修改回覆
const updateReply = async (req, res) => {
  if (!(reply = await Reply.get(req.params.reply_id, req.params.message_id))) {
    return res.status(404).json({ message: "找不到回覆" });
  }

  if (!req.body.content || req.body.content.length > 20) {
    return res.status(400).json({ message: "沒有輸入回覆或長度超過20個字元" });
  }

  if (
    (await Reply.update(
      req.params.reply_id,
      req.params.message_id,
      req.session.userid,
      req.body.content,
      reply["version"]
    )) == "0"
  ) {
    return res.status(400).json({ message: "修改回覆失敗" });
  }
  return res.status(200).json({ message: "修改回覆成功" });
};
```
修改留言因為需要先從資料中取得 `version` 來檢查樂觀鎖，所以會先將 `req.params.reply_id`、`req.params.message_id` 丟到 Reply.get repository 來進行資料庫的查詢，錯誤會回應 404 以及找不到留言。
先檢查輸入內容是否為空以及不能大於20個字元，如果錯誤就回應 400 以及沒有輸入內容或長度超過20個字元，再將 `req.params.reply_id `、`req.params.message_id`、`req.session.userid`、`req.body.content`、`reply["version"]` 丟到 Reply.update repository 來進行資料庫的更新，錯誤會回應 400 以及修改回覆失敗，成功會回應 200 以及修改回覆成功。

<br>

**刪除回覆功能**

```js
//刪除回覆
const deleteReply = async (req, res) => {
  if (
    !(await Reply.delete(
      req.params.reply_id,
      req.params.message_id,
      req.session.userid
    ))
  ) {
    return res.status(400).json({ message: "刪除回覆失敗" });
  }
  return res.status(204).json({ message: "刪除回覆成功" });
};
```
刪除回覆會將 `req.params.reply_id`、`req.params.message_id`、`req.session.userid` 丟到 Reply.delete repository 來進行資料庫的軟刪除，錯誤會回應 400 以及刪除回覆失敗，成功後會回應 204 以及刪除回覆成功。

<br>

### repositories

此會放置與資料庫進行交握的程式。

#### auth.js

```js
const User = require("../models").user;
const bcrypt = require("bcrypt");

const auth = {
  // 註冊
  async register(username, bcrypt_password) {
    return await User.findOrCreate({
      where: { username: username },
      defaults: { username: username, password: bcrypt_password },
    });
  },

  // 登入
  async login(username, password) {
    const user = await User.findOne({ where: { username: username } });
    if (user) {
      return (await bcrypt.compare(password, user.password)) ? user : false;
    }
  },
};

module.exports = auth;
```
auth 內主要有兩個與資料庫進行互動，分別是，註冊以及登入，註冊會將傳入的帳號以及加密過後密碼使用 `User.findOrCreate` 來進行查詢或新增，如果帳號不存在才會進行新增動作。登入會將傳入的帳號及密碼使用 `User.findOne` 來檢查是否存在，如果存在再檢查密碼與資料庫是否正確。

<br>

#### message.js

```js
const Message = require("../models").message;
const Reply = require("../models").reply;

const repository = {
  async getAll() {
    return await Message.findAll({ include: { model: Reply } });
  },

  async get(message_id) {
    return await Message.findOne({
      include: {
        model: Reply,
      },
      where: { id: message_id },
    });
  },

  async create(user_id, content) {
    return await Message.create({ user_id: user_id, content: content });
  },

  async update(message_id, user_id, content, version) {
    return await Message.update(
      {
        content: content,
        version: version + 1,
      },
      {
        where: {
          id: message_id,
          user_id: user_id,
          version: version,
        },
      }
    );
  },

  async delete(message_id, user_id) {
    return await Message.destroy({
      where: { id: message_id, user_id: user_id },
    });
  },
};

module.exports = repository;
```
message.js 裡面有會查詢全部留言、查詢{id}留言、新增留言、修改{id}留言、刪除{id}留言等，以下會說明各別負責功用：
* 查詢全部留言 getAll()：使用 `Message.findAll` 來顯示查詢結果，並且 include model Reply 回覆內容。
* 查詢{id}留言 get()：使用 `Message.findOne` 查詢 message.id 等於 message_id 的結果，並且 include model Reply 回覆內容。
* 新增留言 create()：使用 `Message.create` 新增 message.user_id 以及 message.content。
* 修改{id}留言 update()：使用 `Message.update` 更新 content 以及 version，且 message.id 要等於 message_id 及 message.user_id 等於 user_id 及 message.version 等於 version。
* 刪除{id}留言 delete()：使用 `Message.destroy` 刪除符合 message.id 等於 message_id 及 message.user_id 等於 user_id。
<br>

#### reply.js

```js
const Message = require("../models").message;
const Reply = require("../models").reply;

const reply = {
  async get(reply_id, message_id) {
    return await Reply.findOne({
      where: { id: reply_id, message_id: message_id },
    });
  },

  async create(message_id, user_id, content) {
    is_exist = await Message.findOne({
      where: { id: message_id },
    });
    if (is_exist) {
      return await Reply.create({
        message_id: message_id,
        user_id: user_id,
        content: content,
      });
    }
  },

  async update(reply_id, message_id, user_id, content, version) {
    return await Reply.update(
      {
        content: content,
        version: version + 1,
      },
      {
        where: {
          id: reply_id,
          message_id: message_id,
          user_id: user_id,
          version: version,
        },
      }
    );
  },

  async delete(reply_id, message_id, user_id) {
    return await Reply.destroy({
      where: {
        id: reply_id,
        message_id: message_id,
        user_id: user_id,
      },
    });
  },

  async deleteMessage(message_id) {
    return await Reply.destroy({
      where: {
        message_id: message_id,
      },
    });
  },
};

module.exports = reply;
```
reply.js 裡面有會新增回覆、修改{id}回覆、刪除{id}回覆等，除此之外還多了兩個 `get`、`deleteMessage` 用來取得 version 樂觀鎖以及同步刪除回覆功能，那以下會說明各別負責功用：
* 新增回覆 create()：因為要先確認留言是否被刪除，所以先使用 `Message.findOne` 檢查留言是否被刪除，在用 `Reply.create`  新增 reply.message_id 跟 reply.user_id 以及 reply.content。
* 修改{id}回覆 update()：使用 `Reply.update` 更新 content 以及 version，且 reply.id 要等於 reply_id 及 reply.message_id 等於 message_id 及 reply.user_id 等於 user_id 及 reply.version 等於 version。
* 刪除{id}回覆 delete()：使用 `Reply.destroy ` 刪除符合 reply.id 等於 reply_id 跟 reply.message_id 等於 message_id 及 reply.user_id 等於 user_id。

<br>

## 常用指令 (sequelize-cli)

* sequelize db:migrate：將資料表依照 up 內容執行([migrate 檔案](https://pin-yi.me/node-restful-api-repository-messageboard/#migrations))。
* sequelize db:migrate:undo:all：將資料表依照 down 內容執行([migrate 檔案](https://pin-yi.me/node-restful-api-repository-messageboard/#migrations))。
* sequelize db:seed：產生假資料

<br>

## Postman 測試

### 註冊 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/register-success.webp" width="700" caption="註冊 成功" >}}

<br>

### 註冊 - 失敗(無輸入)

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/register-error-1.webp" width="700" caption="註冊 失敗(無輸入)" >}}

<br>

### 註冊 - 失敗(已經註冊過)

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/register-error-2.webp" width="700" caption="註冊 失敗(已經註冊過)" >}}

<br>

### 登入 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/login-success.webp" width="700" caption="登入 成功" >}}

<br>

### 登入 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/login-error.webp" width="700" caption="登入 失敗" >}}

<br>

### 查詢全部留言 - 成功(無資料)

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/get-success-1.webp" width="700" caption="查詢留言 成功(無資料)" >}}

<br>

### 查詢全部留言 - 成功(有資料)

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/get-success-2.webp" width="700" caption="查詢留言 成功(有資料)" >}}

<br>

### 查詢{id}留言 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/get-id-success.webp" width="700" caption="查詢{id}留言 成功" >}}

<br>

### 查詢{id}留言 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/get-error.webp" width="700" caption="查詢{id}留言 失敗" >}}

<br>

### 新增留言 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/create.webp" width="700" caption="新增留言 成功" >}}

<br>

### 新增留言 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/create-error.webp" width="700" caption="新增留言 失敗" >}}

<br>

### 修改{id}留言 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/patch-success.webp" width="700" caption="修改 {id}留言 成功" >}}

<br>

### 修改{id}留言 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/patch-error.webp" width="700" caption="修改 {id}留言 失敗" >}}

<br>

### 刪除{id}留言 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/delete.webp" width="700" caption="刪除 {id}留言 成功" >}}

<br>

### 刪除{id}留言 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/delete-error.webp" width="700" caption="刪除 {id}留言 失敗" >}}

<br>

### 新增回覆 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-create.webp" width="700" caption="新增回覆 成功" >}}

<br>

### 新增回覆 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-create-error.webp" width="700" caption="新增回覆 失敗" >}}

<br>

### 修改{id}回覆 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-patch-success.webp" width="700" caption="修改 {id}回覆 成功" >}}

<br>

### 修改{id}回覆 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-patch-error.webp" width="700" caption="修改 {id}回覆 失敗" >}}

<br>

### 刪除{id}回覆 - 成功

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-delete.webp" width="700" caption="刪除 {id}回覆 成功" >}}

<br>

### 刪除{id}回覆 - 失敗

{{< figure src="/nodejs/nodejs-restful-api-repository-messageboard/reply-delete-error.webp" width="700" caption="刪除 {id}回覆 失敗" >}}

<br>

## 參考資料

Node.js 官網：[https://nodejs.dev/learn/introduction-to-nodejs](https://nodejs.dev/learn/introduction-to-nodejs)

Sequelize：[https://sequelize.org/v6/index.html](https://sequelize.org/v6/index.html)

透過 sequelize 來達成 DB Schema Migration：[https://hackmd.io/@TSMI_E7ORNeP8YBbWm-lFA/ryCtaVW_M?print-pdf#%E4%BD%BF%E7%94%A8sequelize%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%BC%B5user-table](https://hackmd.io/@TSMI_E7ORNeP8YBbWm-lFA/ryCtaVW_M?print-pdf#%E4%BD%BF%E7%94%A8sequelize%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%BC%B5user-table)

How to create JOIN queries with Sequelize：[https://sebhastian.com/sequelize-join/](https://sebhastian.com/sequelize-join/)

