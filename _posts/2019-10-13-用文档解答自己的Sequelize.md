---
layout:     post
title:      用文档解答自己的Sequelize
date:       2019-10-13
author:    	边黎安
header-img: img/tag-bg.jpg
catalog: true
tags:
    - Sequelize
    - ORM
    - egg
---


## 什么是ORM？
```javascript
简单的讲就是对SQL查询语句的封装，让我们可以用OOP的方式操作数据库，优雅的生成安全、可维护的SQL代码。直观上，是一种Model和SQL的映射关系。  

(Object Oriented Programming) 

```
#### 那么什么是Sequelize？
```javascript

Sequelize是一款基于Nodejs功能强大的异步ORM框架。
同时支持PostgreSQL, MySQL, SQLite and MSSQL多种数据库，很适合作为Nodejs后端数据库的存储接口，为快速开发Nodejs应用奠定扎实、安全的基础。
既然Nodejs的强项在于异步，没有理由不找一个强大的支持异步的数据库框架，与之配合


```

#### Sequelize安装
squelize可以通过npm命令获取，除安装sequelize模块外还要安装所使用数据的驱动模块：
```javascript

$ npm install --save sequelize

# 还需要安装以下之一：
$ npm install --save pg pg-hstore  // postgreSql
$ npm install --save mysql // mysql 或 mariadb
$ npm install --save sqlite3  
$ npm install --save tedious // MSSQL

```


#### Sequelize建立连接
---
## Sequelize会在初始化时设置一个连接池，这样你应该为每个数据库创建一个实例：


```javascript

var sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'mariadb'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    idle: 10000
  },

  // 仅 SQLite 适用
  storage: 'path/to/database.sqlite'
});

// 或者可以简单的使用一个连接 uri
var sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

```
![](https://sequelize.org/master/manual/asset/logo-small.png)

##这一期中 边边 带大家 理解的是 Sequelize中 较为常用的 几个模块 
Transactions Associations Attributes Migrations 
Model definition  Model usage Datatypes



#### Quick example

```javascript
const { Sequelize, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class User extends Model {}
User.init({
  username: DataTypes.STRING,
  birthday: DataTypes.DATE
}, { sequelize, modelName: 'user' });

sequelize.sync()
  .then(() => User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  }))
  .then(jane => {
    console.log(jane.toJSON());
  });

```

##Datatypes

```
Sequelize.STRING                      // VARCHAR(255)  可变字符串  可变长度，可以设置最大长度；适合用在长度可变的属性

Sequelize.STRING(1234)                // VARCHAR(1234)   指定长度的意思
Sequelize.STRING.BINARY               // VARCHAR BINARY  //很少用
Sequelize.TEXT                        // TEXT   当不知道属性的最大长度时，适合用text。
Sequelize.TEXT('tiny')                // TINYTEXT
Sequelize.CITEXT                      // CITEXT      PostgreSQL and SQLite only.

Sequelize.INTEGER                     // INTEGER int -2 的31 次方 - 2 的31 次方 -1 
Sequelize.BIGINT                      // BIGINT  
Sequelize.BIGINT(11)                  // BIGINT(11)  数字的长度 限定11 位

Sequelize.FLOAT                       // FLOAT 整数或者小数加在一起 从左到右只留6位后面整数的话0替代
Sequelize.FLOAT(11)                   // FLOAT 整数或者小数加在一起 从左到右只留11位后面整数的话0替代  
Sequelize.FLOAT(11, 10)               // FLOAT(11,10)  11 表示 整数位数  小数一共10位

Sequelize.REAL                        // REAL        PostgreSQL only.
Sequelize.REAL(11)                    // REAL(11)    PostgreSQL only.
Sequelize.REAL(11, 12)                // REAL(11,12) PostgreSQL only.

Sequelize.DOUBLE                      // DOUBLE 
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 10)              // DOUBLE(11,10)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2) //decimal(10,2)中的“2”表示小数部分的位数，如果插入的值未指定小数部分或者小数部分不足两位则会自动补到2位小数，若插入的值小数部分超过了2为则会发生截断，截取前2位小数。

//“10”指的是整数部分加小数部分的总长度，也即插入的数字整数部分不能超过“10-2”位，否则不能成功插入，会报超出范围的错误。

Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
Sequelize.DATE(6)                     // DATETIME(6) for mysql 5.6.4+. Fractional seconds support with up to 6 digits of precision 6指的就是精度
Sequelize.DATEONLY                    // DATE without time.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // An ENUM with allowed values 'value 1' and 'value 2'
Sequelize.ARRAY(Sequelize.TEXT)       // Defines an array. PostgreSQL only.
Sequelize.ARRAY(Sequelize.ENUM)       // Defines an array of ENUM. PostgreSQL only.

Sequelize.JSON                        // JSON column. PostgreSQL, SQLite and MySQL only.
Sequelize.JSONB                       // JSONB column. PostgreSQL only.

Sequelize.BLOB                        // BLOB (bytea for PostgreSQL)
Sequelize.BLOB('tiny')                // TINYBLOB (bytea for PostgreSQL. Other options are medium and long)

Sequelize.UUID                        // UUID datatype for PostgreSQL and SQLite, CHAR(36) BINARY for MySQL (use defaultValue: Sequelize.UUIDV1 or Sequelize.UUIDV4 to make sequelize generate the ids automatically)

Sequelize.CIDR                        // CIDR datatype for PostgreSQL
Sequelize.INET                        // INET datatype for PostgreSQL
Sequelize.MACADDR                     // MACADDR datatype for PostgreSQL

Sequelize.RANGE(Sequelize.INTEGER)    // Defines int4range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.BIGINT)     // Defined int8range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATE)       // Defines tstzrange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATEONLY)   // Defines daterange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DECIMAL)    // Defines numrange range. PostgreSQL only.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // Defines array of tstzrange ranges. PostgreSQL only.

Sequelize.GEOMETRY                    // Spatial column.  PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT')           // Spatial column with geometry type. PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT', 4326)     // Spatial column with geometry type and SRID.  PostgreSQL (with PostGIS) or MySQL only.
```



### Model usage




##find
```
按照主键
Project.findByPk(123).then(project => {
  // project will be an instance of Project and stores the content of the table entry
  // with id 123. if such an entry is not defined you will get null
})

自定义查找
// search for attributes
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project will be the first entry of the Projects table with the title 'aProject' || null
})

自定义查找 和输出结构
Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(project => {
  // project will be the first entry of the Projects table with the title 'aProject' || null
  // project.get('title') will contain the name of the project
})
```


## findOrCreate
```
The method findOrCreate can be used to check if a certain element already exists in the database. If that is the case the method will result in a respective instance. If the element does not yet exist, it will be created.

Let's assume we have an empty database with a User model which has a username and a job.

where option will be appended to defaults for create case.


User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  .then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
     findOrCreate returns an array containing the object that was found or created and a boolean that
     will be true if a new object was created and false if not, like so:

    [ {
        username: 'sdepold',
        job: 'Technical Lead JavaScript',
        id: 1,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      true ]

 In the example above, the array spread on line 3 divides the array into its 2 parts and passes them
  as arguments to the callback function defined beginning at line 39, which treats them as "user" and
  "created" in this case. (So "user" will be the object from index 0 of the returned array and
  "created" will equal "true".)
    */
  })

  ```
The code created a new instance. So when we already have an instance ...
  
```javascript
User.create({ username: 'fnord', job: 'omnomnom' })
  .then(() => User.findOrCreate({where: {username: 'fnord'}, defaults: {job: 'something else'}}))
  .then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    In this example, findOrCreate returns an array like this:
    [ {
        username: 'fnord',
        job: 'omnomnom',
        id: 2,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      false
    ]
    The array returned by findOrCreate gets spread into its 2 parts by the array spread on line 3, and
    the parts will be passed as 2 arguments to the callback function beginning on line 69, which will
    then treat them as "user" and "created" in this case. (So "user" will be the object from index 0
    of the returned array and "created" will equal "false".)
    */
  })

```



##findAndCountAll 

```javascript
count - an integer, total number records matching the where clause and other filters due to associations
rows - an array of objects, the records matching the where clause and other filters due to associations, within the limit and offset range
Project
  .findAndCountAll({
     where: {
        title: {
          [Op.like]: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
  .then(result => {
    console.log(result.count);
    console.log(result.rows);
  });
It support includes. Only the includes that are marked as required will be added to the count part:

Suppose you want to find all users who have a profile attached:

User.findAndCountAll({
  include: [
     { model: Profile, required: true }
  ],
  limit: 3
});
Because the include for Profile has required set it will result in an inner join, and only the users who have a profile will be counted. If we remove required from the include, both users with and without profiles will be counted. Adding a where clause to the include automatically makes it required:

User.findAndCountAll({
  include: [
     { model: Profile, where: { active: true }}
  ],
  limit: 3
});

```

##findAll



### Model usage
```
class Pub extends Model {}
Pub.init({
  name: { type: Sequelize.STRING },
  address: { type: Sequelize.STRING },
  latitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -90, max: 90 }
  },
  longitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -180, max: 180 }
  },
}, {
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Require either both latitude and longitude or neither')
      }
    }
  },
  sequelize,
})



```

常用的 validate

```
 validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters

}
```

###重点要讲的是Transactions。和 lock

分析代码
```
return sequelize.transaction(t => {

  // chain all your queries here. make sure you return them.
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(user => {
    return user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, {transaction: t});
  });

}).then(result => {
  // Transaction has been committed
  // result is whatever the result of the promise chain returned to the transaction callback
}).catch(err => {
  // Transaction has been rolled back
  // err is whatever rejected the promise chain returned to the transaction callback
});

```

Throw errors to rollback
When using the managed transaction you should never commit or rollback the transaction manually. If all queries are successful, but you still want to rollback the transaction (for example because of a validation failure) you should throw an error to break and reject the chain:
```
return sequelize.transaction(t => {
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(user => {
    // Woops, the query was successful but we still want to roll back!
    throw new Error();
  });
});
```


Automatically pass transactions to all queries
In the examples above, the transaction is still manually passed, by passing { transaction: t } as the second argument. To automatically pass the transaction to all queries you must install the continuation local storage (CLS) module and instantiate a namespace in your own code:
```
const cls = require('continuation-local-storage');
const namespace = cls.createNamespace('my-very-own-namespace');

```
To enable CLS you must tell sequelize which namespace to use by using a static method of the sequelize constructor:
```

const Sequelize = require('sequelize');
Sequelize.useCLS(namespace);

new Sequelize(....);

```
Notice, that the useCLS() method is on the constructor, not on an instance of sequelize. This means that all instances will share the same namespace, and that CLS is all-or-nothing - you cannot enable it only for some instances.

CLS works like a thread-local storage for callbacks. What this means in practice is that different callback chains can access local variables by using the CLS namespace. When CLS is enabled sequelize will set the transaction property on the namespace when a new transaction is created. Since variables set within a callback chain are private to that chain several concurrent transactions can exist at the same time:
```
sequelize.transaction((t1) => {
  namespace.get('transaction') === t1; // true
});

sequelize.transaction((t2) => {
  namespace.get('transaction') === t2; // true
});

```
In most case you won't need to access namespace.get('transaction') directly, since all queries will automatically look for a transaction on the namespace:
```
sequelize.transaction((t1) => {
  // With CLS enabled, the user will be created inside the transaction
  return User.create({ name: 'Alice' });
});

```

Concurrent/Partial transactions
You can have concurrent transactions within a sequence of queries or have some of them excluded from any transactions. Use the {transaction: } option to control which transaction a query belong to:

Warning: SQLite does not support more than one transaction at the same time.

Without CLS enabled

```
sequelize.transaction((t1) => {
  return sequelize.transaction((t2) => {
    // With CLS enable, queries here will by default use t2
    // Pass in the `transaction` option to define/alter the transaction they belong to.
    return Promise.all([
        User.create({ name: 'Bob' }, { transaction: null }),
        User.create({ name: 'Mallory' }, { transaction: t1 }),
        User.create({ name: 'John' }) // this would default to t2
    ]);
  });
});

```
Isolation levels
The possible isolations levels to use when starting a transaction:
```
Sequelize.Transaction.ISOLATION_LEVELS.READ_UNCOMMITTED // "READ UNCOMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED // "READ COMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.REPEATABLE_READ  // "REPEATABLE READ"
Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE // "SERIALIZABLE"
```
By default, sequelize uses the isolation level of the database. If you want to use a different isolation level, pass in the desired level as the first argument:
```
return sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
  }, (t) => {

  // your transactions

  });

  ```
The isolationLevel can either be set globally when initializing the Sequelize instance or locally for every transaction:

// globally
```
new Sequelize('db', 'user', 'pw', {
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```
// locally
```
sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```
Note: The SET ISOLATION LEVEL queries are not logged in case of MSSQL as the specified isolationLevel is passed directly to tedious

Unmanaged transaction (then-callback)
Unmanaged transactions force you to manually rollback or commit the transaction. If you don't do that, the transaction will hang until it times out. To start an unmanaged transaction, call sequelize.transaction() without a callback (you can still pass an options object) and call then on the returned promise. Notice that commit() and rollback() returns a promise.
```
return sequelize.transaction().then(t => {
  return User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, {transaction: t}).then(user => {
    return user.addSibling({
      firstName: 'Lisa',
      lastName: 'Simpson'
    }, {transaction: t});
  }).then(() => {
    return t.commit();
  }).catch((err) => {
    return t.rollback();
  });
});
```
Usage with other sequelize methods
The transaction option goes with most other options, which are usually the first argument of a method. For methods that take values, like .create, .update(), etc. transaction should be passed to the option in the second argument. If unsure, refer to the API documentation for the method you are using to be sure of the signature.

After commit hook
A transaction object allows tracking if and when it is committed.

An afterCommit hook can be added to both managed and unmanaged transaction objects:
```
sequelize.transaction(t => {
  t.afterCommit((transaction) => {
    // Your logic
  });
});

sequelize.transaction().then(t => {
  t.afterCommit((transaction) => {
    // Your logic
  });

  return t.commit();
})

```
The function passed to afterCommit can optionally return a promise that will resolve before the promise chain that created the transaction resolves

afterCommit hooks are not raised if a transaction is rolled back

afterCommit hooks do not modify the return value of the transaction, unlike standard hooks

You can use the afterCommit hook in conjunction with model hooks to know when a instance is saved and available outside of a transaction
```
model.afterSave((instance, options) => {
  if (options.transaction) {
    // Save done within a transaction, wait until transaction is committed to
    // notify listeners the instance has been saved
    options.transaction.afterCommit(() => /* Notify */)
    return;
  }
  // Save done outside a transaction, safe for callers to fetch the updated model
  // Notify
})
```
Locks
Queries within a transaction can be performed with locks
```
return User.findAll({
  limit: 1,
  lock: true,
  transaction: t1
})

```
Queries within a transaction can skip locked rows
```
return User.findAll({
  limit: 1,
  lock: true,
  skipLocked: true,
  transaction: t2
})
```


###Associations

BelongsTo
HasOne
HasMany
BelongsToMany

##BelongsTo
```

class Player extends Model {}
Player.init({/* attributes */}, { sequelize, modelName: 'player' });
class Team extends Model {}
Team.init({/* attributes */}, { sequelize, modelName: 'team' });

Player.belongsTo(Team); // Will add a teamId attribute to Player to hold the primary key value for Team


```


## HasOne

```
class User extends Model {}
User.init({/* ... */}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({/* ... */}, { sequelize, modelName: 'project' })

// One-way associations
Project.hasOne(User)

/*
  In this example hasOne will add an attribute projectId to the User model!
  Furthermore, Project.prototype will gain the methods getUser and setUser according
  to the first parameter passed to define. If you have underscore style
  enabled, the added attribute will be project_id instead of projectId.

  The foreign key will be placed on the users table.

  You can also define the foreign key, e.g. if you already have an existing
  database and want to work on it:
*/

Project.hasOne(User, { foreignKey: 'initiator_id' })

/*
  Because Sequelize will use the model's name (first parameter of define) for
  the accessor methods, it is also possible to pass a special option to hasOne:
*/

Project.hasOne(User, { as: 'Initiator' })
// Now you will get Project.getInitiator and Project.setInitiator

// Or let's define some self references
class Person extends Model {}
Person.init({ /* ... */}, { sequelize, modelName: 'person' })

Person.hasOne(Person, {as: 'Father'})
// this will add the attribute FatherId to Person

// also possible:
Person.hasOne(Person, {as: 'Father', foreignKey: 'DadId'})
// this will add the attribute DadId to Person

// In both cases you will be able to do:
Person.setFather
Person.getFather

// If you need to join a table twice you can double join the same table
Team.hasOne(Game, {as: 'HomeTeam', foreignKey : 'homeTeamId'});
Team.hasOne(Game, {as: 'AwayTeam', foreignKey : 'awayTeamId'});

Game.belongsTo(Team);


```


##hasMany


```
class User extends Model {}
User.init({/* ... */}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({/* ... */}, { sequelize, modelName: 'project' })

```
// OK. Now things get more complicated (not really visible to the user :)).
// First let's define a hasMany association
Project.hasMany(User, {as: 'Workers'})
This will add the attribute projectId to User. Depending on your setting for underscored the column in the table will either be called projectId or project_id. Instances of Project will get the accessors getWorkers and setWorkers.

Sometimes you may need to associate records on different columns, you may use sourceKey option:

```
class City extends Model {}
City.init({ countryCode: Sequelize.STRING }, { sequelize, modelName: 'city' });
class Country extends Model {}
Country.init({ isoCode: Sequelize.STRING }, { sequelize, modelName: 'country' });

// Here we can connect countries and cities base on country code
Country.hasMany(City, {foreignKey: 'countryCode', sourceKey: 'isoCode'});
City.belongsTo(Country, {foreignKey: 'countryCode', targetKey: 'isoCode'});



```

##Belongs-To-Many associations

Belongs-To-Many associations are used to connect sources with multiple targets. Furthermore the targets can also have connections to multiple sources.
```
Project.belongsToMany(User, {through: 'UserProject'});
User.belongsToMany(Project, {through: 'UserProject'});

```
This will create a new model called UserProject with the equivalent foreign keys projectId and userId. Whether the attributes are camelcase or not depends on the two models joined by the table (in this case User and Project).

Defining through is required. Sequelize would previously attempt to autogenerate names but that would not always lead to the most logical setups.

This will add methods getUsers, setUsers, addUser,addUsers to Project, and getProjects, setProjects, addProject, and addProjects to User.

Sometimes you may want to rename your models when using them in associations. Let's define users as workers and projects as tasks by using the alias (as) option. We will also manually define the foreign keys to use:
```
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId' })
Project.belongsToMany(User, { as: 'Workers', through: 'worker_tasks', foreignKey: 'projectId' })
```
foreignKey will allow you to set source model key in the through relation. otherKey will allow you to set target model key in the through relation.
```
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId', otherKey: 'projectId'})
```
Of course you can also define self references with belongsToMany:
```
Person.belongsToMany(Person, { as: 'Children', through: 'PersonChildren' })

```
// This will create the table PersonChildren which stores the ids of the objects.

###Attributes

```
Model.findAll({
  attributes: ['foo', 'bar']
});

```
SELECT foo, bar ...
Attributes can be renamed using a nested array:
```
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});

```
SELECT foo, bar AS baz ...
You can use sequelize.fn to do aggregations:

```
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});
```
SELECT COUNT(hats) AS no_hats ...
When using aggregation function, you must give it an alias to be able to access it from the model. In the example above you can get the number of hats with instance.get('no_hats').

Sometimes it may be tiresome to list all the attributes of the model if you only want to add an aggregation:

// This is a tiresome way of getting the number of hats...
```
Model.findAll({
  attributes: ['id', 'foo', 'bar', 'baz', 'quz', [sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});

// This is shorter, and less error prone because it still works if you add / remove attributes
Model.findAll({
  attributes: { include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']] }
});
```
SELECT id, foo, bar, baz, quz, COUNT(hats) AS no_hats ...
Similarly, it's also possible to remove a selected few attributes:
```
Model.findAll({
  attributes: { exclude: ['baz'] }
});
```
SELECT id, foo, bar, quz ...

### Migrations

详见项目中的演示

```
config/config.json.
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```
Now we should edit this file to insert demo user to User table.

```

'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users', [{
        firstName: 'John',
        lastName: 'Doe',
        email: 'demo@demo.com',
        createdAt: new Date(),
        updatedAt: new Date()
      }], {});
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};

```

```

module.exports = {
    up: (queryInterface, Sequelize) => {
        return queryInterface.sequelize.transaction((t) => {
            return Promise.all([
                queryInterface.addColumn('Person', 'petName', {
                    type: Sequelize.STRING
                }, { transaction: t }),
                queryInterface.addColumn('Person', 'favoriteColor', {
                    type: Sequelize.STRING,
                }, { transaction: t })
            ])
        })
    },

    down: (queryInterface, Sequelize) => {
        return queryInterface.sequelize.transaction((t) => {
            return Promise.all([
                queryInterface.removeColumn('Person', 'petName', { transaction: t }),
                queryInterface.removeColumn('Person', 'favoriteColor', { transaction: t })
            ])
        })
    }
};


```