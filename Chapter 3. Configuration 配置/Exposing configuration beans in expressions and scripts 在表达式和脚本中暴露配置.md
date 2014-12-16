##Exposing configuration beans in expressions and scripts 在表达式和脚本中暴露配置

默认，activiti.cfg.xml 和你自己的 Spring 配置文件中所有 bean 都可以在表达式和脚本中使用。 如果你想限制配置文件中的 bean 的可见性， 可以配置流程引擎配置的 bean 配置。
ProcessEngineConfiguration 的 beans 是一个 map。当你指定了这个参数， 只有包含这个 map 中的 bean 可以在表达式和脚本中使用。 通过在map 中指定的名称来决定暴露的 bean。