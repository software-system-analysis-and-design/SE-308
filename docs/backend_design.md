# 后端软件设计文档

> 主要负责人：邱晓裕
>
> 审核人：邱奕浩，蒲其刚，邵梓硕

## 1. 技术选型及理由

本项目后端的项目架构由以下技术组成：

* Nginx：
  * Nginx使用了最新的epoll（Linux2.6内核）和kqueue（freeBSD）网路I/O模型，是一个高性能的HTTP和反向代理服务器，官方测试Nginx能够支撑5万并发连接，实际生产环境中可以支撑2~4万并发连接数；
  * Nginx是一个高性能的HTTP和反向代理服务器，为后台提供一定的安全保障；作为反向代理，Nginx可以将服务端跟外界隔离开；
  * Nginx的设计极具扩展性，它完全是由多个不同功能、不同层次、不同类型且耦合度极低的模块组成。因此，当对某一个模块修复Bug或进行升级时，可以专注于模块自身，无须在意其他；
  * Nginx学习门槛低，上手较容易，且具有使用经验。
* Go Iris Web 框架：
  * `Iris`以简单而强大的`api`而闻名。 除了`Iris`为您提供的低级访问权限。 `Iris`同样擅长`MVC`。 它是唯一一个拥有`MVC`架构模式丰富支持的`Go Web`框架，性能成本接近于零。
  * `Iris`为您提供构建面向服务的应用程序的结构。 用`Iris`构建微服务很容易。
  * 专注于高性能，简单流畅的API，高扩展性
  * 具有强大的路由和中间件生态系统
* PostgreSQL：
  * PostgreSQL对事务的支持与MySQL相比，经历了更为彻底的测试。对于一个严肃的商业应用来说，事务的支持是不可或缺的。
  * MySQL对于无事务的MyISAM表。采用表锁定，一个长时间运行的查询很可能会长时间地阻碍对表的更新。而PostgreSQL不 存在这样的问题。
  * PostgreSQL支持存储过程，而目前MySQL不支持，对于一个严肃的商业应用来说，作为数据库本身，有众多的商业逻辑的存在， 此时使用存储过程可以在较少地增加数据库服务器的负担的前提下，对这样的商业逻辑进行封装，并可以利用数据库服务器本身的内在机制对存储过程的执行进行优 化。此外存储过程的存在也避免了在网络上大量的原始的SQL语句的传输，这样的优势是显而易见的。
  * 对视图的支持，视图的存在同样可以最大限度地利用数据库服务器内在的优化机制。而且对于视图权限的合理使用，事实上可以提供行级别的权 限，这是MySQL的权限系统所无法实现的。
  * PostgreSQL有着之前数据库课程的学习基础，上手较为快；
* Docker：
  * 标准化应用发布，docker容器包含了运行环境和可执行程序，可以跨平台和主机使用；
  * 节约时间，快速部署和启动，VM启动一般是分钟级，docker容器启动是秒级；
  * 方便构建基于SOA架构或微服务架构的系统，通过服务编排，更好的松耦合；
  * 节约成本，以前一个虚拟机至少需要几个G的磁盘空间，docker容器可以减少到MB级；
  * 方便持续集成，通过与代码进行关联使持续集成非常方便；
  * 一次构建，多次交付：类似于集装箱的“一次装箱，多次运输”，Docker镜像可以做到“一次构建，多次交付”。当涉及到应用程序多副本部署或者应用程序迁移时，更能体现Docker的价值。

## 2. 架构设计

项目的架构设计如下图所示，该项目使用经典的CS架构，使用Nginx进行反向代理，并使用多个dockers运行后台程序，docker与数据库之间进行数据交互，可以很方便提高服务器的抗压能力、性能。

![1561560075556](image/1561560075556.png)

## 3. 模块划分

项目后端代码目录结构如下图所示：

```bash
文件夹 PATH 列表
卷序列号为 0002-F2B8
C:.
├─controllers   # 控制类，管理数据库
├─global		# 全局类，包括一些控制信息
├─helper		# 工具类，包括一些简单的相应类
├─main			# 包括主函数和各种路由函数
├─models		# 模型类
└─test			# 数据库相关测试代码
```

## 4. 软件设计技术

### （1）面向对象编程

面向对象设计与编程是最常见的选择。多年产业实践证明，面向对象具有具有易于理解、易于复用（reuse）和可扩展（extend）的优势。因此，本项目使用面向对象编程的方式：

* 对象：

```bash
├─models
│      answer.go				# 答案类
│      message.go				# 消息类
│      option.go				# 选项类
│      question.go				# 问题类
│      questionnaireFormat.go	# 问卷类
│      record.go				# 答案记录类
│      taskPreview.go			# 问卷简要信息类
│      user.go					# 用户类
```

* 接口：

```bash
├─controllers
│      db_controller.go  		# 数据库核心代码
│      msg_controller.go		# 数据库管理消息模块代码
│      qFormat_controller.go	# 数据库管理问卷模块代码
│      record_controller.go		# 数据库管理用户提交的答案模块代码
│      userdb_controller.go		# 数据库管理用户信息的模块代码
```

实现方式是：

- 定义抽象数据
- 定义接口
- 定义实现类

以用户信息User为例：

* 定义抽象数据：

```go
// user.go

package models

// User is our sample data structure.
// which could wrap by embedding the models.User or
// declare new fields instead butwe will use this models
// as the only one User model in our application,
// for the shake of simplicty.
type User struct {
	Phone       string `json:"phone"`
	Remain      int    `json:"remain"`
	Iscow       bool   `json:"iscow"`
	Name        string `json:"name"`
	Password    string `json:"password"`
	Gender      string `json:"gender"`
	Age         int    `json:"age"`
	University  string `json:"university"`
	Company     string `json:"company"`
	Description string `json:"description"`
	Class       string `json:"class"`
}
```

* 定义接口

```go
// userdb_controller.go 

type User_Interface interface {
    // 对用户信息进行查询
	QueryUser(phone string) (ok bool)

    // 选择指定ID的用户
	SelectUser(phone string) (user *models.User, ok bool)

    // 新建用户
	InsertUser(user *models.User) (ok bool)

    // 更新用户信息
	UpdateUser(phone string, user *models.User) (updatedUser *models.User, ok bool)

    // 更新用户余额
	UpdateMoney(phone string, money int) (ok bool)

    // 删除用户
	DeleteUser(phone string) (ok bool)
}
```

* 接口实现：

```go
// QueryUser 对用户信息进行查询
func (r *DBRepository) QueryUser(phone string) (ok bool) {

	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)
	db, err := sql.Open("postgres", psqlInfo)

	var count int64
	err = db.QueryRow("select count(*) from t_user where phone=$1", phone).Scan(&count)

	ok = true
	if err != nil || count == 0 {
		ok = false
	}

	db.Close()
	return ok
}

// SelectUser 选择指定ID的用户
func (r *DBRepository) SelectUser(phone string) (user *models.User, ok bool) {

	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)
	db, err := sql.Open("postgres", psqlInfo)
	rows, err := db.Query("select * from t_user where phone=$1", phone)

	userObj := models.User{}
	rows.Next()
	err = rows.Scan(&userObj.Phone, &userObj.Remain, &userObj.Iscow, &userObj.Name, &userObj.Password, &userObj.Gender,
		&userObj.Age, &userObj.University, &userObj.Company, &userObj.Description, &userObj.Class)

	if err != nil {
		fmt.Printf("could not find user, %v", err)
		db.Close()
		return nil, false
	}

	db.Close()
	return &userObj, ok
}

// InsertUser 新建用户
func (r *DBRepository) InsertUser(userObj *models.User) (ok bool) {

	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)
	db, err := sql.Open("postgres", psqlInfo)
	ok = true

	stmt, err := db.Prepare("insert into t_user (phone, remain, iscow, name, password, gender, age, university, company, description, class)" +
		" values($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11)")
	if err != nil {
		ok = false
	}
	_, err = stmt.Exec(userObj.Phone, 0, userObj.Iscow, userObj.Name, userObj.Password, userObj.Gender,
		userObj.Age, userObj.University, userObj.Company, userObj.Description, userObj.Class)
	if err != nil {
		fmt.Printf("could not insert user, %v", err)
		ok = false
	}

	db.Close()
	return ok
}

// UpdateUser 更行用户信息
func (r *DBRepository) UpdateUser(phone string, user *models.User) (updatedUser *models.User, ok bool) {

	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)
	db, err := sql.Open("postgres", psqlInfo)
	ok = true

	stmt, err := db.Prepare("update t_user set iscow=$1, name=$2, password=$3, gender=$4, age=$5, " +
		"university=$6, company=$7, description=$8, class=$9 WHERE phone=$10")
	if err != nil {
		ok = false
	}
	_, err = stmt.Exec(user.Iscow, user.Name, user.Password, user.Gender,
		user.Age, user.University, user.Company, user.Description, user.Class, user.Phone)
	if err != nil {
		ok = false
	}
	db.Close()

	return user, ok
}

// UpdateMoney 更新用户余额
func (r *DBRepository) UpdateMoney(phone string, money int) (ok bool) {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)
	db, err := sql.Open("postgres", psqlInfo)
	ok = true

	stmt, err := db.Prepare("update t_user set remain=$1 WHERE phone=$2")
	if err != nil {
		ok = false
	}
	_, err = stmt.Exec(money, phone)
	if err != nil {
		ok = false
	}
	db.Close()

	return ok
}

func checkErr(err error) {
	if err != nil {
		panic(err)
	}
}
```

### （2）MVC架构（Model View Controller）

Iris 有一个非常强大和炽热 [快速](https://github.com/kataras/iris/tree/master/_benchmarks) 的 MVC 支持，你可以从一个方法函数里返回你想要的任何类型的值，并且这个值会发送到你预期的客户端。

项目目录树为：

```bash
文件夹 PATH 列表
卷序列号为 0002-F2B8
C:.
│  
├─controller
│      db_controller.go  		# 数据库核心代码
│      msg_controller.go		# 数据库管理消息模块代码
│      qFormat_controller.go	# 数据库管理问卷模块代码
│      record_controller.go		# 数据库管理用户提交的答案模块代码
│      userdb_controller.go		# 数据库管理用户信息的模块代码
├─global
│      config.go				# 全局配置文件，包含一些全局配置信息
│      
├─helper
│      response.go				# 后台返回信息的一些简单模板类
│      
├─main
│      backend.go				# 后台程序，用于生成消息通知
│      main.go					# 主程序
│      msgRounter.go			# 消息相关路由
│      qFormatRouter.go			# 问卷相关路由
│      recordRouter.go			# 用户提交答案相关路由
│      userRouter.go			# 用户信息相关路由
│      
├─models
│      answer.go				# 答案类
│      message.go				# 消息类
│      option.go				# 选项类
│      question.go				# 问题类
│      questionnaireFormat.go	# 问卷类
│      record.go				# 答案记录类
│      taskPreview.go			# 问卷简要信息类
│      user.go					# 用户类
│      
└─test
        qformat_controller_test.go	# 问卷数据库测试文件
        record_controller_test.go	# 用户提交答案数据库测试文件
        user_controller_test.go		# 用户信息数据库测试文件
```

其中：models目录下为模型，表示应用程序核心，比如用户信息类，controller目录下为控制器，对数据库的各种操作进行控制，main目录下包含各种路由实现，用于将数据信息进行显示（由于后端项目不包括可视化，因此这里只是将数据进行JSON字符串转化然后发送给请求方）；

### （3）单例模式

单例模式（Singleton Pattern）是经典的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。单例对象的类必须保证只有一个实例存在。

在本项目中，我们的数据库管理使用了单例模型，这样整个系统只需要拥有一个的全局对象，有利于我们协调系统整体的行为。

```go
// db_controller.go

package controllers

import (
	"database/sql"
	"fmt"
	"sync"

	"../global"
)

var once sync.Once

type DBRepository struct {
}

var dbRepository *DBRepository

func GetDBInstance() *DBRepository {
	once.Do(func() {
		dbRepository = NewDBRepository()
	})
	return dbRepository
}

// NewUserRepository returns a new user repository,
// the one and only repository type in our example.
func NewDBRepository() *DBRepository {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		global.Host, global.Port, global.User, global.Password, global.Dbname)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}
	defer db.Close()

	err = db.Ping()
	if err != nil {
		panic(err)
	}

	return &DBRepository{}
}
```

### （4）抽象工厂模式

抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

**适用性**

- 一个系统要独立于它的产品的创建、组合和表示时。
- 一个系统要由多个产品系列中的一个来配置时。
- 当你要强调一系列相关的产品对象的设计以便进行联合使用时。
- 当你提供一个产品类库，而只想显示它们的接口而不是实现时。

本项目需要对多个数据库的表进行交互，因此，我们使用抽象工厂模式，根据操作的表不同定义不同的接口，然后对这些接口进行实现：

* 用户数据库操作接口：

```go
// userdb_controller.go
type User_Interface interface {
	// 对用户信息进行查询
	QueryUser(phone string) (ok bool)

	// 选择指定ID的用户
	SelectUser(phone string) (user *models.User, ok bool)

	// 新建用户
	InsertUser(user *models.User) (ok bool)

	// 更新用户信息
	UpdateUser(phone string, user *models.User) (updatedUser *models.User, ok bool)

	// 更新用户余额
	UpdateMoney(phone string, money int) (ok bool)

	// 删除用户
	DeleteUser(phone string) (ok bool)
}
```

* 问卷数据库操作接口：

```go
// qFormat_controller.go
type QFormat_Interface interface {
	// 根据ID查询问卷是否存在
	QueryQFormat(id string) (ok bool)

	// 根据ID获取指定问卷
	SelectQFormat(id string) (qFormat *models.QuestionnaireFormat, ok bool)

	// 新建问卷
	InsertQFormat(qFormat *models.QuestionnaireFormat) (ok bool, id string)

	// 更新问卷
	UpdateQFormat(id string, qFormat *models.QuestionnaireFormat) (updatedQFormat *models.QuestionnaireFormat, ok bool)

	// 将问卷移动到回收站
	TrashQFormat(id string, inTrash int) (ok bool)

	// 删除问卷
	DeleteQFormat(id string) (ok bool)

	// 获取所有问卷
	SelectAllQFormats() (taskPreviews []models.TaskPreview, ok bool)

	// 获取所有有效问卷
	SelectValidFormats() (taskPreviews []models.TaskPreview, ok bool)

	// 根据关键字搜索问卷
	SearchQFormats(str string) (taskPreviews []models.TaskPreview, ok bool)

	// 添加问卷的回答个数
	AddOneQFormats(id string) (ok bool)
}
```

* 问卷提交记录数据库相关接口：

```go
// record_controller.go 
type Record_Interface interface {
	// 查询记录 
	QueryRecord(taskID string, userID string) (ok bool)

	// 获取记录
	SelectRecord(taskID string, userID string) (record *models.Record, ok bool)

	// 新建记录
	InsertRecord(record *models.Record) (ok bool)

	// 更新记录
	UpdateRecord(taskID string, userID string, record *models.Record) (ok bool)

	// 删除记录
	DeleteRecord(taskID string, userID string) (ok bool)

	// 获取所有记录
	SelectAllRecords(taskID string) (records []models.Record, ok bool)
}
```

* 用户消息相关接口

```go
// msg_controller.go
type Message_Interface interface {
	// 获取指定消息
	SelectMessage(messageID string) (message *models.Message, ok bool)

	// 获取未读消息个数
	GetCount(userID string) (count int, ok bool)

	// 获取所有消息
	GetAllMessage(userID string) (messages []models.Message, ok bool)

	// 新建消息
	InsertMessage(message *models.Message) (ok bool)

	// 更改消息的已读状态
	ReadMessage(messageID string, userID string, state int) (ok bool)

	// 删除消息
	DeleteMessage(messageID string, userID string) (ok bool)

	// 获取所有的未读消息
	GetUnReadMessage(userID string) (messages []models.Message, ok bool)

	// 获取所有的已读消息
	GetReadMessage(userID string) (messages []models.Message, ok bool)
}
```

## 5. 数据库设计

### （1）数据库物理模型

* User

| Field       | Type    | Key         | Description                      |
| ----------- | ------- | ----------- | -------------------------------- |
| phone       | text    | PRIMARY KEY | 用户的唯一身份标识               |
| remain      | integer |             | 用户的余额                       |
| iscow       | boolean |             | 是否是奶牛                       |
| name        | text    |             | 用户名                           |
| password    | text    |             | 用户密码                         |
| gender      | text    |             | 用户的性别                       |
| age         | integer |             | 用户的年龄                       |
| university  | text    |             | 用户所在的大学，（学生特有属性） |
| company     | text    |             | 用户所在的组织，（奶牛特有属性） |
| description | text    |             | 用户的个人描述                   |
| class       | text    |             | 用户所在的年级，（学生特有属性） |

* qFormat

| Field          | Type    | Key         | Description            |
| -------------- | ------- | ----------- | ---------------------- |
| taskID         | text    | PRIMARY KEY | 任务的唯一身份标识     |
| taskName       | text    |             | 任务名                 |
| InTrash        | integer |             | 任务是否在回收站之中   |
| taskType       | text    |             | 任务的类型             |
| creator        | text    | FOREIGN KEY | 任务的创建者           |
| description    | text    |             | 任务的描述信息         |
| money          | integer |             | 完成任务可以获得的奖励 |
| number         | integer |             | 任务预计完成的数量     |
| finishedNumber | integer |             | 任务已经完成的数量     |
| publishTime    | text    |             | 任务的发布时间         |
| endTime        | text    |             | 任务的结束时间         |
| chooseData     | text    |             | 任务的信息，json字符串 |

* record

| Field  | Type | Key         | Description                |
| ------ | ---- | ----------- | -------------------------- |
| taskID | text | PRIMARY KEY | 任务的ID                   |
| userID | text | PRIMARY KEY | 用户的ID                   |
| data   | text |             | 提交的答案数据，json字符串 |

- message

| Field    | Type    | Key         | Description              |
| -------- | ------- | ----------- | ------------------------ |
| msgID    | text    | PRIMARY KEY | 消息的ID                 |
| state    | integer |             | 消息的状态（已读和未读） |
| receiver | text    | FOREIGN KEY | 消息的接受者             |
| time     | text    |             | 消息的时间               |
| title    | text    |             | 消息的标题               |
| content  | text    |             | 消息的内容               |

### （2）用户及权限系统数据库设计

![1561349829988](image/db.png)

### （3）ER模型

![1561351902461](image/er.png)

## 6. API 设计

### （1）REST API 设计规范

* 协议：API与用户的通信协议，总是使用HTTPs协议。

* HTTP动词：对于资源的具体操作类型，由HTTP动词表示
  * GET（SELECT）：从服务器取出资源（一项或多项）
  * POST（CREATE）：在服务器新建一个资源。
  * PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
  * DELETE（DELETE）：从服务器删除资源。

* 状态码

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

200 – OK – 一切正常

201 – OK – 新的资源已经成功创建

204 – OK – 资源已经成功擅长

304 – Not Modified – 客户端使用缓存数据

400 – Bad Request – 请求无效，需要附加细节解释如 “JSON无效”

401 – Unauthorized – 请求需要用户验证

403 – Forbidden – 服务器已经理解了请求，但是拒绝服务或这种请求的访问是不允许的。

404 – Not found – 没有发现该资源

422 – Unprocessable Entity – 只有服务器不能处理实体时使用，比如图像不能被格式化，或者重要字段丢失。

500 – Internal Server Error – API开发者应该避免这种错误。

更新和创建操作应该返回对应的状态码，防止用户多次的API调用。

* 错误处理

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。 使用详细的错误包装错误：

```
{
  "errors": [
   {
    "userMessage": "Sorry, the requested resource does not exist",
    "internalMessage": "No car found in the database",
    "code": 34,
    "more info": "http://dev.mwaysolutions.com/blog/api/v1/errors/12345"
   }
  ]
}
```

### （2）API 设计文档

本项目使用ApiPost生成API设计文档：https://swsad-dalaotelephone.github.io/docs/api/