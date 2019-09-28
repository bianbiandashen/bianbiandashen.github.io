
---
layout:     post
title:      egg sequelize 
subtitle:   
date:       2019-9-21
author:    	BIANBIAN
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - sql
    - pgsql
    - windows
---


## 1.1 pgsql

pgsql的安装 已经如何使用图形化工具建表等基础问题就不在这里阐述了，为什么会使用egg sequelize ， 因为egg中 想调用 pgsql

pgsql有个坑。。一定是在public下导入db喔

## 1.2 sequelize的好处

这才是本文的核心，sequelize目前可以说是目前最为成熟的node.js ORM框架了。CRUD操作不可能完全使用SQL语句进行，这样很容易出现各种SQL漏洞，一个成熟的ORM框架可以帮我们避免掉这些风险，并且将CRUD操作封装成对象函数方法之后，操作起来也更加方便，但是这样会提升一定的开发学习成本。


## 1.3 数据模型设计

边边建了3个简单的表

```javascript
CREATE TABLE `group` (
  `id` INTEGER PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(255) NOT NULL,
  `db_create_time` DATETIME,
  `db_update_time` DATETIME
);

CREATE TABLE `user` (
  `id` INTEGER PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(255) NOT NULL,
  `db_create_time` DATETIME,
  `db_update_time` DATETIME
);

CREATE TABLE `group_users` (
  `user` INTEGER NOT NULL,
  `group` INTEGER NOT NULL,
  CONSTRAINT `pk_group_users` PRIMARY KEY (`user`, `group`)
);

CREATE INDEX `idx_group_users` ON `group_users` (`group`);


```



## 1.4 项目结构

清清爽爽的结构


```javascript
|-- app                                     // node服务端相关代码
    |-- controller
        |-- api                             // node端接口controller
            |-- group.js                    // 组相关controller
            |-- user.js                     // 用户相关controller
    |-- extend
        |-- helper.js                       // helper扩展
    |-- middleware
    |-- model                               // sequelize数据模型
        |-- user.js
        |-- group.js
        |-- group_user.js
    |-- service                             // 可复用的数据处理及查询方法
    |-- utils                               // service中拿不到helper，部分utils放在这里
    |-- router.js                           // 路由
|-- build                                   // 构建代码
|-- client                                  // 客户端相关代码
|-- config                                  // 配置文件

```


## 1.5 model
egg-sequelize会自动将sequelize实例挂载到app.model上面，然后静态方法和属性则会直接被绑定到app上，通过app.Sequelize进行获取。
model层作为MVC的最底层，需要注意到数据模型的pure，model文件也应该是纯净的，这个文件里面应该是和数据库中的表一一对应，一个model文件对应一个DB中的表，这个文件中不应该包含任何和逻辑相关的代码，应该完全是数据模型的定义。

```
// app/model/user.js

module.exports = app => {
    // egg-sequelize插件会将Sequelize类绑定到app上线，从里面可以取到各种静态类型
    const { TEXT, INTEGER, NOW } = app.Sequelize;

    const User = app.model.define(
        'user',
        {
            name: TEXT,
            createAt: {
                type: DATE,
                // 可以重写某个字段的字段名
                field: 'db_create_time',
                allowNull: false,
                defaultValue: NOW,
            },
            updateAt: {
                type: DATE,
                field: 'db_update_time',
                allowNull: false,
                defaultValue: NOW,
            },
        },
        {
            timestamps: false,
            freezeTableName: true,
            tableName: 'users',
            underscored: true,
        }
    );

    // 定义关联关系
    User.associate = () => {
        // 定义多对多关联
        User.belongsToMany(app.model.Groups, {
            // 中间表的model
            through: app.model.groupUser,
            // 进行关联查询时，关联表查出来的数据模型的alias
            as: 'project',
            // 是否采用外键进行物理关联
            constraints: false,
        });
        // 这里如果一个模型和多个模型都有关联关系的话，关联关系需要统一定义在这里
    };

    return User;
};

```

上面的代码有非常多需要注意的地方，我们通过这个文件定义了一个数据模型，这个模型可以映射到数据库中的某一个表，这里就是映射到了users表，用来存储用户信息。

在默认情况下，id字段会被设置为主键，并且是AUTO_INCREMENT的，不需要我们自己声明；
timestamps字段可以表示是否采用默认的createAt和updateAt字段，我们通过field字段重写了这两个字段的字段名；
associate字段可以用来设置数据模型的关联关系，如果一个数据模型关联了多个数据模型，那么这个方法里面也可以定义多个关系；
belongsToMany表示n:m的关系映射，这个在官方文档中描述的非常清楚了；
as可以为这个映射设置别名，这样在进行查询的时候，得到的结果就是以别名来标识的；
constraints：这个属性非常重要，可以用来表示这个关联关系是否采用外键关联。在大多数情况下我们是不需要通过外键来进行数据表的物理关联的，直接通过逻辑进行关联即可；
through：这个属性表示关联表的数据模型，也就是保存关联关系的数据库表的模型。

上面的这些属性，在开发过程中多多少少都消耗了我一些时间**-1s**，模型的设置和数据库表之间的关系非常紧密，一定要保证你的数据模型和数据表之间没有歧义。
同样地，我们可以定义到关联表和中间表的模型：
```javascript
// app/model/group.js

module.exports = app => {
    const { TEXT, INTEGER, NOW } = app.Sequelize;

    const Group = app.model.define(
        'group',
        {
            name: TEXT,
            createAt: {
                type: DATE,
                field: 'db_create_time',
                allowNull: false,
                defaultValue: NOW,
            },
            updateAt: {
                type: DATE,
                field: 'db_update_time',
                allowNull: false,
                defaultValue: NOW,
            },
        },
        {
            timestamps: false,
            freezeTableName: true,
            tableName: 'groups',
            underscored: true,
        }
    );

    // 定义关联关系
    Group.associate = () => {
        Group.belongsToMany(app.model.User, {
            through: app.model.groupUser,
            as: 'partner',
            constraints: false,
        });
    };

    return Group;
};

// app/model/group_user.js
// 中间表不需要定义关联关系

module.exports = app => {
    const { INTEGER } = app.Sequelize;

    const GroupUser = app.model.define(
        'group_user',
        {
            user_id: INTEGER,
            group_id: INTEGER,
        },
        {
            timestamps: false,
            freezeTableName: true,
            tableName: 'group_user',
            underscored: true,
        }
    );
    
    return GroupUser;
};

```


## 1.6controller


在egg中，controller模块的作用类似于MVC模式中的控制器，进行从model到view的转换，而在提供接口的时候，controller负责的是提供从model到api的转换，经过model从数据库中查询出来的结果，将在controller里面进行包装，然后返回给接口的调用者。
在进行数据访问的时候，很多的接口请求都可以拆分为几个类似的CRUD操作，比如：

我想查一个用户的注册时间；
我想查一个用户的用户名；
这样类似的操作都可以通过一样的数据库操作拿到，然后再进行单独处理，这些可复用的逻辑，根据egg的建议，都可以写到service里面。而controller只负责请求的响应处理。

当一个接口请求跨过了middleware的处理，经过了router的分发之后：
```javascript
// app/router.js

module.exports = app => {
    app.get('/api/user/get', app.controller.api.user.get);
    
    app.post('/api/group/set', app.controller.api.group.set);
}
```
会被转发到对应的controller进行处理。

```javascript
// app/controller/user.js

module.exports = class UserController extends Controller {
    async get = () => {
        const { uuid } = this.ctx.session;

        if (!uuid) {
            ctx.body = {
                code: 401,
                message: 'unauthorized',
            };
            return;
        }

        const userInfo = await this.ctx.service.user.getUserById({ id: uuid });

        if (userInfo) {
            ctx.body = {
                code: 200,
                message: 'success',
                data: userInfo
            }
        } else {
            ctx.body = {
                code: 500,
                message: 'error',
            }
        }
    }
}
```

##1.7 service
egg官方文档对于service的描述是这样的：

简单来说，Service 就是在复杂业务场景下用于做业务逻辑封装的一个抽象层，提供这个抽象有以下几个好处：

保持 Controller 中的逻辑更加简洁。
保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
将逻辑和展现分离，更容易编写测试用例。


也就是controller中要尽量保持clean，然后，可以复用的业务逻辑被统一抽出来，放到service中，被多个controller进行复用。
我们将CRUD操作，全部提取到service中，封装成一个个通用的CRUD方法，来提供给其他service进行嵌套的时候调用，或者提供给controller进行业务逻辑调用。



比如：读取用户信息的过程：

```javascript
// app/service/user.js

module.exports = class UserService extends Service {
    // 通过id获取用户信息
    async getUserById = ({
        id,
    }) => {
        const { ctx } = this;

        let userInfo = {};
        try {
            userInfo = await ctx.model.User.findAll({
                where: {
                    id,
                },
                // 查询操作的时候，加入这个参数可以直接拿到对象类型的查询结果，否则还需要通过方法调用解析
                raw: true,
            });
        } catch (err) {
            ctx.logger.error(err);
        }

        return userInfo;
    }
}
```
##1.8 sequelize事务

之前有说到，在建立模型的时候，我们建立了User和Group之间的关联关系，并且通过了一个关联表进行两者之间的关联。
由于我们没有建立两者之间的外键关联，所以在写入的时候，我们要进行逻辑的关联写入。
如果我们需要新建一个用户，并且为这个用户新建一个默认的group，由于组和用户有着多对多的关系，所以这里我们采用belongsToMany来建立关系。一个用户可以属于多个组，并且一组也可以包含多个用户。
在建立的时候，需要按照一定的顺序，写入三张表，一旦某个写入操作失败之后，需要对于之前的写入操作进行回滚，防止DB中产生垃圾数据。这里需要用到事务机制进行写入控制，并且人工保证写入顺序。
```javascript
// app/service/user.js

module.exports = class UserService extends Service {
    async setUser = ({
        name,
    }) => {
        const { ctx } = this;
        let transaction;

        try {
            // 这里需要注意，egg-sequelize会将sequelize实例作为app.model对象
            transaction = await ctx.model.transaction();

            // 创建用户
            const user = await ctx.model.User.create({
                name,
            }, {
                transaction,
            });

            // 创建默认组
            const group = await ctx.model.Group.create({
                name: 'default',
            }, {
                transaction,
            });

            const userId = user && user.getDataValue('id');
            const groupId = group && group.getDataValue('id');

            if (!userId || !groupId) {
                throw new Error('创建用户失败');
            }

            // 创建用户和组之间的关联
            const associate = await ctx.mode.GroupUser.create({
                user_id: userId,
                group_id: groupId,
            }, {
                transaction,
            });

            await transaction.commit();

            return userId;
        } catch (err) {
            ctx.logger.error(err);
            await transaction.rollback();
        }
    }
}
```
通过sequelize提供的事务功能，可以将串联写入过程中的错误进行回滚，保证了每次写入操作的原子性。


##1.9关联查询 这个暂时不用太多


既然我们已经创建了关联关系，那么如果通过关联关系，查询到对应的数据库内容呢？

在多对多的关联条件下，如果我们要查询某个用户的所有分组信息，需要通过用户id来查询其关联的所有group。


```javascript

async getGroupByUserId = ({
    id,
}) => {
    const { ctx } = this;
    const group = await ctx.model.User.findAll({
        attributes: ['project.id', 'project.name'],
        include: [
            {
                model: ctx.model.Group,
                as: 'project',
                // 指定关联表查询属性，这里表示不需要任何关联表的属性
                attributes: [],
                through: {
                    // 指定中间表的属性，这里表示不需要任何中间表的属性
                    attributes: []
                }
            }
        ],
        where: {
            id,
        },
        raw: true,
        // 这个需要和上面的中间表属性配合，表示不忽略include属性中的attributes参数
        includeIgnoreAttributes: false,
    });
}


```

通过上面的关联查询方法，可以得到这样的一条SQL语句：
SELECT `project`.`id`, `project`.`name` FROM `users` AS `user` LEFT OUTER JOIN ( `group_user` AS `project->group_user` INNER JOIN `groups` AS `project` ON `project`.`id` = `project->group_user`.`group_id`) ON `user`.`id` = `project->group_user`.`user_id` WHERE `user`.`id` = 1;


结果
[ { id: 1, name: 'default' } ]


而在一对多和多对一的关系下，其本质和多对多基本上是一致的，是在多的方向存储一个冗余字段，来保存其对应的唯一元素的主键，无论是何种关系，其默认在sequelize中实现的数据模型，都是范式化的，如果需要反范式来提高数据库效率，还是需要自己去做冗余的。


##2. hooks
在数据库查询的过程中，难免需要在真正的CRUD前后进行一些数据的处理。
考虑到这样的一个场景：
在客户端，我们前端存储的用户名并不是通过name来表示的，而是通过nickName字段来进行表示的，在每次进行读写操作之前，例如这里允许用户自己修改自己的名字，当请求发送到服务端之后，交予service进行处理。
```javascript
// app/model/user
const User = app.model.define({
    // ...
}, {
    hooks: {
        beforeUpdate: (user, options) => {
            const name = user.nickName;
            delete user.nickName;
            user.name = name;
        }
    }
});


```

我们在定义模型的时候，直接定义好这个hook，beforeUpdate会在User模型每次调用update之前，调用这个hook。这个hook会传入update操作传入的参数实例，可以直接对这个实例进行修改，保证实际update操作的实例是正确的。
hooks可以使用的地方很多，这里只是简单介绍一下使用的方法，hooks中间也可以包含异步操作，但是要注意，如果包含异步操作的话，需要返回一个Promise。我们还可以在进行具有副作用的操作之前，对于用户权限进行校验。
hooks的使用是需要了解到其功能，然后根据自己的业务场景，灵活地进行使用的。


未完待续  sequelize还有很多需要挖掘的地方，它本身提供的很多功能在这次迭代的过程中都没有用到。比如scope、migration，有机会可以尝试下一些新的功能和实现方案





- 参考 egg sequelize 实践 in Alibaba Group 