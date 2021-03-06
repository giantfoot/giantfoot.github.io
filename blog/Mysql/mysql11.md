# MYSQL核心技术总结


> Mysql **字符串索引**

- MySQL 是支持前缀索引的，也就是说，你可以定义字符串的一部分作为索引。默认地，如果你创建索引的语句不指定前缀长度，那么索引就会包含整个字符串。

  ```
  mysql> alter table SUser add index index1(email);
  或
  mysql> alter table SUser add index index2(email(6));
  ```
- 第一个语句创建的 index1 索引里面，包含了每个记录的整个字符串；而第二个语句创建的 index2 索引里面，对于每个记录都是只取前 6 个字节。

- 使用前缀索引就用不上覆盖索引对查询性能的优化。

总结：

1. 直接创建完整索引，这样可能比较占用空间；
2. 创建前缀索引，节省空间，但会增加查询扫描次数，并且不能使用覆盖索引；
3. **倒序存储**，再创建前缀索引，用于绕过字符串本身前缀的区分度不够的问题；
4. 创建 **hash 字段索引** ，查询性能稳定，有额外的存储和计算消耗，跟第三种方式一样，都不支持范围扫描
