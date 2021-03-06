---
layout: post
title: sequelize笔记
excerpt: sequelize笔记
categories: MySQL
keywords: mysql,sequelize,数据库
---
## 安装

```
npm install --save sequelize
# 选择对应的安装:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
```

## 建立连接
```
const Sequelize = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql',
  pool: {
        max: 5,             // 连接池最大连接数量
        min: 0,             // 连接池最小连接数量
        idle: 10000         //如果一个线程超过10秒钟没有被使用过就释放该线程
        acquire: 30000,
    },
    define: {
    // `timestamps` 字段指定是否将创建 `createdAt` 和 `updatedAt` 字段. 该值默认为 true
    timestamps: false,
    //默认情况下,表名自动复数;使用 freezeTableName:true 参数可以为特定模型停止此行为
    freezeTableName:true,
    // 不删除数据库条目,但将新添加的属性deletedAt设置为当前日期(删除完成时). 
    // paranoid 只有在启用时间戳时才能工作
    paranoid: true,
    // 将自动设置所有属性的字段参数为下划线命名方式.
    // 不会覆盖已经定义的字段选项
    underscored: true,
    // 启用乐观锁定. 启用时,sequelize将向模型添加版本计数属性,
    // 并在保存过时的实例时引发OptimisticLockingError错误.
    // 设置为true或具有要用于启用的属性名称的字符串.
    version: true,
  }
});

sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```
- 构造函数采用 define 参数,它将更改所有已定义模型的默认参数
- 如果你希望sequelize处理时间戳,但只想要其中一部分,或者希望你的时间戳被称为别的东西,则可以单独覆盖每个列:

```
class Foo extends Model {}
Foo.init({ /* bla */ }, {
  // 不要忘记启用时间戳！
  timestamps: true,

  // 我不想要 createdAt
  createdAt: false,

  // 我想 updateAt 实际上被称为 updateTimestamp
  updatedAt: 'updateTimestamp',

  // 并且希望 deletedA t被称为 destroyTime(请记住启用paranoid以使其工作)
  deletedAt: 'destroyTime',
  paranoid: true,

  sequelize,
})
```

## 模型定义
```
const User = sequelize.define('user', {
  // 属性
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull 默认为 true
  }
}, {
  // 参数
  timestamps: true 
});
```

```
class Foo extends Model {}
Foo.init({
 // 如果未赋值,则自动设置值为 TRUE
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true},

 // 设置默认时间为当前时间
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },

 // 将allowNull设置为false会将NOT NULL添加到列中,
 // 这意味着当列为空时执行查询时将从DB抛出错误. 
 // 如果要在查询DB之前检查值不为空,请查看下面的验证部分.
 title: { type: Sequelize.STRING, allowNull: false},

 // 创建具有相同值的两个对象将抛出一个错误. 唯一属性可以是布尔值或字符串.
 // 如果为多个列提供相同的字符串,则它们将形成复合唯一键.
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex'},
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex'},

 // unique属性用来创建一个唯一约束.
 someUnique: {type: Sequelize.STRING, unique: true},
 
 // 这与在模型选项中创建索引完全相同.
 {someUnique: {type: Sequelize.STRING}},
 {indexes: [{unique: true, fields: ['someUnique']}]},

 // primaryKey用于定义主键.
 identifier: { type: Sequelize.STRING, primaryKey: true},

 // autoIncrement可用于创建自增的整数列
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },

 // 你可以通过'field'属性指定自定义列名称:
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },

 // 这可以创建一个外键:
 bar_id: {
   type: Sequelize.INTEGER,

   references: {
     // 这是引用另一个模型
     model: Bar,

     // 这是引用模型的列名称
     key: 'id',

     // 这声明什么时候检查外键约束. 仅限PostgreSQL.
     deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
   }
 },

 // 仅可以为 MySQL,PostgreSQL 和 MSSQL 的列添加注释
 commentMe: {
   type: Sequelize.INTEGER,

   comment: '这是一个包含注释的列名'
 }
}, {
  sequelize,
  modelName: 'foo'
});
```
### Getters & setters
Getters和Setters可以通过两种方式定义,如果在两个地方定义了getter或setter,在相关属性定义中找到的函数始终是优先的
- 作为属性定义的一部分

```
class Employee extends Model {}
Employee.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue('title');
      // 'this' 允许你访问实例的属性
      return this.getDataValue('name') + ' (' + title + ')';
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue('title', val.toUpperCase());
    }
  }
}, { sequelize, modelName: 'employee' });

Employee
  .create({ name: 'John Doe', title: 'senior engineer' })
  .then(employee => {
    console.log(employee.get('name')); // John Doe (SENIOR ENGINEER)
    console.log(employee.get('title')); // SENIOR ENGINEER
  })
```
- 作为模型参数的一部分

```
sequelize.define('Foo', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods: {
    fullName() {
      return this.firstname + ' ' + this.lastname;
    }
  },

  setterMethods: {
    fullName(value) {
      const names = value.split(' ');

      this.setDataValue('firstname', names.slice(0, -1).join(' '));
      this.setDataValue('lastname', names.slice(-1).join(' '));
    }
  }
});
```
### 验证
- 验证会自动运行在 create , update 和 save 上

```
class ValidateMe extends Model {}
ValidateMe.init({
  bar: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // 只允许字母
      is: /^[a-z]+$/i,          // 与上一个示例相同,使用了真正的正则表达式
      not: ["[a-z]",'i'],       // 不允许字母
      isEmail: true,            // 检查邮件格式 (foo@bar.com)
      isUrl: true,              // 检查连接格式 (http://foo.com)
      isIP: true,               // 检查 IPv4 (129.89.23.1) 或 IPv6 格式
      isIPv4: true,             // 检查 IPv4 (129.89.23.1) 格式
      isIPv6: true,             // 检查 IPv6 格式
      isAlpha: true,            // 只允许字母
      isAlphanumeric: true,     // 只允许使用字母数字
      isNumeric: true,          // 只允许数字
      isInt: true,              // 检查是否为有效整数
      isFloat: true,            // 检查是否为有效浮点数
      isDecimal: true,          // 检查是否为任意数字
      isLowercase: true,        // 检查是否为小写
      isUppercase: true,        // 检查是否为大写
      notNull: true,            // 不允许为空
      isNull: true,             // 只允许为空
      notEmpty: true,           // 不允许空字符串
      equals: 'specific value', // 只允许一个特定值
      contains: 'foo',          // 检查是否包含特定的子字符串
      notIn: [['foo', 'bar']],  // 检查是否值不是其中之一
      isIn: [['foo', 'bar']],   // 检查是否值是其中之一
      notContains: 'bar',       // 不允许包含特定的子字符串
      len: [2,10],              // 只允许长度在2到10之间的值
      isUUID: 4,                // 只允许uuids
      isDate: true,             // 只允许日期字符串
      isAfter: "2011-11-05",    // 只允许在特定日期之后的日期字符串
      isBefore: "2011-11-05",   // 只允许在特定日期之前的日期字符串
      max: 23,                  // 只允许值 <= 23
      min: 23,                  // 只允许值 >= 23
      isCreditCard: true,       // 检查有效的信用卡号码

      // 自定义验证器的示例:
      isEven(value) {
        if (parseInt(value) % 2 !== 0) {
          throw new Error('Only even values are allowed!');
        }
      }
      isGreaterThanOtherField(value) {
        if (parseInt(value) <= parseInt(this.otherField)) {
          throw new Error('Bar must be greater than otherField.');
        }
      }
    }
  }
}, { sequelize });
```
### 数据库同步

```
// 创建表:
Project.sync()
Task.sync()

// 强制创建!
Project.sync({force: true}) // 这将先丢弃表,然后重新创建它

// 删除表:
Project.drop()
Task.drop()
```

```
// 同步所有尚未在数据库中的模型
sequelize.sync()

// 强制同步所有模型
sequelize.sync({force: true})

// 删除所有表
sequelize.drop()

// 只有当数据库名称以'_test'结尾时,才会运行.sync()
sequelize.sync({ force: true, match: /_test$/ });
```

## 模型使用
### find - 搜索数据库中的一个特定元素

```
// 搜索已知的ids
Project.findByPk(123).then(project => {
  // project 将是 Project的一个实例,并具有在表中存为 id 123 条目的内容.
  // 如果没有定义这样的条目,你将获得null
})

// 搜索属性
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
})


Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
  // project.get('title') 将包含 project 的 name
})
```
## findOrCreate - 搜索特定元素或创建它(如果不可用)

```
User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  . then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    findOrCreate 返回一个包含已找到或创建的对象的数组,找到或创建的对象和一个布尔值,如果创建一个新对象将为true,否则为false,像这样:

    [ {
        username: 'sdepold',
        job: 'Technical Lead JavaScript',
        id: 1,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      true ]

在上面的例子中,第三行的数组将分成2部分,并将它们作为参数传递给回调函数,在这种情况下将它们视为 "user" 和 "created" .(所以“user”将是返回数组的索引0的对象,并且 "created" 将等于 "true".)
    */
  })
```

## findAndCountAll
处理程序成功将始终接收具有两个属性的对象
- count - 一个整数,总数记录匹配where语句和关联的其它过滤器
- rows - 一个数组对象,记录在limit和offset范围内匹配where语句和关联的其它过滤器s

```
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
```
它支持 include. 只有标记为 required 的 include 将被添加到计数部分，
在include中添加一个 where 语句会自动使它成为 required
```
User.findAndCountAll({
  include: [
     { model: Profile, required: true}
  ],
  limit: 3
});
```
### findAll - 搜索数据库中的多个元素

```
// 找到多个条目
Project.findAll().then(projects => {
  // projects 将是所有 Project 实例的数组
})

// 搜索特定属性 - 使用哈希
Project.findAll({ where: { name: 'A Project' } }).then(projects => {
  // projects将是一个具有指定 name 的 Project 实例数组
})

// 在特定范围内进行搜索
Project.findAll({ where: { id: [1,2,3] } }).then(projects => {
  // projects将是一系列具有 id 1,2 或 3 的项目
  // 这实际上是在做一个 IN 查询
})

Project.findAll({
  where: {
    id: {
      [Op.and]: {a: 5},           // 且 (a = 5)
      [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 或 a = 6)
      [Op.gt]: 6,                // id > 6
      [Op.gte]: 6,               // id >= 6
      [Op.lt]: 10,               // id < 10
      [Op.lte]: 10,              // id <= 10
      [Op.ne]: 20,               // id != 20
      [Op.between]: [6, 10],     // 在 6 和 10 之间
      [Op.notBetween]: [11, 15], // 不在 11 和 15 之间
      [Op.in]: [1, 2],           // 在 [1, 2] 之中
      [Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
      [Op.like]: '%hat',         // 包含 '%hat'
      [Op.notLike]: '%hat',       // 不包含 '%hat'
      [Op.iLike]: '%hat',         // 包含 '%hat' (不区分大小写)  (仅限 PG)
      [Op.notILike]: '%hat',      // 不包含 '%hat'  (仅限 PG)
      [Op.overlap]: [1, 2],       // && [1, 2] (PG数组重叠运算符)
      [Op.contains]: [1, 2],      // @> [1, 2] (PG数组包含运算符)
      [Op.contained]: [1, 2],     // <@ [1, 2] (PG数组包含于运算符)
      [Op.any]: [2,3],            // 任何数组[2, 3]::INTEGER (仅限 PG)
    },
    status: {
      [Op.not]: false,           // status 不为 FALSE
    }
  }
})
```
- 复合过滤 / OR / NOT 查询

```
Project.findOne({
  where: {
    name: 'a project',
    [Op.or]: [
      { id: [1,2,3] },
      { id: { [Op.gt]: 10 } }
    ]
  }
})

Project.findOne({
  where: {
    name: 'a project',
    id: {
      [Op.or]: [
        [1,2,3],
        { [Op.gt]: 10 }
      ]
    }
  }
})
```
- 用限制,偏移,顺序和分组操作数据集


```
// 限制查询的结果
Project.findAll({ limit: 10 })

// 跳过前10个元素
Project.findAll({ offset: 10 })

// 跳过前10个元素,并获取2个
Project.findAll({ offset: 10, limit: 2 })
```

- 分组和排序

```
Project.findAll({order: [['title', 'DESC']]})
// 生成 ORDER BY title DESC

Project.findAll({group: 'name'})
// 生成 GROUP BY name
```

```
something.findOne({
  order: [
    // 将返回 `name`
    ['name'],
    // 将返回 `username` DESC
    ['username', 'DESC'],
    // 将返回 max(`age`)
    sequelize.fn('max', sequelize.col('age')),
    // 将返回 max(`age`) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // 将返回 otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // 将返回 otherfunction(awesomefunction(`col`)) DESC,这个嵌套是可以无限的！
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
  ]
})
```
order / group数组的元素可以是以下内容:
- String - 将被引用
- Array - 第一个元素将被引用,第二个将被逐字地追加
- Object -
    - raw 将被添加逐字引用
    - 如果未设置 raw,一切都被忽略,查询将失败
- Sequelize.fn 和 Sequelize.col 返回函数和引用的列名

### count - 计算数据库中元素的出现次数

```
Project.count().then(c => {
  console.log("There are " + c + " projects!")
})

Project.count({ where: {'id': {[Op.gt]: 25}} }).then(c => {
  console.log("There are " + c + " projects with an id greater than 25.")
})
```
### max - 获取特定表中特定属性的最大值

```
/*
   我们假设3个具有属性年龄的对象.
   第一个是10岁,
   第二个是5岁,
   第三个是40岁.
*/
Project.max('age').then(max => {
  // 将返回 40
})

Project.max('age', { where: { age: { [Op.lt]: 20 } } }).then(max => {
  // 将会是 10
})
```

### min - 获取特定表中特定属性的最小值

```
/*
   我们假设3个具有属性年龄的对象.
   第一个是10岁,
   第二个是5岁,
   第三个是40岁.
*/
Project.min('age').then(min => {
  // 将返回 5
})

Project.min('age', { where: { age: { [Op.gt]: 5 } } }).then(min => {
  // 将会是 10
})
```
### sum - 特定属性的值求和

```
/*
   我们假设3个具有属性年龄的对象.
   第一个是10岁,
   第二个是5岁,
   第三个是40岁.
*/
Project.sum('age').then(sum => {
  // 将返回 55
})

Project.sum('age', { where: { age: { [Op.gt]: 5 } } }).then(sum => {
  // 将会是 50
})
```
## Hooks - 钩子
Hook(也称为生命周期事件)是执行 sequelize 调用之前和之后调用的函数，
目前有三种以编程方式添加 hook 
```
// 方法1 通过 .init() 方法
class User extends Model {}
User.init({
  username: DataTypes.STRING,
  mood: {
    type: DataTypes.ENUM,
    values: ['happy', 'sad', 'neutral']
  }
}, {
  hooks: {
    beforeValidate: (user, options) => {
      user.mood = 'happy';
    },
    afterValidate: (user, options) => {
      user.username = 'Toni';
    }
  },
  sequelize
});

// 方法2 通过 .addHook() 方法
User.addHook('beforeValidate', (user, options) => {
  user.mood = 'happy';
});

User.addHook('afterValidate', 'someCustomName', (user, options) => {
  return Promise.reject(new Error("I'm afraid I can't let you do that!"));
});

// 方法3 通过直接方法
User.beforeCreate((user, options) => {
  return hashPassword(user.password).then(hashedPw => {
    user.password = hashedPw;
  });
});

User.afterValidate('myHookAfter', (user, options) => {
  user.username = 'Toni';
});
```
## Querying - 查询
### 属性
想要只选择某些属性,可以使用 attributes 选项. 通常是传递一个数组:

```
Model.findAll({
  attributes: ['foo', 'bar']
});
```
属性可以使用嵌套数组来重命名:

```
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});
```
也可以使用 sequelize.fn 来进行聚合:

```
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});

SELECT COUNT(hats) AS no_hats ...
```
添加聚合,且列出模型的所有属性
```
Model.findAll({
  attributes: { include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']] }
});

SELECT id, foo, bar, baz, quz, COUNT(hats) AS no_hats ...
```
同样,它也可以排除一些指定的表字段:

```
Model.findAll({
  attributes: { exclude: ['baz'] }
});
```
### Where

```
const Op = Sequelize.Op;

Post.findAll({
  where: {
    authorId: 2
  }
});
// SELECT * FROM post WHERE authorId = 2

Post.findAll({
  where: {
    authorId: 12,
    status: 'active'
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';

Post.findAll({
  where: {
    [Op.or]: [{authorId: 12}, {authorId: 13}]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.findAll({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.destroy({
  where: {
    status: 'inactive'
  }
});
// DELETE FROM post WHERE status = 'inactive';

Post.update({
  updatedAt: null,
}, {
  where: {
    deletedAt: {
      [Op.ne]: null
    }
  }
});
// UPDATE post SET updatedAt = null WHERE deletedAt NOT NULL;

Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('status')), 6)
});
// SELECT * FROM post WHERE char_length(status) = 6;
```
## 操作符

```
const Op = Sequelize.Op

[Op.and]: {a: 5}           // 且 (a = 5)
[Op.or]: [{a: 5}, {a: 6}]  // (a = 5 或 a = 6)
[Op.gt]: 6,                // id > 6
[Op.gte]: 6,               // id >= 6
[Op.lt]: 10,               // id < 10
[Op.lte]: 10,              // id <= 10
[Op.ne]: 20,               // id != 20
[Op.eq]: 3,                // = 3
[Op.not]: true,            // 不是 TRUE
[Op.between]: [6, 10],     // 在 6 和 10 之间
[Op.notBetween]: [11, 15], // 不在 11 和 15 之间
[Op.in]: [1, 2],           // 在 [1, 2] 之中
[Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
[Op.like]: '%hat',         // 包含 '%hat'
[Op.notLike]: '%hat'       // 不包含 '%hat'
[Op.iLike]: '%hat'         // 包含 '%hat' (不区分大小写)  (仅限 PG)
[Op.notILike]: '%hat'      // 不包含 '%hat'  (仅限 PG)
[Op.startsWith]: 'hat'     // 类似 'hat%'
[Op.endsWith]: 'hat'       // 类似 '%hat'
[Op.substring]: 'hat'      // 类似 '%hat%'
[Op.regexp]: '^[h|a|t]'    // 匹配正则表达式/~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.notRegexp]: '^[h|a|t]' // 不匹配正则表达式/!~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.iRegexp]: '^[h|a|t]'    // ~* '^[h|a|t]' (仅限 PG)
[Op.notIRegexp]: '^[h|a|t]' // !~* '^[h|a|t]' (仅限 PG)
[Op.like]: { [Op.any]: ['cat', 'hat']} // 包含任何数组['cat', 'hat'] - 同样适用于 iLike 和 notLike
[Op.overlap]: [1, 2]       // && [1, 2] (PG数组重叠运算符)
[Op.contains]: [1, 2]      // @> [1, 2] (PG数组包含运算符)
[Op.contained]: [1, 2]     // <@ [1, 2] (PG数组包含于运算符)
[Op.any]: [2,3]            // 任何数组[2, 3]::INTEGER (仅限PG)

[Op.col]: 'user.organization_id' // = 'user'.'organization_id', 使用数据库语言特定的列标识符, 本例使用 PG
```
### 组合

```
{
  rank: {
    [Op.or]: {
      [Op.lt]: 1000,
      [Op.eq]: null
    }
  }
}
// rank < 1000 OR rank IS NULL

{
  createdAt: {
    [Op.lt]: new Date(),
    [Op.gt]: new Date(new Date() - 24 * 60 * 60 * 1000)
  }
}
// createdAt < [timestamp] AND createdAt > [timestamp]

{
  [Op.or]: [
    {
      title: {
        [Op.like]: 'Boat%'
      }
    },
    {
      description: {
        [Op.like]: '%boat%'
      }
    }
  ]
}
// title LIKE 'Boat%' OR description LIKE '%boat%'
```
### 运算符别名
Sequelize 允许将特定字符串设置为操作符的别名.使用v5,将为你提供弃用警告.
```
const Op = Sequelize.Op;
const operatorsAliases = {
  $gt: Op.gt
}
const connection = new Sequelize(db, user, pass, { operatorsAliases })

[Op.gt]: 6 // > 6
$gt: 6 // 等同于使用 Op.gt (> 6)
```
### 关系 / 关联

```
// 找到所有具有至少一个 task 的  project,其中 task.state === project.state
Project.findAll({
    include: [{
        model: Task,
        where: { state: Sequelize.col('project.state') }
    }]
})
```
### 排序

```
Subtask.findAll({
  order: [
    // 将转义标题,并根据有效的方向参数列表验证DESC
    ['title', 'DESC'],

    // 将按最大值排序(age)
    sequelize.fn('max', sequelize.col('age')),

    // 将按最大顺序(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // 将按 otherfunction 排序(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // 将使用模型名称作为关联的名称排序关联模型的 created_at.
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's created_at using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // 将使用关联的名称由关联模型的created_at排序.
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using an association object. (优选方法)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using association objects. (优选方法)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // 嵌套关联模型的 created_at 简单关联对象排序
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ]
  
  // 将按年龄最大值降序排列
  order: sequelize.literal('max(age) DESC')

  // 按最年龄大值升序排列,当省略排序条件时默认是升序排列
  order: sequelize.fn('max', sequelize.col('age'))

  // 按升序排列是省略排序条件的默认顺序
  order: sequelize.col('age')
  
  // 将根据方言随机排序 (而不是 fn('RAND') 或 fn('RANDOM'))
  order: sequelize.random()
})
```
## Instances - 实例
### 构建非持久性实例

```
// 首先定义模型
class Task extends Model {}
Task.init({
  title: Sequelize.STRING,
  rating: { type: Sequelize.TINYINT, defaultValue: 3 }
}, { sequelize, modelName: 'task' });
 
// 现在实例化一个对象
const task = Task.build({title: 'very important task'})
 
task.title  // ==> 'very important task'
task.rating // ==> 3
```
要将其存储在数据库中,请使用 save 方法并捕获事件(如果需要):

```
project.save().then(() => {
  // 回调
})
 
task.save().catch(error => {
  // 呃
})
 
// 还可以使用链式构建来保存和访问对象:
Task
  .build({ title: 'foo', description: 'bar', deadline: new Date() })
  .save()
  .then(anotherTask => {
    // 你现在可以使用变量 anotherTask 访问当前保存的任务
  })
  .catch(error => {
    // Ooops,做一些错误处理
  })
```
### 创建持久性实例

```
Task.create({ title: 'foo', description: 'bar', deadline: new Date() }).then(task => {
  // 你现在可以通过变量 task 来访问新创建的 task
})
```
也可以通过 create 方法定义哪些属性可以设置

```
User.create({ username: 'barfooz', isAdmin: true }, { fields: [ 'username' ] }).then(user => {
  // 我们假设 isAdmin 的默认值为 false:
  console.log(user.get({
    plain: true
  })) // => { username: 'barfooz', isAdmin: false }
})
```
### 更新 / 保存 / 持久化一个实例

```
// 方法 1
task.title = 'a very different title now'
task.save().then(() => {})
 
// 方法 2
task.update({
  title: 'a very different title now'
}).then(() => {})

task.title = 'foooo'
task.description = 'baaaaaar'
task.save({fields: ['title']}).then(() => {
 // title 现在将是 “foooo”,而 description 与以前一样
})
 
// 使用等效的 update 调用如下所示:
task.update({ title: 'foooo', description: 'baaaaaar'}, {fields: ['title']}).then(() => {
 //  title 现在将是 “foooo”,而 description 与以前一样
})
```
### 销毁 / 删除持久性实例
如果 paranoid 选项为 true,则不会删除该对象,而将 deletedAt 列设置为当前时间戳. 要强制删除,可以将 force: true 传递给 destroy 调用:

```
task.destroy({ force: true })
```
在 paranoid 模式下对象被软删除后,在强制删除旧实例之前,你将无法使用相同的主键创建新实例.
恢复软删除的实例：

```
task.restore();
```
### 批量操作(一次性创建,更新和销毁多行)
- Model.bulkCreate
- Model.update
- Model.destroy

```
Task.bulkCreate([
  {subject: 'programming', status: 'executing'},
  {subject: 'reading', status: 'executing'},
  {subject: 'programming', status: 'finished'}
]).then(() => {})

Task.update(
    { status: 'inactive' }, /* 设置属性的值 */
    { where: { subject: 'programming' }} /* where 规则 */
  );

Task.destroy({
where: {
  subject: 'programming'
},
truncate: true /* 这将忽 where 并用 truncate table 替代  */
});
```
### 一个实例的值
使用选项 plain: true 调用它将只返回一个实例的值，
还可以使用 JSON.stringify(instance) 将一个实例转换为 JSON. 基本上与 values 返回的相同.
```
Person.create({
  name: 'Rambow',
  firstname: 'John'
}).then(john => {
  console.log(john.get({
    plain: true
  }))
})
 
// 结果:
 
// { name: 'Rambow',
//   firstname: 'John',
//   id: 1,
//   createdAt: Tue, 01 May 2012 19:12:16 GMT,
//   updatedAt: Tue, 01 May 2012 19:12:16 GMT
// }
```
### 重载实例
将从数据库中获取当前数据,并覆盖调用该方法的模型的属性.

```
Person.findOne({ where: { name: 'john' } }).then(person => {
  person.name = 'jane'
  console.log(person.name) // 'jane'
 
  person.reload().then(() => {
    console.log(person.name) // 'john'
  })
})
```
### 递增 递减

```
User.findByPk(1).then(user => {
  return user.increment({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* .需要调用 user.reload() 来获取更新的实例.. */)

User.findByPk(1).then(user => {
  return user.decrement({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* ... */)
```
### Raw queries - 原始查询

```
sequelize.query("UPDATE users SET y = 42 WHERE x = 12").then((results, metadata) => {
  // 结果将是一个空数组,元数据将包含受影响的行数.
})
```
- 可以传递一个查询类型来告诉后续如何格式化结果：
```
sequelize.query("SELECT * FROM `users`", { type: sequelize.QueryTypes.SELECT})
  .then(users => {
    // 我们不需要在这里延伸,因为只有结果将返回给选择查询
  })
  
  const QueryTypes = module.exports = { // eslint-disable-line
  SELECT: 'SELECT',
  INSERT: 'INSERT',
  UPDATE: 'UPDATE',
  BULKUPDATE: 'BULKUPDATE',
  BULKDELETE: 'BULKDELETE',
  DELETE: 'DELETE',
  UPSERT: 'UPSERT',
  VERSION: 'VERSION',
  SHOWTABLES: 'SHOWTABLES',
  SHOWINDEXES: 'SHOWINDEXES',
  DESCRIBE: 'DESCRIBE',
  RAW: 'RAW',
  FOREIGNKEYS: 'FOREIGNKEYS',
  SHOWCONSTRAINTS: 'SHOWCONSTRAINTS'
};
```
- 传递模型,返回的数据将是该模型的实例

```
// Callee 是模型定义. 这样你就可以轻松地将查询映射到预定义的模型
sequelize
  .query('SELECT * FROM projects', {
    model: Projects,
    mapToModel: true // 如果你有任何映射字段,则在此处传递true
  })
  .then(projects => {
    // 每个记录现在将是Project的一个实例
  })
```

```
sequelize.query('SELECT 1', {
  // 用于记录查询的函数(或false)
  // 将调用发送到服务器的每个SQL查询.
  logging: console.log,

  // 如果plain为true,则sequelize将仅返回结果集的第一条记录. 
  // 如果是false,它将返回所有记录.
  plain: false,

  // 如果你没有查询的模型定义,请将此项设置为true.
  raw: false,

  // 你正在执行的查询类型. 查询类型会影响结果在传回之前的格式.
  type: Sequelize.QueryTypes.SELECT
})

// 注意第二个参数为null！
// 即使我们在这里声明了一个被调用对象,
// raw: true 也会取代并返回一个原始对象.
sequelize
  .query('SELECT * FROM projects', { raw: true })
  .then(projects => {
    console.log(projects)
  })
```
- "Dotted" 属性,如果表的属性名称包含点,则生成的对象将嵌套

```
sequelize.query('select 1 as `foo.bar.baz`').then(rows => {
  console.log(JSON.stringify(rows))
})

[{
  "foo": {
    "bar": {
      "baz": 1
    }
  }
}]
```
- 替换
    - 如果传递一个数组, ? 将按照它们在数组中出现的顺序被替换
    - 如果传递一个对象, :key 将替换为该对象的键. 如果对象包含在查询中找不到的键,则会抛出异常,反之亦然.

```
sequelize.query('SELECT * FROM projects WHERE status = ?',
  { replacements: ['active'], type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT * FROM projects WHERE status = :status ',
  { replacements: { status: 'active' }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT * FROM projects WHERE status IN(:status) ',
  { replacements: { status: ['active', 'inactive'] }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT * FROM users WHERE name LIKE :search_name ',
  { replacements: { search_name: 'ben%'  }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```
### Associations - 关联
四种类型的关联
- BelongsTo
- HasOne
- HasMany
- BelongsToMany

#### 外键 
- 验证参数是 RESTRICT, CASCADE, NO ACTION, SET DEFAULT, SET NULL
- 对于 1:1 和 1:m 关联,默认选项是 SET NULL 用于删除,CASCADE 用于更新. 对于 n:m,两者的默认值是 CASCADE. 这意味着,如果你从 n:m 关联的一侧删除或更新一行,则引用该行的连接表中的所有行也将被删除或更新.
- Sequelize 允许为 Model 设置 underscored 选项. 当 true 时,此选项会将所有属性的 field 参数设置为其名称的下划线版本. 这也适用于关联生成的外键.
- 循环依赖关系 & 禁用约束：向其中一个关联传递 constraints: false

#### 在没有约束的情况下强制执行外键引用

```
class Trainer extends Model {}
Trainer.init({
  firstName: Sequelize.STRING,
  lastName: Sequelize.STRING
}, { sequelize, modelName: 'trainer' });

// 在我们调用 Trainer.hasMany(series) 后,
// series 将有一个 trainerId = Trainer.id 外键
class Series extends Model {}
Series.init({
  title: Sequelize.STRING,
  subTitle: Sequelize.STRING,
  description: Sequelize.TEXT,
  // 用 `Trainer` 设置 FK 关系(hasMany)
  trainerId: {
    type: Sequelize.INTEGER,
    references: {
      model: Trainer,
      key: 'id'
    }
  }
}, { sequelize, modelName: 'series' });
```

#### HasOne 和 BelongsTo 之间的区别
- HasOne 在 target 模型中插入关联键,而 BelongsTo 将关联键插入到 source 模型中.
- 在已定义 as 的情况下,将使用它代替目标模型名称.
- 在所有情况下,默认外键可以用 foreignKey 选项覆盖
- sourceKey参数,设置源键

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class UserRole extends Model {}
UserRole.init({/* attributes */}, { sequelize, modelName: 'userRole' });

User.belongsTo(UserRole, {as: 'role'}); // 将 role 添加到 user 而不是 userRole
```

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

User.belongsTo(Company, {foreignKey: 'fk_company'}); // 将 fk_company 添加到 User
```


```
// 将 companyName 属性添加到 User
// 使用 Company 的 name 属性作为 source 属性
Company.hasOne(User, {foreignKey: 'companyName', sourceKey: 'name'});
```

#### 一对多关联 (hasMany)

```
class City extends Model {}
City.init({ countryCode: Sequelize.STRING }, { sequelize, modelName: 'city' });
class Country extends Model {}
Country.init({ isoCode: Sequelize.STRING }, { sequelize, modelName: 'country' });

// 在这里,我们可以根据国家代码连接国家和城市
Country.hasMany(City, {foreignKey: 'countryCode', sourceKey: 'isoCode'});
City.belongsTo(Country, {foreignKey: 'countryCode', targetKey: 'isoCode'});
```

#### 多对多关联
- 创建一个名为 UserProject 的新模型,具有等效的外键projectId和userId

```
Project.belongsToMany(User, {through: 'UserProject'});
User.belongsToMany(Project, {through: 'UserProject'});
```
- 需要在关联中使用它们时重命名模型. 通过使用别名(as)选项将 users 定义为 workers 而 projects 定义为t asks. 我们还将手动定义要使用的外键
- foreignKey 将允许你在 through 关系中设置 source model 键.
- otherKey 将允许你在 through 关系中设置 target model 键.
```
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId' })
Project.belongsToMany(User, { as: 'Workers', through: 'worker_tasks', foreignKey: 'projectId' })
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId', otherKey: 'projectId'})
```
- 想要连接表中的其他属性,则可以在定义关联之前为连接表定义一个模型,然后再说明它应该使用该模型进行连接,而不是创建一个新的关联:

```
class User extends Model {}
User.init({}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({}, { sequelize, modelName: 'project' })
class UserProjects extends Model {}
UserProjects.init({
  status: DataTypes.STRING
}, { sequelize, modelName: 'userProjects' })
 
User.belongsToMany(Project, { through: UserProjects })
Project.belongsToMany(User, { through: UserProjects })
```
- 使用多对多你可以基于 through 关系查询并选择特定属性. 例如通过 through 使用findAll

```
User.findAll({
  include: [{
    model: Project,
    through: {
      attributes: ['createdAt', 'startedAt', 'finishedAt'],
      where: {completed: true}
    }
  }]
});
```
- 当通过模型不存在主键时,Belongs-To-Many会创建唯一键. 可以使用 uniqueKey 选项覆盖此唯一键名.

```
Project.belongsToMany(User, { through: UserProjects, uniqueKey: 'my_custom_unique' })
```
#### 关联对象
必须在设置关联后调用 Sequelize.sync. 这样做将允许你进行以下操作:

```
Project.hasMany(Task)
Task.belongsTo(Project)
 
Project.create()...
Task.create()...
Task.create()...
 
// 保存它们.. 然后:
project.setTasks([task1, task2]).then(() => {
  // 已保存!
})
 
// 好的,现在它们已经保存了...我怎么才能得到他们？
project.getTasks().then(associatedTasks => {
  // associatedTasks 是一个 tasks 的数组
})
 
// 你还可以将过滤器传递给getter方法.
// 它们与你能传递给常规查找器方法的选项相同.
project.getTasks({ where: 'id > 10' }).then(tasks => {
  // id大于10的任务
})
 
// 你也可以仅检索关联对象的某些字段.
project.getTasks({attributes: ['title']}).then(tasks => {
  // 使用属性“title”和“id”检索任务
})
```

```
// 删除与 task1 的关联
project.setTasks([task2]).then(associatedTasks => {
  // 你将只得到 task2
})
 
// 删除全部
project.setTasks([]).then(associatedTasks => {
  // 你将得到空数组
})
 
// 或更直接地删除
project.removeTask(task1).then(() => {
  // 什么都没有
})
 
// 然后再次添加它们
project.addTask(task1).then(() => {
  // 它们又回来了
})
```
#### 检查关联

```
// 检查对象是否是关联对象之一:
Project.create({ /* */ }).then(project => {
  return User.create({ /* */ }).then(user => {
    return project.hasUser(user).then(result => {
      // 结果是 false
      return project.addUser(user).then(() => {
        return project.hasUser(user).then(result => {
          // 结果是 true
        })
      })
    })
  })
})
 
// 检查所有关联的对象是否如预期的那样:
// 我们假设我们已经有一个项目和两个用户
project.setUsers([user1, user2]).then(() => {
  return project.hasUsers([user1]);
}).then(result => {
  // 结果是 true
  return project.hasUsers([user1, user2]);
}).then(result => {
  // 结果是 true
})
```
#### 用关联创建

```
class Product extends Model {}
Product.init({
  title: Sequelize.STRING
}, { sequelize, modelName: 'product' });
class User extends Model {}
User.init({
  firstName: Sequelize.STRING,
  lastName: Sequelize.STRING
}, { sequelize, modelName: 'user' });
class Address extends Model {}
Address.init({
  type: Sequelize.STRING,
  line1: Sequelize.STRING,
  line2: Sequelize.STRING,
  city: Sequelize.STRING,
  state: Sequelize.STRING,
  zip: Sequelize.STRING,
}, { sequelize, modelName: 'address' });

Product.User = Product.belongsTo(User);
User.Addresses = User.hasMany(Address);
// 也能用于 `hasOne`
```
可以通过以下方式在一个步骤中创建一个新的Product, User和一个或多个Address:：

```
return Product.create({
  title: 'Chair',
  user: {
    firstName: 'Mick',
    lastName: 'Broadstone',
    addresses: [{
      type: 'home',
      line1: '100 Main St.',
      city: 'Austin',
      state: 'TX',
      zip: '78704'
    }]
  }
}, {
  include: [{
    association: Product.User,
    include: [ User.Addresses ]
  }]
});
```
HasMany / BelongsToMany 关联：

```
class Tag extends Model {}
Tag.init({
  name: Sequelize.STRING
}, { sequelize, modelName: 'tag' });

Product.hasMany(Tag);
// 也能用于 `belongsToMany`.
```
```
Product.create({
  id: 1,
  title: 'Chair',
  tags: [
    { name: 'Alpha'},
    { name: 'Beta'}
  ]
}, {
  include: [ Tag ]
})
```

### Transactions - 事务
支持两种使用事务的方法：
- 一个将根据 promise 链的结果自动提交或回滚事务,(如果启用)用回调将该事务传递给所有调用
- 而另一个 leave committing,回滚并将事务传递给用户.

#### 托管事务(auto-callback)
通过将回调传递给 sequelize.transaction 来启动托管事务.回传传递给 transaction 的回调是否是一个 promise 链,并且没有明确地调用t.commit()或 t.rollback(). 如果返回链中的所有 promise 都已成功解决,则事务被提交. 如果一个或几个 promise 被拒绝,事务将回滚.

```
return sequelize.transaction(t => {

  // 在这里链接你的所有查询. 确保你返回他们.
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
  // 事务已被提交
  // result 是 promise 链返回到事务回调的结果
}).catch(err => {
  // 事务已被回滚
  // err 是拒绝 promise 链返回到事务回调的错误
});
```
#### 非托管事务(then-callback)
务强制你手动回滚或提交交易. 如果不这样做,事务将挂起,直到超时. 要启动非托管事务,请调用 sequelize.transaction() 而不用 callback(你仍然可以传递一个选项对象),并在返回的 promise 上调用 then. 请注意,commit() 和 rollback() 返回一个 promise

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
  }).catch(err => {
    return t.rollback();
  });
});
```
- 后提交 hook,afterCommit 如果事务回滚,hook 不会 被提升.

```
sequelize.transaction(t => {
  t.afterCommit((transaction) => {
    // 你的逻辑片段
  });
});

sequelize.transaction().then(t => {
  t.afterCommit((transaction) => {
    // 你的逻辑片段
  });

  return t.commit();
})
```
### Scopes - 作用域
作用域允许你定义常用查询,以便以后轻松使用. 作用域可以包括与常规查找器 where, include, limit 等所有相同的属性.

```
class Project extends Model {}
Project.init({
  // 属性
}, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    },
    activeUsers: {
      include: [
        { model: User, where: { active: true }}
      ]
    },
    random () {
      return {
        where: {
          someNumber: Math.random()
        }
      }
    },
    accessLevel (value) {
      return {
        where: {
          accessLevel: {
            [Op.gte]: value
          }
        }
      }
    }
    sequelize,
    modelName: 'project'
  }
});
```
- 始终应用默认作用域. 这意味着,通过上面的模型定义,Project.findAll() 将创建以下查询:

```
SELECT * FROM projects WHERE active = true
```
- 可以通过调用 .unscoped(), .scope(null) 或通过调用另一个作用域来删除默认作用域:

```
Project.scope('deleted').findAll(); // 删除默认作用域
SELECT * FROM projects WHERE deleted = true
```
- 通过在模型定义上调用 .scope 来应用作用域,传递一个或多个作用域的名称. .scope 返回一个全功能的模型实例,它具有所有常规的方法:.findAll,.update,.count,.destroy等等.你可以保存这个模型实例并稍后再次使用
- 作用域适用于 .find, .findAll, .count, .update, .increment 和 .destroy.

```
const DeletedProjects = Project.scope('deleted');

DeletedProjects.findAll();
// 过一段时间

// 让我们再次寻找被删除的项目！
DeletedProjects.findAll();
```
- 如果作用域没有任何参数,它可以正常调用. 如果作用域采用参数,则传递一个对象:

```
Project.scope('random', { method: ['accessLevel', 19]}).findAll();
SELECT * FROM projects WHERE someNumber = 42 AND accessLevel >= 19
```
- 通过将作用域数组传递到 .scope 或通过将作用域作为连续参数传递,可以同时应用多个作用域.

```
// 这两个是等价的
Project.scope('deleted', 'activeUsers').findAll();
Project.scope(['deleted', 'activeUsers']).findAll();

SELECT * FROM projects
INNER JOIN users ON projects.userId = users.id
WHERE projects.deleted = true
AND users.active = true
```
- 如果要将其他作用域与默认作用域一起应用,请将键 defaultScope 传递给 .scope:

```
Project.scope('defaultScope', 'deleted').findAll();

SELECT * FROM projects WHERE active = true AND deleted = true
```
- 当调用多个作用域时,后续作用域的键将覆盖以前的作用域(类似于 Object.assign),除了where和include,它们将被合并. 考虑两个作用域:

```
{
  scope1: {
    where: {
      firstName: 'bob',
      age: {
        [Op.gt]: 20
      }
    },
    limit: 2
  },
  scope2: {
    where: {
      age: {
        [Op.gt]: 30
      }
    },
    limit: 10
  }
}

调用 .scope('scope1', 'scope2') 将产生以下查询

WHERE firstName = 'bob' AND age > 30 LIMIT 10
```
