##Activiti database setup 数据库安装

就像在一分钟版本示例里说过的，Activiti Explorer 默认使用 H2 内存数据库。 要让 Activiti 使用独立运行的 H2 数据库或者其他数据库，可以修改 Activiti Explorer web 应用 WEB-INF/
classes 目录下的 db.properties。

另外，注意 Activiti Explorer 自动生成了 demo 用的默认用户和群组，流程定义，数据模型。要想禁用这个功能，要修改  WEB-INF/classes 目录下的 属性文件。 禁用 demo 安装，可以设置所有属性为 false 。从代码中也可以看出，我们可以单独启用或禁用每一项功能。

	# demo data properties
	create.demo.users=true
	create.demo.definitions=true
	create.demo.models=true
	create.demo.reports=true
