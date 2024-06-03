# Dao层

## 数据库创建

在`mysql`中执行以下`sql`语句创建数据库表

```mysql
CREATE DATABASE IF NOT EXISTS mydb;
USE mydb;
CREATE TABLE IF NOT EXISTS user (
    id BIGINT AUTO_INCREMENT COMMENT "user id",
    name VARCHAR(30) COMMENT "user name",
    password VARCHAR(30) COMMENT "user password",
    create_at DATETIME COMMENT "create time",
    update_at DATETIME COMMENT "last update time",
    delete_at DATETIME COMMENT "delete time",
    PRIMARY KEY(id)
)ENGINE = InnoDB DEFAULT CHARSET = utf8;
```

`create_at，update_at，delete_at` 三个字段这样命名时由框架自动维护。

##  ORM配置

### Mysql

下载`mysql`驱动,命令行输入：

```bash
go get github.com/gogf/gf/contrib/drivers/mysql/v2
```

在`main.go`中`import`并初始化驱动

```go
import _ "github.com/gogf/gf/contrib/drivers/mysql/v2"
```

在`Manifest/config/config.yaml`中添加如下配置：

```yaml
"type:username:password@protocol(address)[/dbname][?param1=value1&...&paramN=valueN]" //link格式 类型:账号:密码@协议(地址)/数据库名称?特性配置
```

```yaml
database:
	debug: true
	link: "mysql:root:123456@tcp(127.0.0.1:3306)/mydb?loc=Local&parseTime=true"
```

`loc=Local` 为使用本地时间

`parseTime=true` 时间自动转换为` time.Time` 类型，若为 `false` 自动转换为 `[]byte/string` 类型

`Terminal`

## gen dao 代码生成

在`hack/config.yaml` 中添加如下配置

```
gfcli:
  gen:
    dao:
      - link: "mysql:root:123456@tcp(127.0.0.1:3306)/mydb?loc=Local&parseTime=true"
        tables: "user"gfcli:
  gen:
    gfcli:
  gen:
    dao:
      - link: "mysql:root:123456@tcp(127.0.0.1:3306)/mydb?loc=Local&parseTime=true"
        tables: "user"
```

`tables`:  后的是数据库表名，如果多个按如下格式 `"tablename1,tablename2,tablename3"`

在命令行中输入

```
gf gen dao
```

![image-20240329095900035](E:\workplace\Markdown\goStudyNote\gfUseNote\5 Dao.assets\image-20240329095900035.png)

可以看到生成了多个文件。其中internal\user.go中创建了userDao对象。

```
var (
	// User is globally public accessible object for table user operations.
	User = userDao{
		internal.NewUserDao(),
	}
)
```

可以通过该对象的Ctx方法获取对应表的链式安全的数据模型

```
func (dao *UserDao) Ctx(ctx context.Context) *gdb.Model {
	return dao.DB().Model(dao.table).Safe().Ctx(ctx)
}
```

## NoSQL

### Redis 配置

在`Terminal`输入下面命令获取gf框架的redis驱动

```bash
go get -u github.com/gogf/gf/contrib/nosql/redis/v2
```

在`main.go`中导入并初始化驱动

```go
import _ "github.com/gogf/gf/contrib/nosql/redis/v2"
```

在`Manifest/config/config.yaml`中添加如下配置

```yaml
redis:
  default:
    address: 127.0.0.1:6379
    db:      1
    
  cache:
    address:     127.0.0.1:6379
    db:          1
    pass:        123456
    idleTimeout: 600
```

其中的 `default` 和 `cache` 分别表示配置分组名称，我们在程序中可以通过该名称获取对应配置的 `redis` 单例对象。不传递分组名称时，默认使用 `redis.default` 配置分组项)来获取对应配置的 `redis` 客户端单例对象。

### 使用示例

```go
ctx := gctx.New()
// redis字符串操作
g.Redis().Set(ctx, "k", "v")
v, _ := g.Redis().Get(ctx, "k")
// setex
g.Redis().SetEX(ctx, "keyEx", "v4", 2000)
v3, _ := g.Redis().Get(ctx, "keyEx")	
// list
g.Redis().RPush(ctx, "keyList", "v4")
v4, _ := g.Redis().LPop(ctx, "keyList")
g.Log().Info(ctx, v4.String())
// hash
g.Redis().HSet(ctx, "keyHash", g.Map{"v1": "v5"})
v5, _ := g.Redis().HGet(ctx, "keyHash", "v1")
g.Log().Info(ctx, v5.String())	
// set
g.Redis().SAdd(ctx, "keySet", "v6")
v6, _ := g.Redis().SPop(ctx, "keySet")	
// zset
g.Redis().ZAdd(ctx, "keySortSet", &gredis.ZAddOption{}, gredis.ZAddMember{Score: 1, Member: "v7"})
v7, _ := g.Redis().ZRem(ctx, "keySortSet", "v7")
```

