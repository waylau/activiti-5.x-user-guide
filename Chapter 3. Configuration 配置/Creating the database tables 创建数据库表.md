##Creating the database tables 创建数据库表

下面是创建数据库表最简单的办法：

* 把 activiti-engine 的 jar 放到 classpath 下
* 添加对应的数据库驱动
* 把 Activiti 配置文件 (activiti.cfg.xml) 放到 classpath下， 指向你的数据库（参考[数据库配置](Database configuration 数据库配置.md)章节）
* 执行 DbSchemaCreate 类的 main 方法

不过，一般情况只有数据库管理员才能执行 DDL 语句。 在生产环境，这也是最明智的选择。SQL DDL语句可以从 Activiti 下载页或 Activiti发布目录里找到，在 database 子目录下。 脚本
也包含在引擎的 jar 中(activiti-engine-x.jar)， 在
org/activiti/db/create包下（drop 目录里是删除语句）。 SQL 文件的命名方式如下

	activiti.{db}.{create|drop}.{type}.sql

其中 db 是 [支持的数据库](Supported databases 支持的数据库.md)， type 是

* engine: 引擎执行的表。必须。
* identity: 包含用户，群组，用户与组之间的关系的表。 这些表是可选的，只有使用引擎自带的默认身份管理时才需要。
* history: 包含历史和审计信息的表。可选的：历史级别设为 none 时不会使用。 注意这也会引用一些需要把数据保存到历史表中的功能（比如任务的评论）。

MySQL用户需要注意： 版本低于 5.6.4 的 MySQL 不支持毫秒精度的 timestamp 或 date 类型。 更严重的是，有些版本会在尝试创建这样一列时抛出异常，而有些版本则不会。 在执行自动创建/更新时，引擎会在执行过程中修改 DDL。 当使用 DD L时，可以选择通用版本和名为 mysql55的文件。 （它适合所有版本低于5.6.4的情况）。 后一个文件会将列的类型设置为没有毫秒的情况。

总结一下，对于MySQL版本会执行如下操作

* <5.6: 不支持毫秒精度。可以使用 DDL 文件（包含 mysql55 的文件）。可以实现自动创建/更新。
* 5.6.0 - 5.6.3: 不支持毫秒精度。无法自动创建/更新。建议更新到新的数据库版本。如果真的需要的话，也可以使用 mysql 5.5。
* 5.6.4+:支持毫秒精度。可以使用DDL文件（默认包含 mysql 的文件）。可以实现自动创建、更新。

注意对于已经更新了 MySQL 数据库，而且 Activiti 表已经创建/更新的情况， 必须手工修改列的类型。

