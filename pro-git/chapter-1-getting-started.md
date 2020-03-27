### 起步

#### 关于版本控制
版本控制是一种记录一个或若干文件内容变化, 以便将来查阅特定版本修订情况的系统

##### 本地版本控制系统 (LVCS)
![本地版本控制](https://git-scm.com/book/en/v2/images/local.png)  
其中最流行的一种叫做 RCS, 现今许多计算机系统上都还看得到它的踪影; RCS 的工作原理是在硬盘上保存补丁集 (补丁是指文件修订前后的变化); 通过应用所有的补丁, 可以重新计算出各个版本的文件内容

##### 集中化的版本控制系统 (CVCS)
![集中化的版本控制](https://git-scm.com/book/en/v2/images/centralized.png)  
 如 CVS、Subversion 以及 Perforce 等, 都有一个单一的集中管理的服务器, 保存所有文件的修订版本, 而协同工作的人们都通过客户端连到这台服务器, 取出最新的文件或者提交更新

##### 分布式版本控制系统 (DVCS)
![分布式版本控制](https://git-scm.com/book/en/v2/images/distributed.png)  
如 Git, Mercurial, Bazaar 以及 Darcs 等, 客户端并不只提取最新版本的文件快照, 而是把代码仓库完整地镜像下来, 包括完整的历史记录; 这么一来, 任何一处协同工作用的服务器发生故障, 事后都可以用任何一个镜像出来的本地仓库恢复; 因为每一次的克隆操作, 实际上都是一次对代码仓库的完整备份

#### Git 简史
Linux 内核开源项目有着为数众多的参与者, 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上 (1991－2002年间); 到 2002 年, 整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码  
到了 2005 年, 开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束, 他们收回了 Linux 内核社区免费使用 BitKeeper 的权力; 这就迫使 Linux 开源社区 (特别是 Linux 的缔造者 Linus Torvalds) 基于使用 BitKeeper 时的经验教训, 开发出自己的版本系统; 他们对新的系统制订了若干目标
- 速度
- 简单的设计
- 对非线性开发模式的强力支持 (允许成千上万个并行开发的分支)
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目 (速度和数据量)

#### Git 是什么
尽管 Git 用起来与其它的版本控制系统非常相似, 但它在对信息的存储和认知方式上却有很大差异, 理解这些差异将有助与避免使用中的困惑

##### 直接记录快照而非差异比较
Git 和其它版本控制系统 (包括 Subversion 和近似工具) 的主要差别在于 Git 对待数据的方法; 从概念上来说, 其它大部分系统以文件变更列表的方式存储信息, 这类系统 (CVS, Subversion, Perforce, Bazaar 等等) 将它们存储的信息看作是一组基本文件和每个文件随时间逐步累积的差异 (它们通常称作 **基于差异 (delta-based)** 的版本控制)
![存储每个文件与初始版本的差异](https://git-scm.com/book/en/v2/images/deltas.png)  
Git 不按照以上方式对待或保存数据; 反之, Git 更像是把数据看作是对小型文件系统的一系列快照; 在 Git 中, 每当你提交更新或保存项目状态时, 它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引; 为了效率, 如果文件没有修改, Git 不再重新存储该文件, 而是只保留一个链接指向之前存储的文件; Git 对待数据更像是一个 **快照流**
![存储项目随时间改变的快照](https://git-scm.com/book/en/v2/images/snapshots.png)

##### 近乎所有操作都是本地执行
在 Git 中的绝大多数操作都只需要访问本地文件和资源, 一般不需要来自网络上其它计算机的信息, 因为在本地磁盘上就有项目的完整历史, 所以大部分操作看起来瞬间完成

##### Git 保证完整性
Git 中所有的数据在存储前都计算校验和, 然后以校验和来引用, 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容; 这个功能建构在 Git 底层, 是构成 Git 哲学不可或缺的部分; 若在传送过程中丢失信息或损坏文件 Git 就能发现  
Git 用以计算校验和的机制叫做 SHA-1 散列 (hash, 哈希); 这是一个由 40 个十六进制字符 (0-9 和 a-f) 组成的字符串, 基于 Git 中文件的内容或目录结构计算出来; SHA-1 哈希看起来是这样
```
24b9da6552252987aa493b52f8696cd6d3b00373
```

##### Git 一般只添加数据
执行的 Git 操作, 几乎只往 Git 数据库中 **添加** 数据; 很难让 Git 执行任何不可逆操作, 或者让它以任何方式清除数据; 同别的 VCS 一样, 未提交更新时有可能丢失或弄乱修改的内容, 但是一旦提交快照到 Git 中, 就难以再丢失数据

##### 三种状态
Git 有三种状态, 被 Git 跟踪的文件可能处于其中之一: 已提交 (committed), 已修改 (modified) 和已暂存 (staged)
- 已修改表示修改了文件, 但还没保存到数据库中
- 已暂存表示对一个已修改文件的当前版本做了标记, 使之包含在下次提交的快照中
- 已提交表示数据已经安全地保存在本地数据库中

这会让 Git 管理的项目拥有三个阶段: 工作区, 暂存区, Git 目录
![工作目录, 暂存区域, Git 仓库](https://git-scm.com/book/en/v2/images/areas.png)  
工作区是对项目的某个版本独立提取出来的内容, 这些从 Git 仓库的压缩数据库中提取出来的文件, 放在磁盘上供你使用或修改  
暂存区是一个文件, 保存了下次将要提交的文件列表信息, 一般在 Git 仓库目录中; 按照 Git 的术语叫做 `索引`, 不过一般说法还是叫 `暂存区`  
Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方; 这是 Git 中最重要的部分, 从其它计算机克隆仓库时, 复制的就是这里的数据  
基本的 Git 工作流程如下:
- 在工作区中修改文件
- 将想要下次提交的更改选择性地暂存, 这样只会将更改的部分添加到暂存区
- 提交更新, 找到暂存区的文件, 将快照永久性存储到 Git 目录

如果 Git 目录中保存着特定版本的文件, 就属于 **已提交** 状态; 如果文件已修改并放入暂存区, 就属于 **已暂存** 状态; 如果自上次检出后, 作了修改但还没有放到暂存区域, 就是 **已修改** 状态

#### 命令行
Git 有多种使用方式, 可以使用原生的命令行模式, 也可以使用 GUI 模式; 在命令行模式下能执行 Git 的 **所有** 命令

#### 安装 Git
TODO

#### 初次运行 Git 前的配置
Git 自带一个 `git config` 的工具来帮助设置控制 Git 外观和行为的配置变量; 这些变量存储在三个不同的位置
- `/etc/gitconfig` 文件
包含系统上每一个用户及他们仓库的通用配置; 如果在执行 `git config` 时带上 `--system` 选项, 那么它就会读写该文件中的配置变量 (由于是系统配置文件, 因此需要管理员或超级用户权限来修改它)
- `~/.gitconfig` 或 `~/.config/git/config` 文件
只针对当前用户; 可以传递 `--global` 选项让 Git 读写此文件, 这会对当前用户的系统上 **所有** 的仓库生效
- 当前使用仓库的 Git 目录中的 `config` 文件 (即 `.git/config`)
针对该仓库; 可以传递 `--local` 选项让 Git 强制读写此文件, 虽然默认情况下用的就是它 (当然需要进入某个 Git 仓库中才能让该选项生效)

可以通过以下命令查看所有的配置以及它们所在的文件
```
git config --list --show-origin
```

##### 用户信息
安装完 Git 之后, 要做的第一件事就是设置你的用户名和邮件地址; 这一点很重要, 因为每一个 Git 提交都会使用这些信息, 它们会写入到你的每一次提交中, 不可更改
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

##### 文本编辑器
如果想使用不同的文本编辑器, 例如 Emacs, 可以这样做
```
$ git config --global core.editor emacs
```

##### 检查配置信息
如果想要检查你的配置, 可以使用 `git config --list` 命令来列出所有 Git 当时能找到的配置  
另外, 可以通过输入 `git config <key>` 来检查 Git 的某一项配置
```
$ git config user.name
John Doe
```

#### 获取帮助
你使用 Git 时需要获取帮助, 有三种等价的方法可以找到 Git 命令的综合手册 (manpage)
```
git help <verb>
$ git <verb> --help
$ man git-<verb>
```
此外, 如果你不需要全面的手册, 只需要可用选项的快速参考, 那么可以用 `-h` 选项获得更简明的输出
```
$ git add -h
usage: git add [<options>] [--] <pathspec>...

    -n, --dry-run         dry run
    -v, --verbose         be verbose

    -i, --interactive     interactive picking
    -p, --patch           select hunks interactively
    -e, --edit            edit current diff and apply
    -f, --force           allow adding otherwise ignored files
    -u, --update          update tracked files
    --renormalize         renormalize EOL of tracked files (implies -u)
    -N, --intent-to-add   record only the fact that the path will be added later
    -A, --all             add changes from all tracked and untracked files
    --ignore-removal      ignore paths removed in the working tree (same as --no-all)
    --refresh             don't add, only refresh the index
    --ignore-errors       just skip files which cannot be added because of errors
    --ignore-missing      check if - even missing - files are ignored in dry run
    --chmod (+|-)x        override the executable bit of the listed files
```

#### 总结
TODO

>**参考:**
- [起步](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)
