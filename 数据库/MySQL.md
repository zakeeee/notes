# MySQL

[TOC]

## InnoDB 和 MyISAM 区别

1. InnoDB 支持事务，MyISAM 不支持。对于 InnoDB 每一条 SQL 语句都默认封装成事务，自动提交，这样会影响速度，所以最好把多条 SQL 语句放在 begin 和 commit 之间，组成一个事务。
2. InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败。
3. InnoDB 采用聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 采用非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
4. InnoDB 不保存表的具体行数，执行 `select count(*) from table` 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快。
5. InnoDB 同时支持行级锁和表级锁，而 MyISAM 只支持表级锁，不支持行级锁。
6. ~~InnoDB 不支持全文索引，而 MyISAM 支持全文索引，查询效率上 MyISAM 要高。~~（5.6.4 以后版本开始支持 InnoDB 引擎的 FULLTEXT 类型的索引）

## 参考文章

[MySQL 面试高频100问](https://zhuanlan.zhihu.com/p/76873520)