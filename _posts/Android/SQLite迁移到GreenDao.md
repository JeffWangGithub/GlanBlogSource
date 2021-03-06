---
title: SQLite迁移到GreenDao
date: 2017-08-17 11:48:21
tags: sqltie, GreenDao
categories: Android
---
从原生的 SQLite 迁移到 GreenDao 数据库框架整体上来讲还是相对比较容易的，这也归功与 GreenDao 易用的 API 和编译期注解。
### 整体的设计思路
![db_framework](SQLite迁移到GreenDao/db_framework.png)
- DAO 层主要由各个表的 Manager 类来进行承担，
    1. 负责与数据存储层进行 CURD 的操作。
    2. CommonDao 其实是对 GreenDao 框架 API 的封装，同时承担当数据变更时，通知业务层 Observer 的任务。
    3. 将 Entity 实体类转换成业务层的 Bean，以便对业务层的变更最小化
- Entity 实体主要定义了数据表的格式。
### 出现的问题
#### 旧数据库关键数据的迁移
问题描述:
![db_transform](SQLite迁移到GreenDao/db_transform.jpg)
存在两个问题:
1. 两个独立的数据库文件如何迁移
2. 何时进行迁移

解决方案:
** I 两个独立的数据库文件如何迁移  :  使用 SQLite 附加数据的功能 **
由于是两个独立的数据库，操作起来是非常困难的。因此我们最终目的就是通过一定的方式像操作一个数据库一样操作两个数据库文件。如何做到呢？将 old 数据库附加到 新建的数据库上，这样就可以像操作一个数据一样操作两个数据库文件了。
` ATTACH DATABASE 'DatabaseName' As 'Alias-Name'` 
如果数据库尚未被创建，上面的命令将创建一个数据库，如果数据库已存在，则把数据库文件名称与逻辑数据库 'Alias-Name' 绑定在一起。
** II 何时进行数据的迁移: 新数据库创建表完成的时候 **
可悲的是，DBHelper 的 onCreate 是在一个数据库事务中执行的，然而 SQLite 附加数据的操作是不允许在数据库事务中执行的，如果在数据库事务中执行会抛出异常。那么如何搞定这个事情呢？ 其实我们采取了变通的方式，即创建完表之后结束事务，进行 attach database 的操作，然后再开启事务。

```java
    @Override
    public void onCreate(Database db) {
         DaoMaster.createAllTables(db, false); //创建所有的声明的表
        boolean isOldDBExist = DBUtil.isDatabaseExist(DBMigrateHelper.OLD_DB_NAME);       
        if (isOldDBExist) {
            db.setTransactionSuccessful();
            db.endTransaction(); //处于数据库事务中的时候不允许进行 attach database 的操作，因此这里先结束事务，attach 完成之后再进行开启事务
            SQLiteDatabase oldDB = BaseApplication.getInstance().openOrCreateDatabase(DBMigrateHelper.OLD_DB_NAME, Context.MODE_PRIVATE, null);
            if (oldDB != null) {
                attachDB(); //执行 ATTACH DATABASE的操作和数据库表迁移的操作
                db.beginTransaction(); //重新开启事务
            }
        }
        initTableData(db);
    }
```
#### 数据库升级过程中创建新表和修改表
** I 创建表 **
创建表就是添加了新的 Entity，在 DBHelper 的 onUpgrade 方法中直接调用 Entity 对应 Dao类中的createTable()方法即可。
``` java
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        if(newVersion < oldVersion){
            Database wrapDB = wrap(db);
            DaoMaster.dropAllTables(wrapDB, false);
            onCreate(wrapDB);
        }
        TestUpdateEntityDao.createTable(db, true); //创建新表
    }
```
** II 修改表 **
修改表结构其实没什么好办法，只能在onUpgrade 方法中直接使用 raw sql 的方式进行修改了
#### 如何兼容业务中使用了ContentObserver 对数据变更进行监听
由于业务均是使用的 ContentProvider 的方式对数据库进行的操作，同时也是使用的 ContentObserver 的形式对数据变更进行的监听，为了保证业务变更最小，使用 CommonDao类对 GreenDao 的 CURD 操作的 api 进行了一层封装，并CommonDao 的 update和 insert 方法执行后进行ContentResolver 的 notify 操作。