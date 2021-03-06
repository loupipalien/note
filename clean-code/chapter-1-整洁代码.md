### 整洁代码
阅读本书有两种原因: 第一, 你是个程序员; 第二, 你想成为更好的程序员

#### 要有代码
我们永远抛不掉代码, 因为代码呈现了需求的细节; 在某些层面上, 这些细节无法被忽略或抽象, 必须明确之; 将需求明确到机器可以执行的细节程度, 就是**编程**要做的事, 而这种规约正是代码

#### 糟糕的代码
我们都曾经瞟一眼自己亲手造成的混乱, 决定弃之不顾, 走向新的一天; 我们都曾经看到自己的烂程序居然能运行, 然后断言能运行的烂程序总比什么都没有强; 我们都曾经说过有朝一日再回头清理; 当然, 在那些日子里, 我们都没听过勒布朗 (LeBlanc) 法则: **稍后等于永不 (Later equals never)**

#### 混乱的代价
随着混乱的增加, 团队生产力也持续下降, 趋向于零; 当生产力下降时, 管理层就只有一件事可做了: 增加更多的人手到项目中, 期望提高生产力; 但是新人加入往往制造更多的混乱, 驱动生产力那端不断下降

##### 华丽的设计
最后, 开发团队造反了, 无法在这令人生厌的代码基础上做开发, 他们要求做全新的设计; 管理层不愿意投入资源完全重启炉灶, 但他们也不能否认生产力低得可怕, 治好同意开发者得要求,授权做一套看上去很美得华丽新设计; 于是就有了两只队伍做同一个需求... (但感觉一般项目基本都这样... 开发者挂项目好跳槽? 管理层新增项目好挣钱?)
花时间保持代码整洁不但有关效率, 还有关生存 (我一个搬砖? 惑)

##### 态度
多数经理想要知道实情, 即便他们看起来不喜欢实情; 多数经理想要好代码, 即便他们总是痴缠于进度; 他们会奋力卫护进度和需求; 那是他们该干的, 你则当以同等的热情卫护代码  
程序员遵从不了解混乱风险的经理的意愿, 是不专业的做法...

##### 谜题
程序员面临着一种基础价值谜题; 有那么几年经验的开发者都知道, 之前混乱拖了自己的后腿, 但开发者们背负期限的压力, 只好制造混乱, 简言之, 他们没花时间让自己做得更快  
真正的专业人士明白, 谜题的第二部分说错了; 制造混乱无助于赶上期限, 混乱只会立刻拖慢你, 叫你错过期限; 赶上期限的唯一方法 --- 就是时钟时刻保持代码整洁

##### 代码整洁的艺术
假设接受做的快的唯一方法是保持代码整洁的说法, 那么如何才能写出整洁的代码呢? 如果不明白整洁对代码有何意义, 尝试着去写整洁代码就毫无所益; 培养 "代码感", 编写整洁代码的程序员就像是艺术家, 他能用一系列变化把一块白板变作由优雅代码构成的系统

##### 什么是整洁代码
- Bjarne Stroustrup, C++ 语言发明者, <<C++ Programming Language>>  一书作者
我喜欢优雅和高效的代码; 代码逻辑应当直截了当, 叫缺陷难以隐藏; 尽量减少依赖关系, 使之便于维护; 依据某种分层战略完善错误处理代码; 性能调至最优, 省得引诱别人做没规矩的优化, 搞出一堆混乱来; 整洁的代码只做好一件事
- Grady Booch, <<Object Orientd Analysis and Design with Applications>> 一书作者
整洁的代码简单直接; 整洁的代码如同优美的散文; 整洁的代码从不隐藏设计者的意图, 充满了干净利落的抽象和直截了当的控制语句
- Dave Thomas, OTI 公司创始人, Eclipse 战略教父
整洁的代码应可由作者之外的开发者阅读和增补; 它应当有单元测试和验收测试; 它使用有意义的命名; 它只提供一种而非多种做一件事的途径; 它只有尽量少的依赖关系, 而且要明确地定义和提供清晰, 尽量少的 API; 代码应通过其字面表达含义, 因为不同的语言导致并非所有必需信息均可通过代码自身清晰表达
- Michael Feathers, <<Working Effectively with Legacy Code>> 一书作者
我可以列出我留意到的整洁代码的所有特点, 但其中有一条是根本性的; 整洁的代码总是看起来像是某位特别在意它的人写的; 几乎没有改进的余地; 代码作者什么都想到了, 如果你企图改进它, 总会回到原点, 赞叹某人留给你的代码 --- 全心投入的某人留下的代码
- Ron Jeffries, <<Extreme Programming Installed>> 和 <<Extreme Programming Adventures in C#>> 的作者
减少重复代码, 提高表达力, 提早构建简单抽象; 这就是我写整洁代码的方法
  - 能通过所有测试
  - 没有重复代码
  - 体现系统着那个的全部设计理念
  - 包括尽量小的实体, 比如类, 方法, 函数等
- Ward CunningHam, Wiki 发明者, eXtreme Programming 的创始人之一, Smalltalk 语言和面向对象的思想领袖
如果每个例程都让你感到深合己意, 那就是整洁代码, 如果代码让编程语言看起来像是专门为解决那个问题而存在的, 就可以称之为漂亮的代码

#### 思想流派
书中很多建议都存在争议, 或许你并不完全同意这些建议; 你可能会强烈反对其中一些建议, 这样挺好的; 我们不能要求做最终权威; 另一方面, 书中所列的建议, 乃是我们长久苦思, 从数十年的从业经验和无数尝试与错误中得来; 无论你同意与否, 如果你没看到或是不尊敬我们的观点, 就真该为自己害臊 :thumbsup::thumbsup::thumbsup:

#### 我们是作者
在写代码的过程中, 实际上读写花费时间比例超过 10 : 1, 不读周边代码的话就无法写代码; 编写代码的难度, 取决于读周边代码的难度; 要想干得快, 要想早点做完, 要想轻松写代码, 先让代码易读吧

#### 童子军军规
**让营地比你来时更干净**

#### 前传与原则
...

#### 小结
...
