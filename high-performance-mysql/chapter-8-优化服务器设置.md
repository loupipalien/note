### 优化服务器设置
MySQL 有大量可以修改的参数, 但不应该随便去修改, 通常只需要把基本的项配置正确 (大部分情况下只有很少一些参数是真正重要的), 应该将更多的时间花在 schema 的优化, 索引, 以及查询上; 正确的配置了 MySQL 的基本配置项后, 再花力气去修改其他配置项的收益通常就比较小了

#### MySQL 配置的工作原理
MySQL 可以从命令行参数和配置文件获取配置信息; 在类 UNIX 系统中, 配置文件的位置一般在 `/etc/my.cnf` 或者 `/etc/mysql/my.cnf`   
配置文件通常分成多个部分, 每个部分的开头是一个用方括号括起来的分段名称; MySQL 程序通常读取跟它同名的分段部分, 许多客户端还会读取 client 部分, 这时一个存放公用配置的地方; 服务器通常读取 mysqld 这一段, 所以一定要确认把配置项放在了文件正确的分段中, 否则配置是不会生效的

##### 语法, 作用域, 动态性
配置项设置都使用小写, 单词之间用下划线或者横线隔开; 例如
```
/usr/sbin/mysqld --autu-increment-offset=5;
/usr/sbin/mysqld --autu_increment_offset=5;
```
配置项可以有多个作用域, 有些设置是服务器级的 (全局作用域), 有些对每个连接是不同的 (会话作用域), 剩下的一些是对象级的; 许多会话级变量和全局变量相等, 可以认为是默认值, 如果改变会话变量, 它只影响改动的当前连接, 当连接关闭时所有参数变更都会失效; 下面语句展示了动态改变了 `sort_buffer_size` 的会话值和全局值的不同方式
```
SET sort_buffer_size = value;
SET GLOBAL sort_buffer_size = value;
SET @@sort_buffer_size = value;
SET @@session.sort_buffer_size = value;
SET @@global.sort_buffer_size = value;
```

##### 设置变量的副作用
TODO

##### 入门
TODO

##### 通过基准测hi迭代优化
TODO

#### 什么不该做
TODO

#### 创建 MySQL 配置文件
