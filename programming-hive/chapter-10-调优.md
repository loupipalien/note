### 调优
HiveQL 是一种声明式语言, 用户会提交声明式的查询, 而 Hive 会将其转换为 MapReduce job

#### 使用 EXPLAIN
```
explain select sum(number) from onecol;

ABSTRACT SYNTAX TREE:
(TOK_QUERY
  (TOK_FROM (TOK_TABREF (TOK_TABNAME onecol)))
  (TOK_INSERT (TOK_DESTINATION (TOK_DIR TOK_TMP_FILE))
    (TOK_SELECT (TOK_SELEXPR (TOK_FUNCTION sum (TOK_TABLE_OR_COL number))))
  )
)
```
正如以上抽象语法树所示, Hive 实际上会将输出写入一个临时文件中  
一个 Hive 任务会包含有一个或多个 stage (阶段), 不同的 stage 间会存在着依赖关系, 越复杂的查询通常将会引入越多的 stage, 通常 stage 越多就需要越多的时间来完成; 一个 stage 可以是一个 MapReduce 任务, 也可以是一个抽样阶段, 或者一个合并阶段, 或者一个 limit 阶段, 以及 Hive 需要的其他某个任务的一个阶段; 默认情况下, Hive 会一次只执行一个 stage (阶段)
```
STAGE DEPENDENCIES
  Stage-0 is a root stage
  Stage-1 is a root stage
```
STAGE PLANS 部分比较冗长也比较复杂; Stage-1 包含了这个 job 的大部分处理过程, 而且会触发一个 MapReduce job; TableScan 以这个表作为输入, 然后产生一个只有字段 number 的输出, Group By Operator 会应用到 sum(number), 然后会产生一个输出字段 \_col0 (这是临时结果字段按规则起的临时字段名); 这些都发生在 job 的 map 处理阶段过程, 也就是 Map Operator Tree 下
```
STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Alias -> Map Operator Tree:
        onecol
          TableScan
            alias: onecol
              Select Operator
                expressions:
                    expr: number
                    type: int
                outputColumnNames: number
                Group By Operator
                  aggregations:
                      expr: sum(number)
                  bucketGroup: false
                  mode: hash
                  outputColumnNames: _col0
                  Reduce Output Operator
                    sort order:
                    tag: -1
                    value expressions:
                        expr: _col0
                        type: bigint
```
在 reduce 过程这边, 也就是 Reduce Operator Tree 下可以看到相同 的 Group By Operator, 但是这次其应用到的是对 \_col0 字段进行 sum 操作, 最后在 reducer 中可以看到 File Output Operator, 其说明了输出结果将是文本格式, 是基于字符串的输出格式: HiveIgnoreKeyTextOutputFormat:
```
Reducer Operator Tree:
  Group By Operator
    aggregations:
        expr: sum(VALUE._col0)
    bucketGroup: false
    mode: mergepartial
    outputColumnNames: _col0
    Select Operator
      expressions:
          expr: _col0
          type: bigint
      outputColumnNames: _col0
      File Output Operator
        compressed: false
        GlobalTableId: 0
        table:
            input format: otg.apche.hadoop.mapred.TextInputFormat
            output format: org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
```
因为这个 job 没有 limit 语句, 因此 Stage-0 阶段是一个没有任何操作的阶段
```
Stage: Stage-0
  Fetch Operator
    limit -1
```

#### EXPLAIN EXTENDED
使用 explain extended 语句可以产生更多的输出信息

#### 限制调整
在大多数情况下 limit 语句还是需要执行整个查询语句, 然后再返回部分结果的, 因为这种情况通常是浪费的, 所以应该尽可能避免出现这种情况; 当 Hive 将 hive.limit.optimize.enable = true, 使用 limit 语句时可以对源数据进行抽样, 此外当此参数开启时, hive.limit.row.max.size 可以设置至少保证可以返回多少条, hive.limit.optimize.limit.file 可以设置最多可以抽样多少个文件; 此功能的缺点是, 有可能输入中有用的数据永远都不会被用到

#### JOIN 优化
TODO

#### 本地模式
有时 Hive 的输入数据量是非常小的, 在这种情况下为查询触发执行任务的时间消耗可能会比实际 job 的执行时间要多得多; 对于大多数这种情况, Hive 可以通过本地模式处理所有的任务, 对于小数据集执行时间会被明显缩短; hive.exec.more.local.auto = true, 会尽可能的将执行转化为本地模式

#### 并行执行
Hive 会将一个查询转换为一个或多个查询阶段, 这个阶段可以是 MapReduce 阶段, 抽样阶段, 合并阶段, limit 阶段, 或者 Hive 执行过程中可能需要的其他阶段; 默认情况下, Hive 一次只会执行一个阶段, 但某个特定的 job 可能包含众多的阶段, 而这些阶段可能并非完全互相依赖, 也就是说有些阶段是可以并行执行的, 这样可能使得整个 job 的执行时间缩短; 通过设置参数 hive.exec.parallel = true 就可以开启并发执行

#### 严格模式
Hive 提供一个严格模式, 可以防止用户执行哪些可能产生意想不到的不好影响的查询, 通过设置属性 hive.mapred.mode = strict 可以禁止 3 中类型的查询
- 对于分区表, 除非 where 语句中含有分区字段过滤条件来限制数据范围, 否则不允许执行; 即不允许用户扫描所有分区, 进行这个限制的原因是分区表通常都拥有非常大的数据集, 而且数据增加迅速, 没有进行分区限制的查询可能会消耗令人不可接受的巨大资源来处理这个表
- 对于使用了 order by 语句的查询, 要求必须使用 limit 语句, 因为 order by 为了执行排序过程会将所有的结果数据分发到同一个 reducer 中进行处理, 强制要求增加这个 limit 语句可以防止 reducer 额外执行很长的时间
- 最后一种情况是限制笛卡儿积的查询, 在关系型数据库章用户可能期望在执行 join 查询的时候不使用 on 语句而是 where 语句, 关系型数据库的执行优化器可以高效的将 where 语句转化成那个 on 语句; 不幸的是 Hive 并不会执行这种优化, 因此如果表足够大, 那么这个查询就会出现不可控的情况

#### 调整 mapper 和 reducer 的个数
Hive 通过将查询划分成一个或多个的 MapReduce 任务达到并行的目的, 每个任务都可能具有多个 mapper 和 reducer 任务, 确定最佳的 mapper 个数和 reducer 个数取决于多个变量, 例如输入的数据量的大小以及对这些数据执行的操作类型等; 保持平衡性是非常有必要的, 如果有太多的 mapper 和 reducer 任务, 就会导致启动阶段, 调度和运行 job 运行过程中产生过多的开销, 而如果设置的数量太少, 那么就可能没有充分利用好集群内在的并行性  
以下是三个可以直接或间接的调整 job 的 reducer 个数的参数
```
# 设置每个 reducer 处理多少字节的数据
hive.exec.reducers.bytes.per.reducer = 256000000
# 每个 job 最多可以起多少个 reducer
hive.exec.reducers.max = 1009
# job 会起多少个 reducer
mapred.reduce.tasks = -1
```

#### JVM 重用
JVM 重用是 Hadoop 的调优参数, 但对 Hive 的性能有非常大的影响, 特别是对于很难避免小文件的场景或 task 特别多的场景; Hadoop 的默认配置通常是派生 JVM 来执行 map 和 reduce 任务的, 这时 JVM 的启动过程可能会造成相当大的开销, 尤其是 job 包含有成千上万个 task 任务的时候; JVM 重用可以使得 JVM 实例在同一个 job 中重新使用 N 次, 此参数 mapred.job.reuse.jvm.num.tasks = n 可以在 Hadoop 的 mapred-site.xml 的文件中配置  
这个功能带来的代价是, 开启 JVM 重用会一直占用使用的 task 插槽以便进行重用, 直到完成任务后才释放; 如果某个 "不平衡的" job 中有某几个 reduce task 执行的时间比其他 reduce task 消耗的时间多的多的话, 那么保留的插槽就会一直空闲却无法被其他 job 使用, 直到所有的 task 都结束了才会释放

#### 索引
索引可以用来加速含有 GROUP BY 语句查询的计算速度

#### 动态分区调整
动态分区 insert 语句可以通过简单的 select 语句先分区表中创建很多新的分区, 这是一个非常强大的功能, 但如果分区数过多, 那么就会在系统中产生大量的输出控制流, 对于 Hadoop 来说, 这种情况并不是常见的使用场景, 这样通常会创建非常多的文件, 然后像其写入数据; Hive 可以通过配置限制动态分区插入允许创建的分区数在 1000 个左右, 虽然太对的分区对于表来说并不好, 但通常还是会将这个值设置的更大以便执行; 通常 Hive 设置动态分区为严格模式 (hive.exec.dynamic.partition.mode = strict), 但开启严格模式的时候必须保证至少有一个分区是静态的; 还可以设置 hive.exec.max.dynamic.partitions = 500000 和 hive.exec.max.dynamic.partitions.pernode = 50000 来限制查询可以创建的最大分区数; hdfs-site.xml 配置文件中 dfs.datanode.max.xcievers = 8192 可以控制 DataNode 一次可以打开的文件数的个数, 这个配置必须在 DataNode 节点上的配置文件中, 并且需要重启集群才可生效

#### 推测执行
推测执行是 Hadoop 中的一个功能, 其可以触发一些重复性的任务 (task), 尽管这样因对重复数据进行计算而导致消耗更多的计算资源, 不过这个功能的目标是通过加快获取单个 task 的结果以及进行侦测将执行慢的 TaskTracker (ResourceNode) 加入到黑名单的方式来提供整体的任务执行效率; Hadoop 推测执行的功能在 hdfs-site.xml 文件的以下两个参数控制: mapred.map.tasks.speculative.execution = true 和  mapred.reduce.tasks.speculative.execution = true  
在 Hive 端也提供了配置项来控制 reduce-side 的推测执行: hive.mapred.reduce.tasks.speculative.execution = true; 关于调优的推测执行较难给出建议, 如果用户对于运行时的偏差非常敏感的话, 建议将其功能关闭, 因为用户如果输入了数据量很大而需要执行很长时间的 map 或者 reduce 的话, 那么启动推测执行造成的浪费是非常巨大的

#### 单个 MapReduce 中多个 GROUP BY
另外一个特别的优化试图将查询中的多个 GROUP BY 操作组装到单个 MapReduce 任务中, 如果想启动这个优化, 那么需要一组共用的 GROUP BY 键, 配置参数为: hive.multigroupby.singlemr

#### 虚拟列
Hive 提供了用于将要进行划分的输入文件名, 用于文件中块内偏移量的虚拟列, 当 Hive 产生了非预期的或 null 的返回结果时, 可以通过这些虚拟列诊断查询, 通过查询这些字段, 用户可以查看到哪个文件甚至哪行数据导致出现问题
```
set hive.exec.rowoffset = true;
select INPUT__FILE__NAME, BLOCK__OFFSET__INSIDE__FILE, line from hive_text
where line like '%hive%' limit 2
```
Hive 还提供了文件的行偏移量
```
set hive.exec.rowoffset = true;
select INPUT__FILE__NAME, BLOCK__OFFSET__INSIDE__FILE, ROW__OFFSET__INSIDE__BLOCK from hive_text 
where line like '%hive%' limit 2
```
