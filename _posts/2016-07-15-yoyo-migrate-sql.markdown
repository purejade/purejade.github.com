---
layout: post
title:  "用yoyo-migrate来管理数据库结构迁移"
date:   2016-07-15 15:40:29 +0800
categories: mysql
---

## 用yoyo-migrate来管理数据库结构迁移

在项目开发初期，由于业务等不确定因素，数据库表结构需要频繁修改，那么如何记录数据库表结构的演化过程是一个复杂的问题。一般来说我们仅保存一份表结构，该表结构仅保存当前可用最新的数据库表结构。但问题来了，要是在使用过程中发现当前表结构不适应业务，想要回退到之前版本如何处理呢？ 

这个时候就想到了git这样的工具来进行schema管理，git主要用来管理code的，对于表结构有些大材小用，且不够直观，因此我们选取了了[yoyo-migrate][yoyo] 来进行表结构管理。

### yoyo-migrate的主要使用方法

1. 文件命名 

    date-xyz.py格式，date最好是一个序列或者时间

2. 文件内容格式
    
    <pre>
    from yoyo import step
    step(
     """
        ALTER TABLE `activities`
        ADD COLUMN `activity_type` TINYINT NOT NULL DEFAULT 0 AFTER `state`,
        ADD COLUMN `created_by` VARCHAR(50) NULL AFTER `home_key`,
        ADD COLUMN `checkpoints` TEXT NULL AFTER `created_by`,
        ADD COLUMN `pages` TEXT NULL AFTER `checkpoints`;
     """,
     """
       ALTER TABLE `activities`
       DROP COLUMN  `activity_type`,
       DROP COLUMN `created_by`,
       DROP COLUMN `checkpoints`,
       DROP COLUMN `pages`;
     """ 
    )
    step中第一个引号中内容为执行，第二个引号中为回退
    </pre>

3. 执行
 
    <pre>yoyo-migrate apply . mysql://user:pwd@localhost/db </pre>

    该命令会找到当前目录中满足yoyo-migrate格式的文件，然后将文件中数据库操作导入到对应的数据库中；第一次执行会生成一个.yoyo-migrate文件，保存着数据库连接配置；之后再执行就不需要加数据库连接了。

4. 回滚

    <pre>yoyo-migrate rollback .  mysql://user:pwd@localhost/db</pre>

通过以上方法，可以将每次数据表结构的操作都记录下来，并可以进行应用或者回滚，也方便数据库表结构的迁移。

### 注意事项    
如果迁移文件之间存在依赖关系，要注意它们之前的命名关系，yoyo-migrate按照文件名字顺序进行执行。如果显示的来调用依赖关系，则可以添加__depends__ = ['0001.create-foo']
 

[yoyo]: https://pypi.python.org/pypi/yoyo-migrations
