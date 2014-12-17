##External resources 外部资源

流程定义保存在 Activiti 所支持的数据库中。当使用服务任务、执行监听器或者从 Activiti 配置文件中配置的 Spring beans 时，流程定义能够引用这些委托类。 这些类或者 Spring 配置文件对于所有流程引擎中可能执行的流程定义必须是可用的。

###Java classes

当流程实例被启动的时候，在流程中被使用的所有自定义类（例如：服务任务中使用的 JavaDelegates、事件监听器、任务监听器,...）应该存在与流程引擎的类路径下。

然后，在部署业务文档时，这些类不必都存在于类路径下。当使用 Ant 部署一个新的业务文档时，这意味着你的委托类不必存在与类路径下。

当你使用示例设置并添加你自定义的类，你应该添加包含自定义类的 jar 包到 activitiexplorer 控制台或者 activiti-rest 的 webapp lib文件夹中。以及不要忽略包含你自定义类的依赖关系（如果有）。另外，你还可以包含你自己的依赖添加到你的Tomcat容器的安装目录中的 ${tomcat.home}/lib。

###Using Spring beans from a process 在流程中使用 Spring beans

当表达式或者脚本使用 Spring beans 时，这些 beans 对于引擎执行流程定义时必须是可用的。如果你将要构建你自己的 web 应用并且按照Spring 集成这一章中描述那样在你的应用上下文配置流程引擎，这个看上去非常的简单。但是要记住，如果你也在使用 Activiti rest web
应用，那么也应该更新 Activiti rest web应用的上下文。 你可以把在 activiti-rest/lib/activiti-cfg.jar 文件中的 activiti.cfg.xml 替换成你的 Spring 上下文配置的 activiti-context.xml文件。

###Creating a single app 创建独立应用

你可以考虑把 Activiti rest web 应用加入到你的 web 应用之中，因此，就仅仅只需要配置一个 ProcessEngine，从而不用确保所有的流程引擎的所有委托类在类路径下面并且是否使用正确的 spring 配置。

