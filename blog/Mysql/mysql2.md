# MYSQL核心技术总结



> Mysql **更新流程**

- 更新过程主要涉及到 **redo log(重做日志)** 和 **bin log(归档日志)** 两个主角，其中redo log是InnoD引擎专有的日志，而bin log属于Server层，是所有引擎公用的。

- redo log记录的是数据页的物理变化，主要用于保证Crash-Safe，bin log记录的是逻辑改变（prepare模式直接记录SQL,row模式记录改变前后的两条数据），主要用于备份恢复，不能做崩溃恢复，历史原因，它就是这么设计的，没有这个功能。

- redo log有固定的大小，循环利用，空间不够便擦除最老的记录，无法用来备份恢复。bin log是追加模式，保存了跟现有数据库完全一样的数据，用来备份恢复。

- redo log来由：更新操作很频繁，如果每次更新都要把数据写入磁盘，再在磁盘中找到对应的数据，IO成本和查找成本都很高，所以**WAL**技术就应运而生（Write-Ahead Logging），先把更改逻辑写入日志，等空闲资源充沛的时候再把逻辑同步到磁盘。也就是说当一个更新逻辑出现的时候，会先把逻辑写入日志，再更新到内存，更新操作就算完成了，等到空闲的时候再这部分逻辑同步到数据页。

- 更新步骤：

    - 执行器根据条件通过引擎接口获取需要更新的数据行。如果这一行所在的数据页在内存中则直接返回，否则从磁盘中读取后先读入内存再返回。

    - 执行器拿到数据后根据逻辑更新成新的数据行，再调用引擎接口写入这行改变后的数据。

    - 引擎会把新数据写入内存，再把这个更新操作记入redo log并通知执行器，redo log此时处于prepare状态随时可以提交。

    - 执行器把这个操作逻辑写入bin log。

    - 执行器再调用引擎的提交接口，更新操作完成。
