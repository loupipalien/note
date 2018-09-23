### 其他文件格式和压缩方法
Hive 的一个独特的功能是: Hive 不会强制要求将数据转换成特定的格式才可以使用, Hive 利用 Hadoop 的 InputFormat API 来从不同的数据源读取数据, 同样的 OutputFormat API 也可以将数据写成不同的格式  
Hadoop 的 job 通常是 I/O 密集型而不是 CPU 密集型的, 如果是这样压缩可以提高性能, 但用户的 job 是 CPU 密集型的话, 那么使用压缩可能会降低执行的效率

#### 确定安装编/解码器
基于所使用的 Hadoop 版本, 会提供不同的编/解码器, 使用以下语句可以查看
```
set io.compression.codecs;
```

#### 选择一种压缩编/解码器
最新的 Hadoop 版本已经内置了 GZIP 和 BZip2 压缩方案, 其次也支持自行整合 Snappy, LZO 等压缩方案; 多种压缩方案在选择时, 通常在在压缩/解压缩速度和压缩率进行权衡  
BZip2 压缩率最高, 但同时需要消耗更多的 CPU 开销, GZIP 是压缩率和压缩/解压缩速度的下一个选择, 如果磁盘空间利用率和 I/O 开销需要考虑的话, 则这两种方案都是非常有吸引力的; LZO 和 Snappy 压缩率相比前面两种要小但是压缩解压缩的速度要快, 特别是解压缩过程, 如果相对与磁盘空间和 I/O 开销, 频繁读取数据所需的解压缩速度更重要的话, 则这两种是不错的选择  
另一个需要考虑的因素是压缩格式的文件是否是可以分割的, MapReduce 需要将输入文件切割成多个划分, 其中每个划分会被分发到一个 map 进程中处理, 只有当 Hadoop 知道文件中记录的边界时才可以进行这样的分割; 对于文本文件, 每一行都是一条记录, 但是 GZIP 和 Snappy 将这些边界信息掩盖掉了, BZip2 和 LZO 提供了块级别的压缩, 也就是每个块中都含有完整的记录信息, 因此 Hadoop 可以在这些边界级别对这些文件划分; 虽然 GZIP 和 Snappy 压缩的文件不可以划分, 但是并不能因此而排除它们

#### 开启中间压缩
对中间数据进行压缩可以减少 job 中 map 和 reduce task 间的数据传输量; 对于中间数据压缩, 选择一个低 CPU 开销的编/解码器要比选择一个压缩率高的编/解码器要重要的多, 将 hive.exec.compress.intermediate = true 可开启中间结果压缩  
对于其他 Hadoop job 来说控制中间数据压缩的属性是 mapred.compress.map.output, Hadoop 压缩默认的编/解码器是 DefaultCodec, 可以通过修改属性 mapred.map.output.compression.codec 来设置, 这是一个 Hadoop 的配置项, 在 mapred-site.xml 配置文件中设置

#### 最终的输出结果压缩
当 Hive 将输出写入到表中的时候, 输出内容同样可以进行压缩, 默认是非压缩的纯文本文件, 可以设置属性 hive.exec.compress.output = true 来开启  
对于其他 Hadoop job 开启最终输出结果压缩功能的属性是 mapred.output.compress, 开启后需要设置 mapred.output.compression.codec 为其指定一个编码器, 对于输出文件使用 GZIP 压缩是不错的选择, 通常可以大幅度降低文件大小, 但需要注意的是 GZIP 文件对于后面的 MapReduce job 而言是不可分割的

#### sequence file 存储格式
压缩文件确实能节约存储空间, 但是在 Hadoop 中存储裸压缩文件的一个缺点是, 通常这些文件是不可分割的, 可分割的文件可以划分为多个部分, 由多个 mapper 并行处理, 大多数压缩文件是不可分割的, 也就是说只能从头读到尾; Hadoop 所支持的 sequence file 存储格式可以将一个文件划分为多个块, 然后采用一种可分割的方式对块进行压缩; sequence file 提供了 3 种压缩方式: NONE, RECORD, BLOCK; 默认是 RECORD 级别的, 但通常来说 BLOCK 级别压缩性能更好且可以分割; 与其他的压缩属性一样, 此属性并非 Hive 所特有的, 可以在 Hadoop 的 hdfs-site.xml 文件中或者 Hive 的 hive-site.xml 中配置

#### 使用压缩实践
TODO  
/bin/zcat 可以查看 .gz 文件  
dfs -text 可以查看 sequence file 文件

#### 存档分区
Hadoop 中有一种存储格式为 HAR, 也就是 Hadoop Archive (Hadoop 归档文件) 的简写; 一个 HAR 文件就像在 HDFS 文件系统中的一个 TAR 文件一样是一个单独的文件, 不过其内部可以存放多个文件和文件夹; 在一些使用场景下, 一些旧的文件夹和文件被访问大概率很低, 但是又有非常多的文件, 这消耗了较多的 NameNode 内存来管理; 将其归档成一个巨大的 HAR 文件可以减轻 NameNode 压力, 但同时可以被 Hive 访问; 其缺点是 HAR 文件查询效率不高, 同时 HAR 文件是非压缩的, 也并不会节约存储空间

#### 压缩: 包扎
