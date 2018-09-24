### 函数
用户自定义函数 (UDF) 是一个允许用户扩展 HiveQL 的强大的功能, 可以将用户自定义函数加入会话中, 就可以像内置函数一样使用, 此外 UDF 是在 Hive 产生的相同的 task 进程中执行的, 因此可以高效的执行

#### 发现和描述函数
```
# 展示所有函数
show functions;
# 展示函数 concat 的描述
desc funtion concat;
# 展示函数 concat 的扩展描述
desc funtion extended concat;
```

#### 调用函数
```
select concat(column1, column2) as x from table;
```

#### 标准函数
用户自定义函数 (UDF) 在狭义上的概念上还表示以一行数据中的一列或多列数据作为参数然后返回结果是一个值的函数, UDF 也可以返回一个复杂对象, 如 array, map, object 等

#### 聚合函数
所有的聚合函数, 用户自定义函数, 内置函数都统称为用户自定义聚合函数 (UDAF)

#### 表生成函数
TODO

#### 一个通过日期计算其星座的 UDF
TODO

#### UDF 和 GenericUDF
TODO

#### 不变函数
TODO

#### 用户自定义聚合函数
TODO

#### 用户自定义表函数
TODO

#### 在 UDF 中访问分布式缓存
UDF 是可以访问分布式缓存, 本地文件系统, 甚至分布式文件系统中的文件的, 不过这样的访问应该小心使用, 因为这种使用会显著降低执行效率

#### 以函数的方式使用注解
TODO

#### 宏命令
TODO
