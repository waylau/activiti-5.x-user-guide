##Expressions 表达式
 
Activiti 使用 UEL 处理表达式。UEL 即 Unified Expression Language (统一表达式语言)，它是 EE6 规范的一部分（参考 [EE6规范](http://docs.oracle.com/javaee/6/tutorial/doc/gjddd.html)）。为了在所有运行环境都支持最新 UEL 的所有功能，我们使用了一个 JUEL 的修改版本。

表达式可以用在很多场景下，比如 [Java 服务任务](http://www.activiti.org/userguide/index.html#bpmnJavaServiceTaskXML)，[执行监听器](http://www.activiti.org/userguide/index.html#executionListeners)，[任务监听器](http://www.activiti.org/userguide/index.html#taskListeners)和[条件流](http://www.activiti.org/userguide/index.html#conditionalSequenceFlowXml)。 虽然有两重表达式，值表达式和方法表达式，Activiti进行了抽象，所以两者可以同样使用在需
要表达式的场景中。

* Value expression(值表达式)：解析为值。默认，所有流程变量都可以使用。所有 spring bean（spring环境中）也可以使用在表达式中。 一些实例：

	${myVar}
	${myBean.myProperty}

* Method expression(方法表达式)：调用一个方法，使用或不使用参数。当调用一个无参数的方法时，记得在方法名后添加空的括号（以区分值表达式）。 传递的参数可以是字符串也可以是表达式，它们会被自动解析。例子：

	${printer.print()}
	${myBean.addNewOrder('orderName')}
	${myBean.doSomething(myVar, execution)}

注意这些表达式支持解析原始类型（包括比较），bean，list，数组和map。

在所有流程实例中，表达式中还可以使用一些默认对象：

* execution：DelegateExecution 提供外出执行的额外信息。
* task：DelegateTask 提供当前任务的额外信息。注意，只对任务监听器的表达式有效。
* authenticatedUserId：当前登录的用户id。如果没有用户登录，这个变量就不可用。

想要更多具体的使用方式和例子，参考 [spring 中的表达式](../Chapter 5. Spring integration 集成 Spring/Expressions 表达式.md)，[Java 服务任务](http://www.activiti.org/userguide/index.html#bpmnJavaServiceTaskXML)，[执行监听器](http://www.activiti.org/userguide/index.html#executionListeners)，[任务监听器](http://www.activiti.org/userguide/index.html#taskListeners)和[条件流](http://www.activiti.org/userguide/index.html#conditionalSequenceFlowXml)。

