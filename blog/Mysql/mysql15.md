# MYSQL核心技术总结


> Mysql  **order by**

- Extra 这个字段中的“Using filesort”表示的就是需要排序，MySQL 会给每个线程分配一块内存用于排序，称为 sort_buffer。

  ```

  CREATE TABLE `t` (
    `id` int(11) NOT NULL,
    `city` varchar(16) NOT NULL,
    `name` varchar(16) NOT NULL,
    `age` int(11) NOT NULL,
    `addr` varchar(128) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `city` (`city`)
  ) ENGINE=InnoDB;

  select city,name,age from t where city='杭州' order by name limit 1000  ;
  ```
- 需要返回的字段长度很大，使得排序内存能存储的行很少，使用rowid 排序，否则使用全字段排序

  **全字段排序** 流程：
  1. 初始化 sort_buffer，确定放入 name、city、age 这三个字段；

  2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；到主键 id 索引取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中；
  3. 从索引 city 取下一个记录的主键 id；重复步骤 3、
  4. 直到 city 的值不满足查询条件为止，对应的主键 id 也就是图中的 ID_Y；对 sort_buffer 中的数据按照字段 name 做快速排序；按照排序结果取前 1000 行返回给客户端。

  >按 name 排序”这个动作，**可能在内存中完成，也可能需要使用外部排序**，这取决于排序所需的内存和参数 **sort_buffer_size**。sort_buffer_size，就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。外部排序一般使用 **归并排序算法** 。可以这么简单理解，MySQL 将需要排序的数据分成 12 份，每一份单独排序后存在这些临时文件中。然后把这 12 个有序文件再合并成一个有序的大文件。

  **rowid 排序** 流程：
  1. 初始化 sort_buffer，确定放入两个字段，即 name 和 id；从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；

  2. 到主键 id 索引取出整行，取 name、id 这两个字段，存入 sort_buffer 中；
  3. 从索引 city 取下一个记录的主键 id；重复步骤 3、
  4. 直到不满足 city='杭州’条件为止，也就是图中的 ID_Y；
  5. 对 sort_buffer 中的数据按照字段 name 进行排序；遍历排序结果，取前 1000 行，并按照 id 的值回到原表中取出 city、name 和 age 三个字段返回给客户端。
  6. 实际上 MySQL 服务端从排序后的 sort_buffer 中依次取出 id，然后到原表查到 city、name 和 age 这三个字段的结果，不需要在服务端再耗费内存存储结果，是直接返回给客户端的。

  **建联合索引，避免排序**：
  ```
    alter table t add index city_user(city, name);
  ```
  我们依然可以用树搜索的方式定位到第一个满足 city='杭州’的记录，并且额外确保了，接下来按顺序取“下一条记录”的遍历过程中，只要 city 的值是杭州，name 的值就一定是有序的。这样整个查询过程的流程就变成了：

  1. 从索引 (city,name) 找到第一个满足 city='杭州’条件的主键 id；

  2. 到主键 id 索引取出整行，取 name、city、age 三个字段的值，作为结果集的一部分直接返回；
  3. 从索引 (city,name) 取下一个记录主键 id；
  4. 重复步骤 2、3，直到查到第 1000 条记录，或者是不满足 city='杭州’条件时循环结束。

  **覆盖联合索引再优化**：
  ```
    alter table t add index city_user_age(city, name, age);
  ```
  1. 从索引 (city,name,age) 找到第一个满足 city='杭州’条件的记录，取出其中的 city、name 和 age 这三个字段的值，作为结果集的一部分直接返回；

  2. 从索引 (city,name,age) 取下一个记录，同样取出这三个字段的值，作为结果集的一部分直接返回；
  3. 重复执行步骤 2，直到查到第 1000 条记录，或者是不满足 city='杭州’条件时循环结束。
