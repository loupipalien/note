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
