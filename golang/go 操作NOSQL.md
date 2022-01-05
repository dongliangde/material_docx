# go 操作NOSQL

## redis 

```
package main

import (
   "fmt"
   "github.com/go-redis/redis"
)

//操作redis
func main() {
   //初始化配置
   client := redis.NewClient(&redis.Options{
      Addr:     "127.0.0.1:6379",
      Password: "", // no password set
      DB:       0,  // use default DB
   })
   //测试redis是否访问正常
   pong, err := client.Ping().Result()
   fmt.Println(pong, err)

   err = client.Set("key", "value", 0).Err()
   if err != nil {
      panic(err)
   }

   val, err := client.Get("key").Result()
   if err != nil {
      panic(err)
   }
   fmt.Println("key", val)

   val2, err := client.Get("key2").Result()
   if err == redis.Nil {
      fmt.Println("key2 does not exists")
   } else if err != nil {
      panic(err)
   } else {
      fmt.Println("key2", val2)
   }
}
```

## Mongodb

## 获取包

```
go get go.mongodb.org/mongo-driver/mongo
go mod vendor
```

## 连接数据库

```
func main() {
   var (
      client     *mongo.Client
      err        error
      db         *mongo.Database
      collection *mongo.Collection
   )
   //1.建立连接
   if client, err = mongo.Connect(context.TODO(), options.Client().ApplyURI("mongodb://localhost:27017").SetConnectTimeout(5*time.Second)); err != nil {
      fmt.Print(err)
      return
   }
   //2.选择数据库 my_db
   db = client.Database("my_db")

   //3.选择表 my_collection
   collection = db.Collection("my_collection")
   collection = collection
}
```

### 将连接到mongoDB的函数抽出来

```
package util

import (
    "context"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
    "log"
)

var mgoCli *mongo.Client

func initEngine() {
    var err error
    clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")

    // 连接到MongoDB
    mgoCli, err = mongo.Connect(context.TODO(), clientOptions)
    if err != nil {
        log.Fatal(err)
    }
    // 检查连接
    err = mgoCli.Ping(context.TODO(), nil)
    if err != nil {
        log.Fatal(err)
    }
}
func GetMgoCli() *mongo.Client {
    if mgoCli == nil {
        initEngine()
    }
    return mgoCli
}
package main

import (
   "awesomeProject/mongodb/util"
   "context"
   "fmt"
   "go.mongodb.org/mongo-driver/mongo"
   "go.mongodb.org/mongo-driver/mongo/options"
   "time"
)
func main() {
    var (
        client = util.GetMgoCli()
        db         *mongo.Database
        collection *mongo.Collection
    )
    //2.选择数据库 my_db
    db = client.Database("my_db")

    //选择表 my_collection
    collection = db.Collection("my_collection")
    collection = collection
}
```

## 插入一条数据

构建几个结构体

```
package model
type TimePorint struct {
   StartTime int64 `bson:"startTime"` //开始时间
   EndTime   int64 `bson:"endTime"`   //结束时间
}
type LogRecord struct {
   JobName string     `bson:"jobName"` //任务名
   Command string     `bson:"command"` //shell命令
   Err     string     `bson:"err"`     //脚本错误
   Content string     `bson:"content"` //脚本输出
   Tp      TimePorint //执行时间
}
```

main函数：

```
package main

import (
    "awesomeProject/mongodb/model"
    "awesomeProject/mongodb/util"
    "context"
    "fmt"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "go.mongodb.org/mongo-driver/mongo"
)
func main() {
    var (
        client = util.GetMgoCli()
        err        error
        collection *mongo.Collection
        lr         *model.LogRecord
        iResult    *mongo.InsertOneResult
        id         primitive.ObjectID
    )
    //2.选择数据库 my_db里的某个表
    collection = client.Database("my_db").Collection("my_collection")

    //插入某一条数据
    if iResult, err = collection.InsertOne(context.TODO(), lr); err != nil {
        fmt.Print(err)
        return
    }
    //_id:默认生成一个全局唯一ID
    id = iResult.InsertedID.(primitive.ObjectID)
    fmt.Println("自增ID", id.Hex())
}
```

**去数据库中检验**

![img](https://pic3.zhimg.com/80/v2-bb2cd63ce5c7c52f85b72ae765b8a49e_720w.jpg)

## 批量插入数据

```
package main

import (
    "awesomeProject/mongodb/model"
    "awesomeProject/mongodb/util"
    "context"
    "fmt"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "go.mongodb.org/mongo-driver/mongo"
    "log"
    "time"
)
func main() {
    var (
        client = util.GetMgoCli()
        err        error
        collection *mongo.Collection
        result     *mongo.InsertManyResult
        id         primitive.ObjectID
    )
    collection = client.Database("my_db").Collection("test")

    //批量插入
    result, err = collection.InsertMany(context.TODO(), []interface{}{
        model.LogRecord{
            JobName: "job10",
            Command: "echo 1",
            Err:     "",
            Content: "1",
            Tp: model.TimePorint{
                StartTime: time.Now().Unix(),
                EndTime:   time.Now().Unix() + 10,
            },
        },
        model.LogRecord{
            JobName: "job10",
            Command: "echo 2",
            Err:     "",
            Content: "2",
            Tp: model.TimePorint{
                StartTime: time.Now().Unix(),
                EndTime:   time.Now().Unix() + 10,
            },
        },
        model.LogRecord{
            JobName: "job10",
            Command: "echo 3",
            Err:     "",
            Content: "3",
            Tp: model.TimePorint{
                StartTime: time.Now().Unix(),
                EndTime:   time.Now().Unix() + 10,
            },
        },
        model.LogRecord{
            JobName: "job10",
            Command: "echo 4",
            Err:     "",
            Content: "4",
            Tp: model.TimePorint{
                StartTime: time.Now().Unix(),
                EndTime:   time.Now().Unix() + 10,
            },
        },
    })
    if err != nil{
        log.Fatal(err)
    }
    if result == nil {
        log.Fatal("result nil")
    }
    for _, v := range result.InsertedIDs {
        id = v.(primitive.ObjectID)
        fmt.Println("自增ID", id.Hex())
    }
}
```

运行结果:

![img](https://pic3.zhimg.com/80/v2-49c411a6ddbaceee6ef6e508cf1e7c3e_720w.jpg)

**去数据库中检验**

![img](https://pic4.zhimg.com/80/v2-468f819f78bd8ca78721b267f7c670ef_720w.jpg)

## 查询数据

添加一个查询结构体：

```
//查询实体
type FindByJobName struct {
   JobName string `bson:"jobName"` //任务名
}
```

main代码：

```
package main

import (
    "awesomeProject/mongodb/model"
    "awesomeProject/mongodb/util"
    "context"
    "fmt"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

func main() {
    var (
        client     = util.GetMgoCli()
        err        error
        collection *mongo.Collection
        cursor     *mongo.Cursor
    )
    //2.选择数据库 my_db里的某个表
    collection = client.Database("my_db").Collection("test")

    //如果直接使用 LogRecord{JobName: "job10"}是查不到数据的，因为其他字段有初始值0或者“”
    cond := model.FindByJobName{JobName: "job10"}

    //按照jobName字段进行过滤jobName="job10",翻页参数0-2
    if cursor, err = collection.Find(context.TODO(), cond, options.Find().SetSkip(0), options.Find().SetLimit(2)); err != nil {
        fmt.Println(err)
        return
    }
    //延迟关闭游标
    defer func() {
        if err = cursor.Close(context.TODO()); err != nil {
            log.Fatal(err)
        }
    }()

    //遍历游标获取结果数据
    for cursor.Next(context.TODO()) {
        var lr model.LogRecord
        //反序列化Bson到对象
        if cursor.Decode(&lr) != nil {
            fmt.Print(err)
            return
        }
        //打印结果数据
        fmt.Println(lr)
    }

    //这里的结果遍历可以使用另外一种更方便的方式：
    var results []model.LogRecord
    if err = cursor.All(context.TODO(), &results); err != nil {
        log.Fatal(err)
    }
    for _, result := range results {
        fmt.Println(result)
    }
}
```

执行结果：

![img](https://pic3.zhimg.com/80/v2-76eee87774a34d75f3bad6f8be982c1a_720w.jpg)

## BSON

使用文档前面的方法进行查询显然是很麻烦的，我们不可能每次查询都定义一个新的struct，是否有一种通用的struct来帮助我们作为过滤条件呢，这时候就需要使用到BSON包

MongoDB中的JSON文档存储在名为BSON(二进制编码的JSON)的二进制表示中。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示，使其包含额外的类型，如int、long、date、浮点数和decimal128。这使得应用程序更容易可靠地处理、排序和比较数据。

连接MongoDB的Go驱动程序中有两大类型表示BSON数据：`D`和`Raw`。

类型`D`家族被用来简洁地构建使用本地Go类型的BSON对象。这对于构造传递给MongoDB的命令特别有用。`D`家族包括四类:

- D：一个BSON文档。这种类型应该在顺序重要的情况下使用，比如MongoDB命令。
- M：一张无序的map。它和D是一样的，只是它不保持顺序。
- A：一个BSON数组。
- E：D里面的一个元素。

使用BSON可以更方便的使用Golang完成对数据库的CURD操作

要使用BSON，需要先导入下面的包：

```
import "go.mongodb.org/mongo-driver/bson"
```

下面是一个使用D类型构建的过滤器文档的例子，它可以用来查找name字段与’张三’或’李四’匹配的文档:

```
bson.D{{
   "name",
   bson.D{{
      "$in",
      bson.A{"张三", "李四"},
   }},
}}
```

`Raw`类型用于验证字节切片。你还可以使用`Lookup()`从原始类型检索单个元素。如果你不想要将BSON反序列化成另一种类型的开销，那么这是非常有用的。

那么如何使用这样的方式进行查询呢？代码如下：

```
package main

import (
   "awesomeProject/mongodb/model"
   "awesomeProject/mongodb/util"
   "context"
   "fmt"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "go.mongodb.org/mongo-driver/mongo/options"
   "log"
)

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      cursor     *mongo.Cursor
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")
   filter := bson.M{"jobName":"job10"}
   if cursor, err = collection.Find(context.TODO(), filter,options.Find().SetSkip(0), options.Find().SetLimit(2) ); err != nil {
      log.Fatal(err)
   }
    //延迟关闭游标
    defer func() {
        if err = cursor.Close(context.TODO()); err != nil {
            log.Fatal(err)
        }
    }()

    //这里的结果遍历可以使用另外一种更方便的方式：
    var results []model.LogRecord
    if err = cursor.All(context.TODO(), &results); err != nil {
        log.Fatal(err)
    }
    for _, result := range results {
        fmt.Println(result)
    }
}
```

## 聚合查询

有时候我们需要对数据进行聚合查询，那么就要用到group等聚合方法

```
package main

import (
   "awesomeProject/mongodb/util"
   "context"
   "fmt"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
)

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      cursor     *mongo.Cursor
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")
   //filter := bson.M{"jobName": "job10"}

  //按照jobName分组,countJob中存储每组的数目
   groupStage := mongo.Pipeline{bson.D{
      {"$group", bson.D{
         {"_id", "$jobName"},
         {"countJob", bson.D{
            {"$sum", 1},
         }},
      }},
   }}
   if cursor, err = collection.Aggregate(context.TODO(), groupStage, ); err != nil {
      log.Fatal(err)
   }
   //延迟关闭游标
   defer func() {
      if err = cursor.Close(context.TODO()); err != nil {
         log.Fatal(err)
      }
   }()
   //遍历游标
   var results []bson.M
   if err = cursor.All(context.TODO(), &results); err != nil {
      log.Fatal(err)
   }
   for _, result := range results {
      fmt.Println(result)
   }

}
```

执行结果:

![img](https://pic1.zhimg.com/80/v2-fba2cfcdf79026f30b8803459fa2e15c_720w.jpg)

## 更新数据

同样的，使用mongo-go-driver进行更新也需要建立专门用于更新的实体，在这里我建立的实体中存在Command，和Content两个字段，更新时需要同时对这两个字段进行赋值，否则未被赋值的字段会被更新为golang的数据类型初始值。为更新方便可以采用`bson.M{"$set": bson.M{"command": "ByBsonM",}}`来进行更新。

```
package model
//更新实体
type UpdateByJobName struct {
   Command string     `bson:"command"` //shell命令
   Content string     `bson:"content"` //脚本输出
}
package main

import (
   "awesomeProject/mongodb/model"
   "awesomeProject/mongodb/util"
   "context"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
)

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      uResult    *mongo.UpdateResult
      //upsertedID model.LogRecord
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")
   filter := bson.M{"jobName": "job10"}
   //update := bson.M{"$set": bson.M{"command": "ByBsonM",}}
   update := bson.M{"$set": model.UpdateByJobName{Command: "byModel",Content:"model"}}
   //update := bson.M{"$set": model.LogRecord{JobName:"job10",Command:"byModel"}}
   if uResult, err = collection.UpdateMany(context.TODO(), filter, update); err != nil {
      log.Fatal(err)
   }
    //uResult.MatchedCount表示符合过滤条件的记录数，即更新了多少条数据。
   log.Println(uResult.MatchedCount)
}
```

`bson.M{"$set": model.UpdateByJobName{Command: "byModel",Content:"model"}}`中的$set表示修改字段的值。

使用$inc可以对字段的值进行增减计算，例如`bson.M{"$inc": bson.M{ "age": -1, }}`表示对age的值减一。

使用$push可以对该字段增加一个元素，例如`bson.M{"$push": bson.M{ "interests": "Golang", }}`表示对interests字段的元素数组增加Golang元素。

使用$push可以对该字段删除一个元素，例如`bson.M{"$pull": bson.M{ "interests": "Golang", }}`表示对interests字段的元素数组删除Golang元素。

## 删除数据

```
package main

import (
   "awesomeProject/mongodb/util"
   "context"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
)

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      uResult    *mongo.DeleteResult
      //upsertedID model.LogRecord
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")
   filter := bson.M{"jobName": "job0"}
   //3.删除开始时间早于当前时间的数据
   //
   if uResult, err = collection.DeleteMany(context.TODO(), filter); err != nil {
      log.Fatal(err)
   }
   log.Println(uResult.DeletedCount)
}
```

运行结果：

![img](https://pic3.zhimg.com/80/v2-15181df6cc792d653b7e87b793a900a2_720w.jpg)

带过滤条件删除：

```
package main

import (
   "awesomeProject/mongodb/util"
   "context"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
   "time"
)

type DeleteCond struct {
   BeforeCond TimeBeforeCond `bson:"tp.startTime"`
}

//startTime小于某时间，使用这种方式可以对想要进行的操作($set、$group等)提前定义
type TimeBeforeCond struct {
   BeforeTime int64 `bson:"$lt"`
}

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      uResult    *mongo.DeleteResult
      delCond    *DeleteCond
      //upsertedID model.LogRecord
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")

   //3.删除jobName为job0的数据
   delCond = &DeleteCond{BeforeCond: TimeBeforeCond{BeforeTime: time.Now().Unix()}}
   if uResult, err = collection.DeleteMany(context.TODO(), delCond); err != nil {
      log.Fatal(err)
   }
   log.Println(uResult.DeletedCount)
}
```

## 最终使用的过滤方式

在经过以上的学习，发现使用struct来进行过滤还是比较方便的，但是如何能忽略被初始化的值呢?经过多次尝试，发现可以通过以下方式完成过滤：

```
package model

type TimePorintFilter struct {
    StartTime interface{} `bson:"tp.startTime,omitempty"` //开始时间
    EndTime   interface{} `bson:"tp.endTime,omitempty"`   //结束时间
}

type LogRecordFilter struct {
    ID      interface{} `bson:"_id,omitempty"`
    JobName interface{} `bson:"jobName,omitempty" json:"jobName"` //任务名
    Command interface{} `bson:"command,omitempty" `               //shell命令
    Err     interface{} `bson:"err,omitempty"`                    //脚本错误
    Content interface{} `bson:"content,omitempty"`                //脚本输出
    Tp      interface{} `bson:"tp,omitempty"`                     //执行时间
}

//小于
type Lt struct {
    Lt int64 `bson:"$lt"` //小于
}

//分组
type Group struct {
    Group interface{} `bson:"$group"` //小于
}

//求和
type Sum struct {
    Sum interface{} `bson:"$sum"` //小于
}
package main

import (
   "awesomeProject/mongodb/util"
   "context"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
   "time"
)

type DeleteCond struct {
   BeforeCond TimeBeforeCond `bson:"tp.startTime"`
}

//startTime小于某时间
type TimeBeforeCond struct {
   BeforeTime int64 `bson:"$lt"`
}

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      uResult    *mongo.DeleteResult
      delCond    *DeleteCond
      //upsertedID model.LogRecord
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")

   //3.删除jobName为job0的数据
   //delCond = &model.LogRecordFilter{Tp: model.TimePorintFilter{StartTime: model.Lt{Lt: time.Now().Unix()}}}
   delCond = &DeleteCond{BeforeCond: TimeBeforeCond{BeforeTime: time.Now().Unix()}}
   if uResult, err = collection.DeleteMany(context.TODO(), delCond); err != nil {
      log.Fatal(err)
   }
   log.Println(uResult.DeletedCount)
}
package main

import (
   "awesomeProject/mongodb/model"
   "awesomeProject/mongodb/util"
   "context"
   "fmt"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "log"
)

func main() {
   var (
      client     = util.GetMgoCli()
      collection *mongo.Collection
      err        error
      cursor     *mongo.Cursor
   )
   //2.选择数据库 my_db里的某个表
   collection = client.Database("my_db").Collection("test")

   // 这一段分组求和其实可以使用某个方法做个封装
   groupStage := []model.Group{}
   groupStage = append(groupStage, model.Group{
      Group: bson.D{
         {"_id", "$jobName"},
         {"countJob", model.Sum{Sum: 1}},
      },
   })

   if cursor, err = collection.Aggregate(context.TODO(), groupStage, ); err != nil {
      log.Fatal(err)
   }
   //延迟关闭游标
   defer func() {
      if err = cursor.Close(context.TODO()); err != nil {
         log.Fatal(err)
      }
   }()
   //遍历游标
   var results []bson.M
   if err = cursor.All(context.TODO(), &results); err != nil {
      log.Fatal(err)
   }
   for _, result := range results {
      fmt.Println(result)
   }
}
```