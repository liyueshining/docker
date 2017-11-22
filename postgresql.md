
# PostgreSQL

数据库要从MySQL切换到PostgreSQL,在使用定义表对象时，之前MySQL表示布尔型 用的是tinyint，而PostgreSQL没有这种类型，故程序运行的时候会出现tinyint不识别，hibernate无法自动创建表结果的情况，将列的注解中的列定义修改为boolean之后，问题解决。

PostgreSQL还可以用作缓存。


网上阮大神总结了的入门教程。

http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html
