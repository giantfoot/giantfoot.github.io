# MYSQL核心技术总结


> Mysql  **索引选择的特殊情况**

- **对索引字段做函数操作**，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。

- 需要注意的是，优化器并不是要放弃使用这个索引。而是扫描了整个索引的所有值：全索引扫描；

- 函数操作会使MySQL 无法再使用索引快速定位功能，而只能使用全索引扫描。

- **有数据类型转换，就需要走全索引扫描**
  ```
  tradeid(varchar);
  mysql> select * from tradelog where tradeid=110717;
  相当于
  mysql> select * from tradelog where  CAST(tradid AS signed int) = 110717;
  ```

- **隐式字符编码转换**
```
mysql> select d.* from tradelog l, trade_detail d where d.tradeid=l.tradeid and l.id=2; /*语句Q1*/
```
两表字符集不同，tradelog(utf8mb4),trade_detail(utf8),字符集 utf8mb4 是 utf8 的超集，所以当这两个类型的字符串在做比较的时候，MySQL 内部的操作是，先把 utf8 字符串转成 utf8mb4 字符集，再做比较。

  上面sql相当于：
```
select * from trade_detail  where CONVERT(traideid USING utf8mb4)=$L2.tradeid.value;
```
所以，字符集不同只是条件之一，连接过程中要求在被驱动表的索引字段上加函数操作，是直接导致对被驱动表做全表扫描的原因。

  优化：
  ```
  mysql> select d.* from tradelog l , trade_detail d where d.tradeid=CONVERT(l.tradeid USING utf8) and l.id=2;
  ```
