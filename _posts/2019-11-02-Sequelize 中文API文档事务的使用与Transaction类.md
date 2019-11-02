---
layout:     post
title:      Sequelize 中文API文档事务的使用与Transaction类
date:       2019-11-2
author:    	边黎安
header-img: img/tag-bg.jpg
catalog: true
tags:
    - Sequelize
    - ORM
    - egg
---


##Transaction是Sequelize中用于实现事务功能的子类，通过调用Sequelize.transaction()方法可以创建一个该类的实例。在Sequelize中，支持自动提交/回滚，也可以支持用户手动提交/回滚。

###1. 事务的使用
Sequelize有两种使用事务的方式：

基于Promise结果链的自动提交/回滚
另一种是不自动提交和回滚，而由用户控制事务


####1.1 受管理的事务（auto-callback）
受管理的事务会自动提交或回滚，你可以向sequelize.transaction方法传递一个回调函数来启动一个事务。

需要注意，在这种方式下传递给回调函数的transaction会返回一个promise链，在promise链中（then或catch）中并不能调用t.commit()或t.rollback()来控制事务。在这种方式下，如果使用事务的所有promise链都执行成功，则自动提交；如果其中之一执行失败，则自动回滚。
```
return sequelize.transaction(function (t) {

  // 要确保所有的查询链都有return返回
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(function (user) {
    return user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, {transaction: t});
  });

}).then(function (result) {
  // Transaction 会自动提交
  // result 是事务回调中使用promise链中执行结果
}).catch(function (err) {
  // Transaction 会自动回滚
  // err 是事务回调中使用promise链中的异常结果
});
```
抛出错误并回滚

使用受管理的事务时，不能通过手工调用的方式来提交或回滚事务。但在需要时（如验证失败），可以通过throw来抛出异常回滚事务。
```
return sequelize.transaction(function (t) {
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(function (user) {
    // 注意，虽然所有操作成功但仍会回滚
    throw new Error();
  });
});
```
自动传递事务到所有的查询中

在上面的示例中，我们通过向第二个参数中添加{ transaction: t }选项来手工传递事务。如果要自动传递事务到所有的查询中，需要安装continuation local storage（CLS）模块并在代码中创建一个命名空间实例：
```
var cls = require('continuation-local-storage'),
    namespace = cls.createNamespace('my-very-own-namespace');
```
启用CLS，需要在Sequlize构造函数属性中设置命名空间：
```
var Sequelize = require('sequelize');
Sequelize.cls = namespace;

new Sequelize(....);
```
注意cls必须在constructor构造函数中设置，而不能在sequlize实例中设置。

CLS的工作方式就像一个回调函数的本地线程存储。在sequlize中启用CLS后，需要在启动事务时设置transaction命名空间。在一个回调链中设置的变量时私有的，所以几个并发事务可以同时存在。
```
sequelize.transaction(function (t1) {
  namespace.get('transaction') === t1; // true
});

sequelize.transaction(function (t2) {
  namespace.get('transaction') === t2; // true
});
大多数情况下，你不用通过namespace.get('transaction')直接访问命名空间，因为所有的查询都会自动查找事务的命名空间。

sequelize.transaction(function (t1) {
  // 启用 CLS 后，会在自动在事务中执行create 操作
  return User.create({ name: 'Alice' });
});
```

####1.2 不受管理的事务（then-callback）
不受管理的事务需要你强制提交或回滚，如果不进行这些操作，事务会一直保持挂起状态直到超时。

启动一个不受管理的事务，同样是调用sequelize.transaction()方法，但不传递回调函数参数（仍然可以传递选项参数）。然后可以在其返回的promisethen方法中手工控制事务：
```
return sequelize.transaction().then(function (t) {
  return User.create({
    firstName: 'Homer',
    lastName: 'Simpson'
  }, {transaction: t}).then(function (user) {
    return user.addSibling({
      firstName: 'Lisa',
      lastName: 'Simpson'
    }, {transaction: t});
  }).then(function () {
    return t.commit();
  }).catch(function (err) {
    return t.rollback();
  });
});

```
####1.3 并行/部分事务
在一系列的查询中可以有多个并行的事务或者其中的一些查询不使用事务。通过{transaction: }选项来指定查询属于哪个事务：
```
sequelize.transaction(function (t1) {
  return sequelize.transaction(function (t2) {
    // 启用 CLS 时, 查询会自动使用 t2
    // 通过 `transaction` 选项可以改变查询所属的事务
    return Promise.all([
        User.create({ name: 'Bob' }, { transaction: null }),
        User.create({ name: 'Mallory' }, { transaction: t1 }),
        User.create({ name: 'John' }) // 默认使用 t2
    ]);
  });
});
```

####1.4 隔离级别
在启动事务时，可以设置事务的隔离级别。可选值有：
```
Sequelize.Transaction.ISOLATION_LEVELS.READ_UNCOMMITTED // "READ UNCOMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED // "READ COMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.REPEATABLE_READ  // "REPEATABLE READ"
Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE // "SERIALIZABLE"
默认的隔离级别为REPEATABLE READ，如果需要修改可以在启动事务时在第一个参数中设置：

return sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
  }, function (t) {

  // your transactions

});
```

####1.5 选项参数
调用transaction方法时，可以向其第一个参数中传递一个选项参数，通过该参数可以对事务进行一些配置：
```
return sequelize.transaction({ /* options */ });
```
默认配置选项如下：
```
{
  autocommit: true,
  isolationLevel: 'REPEATABLE_READ',
  deferrable: 'NOT DEFERRABLE' // implicit default of postgres
}
```
isolationLevel选项可以在初始化Sequelize全局设置，也可以启动每个事务时局部设置：
// 全局设置
```
new Sequelize('db', 'user', 'pw', {
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```
// 局部设置
```
sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
deferrable选项会在事务执行前或执行后启动一个额外的查询，但需要注意此选项仅适用于PostgreSQL。

sequelize.transaction({
  // to defer all constraints:
  deferrable: Sequelize.Deferrable.SET_DEFERRED,

  // to defer a specific constraint:
  deferrable: Sequelize.Deferrable.SET_DEFERRED(['some_constraint']),

  // to not defer constraints:
  deferrable: Sequelize.Deferrable.SET_IMMEDIATE
})
```

##2. Transaction类
2.1 访问与初始化
Transaction对象用于标识一个要执行的事务，可以通过以下方式创建该对象的实例：

sequelize.transaction(options, autoCallback);    // sequelize 是一个Sequelize对象实例
在创建事务实例，并在事务中执行查询时，需要传入一个可选参数对象。各参数值如下：

```
名称  类型  说明
sequelize   Sequelize   一个已配置的sequelize实例
options Object  选项对象
options.autocommit=true Boolean 设置事务的autocommit（自动完成）属性
options.type=true   String  设置事务类型，详见TYPES
options.isolationLevel=true String  设置事务的隔离级别，详见ISOLATION_LEVELS
options.deferrable  String  设置立即或延迟检查约束
TYPES
```
此选项仅Sqlite适用，用于设置事务类型，默认为DEFERRED，也可以在new Sequelize()时由options.transactionType选项指定。可选值如下：
```
{
  DEFERRED: "DEFERRED",
  IMMEDIATE: "IMMEDIATE",
  EXCLUSIVE: "EXCLUSIVE"
}
我可以像下面这样设置事务类型：

return sequelize.transaction({
  type: Sequelize.Transaction.EXCLUSIVE
}, function (t) {

 // 事务

}).then(function(result) {
  // 事务已完成（commit）
}).catch(function(err) {
  // 出现异常
});
ISOLATION_LEVELS
```

通过options.isolationLevel选项设置sequelize.transaction启动事务时的隔离级别。默认为REPEATABLE_READ，也可以在new Sequelize()时由options.isolationLevel选项指定。可选值如下：

```
{
  READ_UNCOMMITTED: "READ UNCOMMITTED",
  READ_COMMITTED: "READ COMMITTED",
  REPEATABLE_READ: "REPEATABLE READ",
  SERIALIZABLE: "SERIALIZABLE"
}
如，可以像下面这样设置事务的隔离级别：

return sequelize.transaction({
  isolationLevel: Sequelize.Transaction.SERIALIZABLE
}, function (t) {

 // 事务

}).then(function(result) {
  // 事务已完成（commit）
}).catch(function(err) {
  // 出现异常
});
```

LOCK

在调用find等方法时，可以添加一个lock选择用于锁定正在使用的数据行：
```

t1 // 一个事务
t1.LOCK.UPDATE,
t1.LOCK.SHARE,
t1.LOCK.KEY_SHARE, // Postgres 9.3+ only
t1.LOCK.NO_KEY_UPDATE // Postgres 9.3+ only
使用

t1 // 一个事务
Model.findAll({
  where: ...,
  transaction: t1,
  lock: t1.LOCK...
});
Postgres还支持通过of选项实现预锁定：

UserModel.findAll({
  where: ...,
  include: [TaskModel, ...],
  transaction: t1,
  lock: {
    level: t1.LOCK...,
    of: UserModel
  }
});
```


##2.2 实例方法
通过sequelize.transaction()启动事务后，在实例中包含两个方法commit()和rollback()分别用于完成事务和事务回滚。

commit()完成事务

该方法用于手动完成事务。

如，我们可以像下面这样完成一个事务：
```

sequelize.transaction(function (t) {
  // 事务
  Model.findAll().then(function(result){
    // 手动提交/完成事务
    t.commit();
  });
}).then(function(result) {
  // 事务已完成（commit）
}).catch(function(err) {
  // 出现异常
});
rollback()回滚事务

```

该方法用于手动回滚事务。
```

sequelize.transaction(function (t) {
  // 事务
  Model.findAll().then(function(result){
    // 手动回滚事务
    t.rollback();
  });
}).then(function(result) {
  // 事务已完成（commit）
}).catch(function(err) {
  // 出现异常
});
```

