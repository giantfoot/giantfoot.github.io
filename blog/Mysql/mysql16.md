# MYSQL核心技术总结


> Mysql  **随机排序**

- 随机排序
  ```
  mysql> select word from words order by rand() limit 3;
  ```

- 直接使用 order by rand()，这个语句需要 Using temporary 和 Using filesort，查询的执行代价往往是比较大的。所以，在设计的时候你要尽量避开这种写法。

- 尽量将业务逻辑写在业务代码中，让数据库只做“读写数据”的事情。
