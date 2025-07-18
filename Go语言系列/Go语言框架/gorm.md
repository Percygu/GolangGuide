---
tags:
  - Go
  - golang
  - gorm
  - mysql
  - 数据库
---

# GORM

GORM是一个功能强大的Go语言ORM库，提供了简洁的API来操作数据库。本文将详细介绍如何使用GORM与MySQL数据库进行交互。

## 安装GORM和MySQL驱动

首先，我们需要安装GORM和MySQL驱动。可以使用以下命令：

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

## 连接数据库

在使用GORM连接MySQL数据库时，我们需要提供数据源名称（DSN），其中包含用户名、密码、主机地址、端口号和数据库名称等信息。

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    // 定义数据源名称
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    
    // 连接数据库
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatalf("failed to connect database: %v", err)
    }
}
```

## 初始化连接

GORM会自动管理数据库连接池，因此我们不需要手动设置最大连接数和最大空闲连接数。不过，如果有特殊需求，可以通过`sql.DB`对象进行设置。

```go
sqlDB, err := db.DB()
if err != nil {
    log.Fatalf("failed to get database object: %v", err)
}

// 设置最大打开连接数
sqlDB.SetMaxOpenConns(100)

// 设置最大空闲连接数
sqlDB.SetMaxIdleConns(10)
```

## 定义模型

在GORM中，我们通过定义结构体来映射数据库表。每个字段对应表中的一列。

```go
// User 结构体映射到数据库中的users表
type User struct {
    ID   uint   `gorm:"primaryKey;column:id"` // ID为主键
    Name string `gorm:"size:255;column:name"` // Name字段最大长度为255
    Age  int    `gorm:"column:age"` // Age字段
}
```

## 自动迁移

GORM提供了自动迁移功能，可以根据模型结构自动创建或更新数据库表。

```go
// 自动迁移User结构体
db.AutoMigrate(&User{})
```

## CRUD操作

### 创建记录

使用`Create`方法可以向数据库中插入一条新记录。

```go
// 创建一个新的User记录
user := User{Name: "张三", Age: 25}
result := db.Create(&user)
if result.Error != nil {
    log.Fatalf("failed to create user: %v", result.Error)
}
```

### 查询记录

#### 单行查询

使用`First`方法可以查询符合条件的第一条记录。

```go
// 查询ID为1的User记录
var user User
result := db.First(&user, 1) // 根据主键查询
if result.Error != nil {
    log.Printf("failed to find user: %v", result.Error)
}
```

#### 多行查询

使用`Find`方法可以查询符合条件的多条记录。

```go
// 查询年龄大于20的所有User记录
var users []User
result := db.Where("age > ?", 20).Find(&users)
if result.Error != nil {
    log.Printf("failed to find users: %v", result.Error)
}
```

### 更新记录

使用`Save`方法可以更新记录。

```go
// 更新User记录的Age字段
user.Age = 26
result := db.Save(&user)
if result.Error != nil {
    log.Fatalf("failed to update user: %v", result.Error)
}
```

### 删除记录

使用`Delete`方法可以删除记录。

```go
// 删除User记录
result := db.Delete(&user)
if result.Error != nil {
    log.Fatalf("failed to delete user: %v", result.Error)
}
```

## 事务支持

GORM支持事务操作，可以通过`Transaction`方法来执行事务。

```go
// 使用事务创建两个User记录
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&User{Name: "李四", Age: 30}).Error; err != nil {
        return err
    }
    if err := tx.Create(&User{Name: "王五", Age: 35}).Error; err != nil {
        return err
    }
    return nil
})
if err != nil {
    log.Fatalf("transaction failed: %v", err)
}
```

## 预处理和SQL注入

GORM会自动处理SQL注入问题，因此我们不需要手动拼接SQL语句。所有的查询条件都可以通过链式调用来实现。

```go
// 查询Name为"张三"的User记录
db.Where("name = ?", "张三").First(&user)
```

## 完整示例
以下是一个完整的示例代码，展示了如何使用GORM连接MySQL数据库并进行CRUD操作。
```go
package main

import (
    "fmt"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

// User 结构体映射到数据库中的users表
type User struct {
    ID   uint   `gorm:"primaryKey;column:id"` // ID为主键
    Name string `gorm:"size:255;column:name"` // Name字段最大长度为255
    Age  int    `gorm:"column:age"`           // Age字段
}

func main() {
    // 定义数据源名称
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    
    // 连接数据库
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatalf("failed to connect database: %v", err)
    }

    // 自动迁移User结构体
    db.AutoMigrate(&User{})

    // 创建一个新的User记录
    user := User{Name: "张三", Age: 25}
    result := db.Create(&user)
    if result.Error != nil {
        log.Fatalf("failed to create user: %v", result.Error)
    }

    // 查询ID为1的User记录
    var queriedUser User
    result = db.First(&queriedUser, user.ID) // 根据主键查询
    if result.Error != nil {
        log.Printf("failed to find user: %v", result.Error)
    } else {
        fmt.Printf("Queried User: %+v\n", queriedUser)
    }

    // 更新User记录的Age字段
    queriedUser.Age = 26
    result = db.Save(&queriedUser)
    if result.Error != nil {
        log.Fatalf("failed to update user: %v", result.Error)
    }

    // 删除User记录
    result = db.Delete(&queriedUser)
    if result.Error != nil {
        log.Fatalf("failed to delete user: %v", result.Error)
    }
}
```

### 程序输出

假设数据库连接成功，程序将输出如下信息：

```
Queried User: {ID:1 Name:张三 Age:25}
```

这表示程序成功创建、查询、更新并删除了一条用户记录。

### 代码说明

1. **连接数据库**：使用`gorm.Open`方法连接MySQL数据库，数据源名称（DSN）中包含了数据库连接的必要信息。

2. **自动迁移**：使用`AutoMigrate`方法根据`User`结构体自动创建或更新数据库表。

3. **创建记录**：使用`Create`方法向数据库中插入一条新记录。

4. **查询记录**：使用`First`方法查询符合条件的第一条记录。

5. **更新记录**：使用`Save`方法更新记录。

6. **删除记录**：使用`Delete`方法删除记录。

---

## 小结
这篇文章简单介绍了GORM的基本使用方法，包括连接数据库、定义模型、CRUD操作、事务支持等内容。希望对您有所帮助！，如需了解GORM详细的用法，可以参考：[GORM中文官网](https://gorm.io/zh_CN/docs/) 