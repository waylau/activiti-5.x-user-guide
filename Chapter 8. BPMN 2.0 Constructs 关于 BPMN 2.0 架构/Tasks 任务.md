##Tasks 任务

###User Task 用户任务

####Description 描述

用户任务用来设置必须由人员完成的工作。当流程执行到用户任务，会创建一个新任务，并把这个新任务加入到分配人或群组的任务列表中。

####Graphical notation 图形标记

用户任务显示成一个普通任务（圆角矩形），左上角有一个小用户图标

![](http://99btgc01.info/uploads/2014/12/bpmn.user.task.png)

####XML representation 内容

XML 中的用户任务定义如下。id 属性是必须的。name 属性是可选的。

	<userTask id="theTask" name="Important task" />                   
                                  

用户任务也可以设置描述。实际上所有 BPMN 2.0 元素 都可以设置描述。 添加 documentation 元素可以定义描述。

	<userTask id="theTask" name="Schedule meeting" >
	  <documentation>
	          Schedule an engineering meeting for next week with the new hire.
	  </documentation>

描述文本可以通过标准的java方法来获得:

	task.getDescription()

####Due Date 持续时间

任务可以用一个字段来描述任务的持续时间。可以使用查询 API 来对持续时间进行搜索， 根据在时间之前或之后进行搜索。

我们提供了一个节点扩展，在任务定义中设置一个表达式，这样在任务创建时就可以为它设置初始持续时间。表达式应该是 java.util.Date， java.util.String (ISO8601格式)，ISO8601 持续时间 (比如 PT50M )或null。 例如：你可以在流程中使用上述格式输入日期，或在前一个服务任务中计算一个时间。 这里使用了持续时间，持续时间会基于当前时间进行计算，再通过给定的时间段累加。 比如，使用"PT30M"作为持续时间，任务就会从现在开始持续 30 分钟。

	<userTask id="theTask" name="Important task" activiti:dueDate="${dateVariable}"/>

####User assignment 用户分配

用户任务可以直接分配给一个用户。 这可以通过 humanPerformer 元素定义。humanPerformer 定义需要一个 resourceAssignmentExpression 来实际定义用户。 当前，只支持 formalExpressions。

	<process ... >
	  
	  ...
	  
	  <userTask id='theTask' name='important task' >
	    <humanPerformer>
	      <resourceAssignmentExpression>
	        <formalExpression>kermit</formalExpression>
	      </resourceAssignmentExpression>
	    </humanPerformer>
	  </userTask>

只有一个用户可以坐拥任务的执行者分配给用户。 在 activiti 中，用户叫做执行者。 拥有执行者的用户不会出现在其他人的任务列表中， 只能出现执行者的个人任务列表中。

直接分配给用户的任务可以通过 TaskService 像下面这样获取：

	List<Task> tasks = taskService.createTaskQuery().taskAssignee("kermit").list();

任务也可以加入到人员的候选任务列表中。 这时，需要使用potentialOwner 元素。 用法和 humanPerformer 元素类似。注意它需要指定表达式中的每个项目是人员还是群组（引擎猜不出来）。

	<process ... >
	  
	  ...
	  
	  <userTask id='theTask' name='important task' >
	    <potentialOwner>
	      <resourceAssignmentExpression>
	        <formalExpression>user(kermit), group(management)</formalExpression>
	      </resourceAssignmentExpression>
	    </potentialOwner>
	  </userTask>

使用 potential owner 元素定义的任务，可以像下面这样获取 （使用TaskQuery 的发那个发与查询设置了执行者的任务类似）：

	List<Task> tasks = taskService.createTaskQuery().taskCandidateUser("kermit");

这会获取所有kermit为候选人的任务， 例如：表达式中包含 user(kermit)。 这也会获得所有分配包含 kermit 这个成员的群组 （比如，group(management)，前提是 kermit 是这个组的成员， 并且使用了activiti 的账号组件）。 用户所在的群组是在运行阶段获取的，它们可
以通过 IdentityService 进行管理。

如果没有显示指定设置的是用户还是群组，引擎会默认当做群组处理。所以下面的设置与使用 group(accountancy) 效果一样。

	<formalExpression>accountancy</formalExpression>

####Activiti extensions for task assignment 对任务分配的扩展

当分配不复杂时，用户和组的设置非常麻烦。 为避免复杂性，可以使用用户任务的自定义扩展。

* assignee属性：这个自定义扩展可以直接把用户任务分配给指定用户。

	<userTask id="theTask" name="my task" activiti:assignee="kermit" />

它和使用上面定义的 humanPerformer 效果完全一样。

* candidateUsers 属性：这个自定义扩展可以为任务设置候选人。

	<userTask id="theTask" name="my task" activiti:candidateUsers="kermit, gonzo" />

它和使用上面定义的 potentialOwner 效果完全一样。 注意它不需要像使用potential owner 通过 user(kermit)声明， 因为这个属性只能用于人员。

* candidateGroups 属性：这个自定义扩展可以为任务设置候选组。
	<userTask id="theTask" name="my task" activiti:candidateGroups="management, accountancy" />

它和使用上面定义的 potentialOwner 效果完全一样。 注意它不需要像使用 potentialOwner 通过 group(management) 声明， 因为这个属性只能用于群组。

* candidateUsers 和 candidateGroups 可以同时设置在同一个用户任务中。

注意：虽然 activiti 提供了一个账号管理组件， 也提供了IdentityService， 但是账号组件不会检测设置的用户是否村爱。 它嵌入到应用中，也允许 activiti 与其他已存的账户管理方
案集成。

####Custom identity link types (Experimental) 自定义身份链接类型(试验)

[试验]

BPMN 标准支持一个指定的用户或 humanperformer 或一组用户，形成一个潜在的池 potentialowners 为用户分配的定义。此外，Activiti 定义扩展属性的元素，可以代表任务受让人或候选人的所有者的用户任务。

支持的身份链接类型有：

	public class IdentityLinkType {
	  /* Activiti native roles */
	  public static final String ASSIGNEE = "assignee";
	  public static final String CANDIDATE = "candidate";
	  public static final String OWNER = "owner";
	  public static final String STARTER = "starter";
	  public static final String PARTICIPANT = "participant";
	}

BPMN 标准和 Activiti 实例授权身份是 user（用户）和 group（组）。如前所述，Activiti 身份管理的实施不是用于生产使用，但应扩展取决于支持授权方案。

如果额外的链接类型是必需的，自定义资源可以被定义为与下面的语法扩展元素：

	<userTask id="theTask" name="make profit">
	  <extensionElements>
	    <activiti:customResource activiti:name="businessAdministrator">
	      <resourceAssignmentExpression>
	        <formalExpression>user(kermit), group(management)</formalExpression>
	      </resourceAssignmentExpression>
	    </activiti:customResource>
	  </extensionElements>
	</userTask>

自定义链接表达式添加到 TaskDefinition 类：

	  protected Map<String, Set<Expression>> customUserIdentityLinkExpressions = 
	      new HashMap<String, Set<Expression>>(); 
	  protected Map<String, Set<Expression>> customGroupIdentityLinkExpressions = 
	      new HashMap<String, Set<Expression>>();
	  
	  public Map<String, 
	         Set<Expression>> getCustomUserIdentityLinkExpressions() {
	    return customUserIdentityLinkExpressions;
	  }
	  
	  public void addCustomUserIdentityLinkExpression(String identityLinkType, 
	                                                  Set<Expression> idList) {
	    customUserIdentityLinkExpressions.put(identityLinkType, idList);
	  }
	  
	  public Map<String, 
	         Set<Expression>> getCustomGroupIdentityLinkExpressions() {
	    return customGroupIdentityLinkExpressions;
	  }
	  
	  public void addCustomGroupIdentityLinkExpression(String identityLinkType, 
	                                                   Set<Expression> idList) {
	    customGroupIdentityLinkExpressions.put(identityLinkType, idList);
	  }

这是由在运行时的 UserTaskActivityBehavior handleAssignments 方法落实的。

最后，该 IdentityLinkType 类必须扩展支持自定义身份链接类型：

	package com.yourco.engine.task;
	
	public class IdentityLinkType
	    extends org.activiti.engine.task.IdentityLinkType
	{
	    public static final String ADMINISTRATOR = "administrator";
	
	    public static final String EXCLUDED_OWNER = "excludedOwner";
	}

这是一个数据库实体，包含了自定义身份链接类型 administrator 与一个单一的用户 (gonzo) 和组 (engineering)。

![](http://99btgc01.info/uploads/2014/12/custom.identityLinkType.example.png)

####Custom Assignment via task listeners 通过任务监听器自定义分配任务

如果上面的方式还不满足需求，可以使用任务监听器在创建事件委托自定义任务逻辑：

	<userTask id="task1" name="My task" >
	  <extensionElements>
	    <activiti:taskListener event="create" class="org.activiti.MyAssignmentHandler" />
	  </extensionElements>
	</userTask>

该 DelegateTask ，传递到 TaskListener 的实现，允许设置的受让人和候选的用户或组：

	public class MyAssignmentHandler implements TaskListener {
	
	  public void notify(DelegateTask delegateTask) {
	    // Execute custom identity lookups here
	    
	    // and then for example call following methods:
	    delegateTask.setAssignee("kermit");
	    delegateTask.addCandidateUser("fozzie");
	    delegateTask.addCandidateGroup("management");
	    ...
	  }
	  
	}

使用 spring 时，可以使用向上面章节中介绍的自定义分配属性，使用表达式 把任务监听器设置为 spring 代理的 bean， 让这个监听器监听任务的创建事件。 下面的例子中，执行者会通过调用 ldapService 这个spring bean 的 findManagerOfEmployee 方法获得。 流程变量 emp 会作为参数传递给 bean。

	<userTask id="task" name="My Task" activiti:assignee="${ldapService.findManagerForEmployee(emp)}"/>

也可以用来设置候选人和候选组：

	<userTask id="task" name="My Task" activiti:candidateUsers="${ldapService.findAllSales()}"/>

注意方法返回类型只能为 String 或 Collection<String> （对应候选人和候选组）：
	          
	public class FakeLdapService {
	  
	  public String findManagerForEmployee(String employee) {
	    return "Kermit The Frog";
	  }
	  
	  public List<String> findAllSales() {
	    return Arrays.asList("kermit", "gonzo", "fozzie");
	  }
	
	}

###Script Task 脚本任务

####Description 描述

脚本任务时一个自动节点。当流程到达脚本任务， 会执行对应的脚本。

####Graphical notation 图形标记

脚本任务显示为标准 BPMN 2.0 任务（圆角矩形）， 左上角有一个脚本小图标。

![](http://99btgc01.info/uploads/2014/12/bpmn.scripttask.png)

####XML representation 内容

脚本任务定义需要指定 script 和 scriptFormat。

	<scriptTask id="theScriptTask" name="Execute script" scriptFormat="groovy">
	  <script>
	    sum = 0
	    for ( i in inputArray ) {
	      sum += i
	    }
	  </script>
	</scriptTask>

scriptFormat 的值必须兼容 [JSR-223](http://jcp.org/en/jsr/detail?id=223)（java 平台的脚本语言）。默认 Javascript 会包含在 JDK 中，不需要额外的依赖。 如果你想使用其他（JSR-223兼容）的脚本引擎， 需要把对应的 jar 添加到 classpath 下，并使用合适的名称。比如，activiti 单元测试经常使用 groovy， 因为语法比 java 简单太多。(译者注：新出的项目管理工具 [Gradle](http://gradle.org/) 也是使用了 groovy,详见 《[Gradle 2 用户指南](https://github.com/waylau/Gradle-2-User-Guide)》)

注意，groovy 脚本引擎放在 groovy-all.jar中。在 2.0 版本之前， 脚本引擎是 Groovy jar 的一部分。这样，需要添加如下依赖：

	<dependency>
	      <groupId>org.codehaus.groovy</groupId>
	      <artifactId>groovy-all</artifactId>
	      <version>2.x.x<version>
	</dependency>

####脚本中的变量

到达脚本任务的流程可以访问的所有流程变量，都可以在脚本中使用。实例中，脚本变量'inputArray'其实是流程变量（整数数组）。

	<script>
	    sum = 0
	    for ( i in inputArray ) {
	      sum += i
	    }
	</script>

也可以在脚本中设置流程变量，直接调用 execution.setVariable("variableName",variableValue)。 默认，不会自动保存变量（注意 ：activiti 5.12 之前存在这个问题）。 可以在脚本中自动保存任何变量。 （比如上例中的 sum ），只要把 scriptTask 的autoStoreVariables 属性设置为 true。 然而，最佳实践是不要用它，而是显示调用 execution.setVariable()， 因为一些当前版本的 JDK对于一些脚本语言，无法实现自动保存变量。 参考[这里](http://www.jorambarrez.be/blog/2013/03/25/bug-on-jdk-1-7-0_17-when-using-scripttask-in-activiti/) 获得更多信息。

	<scriptTask id="script" scriptFormat="JavaScript" activiti:autoStoreVariables="false">

参数默认为 false，意思是如果没有为脚本任务定义设置参数， 所有声明的变量将只存在于脚本执行的阶段。

如何在脚本中设置变量的例子：

	<script>
	    def scriptVar = "test123"
	    execution.setVariable("myVar", scriptVar)
	</script>

注意：下面这些命名已被占用，不能用作变量名： : out, out:print, lang:import, context, elcontext

####Script results 脚本结果

脚本任务的返回值可以通过制定流程变量的名称，分配给已存或一个新流程变量， 使用脚本任务定义的 'activiti:resultVariable' 属性。 任何已存的流程变量都会被脚本执行的结果覆盖。 如果没有指定返回变量名，脚本的返回值会被忽略。

	<scriptTask id="theScriptTask" name="Execute script" scriptFormat="juel" activiti:resultVariable="myVar">
	  <script>#{echo}</script>
	</scriptTask>

上例中，脚本的结果（表达式'#{echo}'的值） 在脚本完成后，会设置到'myVar'变量中。

###Java Service Task 服务任务

####Description 描述

Java 服务任务用来调用外部 Java  类

####Graphical notation 图形标记

服务任务显示为圆角矩形，左上角有一个齿轮小图标

![](http://99btgc01.info/uploads/2014/12/bpmn.java.service.task.png)

####XML representation 内容

有4钟方法来声明 java 调用逻辑：

* 实现 JavaDelegate 或 ActivityBehavior
* 执行解析代理对象的表达式
* 调用一个方法表达式
* 调用一直值表达式

执行一个在流程执行中调用的类， 需要在'activiti:class'属性中设置全类名。

	<serviceTask id="javaService" 
	             name="My Java Service Task" 
	             activiti:class="org.activiti.MyJavaDelegate" />

参考实现章节 了解更多使用类的信息

也可以使用表达式调用一个对象。对象必须遵循一些规则， 并使用activiti:class 属性进行创建。（了解更多）。

    <serviceTask id="serviceTask" activiti:delegateExpression="${delegateExpressionBean}" />

这里，delegateExpressionBean 是一个实现了 JavaDelegate 接口的bean， 它定义在实例的 spring 容器中。

要指定执行的 UEL 方法表达式， 需要使用 activiti:expression。

	<serviceTask id="javaService" 
             name="My Java Service Task" 
             activiti:expression="#{printer.printMessage()}" />	

方法 printMessage（无参数）会调用 名为 printer 对象的方法。
也可以为表达式中的方法传递参数。

	<serviceTask id="javaService" 
             name="My Java Service Task" 
             activiti:expression="#{printer.printMessage(execution, myVar)}" />

这会调用名为 printer 对象上的方法 printMessage。 第一个参数是DelegateExecution，在表达式环境中默认名称为 execution。 第二个参数传递的是当前流程的名为 myVar 的变量。

要指定执行的 UEL 值表达式， 需要使用 activiti:expression 属性。

	<serviceTask id="javaService" 
             name="My Java Service Task" 
             activiti:expression="#{split.ready}" />

ready 属性的 getter 方法，getReady（无参数）， 会作用于名为split 的 bean 上。 这个对象会被解析为流程对象和 （如果合适）Spring  环境中的对象。

####Implementation 实现

要在流程执行中实现一个调用的类，这个类需要实现 org.activiti.engine.delegate.JavaDelegate 接口， 并在execute 方法中提供对应的业务逻辑。 当流程执行到特定阶段，它会指定方法中定义好的业务逻辑， 并按照默认 BPMN 2.0 中的方式离开节点。
让我们创建一个 java 类的例子，它可以流程变量中字符串转换为大写。 这个类需要实现 org.activiti.engine.delegate.JavaDelegate 接口， 这要求我们实现 execute(DelegateExecution) 方法。 它包含的业务逻辑会被引擎调用。流程实例信息，如流程变量和其他信息， 可以通过 [DelegateExecution](http://activiti.org/javadocs/org/activiti/engine/delegate/DelegateExecution.html)接口访问和操作（点击对应操作的 javadoc 的链接，获得更多信息）。

	public class ToUppercase implements JavaDelegate {
	  
	  public void execute(DelegateExecution execution) throws Exception {
	    String var = (String) execution.getVariable("input");
	    var = var.toUpperCase();
	    execution.setVariable("input", var);
	  }
	  
	}

注意：serviceTask 定义的 class 只会创建一个 java 类的实例。 所有流程实例都会共享相同的类实例，并调用 execute(DelegateExecution)。 这意味着，类不能使用任何成员变量，必须
是线程安全的，它必须能模拟在不同线程中执行。 这也影响着属性注入的处理方式。

流程定义中引用的类（比如，使用 activiti:class ）不会在部署时实例化。只有当流程第一次执行到使用类的时候， 类的实例才会被创建。如果找不到类，会抛出一个 ActivitiException。这个原因是部署环境（更确切是的 classpath ）和真实环境往往是不同的。 比如当使用 ant 或
业务归档上传到 Activiti Explorer 来发布流程 classpath 没有包含引用的类。


[内部：非公共实现类] 也可以提供实现
org.activiti.engine.impl.pvm.delegate.ActivityBehavior 接口的类。 实现可以访问更强大的 ActivityExecution, 它可以影响流程的流向。注意，这不是一个很好的实践， 应该尽量避免。所以，建议只有在高级情况下并且你确切知道你要做什么的情况下， 再使用ActivityBehavior 接口。

####Field Injection 属性注入

可以为代理类的属性注入数据。支持如下类型的注入：

* 固定的字符串
* 表达式

如果有效的话，数值会通过代理类的setter方法注入，遵循java bean的命名规范（比如 fistName 属性对应 setFirstName(...) 方法）。 如果属性没有对应的 setter 方法，数值会直接注入到私有属性中。 一些环境的 SecurityManager 不允许修改私有属性，所以最好还是把你想注入的属性暴露出对应的 setter 方法来。 无论流程定义中的数据是什么类型，注入目标的属性类型都应该是   org.activiti.engine.delegate.Expression。

下面代码演示了如何把一个常量注入到属性中。 属性注入可以使用 'class' 属性。 注意我们需要定义一个'extensionElements' XML元素， 在声明实际的属性注入之前，这是 BPMN 2.0 XML 格式要求的。

	<serviceTask id="javaService" 
	    name="Java service invocation" 
	    activiti:class="org.activiti.examples.bpmn.servicetask.ToUpperCaseFieldInjected">
	    <extensionElements>
	      <activiti:field name="text" stringValue="Hello World" />
	  </extensionElements>           
	</serviceTask>       

ToUpperCaseFieldInjected 类有一个 text 属性， 类型是org.activiti.engine.delegate.Expression。 调
用 text.getValue(execution) 时，会返回定义的字符串 Hello World。

也可以使用长文字（比如，内嵌的 email），可以使用 'activiti:string' 子元素：

	<serviceTask id="javaService" 
	    name="Java service invocation" 
	    activiti:class="org.activiti.examples.bpmn.servicetask.ToUpperCaseFieldInjected">
	  <extensionElements>   
	    <activiti:field name="text">
	        <activiti:string>
	          Hello World
	      </activiti:string>
	    </activiti:field>
	  </extensionElements>        
	</serviceTask>       

可以使用表达式，实现在运行期动态解析注入的值。这些表达式可以使用流程变量或 spring 定义的 bean（如果使用了 spring ）。像服务任务实现里说的那样，服务任务中的java类实例会在所有流程实例中共享。 为了动态注入属性的值，我们可以在org.activiti.engine.delegate.Expression 中使用值和方法表达式，它会使用传递给 execute 方法的 DelegateExecution 参数进行解析。 

	<serviceTask id="javaService" name="Java service invocation" 
	  activiti:class="org.activiti.examples.bpmn.servicetask.ReverseStringsFieldInjected">
	  
	  <extensionElements>
	    <activiti:field name="text1">
	      <activiti:expression>${genderBean.getGenderString(gender)}</activiti:expression>
	    </activiti:field>
	    <activiti:field name="text2">
	       <activiti:expression>Hello ${gender == 'male' ? 'Mr.' : 'Mrs.'} ${name}</activiti:expression>
	    </activiti:field>
	  </ extensionElements>
	</ serviceTask>

下面的例子中，注入了表达式，并使用在传入的当前DelegateExecution解析它们。 完整代码可以参考 org.activiti.examples.bpmn.servicetask.JavaServiceTaskTest.testExpressionFieldInjection。


	public class ReverseStringsFieldInjected implements JavaDelegate {
	
	  private Expression text1;
	  private Expression text2;
	
	  public void execute(DelegateExecution execution) {
	    String value1 = (String) text1.getValue(execution);
	    execution.setVariable("var1", new StringBuffer(value1).reverse().toString());
	
	    String value2 = (String) text2.getValue(execution);
	    execution.setVariable("var2", new StringBuffer(value2).reverse().toString());
	  }
	}

另外，你也可以把表达式设置成一个属性，而不是字元素，让XML更简单一些。

	<activiti:field name="text1" expression="${genderBean.getGenderString(gender)}" />
	<activiti:field name="text1" expression="Hello ${gender == 'male' ? 'Mr.' : 'Mrs.'} ${name}" />

因为 java 类实例会被重用，注入只会发生一次，当服务任务调用第一次的时候。 当你的代码中的属性改变了，值也不会重新注入，所以你应该把它们看做是不变的，不用修改它们。

####Service task results 服务任务结果

服务流程返回的结果（使用表达式的服务任务）可以分配给已经存在的或新的流程变量， 可以通过指定服务任务定义的 'activiti:resultVariable' 属性来实现。 指定的路程比那两的值会被服务流程的返回结果覆盖。如果没有指定返回变量名，就会忽略返回结果。

	<serviceTask id="aMethodExpressionServiceTask"
	    activiti:expression="#{myService.doSomething()}"
	    activiti:resultVariable="myVar" />

在上面的例子中，服务流程的返回值（在'myService'上调用 'doSomething()' 方法的返回值， myService 可能是流程变量，也可能是 spring 的 bean ），会设置到名为 'myVar' 的流程变量里，在服务执行完成之后。

####Handling exceptions处理异常

执行自定义逻辑时，常常需要捕获对应的业务异常，在流程内部进行处理。 activiti 提供了不同的方式来处理这个问题

#####Throwing BPMN Errors 抛出BPMN Errors

可以在服务任务或脚本任务的代码里抛出 BPMN error。 为了实现这个，要从 JavaDelegate，脚本，表达式和代理表达式中抛出名为 BpmnError的特殊 ActivitiExeption。 引擎会捕获这个异常，把它转发到对应的错误处理中。比如，边界错误事件或错误事件子流程。
	
	public class ThrowBpmnErrorDelegate implements JavaDelegate {
	
	  public void execute(DelegateExecution execution) throws Exception {
	    try {
	      executeBusinessLogic();
	    } catch (BusinessException e) {
	      throw new BpmnError("BusinessExceptionOccurred");
	    }
	  }
	
	}

构造参数是错误代码，会被用来决定 哪个错误处理器会来响应这个错误。 参考边界错误事件 获得更多捕获 BPMN error 的信息。

这个机制应该只用于业务失败， 它应该被流程定义中设置的边界错误事件或错误事件子流程处理。技术上的错误应该使用其他异常类型，通常不会在流程里处理。

#####Exception Sequence Flow 异常顺序流

[内部，公开实现类] 另一种选择是在一些异常发生时，让路程进入其他路径。下面的代码演示了如何实现。

	<serviceTask id="javaService" 
	  name="Java service invocation" 
	  activiti:class="org.activiti.ThrowsExceptionBehavior">            
	</serviceTask>
	    
	<sequenceFlow id="no-exception" sourceRef="javaService" targetRef="theEnd" />
	<sequenceFlow id="exception" sourceRef="javaService" targetRef="fixException" />

这里的服务任务有两个外出顺序流，分别叫 exception 和 no-exception 。异常出现时会使用顺序流的 id 来决定流向

	public class ThrowsExceptionBehavior implements ActivityBehavior {
	
	  public void execute(ActivityExecution execution) throws Exception {
	    String var = (String) execution.getVariable("var");
	
	    PvmTransition transition = null;
	    try {
	      executeLogic(var);
	      transition = execution.getActivity().findOutgoingTransition("no-exception");
	    } catch (Exception e) {
	      transition = execution.getActivity().findOutgoingTransition("exception");
	    }
	    execution.take(transition);
	  }
	  
	}

####Using an Activiti service from within a JavaDelegate 在 JavaDelegate 里使用 activiti 服务

一些场景下，需要在 java 服务任务中使用 activiti 服务 （比如，通过 RuntimeService 启动流
程实例，而 callActivity 不满足你的需求）。
org.activiti.engine.delegate.DelegateExecution 允许通过
org.activiti.engine.EngineServices 接口直接获得这些服务：

	public class StartProcessInstanceTestDelegate implements JavaDelegate {
	
	  public void execute(DelegateExecution execution) throws Exception {
	    RuntimeService runtimeService = execution.getEngineServices().getRuntimeService();
	    runtimeService.startProcessInstanceByKey("myProcess");
	  }
	
	}       

所有 activiti 服务的 API 都可以通过这个接口获得。

使用这些 API 调用出现的所有数据改变，都是在当前事务中的。在像spring 和 CDI 这样的依赖注入环境也会起作用，无论是否启用了 JTA 数据源。 比如，下面的代码功能与上面的代码一致， 这是 RuntimeService 是通过依赖注入获得的，而不是通过 org.activiti.engine.EngineServices 接口。 

	@Component("startProcessInstanceDelegate")
	public class StartProcessInstanceTestDelegateWithInjection {
	  
	    @Autowired
	    private RuntimeService runtimeService;
	    
	    public void startProcess() {
	      runtimeService.startProcessInstanceByKey("oneTaskProcess");
	    }
	    
	}
	            
重要技术提示：因为服务调用是在当前事务里， 数据的产生或改变，在服务任务执行完之前，还没有提交到数据库。 所有 API 对于数据库数据的操作，意味着未提交的操作在服务任务的 API 调用中都是不可见的

###Web Service Task

[试验]

####Description 描述

Web Service任务可以用来同步调用一个外部的Web service。

####Graphical notation 图形标记

Web Service 任务与 Java服务任务显示效果一样。

![](http://99btgc01.info/uploads/2014/12/bpmn.web.service.task.png)

####XML representation 内容

要使用 Web Service 我们需要导入它的操作和类型。 可以自动使用import 标签来指定 Web Service 的 WSDL：

	<import importType="http://schemas.xmlsoap.org/wsdl/"
        location="http://localhost:63081/counter?wsdl"
        namespace="http://webservice.activiti.org/" />

上面的声明告诉 activiti 导入 WSDL 定义，但没有创建 item 定义和消息。 假设我们想调用一个名为 'prettyPrint' 的方法， 我们必须创建为请求和响应信息对应的消息和 item 定义：

	<message id="prettyPrintCountRequestMessage" itemRef="tns:prettyPrintCountRequestItem" />
	<message id="prettyPrintCountResponseMessage" itemRef="tns:prettyPrintCountResponseItem" />
	  
	<itemDefinition id="prettyPrintCountRequestItem" structureRef="counter:prettyPrintCount" />
	<itemDefinition id="prettyPrintCountResponseItem" structureRef="counter:prettyPrintCountResponse" />

在申请服务任务之前，我们必须定义实际引用 Web Service 的 BPMN 接口和操作。 基本上，我们定义接口和必要的操作。对每个奥做我们都会重用上面定义的信息作为输入和输出。 比如，下面定义了 'counter' 接口和 'prettyPrintCountOperation' 操作：

	<interface name="Counter Interface" implementationRef="counter:Counter">
	        <operation id="prettyPrintCountOperation" name="prettyPrintCount Operation" 
	                        implementationRef="counter:prettyPrintCount">
	                <inMessageRef>tns:prettyPrintCountRequestMessage</inMessageRef>
	                <outMessageRef>tns:prettyPrintCountResponseMessage</outMessageRef>
	        </operation>
	</interface>

然后我们可以定义 Web Service 任务使用 ##WebService 实现， 并引用 Web Service 操作。

	<serviceTask id="webService" 
        name="Web service invocation"
        implementation="##WebService"
        operationRef="tns:prettyPrintCountOperation">

####Web Service Task IO Specification 规范

除非我们使用简化方式处理数据输入和输出关联（如下所示），每个 Web Service 任务可以定义任务的输入输出 IO 规范。 配置方式与 BPMN 2.0 完全兼容，下面格式化后的例子，我们根据之前定义 item 定义，定义了输入和输出。

	<ioSpecification>
	        <dataInput itemSubjectRef="tns:prettyPrintCountRequestItem" id="dataInputOfServiceTask" />
	        <dataOutput itemSubjectRef="tns:prettyPrintCountResponseItem" id="dataOutputOfServiceTask" />
	        <inputSet>
	                <dataInputRefs>dataInputOfServiceTask</dataInputRefs>
	        </inputSet>
	        <outputSet>
	                <dataOutputRefs>dataOutputOfServiceTask</dataOutputRefs>
	        </outputSet>
	</ioSpecification>

####Web Service Task data input associations 任务数据输入关联

有两种方式指定数据输入关联：

* 使用表达式
* 使用简化方式

要使用表达式指定数据输入关联，我们需要定义来源和目的 item，并指定每个 item 属性之间的对应关系。 下面的例子中我们分配了这些 item 的前缀和后缀：

	<dataInputAssociation>
	        <sourceRef>dataInputOfProcess</sourceRef>
	        <targetRef>dataInputOfServiceTask</targetRef>
	        <assignment>
	                <from>${dataInputOfProcess.prefix}</from>
	                <to>${dataInputOfServiceTask.prefix}</to>
	        </assignment>
	        <assignment>
	                <from>${dataInputOfProcess.suffix}</from>
	                <to>${dataInputOfServiceTask.suffix}</to>
	        </assignment>
	</dataInputAssociation>

另外，我们可以使用更简单的简化方式。'sourceRef' 元素是 activiti 的变量名，'targetRef' 元素是 item 定义的一个属性。在下面的例子中，我们把 'PrefixVariable' 变量的值分配给 'field' 属性， 把 'SuffixVariable' 变量的值分配给 'suffix' 属性。

	<dataInputAssociation>
	        <sourceRef>PrefixVariable</sourceRef>
	        <targetRef>prefix</targetRef>
	</dataInputAssociation>
	<dataInputAssociation>
	        <sourceRef>SuffixVariable</sourceRef>
	        <targetRef>suffix</targetRef>
	</dataInputAssociation>

####Web Service Task data output associations 任务数据输出关联

有两种方式指定数据输出关联：

* 使用表达式
* 使用简化方式

要使用表达式指定数据输出关联，我们需要定义目的变量和来源表达式。 方法和数据输入关联完全一样：
	
	<dataOutputAssociation>
	        <targetRef>dataOutputOfProcess</targetRef>
	        <transformation>${dataOutputOfServiceTask.prettyPrint}</transformation>
	</dataOutputAssociation>

另外，我们可以使用更简单的简化方式。'sourceRef' 元素是 item 定义的一个属性，'targetRef' 元素是 activiti 的变量名。 方法和数据输入关联完全一样：

	<dataOutputAssociation>
	        <sourceRef>prettyPrint</sourceRef>
	        <targetRef>OutputVariable</targetRef>
	</dataOutputAssociation>

###Business Rule Task 业务规则任务

[试验]

####Description 描述

业务规则用户用来同步执行一个或多个规则。activiti 使用 drools 规则引擎执行业务规则。目前，包含业务规则的 .drl 文件必须和流程定义一起发布，流程定义里包含了执行这些规则的业务规则任务。意味着流程使用的所有.drl文件都必须打包在流程BAR文件里，比如任务表单。更多使用Drools Expert 创建业务规则的信息，请参考 [JBoss Drools](http://www.jboss.org/drools/documentation) 的文档。如果想要使用你的规则任务的实现，比如，因为你想用不同方式使用 drools，或你想使用完全不同的规则引擎，你可以使用 BusinessRuleTask 上的 class 或表达式属性，它用起来就和 ServiceTask 一样。

####Graphical notation 图形标记

业务规则任务使用一个表格小图标进行显示。

![](http://99btgc01.info/uploads/2014/12/bpmn.business.rule.task.png)

####XML representation 内容

要执行部署流程定义的 BAR 文件中的一个或多个业务规则，我们需要定义输入和输出变量。对于输入变量定义，可以使用逗号分隔的一些流程变量。 输出变量定义智能包含一个变量名，它会把执行业务规则后返回的对象保存到对应的流程变量中。 注意，结果变量会包含一个对象列表。如果没有指定输出变量名称，默认会使用 org.activiti.engine.rules.OUTPUT。

下面的业务规则任务会执行和流程定义一起部署的素有业务规则：

	<process id="simpleBusinessRuleProcess">
	  
	  <startEvent id="theStart" />
	  <sequenceFlow sourceRef="theStart" targetRef="businessRuleTask" />
	  
	  <businessRuleTask id="businessRuleTask" activiti:ruleVariablesInput="${order}"
	      activiti:resultVariable="rulesOutput" />
	        
	  <sequenceFlow sourceRef="businessRuleTask" targetRef="theEnd" />
	
	  <endEvent id="theEnd" />
	 
	</process>
                
业务规则任务也可以配置成只执行部署的 .drl 文件中的一些规则。 这时要设置逗号分隔的规则名。

	<businessRuleTask id="businessRuleTask" activiti:ruleVariablesInput="${order}"
      activiti:rules="rule1, rule2" />

这时，只会执行 rule1 和 rule2。

你也可以定义哪些规则不用执行。
      
	<businessRuleTask id="businessRuleTask" activiti:ruleVariablesInput="${order}"
      activiti:rules="rule1, rule2" exclude="true" />

这时除了 rule1 和 rule2 以外，所有部署到流程定义同一个BAR文件中的规则都会执行。

像之前提到的，可以用一个选项修改 BusinessRuleTask 的实现：
	<businessRuleTask id="businessRuleTask" activiti:class="${MyRuleServiceDelegate}" />


注意 BusinessRuleTask 的功能和 ServiceTask 一样，但是我们使用BusinessRuleTask 的图标来表示 我们在这里要执行业务规则。

###Email Task

activiti 强化了业务流程，支持了自动邮件任务，它可以发送邮件给一个或多个参与者， 包括支持 cc, bcc, HTML 内容等等。 注意邮件任务不是 BPMN 2.0 规范定义的官方任务。 （它也没有对应的图标）。 因此，activiti 中邮件任务是用专门的服务任务实现的。

####Mail server configuration 邮件服务器配置

activiti 引擎要通过支持SMTP功能的外部邮件服务器发送邮件。 为了实际发送邮件，引擎需要知道如何访问邮件服务器。下面的配置可以设置到activiti.cfg.xml 配置文件中：
                       
Table 8.1. Mail server configuration

<table border="1" summary="Mail server configuration"><colgroup><col><col><col></colgroup>
<thead>
<tr><th>属性</th><th>是否需要?</th><th>描述</th></tr></thead>
<tbody><tr><td>mailServerHost</td><td>no</td><td>
mail服务器名称 (如 mail.mycorp.com). 默认是 localhost</td></tr>
<tr><td>mailServerPort</td><td>yes, 但不是在默认端口</td><td>邮件服务器上的SMTP传输端口。默认为25</td></tr>
<tr><td>mailServerDefaultFrom</td><td>no</td><td>如果用户没有指定发送邮件的邮件地址，默认设置的发送者的邮件地址。默认为activiti@activiti.org</td></tr>
<tr><td>mailServerUsername</td><td>如果服务器需要</td><td>一些邮件服务器需要认证才能发送邮件。默认不设置。</td></tr>
<tr><td>mailServerPassword</td><td>如果服务器需要</td><td>一些邮件服务器需要认证才能发送邮件。默认不设置。</td></tr><tr><td>mailServerUseSSL</td><td>如果服务器需要</td><td>一些邮件服务器需要ssl交互。默认为false。</td></tr><tr><td>mailServerUseTLS</td><td>如果服务器需要</td><td>一些邮件服务器（比如gmail）需要支持TLS。默认为false。</td></tr></tbody></table>

####Defining an Email Task 定义

邮件任务是一个专用的服务任务， 这个服务任务的type设置为 'mail'。

	<serviceTask id="sendMail" activiti:type="mail">   

邮件任务是通过属性注入进行配置的。 所有这些属性都可以使用 EL 表达式，可以在流程执行中解析。 下面的属性都可以设置：

Table 8.2. Mail task configuration

<table border="1" summary="Mail task configuration"><colgroup><col><col><col></colgroup>
<thead><tr><th>属性</th><th>是否需要?</th><th>描述</th></tr></thead>
<tbody>
<tr><td>to</td><td>yes</td><td>邮件的接受者。可以使用逗号分
隔多个接受者</td></tr>
<tr><td>from</td><td>no</td><td>邮件发送者的地址。如果不提
供，会使用默认配置的地址</td></tr>
<tr><td>subject</td><td>no</td><td>邮件的主题</td></tr>
<tr><td>cc</td><td>no</td><td>邮件抄送人。可以使用逗号分隔
多个接收者</td></tr>
<tr><td>bcc</td><td>no</td><td>邮件暗送人。可以使用逗号分隔
多个接收者</td></tr>
<tr><td>charset</td><td>no</td><td>可以修改邮件的字符集，对很多非英语语言是必须设置的。
</td></tr>
<tr><td>html</td><td>no</td><td>作为邮件内容的HTML</td></tr>
<tr><td>text</td><td>no</td><td>邮件的内容，在需要使用原始文
字（非富文本）的邮件时使用。可以与html一起使用，对于不支持富客户端的邮件客户端。 客户端会降级到仅显示文本的方式。
</td></tr>
<tr><td>htmlVar</td><td>no</td><td>使用对应的流程变量作为e-mail的内容。它和html的不同之处是它内容中包含的表达式会在 mail
任务发送之前被替换掉。</td></tr>
<tr><td>textVar</td><td>no</td><td>使用对应的流程变量作为e-mail 的纯文本内容。它和 html 的不同之处是它内容中包含的表达式会
在mail任务发送之前被替换掉</td></tr>
<tr><td>ignoreException</td><td>no</td><td>处理邮件失败时，是否忽略异常，不抛出ActivitiException，默认为false。</td></tr>
<tr><td>exceptionVariableName</td><td>no</td><td>当设置了ignoreException =true 处理 email 时不抛出异常，可以指定一个变量名来存储失败信息。</td></tr></tbody></table>

####Example usage 使用实例

下面的XML演示了使用邮件任务的例子

	<serviceTask id="sendMail" activiti:type="mail">
	  <extensionElements>
	    <activiti:field name="from" stringValue="order-shipping@thecompany.com" />
	    <activiti:field name="to" expression="${recipient}" />
	    <activiti:field name="subject" expression="Your order ${orderId} has been shipped" />
	    <activiti:field name="html">
	      <activiti:expression>
	        <![CDATA[
	          <html>
	            <body>
	              Hello ${male ? 'Mr.' : 'Mrs.' } ${recipientName},<br/><br/>
	                 
	              As of ${now}, your order has been <b>processed and shipped</b>.<br/><br/>
	                  
	              Kind regards,<br/>
	                  
	              TheCompany.
	            </body>
	          </html>
	        ]]>
	      </activiti:expression>
	    </activiti:field>      
	  </extensionElements>
	</serviceTask>         

结果如下：

![](http://99btgc01.info/uploads/2014/12/email.task.result.png)

###Mule Task

mule 任务可以向 mule 发送消息，以强化activiti的集成能力。注意mule 任务不是 BPMN 2.0 规范定义的官方任务。它也没有对应的图标）。 因此，activiti 中 mule 任务是用专门的服务任务实现的。

####Defining an Mule Task 定义

mule 任务是一个专用的服务任务， 这个服务任务的 type 设置为 'mule'。

	<serviceTask id="sendMule" activiti:type="mule"> 

mule 任务是通过属性注入进行配置的。 所有这些属性都可以使用 EL 表达式，可以在流程执行中解析。 下面的属性都可以设置：    

Table 8.3. Mule server configuration

<table border="1" summary="Mule server configuration"><colgroup><col><col><col></colgroup>
<thead><tr><th>属性</th><th>是否需要?</th><th>描述</th></tr></thead><tbody>
<tr><td>endpointUrl</td><td>yes</td><td>希望调用的Mule终端</td></tr>
<tr><td>language</td><td>yes</td><td>你要使用解析荷载表达式
（payloadExpression）属性的语言</td></tr>
<tr><td>payloadExpression</td><td>yes</td><td>作为消息荷载的表达式。</td></tr><tr><td>resultVariable</td><td>no</td><td>将要保存调用结果的变量名称</td></tr></tbody></table> 

####Example usage 应用实例

下面是一个使用 mule 任务的例子  

	<extensionElements>
	<activiti:field name="endpointUrl">
	  <activiti:string>vm://in</activiti:string>
	</activiti:field>
	<activiti:field name="language">
	  <activiti:string>juel</activiti:string>
	</activiti:field>
	<activiti:field name="payloadExpression">
	  <activiti:string>"hi"</activiti:string>
	</activiti:field>
	<activiti:field name="resultVariable">
	  <activiti:string>theVariable</activiti:string>
	</activiti:field>
	</extensionElements> 

###Camel Task

Camel 任务可以从 Camel 发送和介绍消息，由此强化了 activiti 的集成功能。 注意 camel 任务不是 BPMN 2.0 规范定义的官方任务。 （它也没有对应的图标）。 在 activiti 中，camel 任务时由专用的服务任务实现的。 要使用 camel 任务功能时，也要记得吧 activiti camel 包含到项目里。

####Defining a Camel Task 定义

camel 任务是一个专用的服务任务，这个服务任务的type设置为 'camel'。

	<serviceTask id="sendCamel" activiti:type="camel">  

流程定义只需要在服务任务中定义 camel 类型。 集成逻辑都会代理给camel 容器。默认 activiti 引擎会在 spring 容器中查找camelContext bean。 camelContext 定义了 camel 容器
加载的路由规则。下面的例子中路由规则是从指定的java包下加载的。 但是你也可以通过 spring 配置直接定义路由规则。
	
	<camelContext id="camelContext" xmlns="http://camel.apache.org/schema/spring">
	  <packageScan>
	    <package>org.activiti.camel.route</package>
	  </packageScan>
	</camelContext>

如果想了解更多关于 camel 路由的信息，可以访问 Camel 的[网站](http://camel.apache.org/)在这里只通过很小的例子演示了基础的概念。 在第一个例子中，我们会通过 activiti 工作流实现最简单的Camel调用。我们称其为 SimpleCamelCall。

如果想定义多个 Camel 环境 bean，并且（或者）想使用不同的 bean 名称，可以重载 CamelTask 的定义，如下所示：
	
	<serviceTask id="serviceTask1" activiti:type="camel">
	  <extensionElements>
	    <activiti:field name="camelContext" stringValue="customCamelContext" />
	  </extensionElements>
	</serviceTask>
                
####Simple Camel Call example 简单Camel调用

这个例子对应的文件都可以在 activiti camel 模块的
org.activiti.camel.examples.simpleCamelCall 包下找到。我们的目标是简单激活一个特定的 camel 路由。 首先，我们需要一个 Spring 环境，它要包含之前介绍的路由。这些文件的目的如下：
	
	<camelContext id="camelContext" xmlns="http://camel.apache.org/schema/spring">
	  <packageScan>
	    <package>org.activiti.camel.examples.simpleCamelCall</package>
	  </packageScan>
	</camelContext>
         
包含名为 SimpleCamelCallRoute 的路由的类文件，放在 PackageScan标签的扫描目录下。 下面就是路由的定义：

	public class SimpleCamelCallRoute extends RouteBuilder {
	
	  @Override
	  public void configure() throws Exception {
	    from("activiti:SimpleCamelCallProcess:simpleCall").to("log:org.activiti.camel.examples.SimpleCamelCall");
	  }
	}

这个规则仅仅打印消息体，不会做其他事情。注意终端的格式。它包含三部分：

Table 8.4. Endpoint URL parts:

<table border="1" summary="Endpoint URL parts:"><colgroup><col><col></colgroup>
<thead><tr><th>Endpoint Url 部分</th><th>描述</th></tr></thead><tbody>
<tr><td>activiti</td><td>引用 Activiti endpoint</td></tr><tr><td>SimpleCamelCallProcess</td><td>进程名字</td></tr><tr><td>simpleCall</td><td>流程中的 Camel 服务</td></tr></tbody></table>

OK，我们的规则已经配置好，也可以让 Camel 使用了。 现在看工作流部分。工作流看起来像这样：

	<process id="SimpleCamelCallProcess">
	  <startEvent id="start"/>
	  <sequenceFlow id="flow1" sourceRef="start" targetRef="simpleCall"/>
	  
	  <serviceTask id="simpleCall" activiti:type="camel"/>
	  
	  <sequenceFlow id="flow2" sourceRef="simpleCall" targetRef="end"/>      
	  <endEvent id="end"/>
	</process>  

在 serviceTask 部分，它只注明服务的类型是 Camel，目标规则名为simpleCall。这与上面的 activiti 终端相匹配。初始化流程后，我们会看到一个空的日志。 好，我们已经完成了这个最简单的例子了

####Ping Pong example 乒乓实例

我们的例子成功执行了，但是 Camel 和 Activiti 之间没有任何交互，而且这样做也没有任何优势。在这个例子里，我们尝试向 Camel 发送和接收数据。 我们发送一个字符串，camel 进行一些处理，然后返回结果。 发送部分很简单，我们把变量里的消息发送给 camel。这里是我们的调用代码

	@Deployment
	public void testPingPong() {
	  Map<String, Object> variables = new HashMap<String, Object>();
	
	  variables.put("input", "Hello");
	  Map<String, String> outputMap = new HashMap<String, String>();
	  variables.put("outputMap", outputMap);
	  
	  runtimeService.startProcessInstanceByKey("PingPongProcess", variables);
	  assertEquals(1, outputMap.size());
	  assertNotNull(outputMap.get("outputValue"));
	  assertEquals("Hello World", outputMap.get("outputValue"));
	}         

变量"input"是 Camel 规则的实际输入，outputMap 会记录 camel 返回的结果。流程应该像是这样：	        

	<process id="PingPongProcess">
	  <startEvent id="start"/>
	  <sequenceFlow id="flow1" sourceRef="start" targetRef="ping"/>
	  <serviceTask id="ping" activiti:type="camel"/>
	  <sequenceFlow id="flow2" sourceRef="ping" targetRef="saveOutput"/>
	  <serviceTask id="saveOutput"  activiti:class="org.activiti.camel.examples.pingPong.SaveOutput" />
	  <sequenceFlow id="flow3" sourceRef="saveOutput" targetRef="end"/>
	  <endEvent id="end"/>
	</process>
	    
注意，SaveOuput 这个 serviceTask，会把"Output"变量的值从上下文保存到上面提到的 OutputMap 中。 现在，我们必须了解变量是如何发送给 Camel，再返回的。这里就要涉及到 camel 实际执行的行为了。 变量提交给 camel 的方法是由 CamelBehavior 控制的。这里我们使用默认的配置，其他的会在后面提及。 使用这些代码，我们就可以配置一个期望的 camel 行为：

	<serviceTask id="serviceTask1" activiti:type="camel">
	  <extensionElements>
	    <activiti:field name="camelBehaviorClass" stringValue="org.activiti.camel.impl.CamelBehaviorCamelBodyImpl" />
	  </extensionElements>
	</serviceTask>

如果你没有特别指定一个行为，就会使用
org.activiti.camel.impl.CamelBehaviorDefaultImpl。 这个行为会把变量复制成名称相同的 Camel 属性。 在返回时，无论选择什么行为，如果 camel 消息体是一个 map，每个元素都会复制成一个变量， 否则整个对象会复制到指定名称为"camelBody"的变量中。 了解这些后，
就可以看看我们第二个例子的 camel 规则了：

	@Override
	public void configure() throws Exception {
	  from("activiti:PingPongProcess:ping").transform().simple("${property.input} World");
	}

在这个规则中，字符串"world"会被添加到"input"属性的后面，结果会写入消息体。 这时可以检查 javaServiceTask 中的"camelBody"变量，复制到"outputMap"中，并在 testcase 进行判断。现在这个例子是在默认的行为下运行的，然后我们看一起其他的方案。在启动的所有 camel 规则中，流程实例 id 会复制到 camel 的名为"PROCESS_ID_PROPERTY"的属性中。 后续可以用它关联流程实例和 camel 规则。他也可以在 camel规则中直接使用。

Activiti 中可以使用三种不同的行为。这些行为可以通过在规则URL中指定对应的环节来实现覆盖。 这里有一个在URL中覆盖现存行为的例子：

	from("activiti:asyncCamelProcess:serviceTaskAsync2?copyVariablesToProperties=true").
                
下面的表格提供了三种 camel 行为的概述：

Table 8.5. Existing camel behaviours:

<table border="1" summary="Existing camel behaviours:"><colgroup><col><col><col></colgroup><thead><tr><th>行为</th><th>Url</th><th>描述</th></tr></thead><tbody>
<tr><td>CamelBehaviorDefaultImpl</td><td>copyVariablesToProperties</td><td>复制 Activiti 属性作为 Camel 属性</td></tr>
<tr><td>CamelBehaviorCamelBodyImpl</td><td>copyCamelBodyToBody</td><td>只把名为 "camelBody" Activiti变量复制成camel的消息体</td></tr>
<tr><td>CamelBehaviorBodyAsMapImpl</td><td>copyVariablesToBodyAsMap</td><td>把activiti的所有变量复制到一个map里，作为Camel的消息体</td></tr></tbody></table>

上面的表格解释和 activiti 变量如何传递给 camel。下面的表格解释和camel 的变量如何返回给 activiti。 它只能配置在规则URL中。

Table 8.6. Existing camel behaviours:

<table border="1" summary="Existing camel behaviours:"><colgroup><col><col><col></colgroup><thead><tr><th>Url</th><th>描述</th><td class="auto-generated">&nbsp;</td></tr></thead><tbody>
<tr><td>Default</td><td>如果Camel消息体是一个map，把
每个元素复制成activiti的变量，否则把整个camel消息体作
为activiti的"camelBody"变量。</td><td class="auto-generated">&nbsp;</td></tr>
<tr><td>copyVariablesFromProperties</td><td>将Camel属性以相同名称复制为Activiti变量</td><td class="auto-generated">&nbsp;</td></tr>
<tr><td>copyCamelBodyToBodyAsString</td><td>和默认一样，但是如果camel消息体不是map时，先把它转换成字符串，再设置为 "camelBody"。</td><td class="auto-generated">&nbsp;</td></tr><tr><td>copyVariablesFromHeader</td><td>额外把camel头部以相同名称复制成Activiti变量</td><td class="auto-generated">&nbsp;</td></tr></tbody></table>

例子的源码放在 activiti-camel 模块的org.activiti.camel.examples.pingPong 包下。

####Asynchronous Ping Pong example 异步乒乓实例

之前的例子都是同步的。流程会等到 camel 规则返回之后才会停止。 一些情况下，我们需要 activiti 工作流继续运行。这时camelServiceTask 的异步功能就特别有用。 你可以通过设置
camelServiceTask 的 async 属性来启用这个功能。

	<serviceTask id="serviceAsyncPing" activiti:type="camel" activiti:async="true"/>

通过设置这个功能，camel 规则会被 activiti 的 jobExecutor 异步执行。 当你在 camel 规则中定义了一个队列，activiti 流程会在camelServiceTask 执行时继续运行。 camel 规则会以完全异步的方式执行。 如果你想在什么地方等待 camelServiceTask 的返回值，你可以使用一个 receiveTask。
  
	<receiveTask id="receiveAsyncPing" name="Wait State" />
              
流程实例会等到接收一个 signal，比如来自 camel。在 camel 中你可以发送一个 signal 给流程实例，通过对应的 activiti 终端发送消息

	from("activiti:asyncPingProcess:serviceAsyncPing").to("activiti:asyncPingProcess:receiveAsyncPing");           
                
对于一个常用的终端，会使用冒号分隔的三个部分：

* 常量字符串"activiti"
* 流程名称
* 接收任务名

####Instantiate workflow from Camel route 从camel规则中实例化工作流

之前的所有例子中，activiti 工作流会先启动，然后在流程中启动 camel 规则。 也可以使用另外一种方法。在已经启动的 camel 规则中启动一个工作流。 这会触发一个 receiveTask 十分类似，除了最后的部分。这是一个实例规则：

	from("direct:start").to("activiti:camelProcess");

我们看到 url 有两个部分，第一个部分是常量字符串"activiti"，第二部分是流程的名称。很明显，流程应该已经部署完成，并且是可以启动的。

也可以设置流程发起人到 Camel 头提供的身份验证的用户 ID 。为了实现这个，发起人变量必须在流程定义中指定的：

	<startEvent id="start" activiti:initiator="initiator" />

接着 用户 Id 包含在 Camel 头 名字叫 CamelProcessInitiatorHeader ，定义如下

	from("direct:startWithInitiatorHeader")
	    .setHeader("CamelProcessInitiatorHeader", constant("kermit"))
	    .to("activiti:InitiatorCamelCallProcess?processInitiatorHeaderName=CamelProcessInitiatorHeader");

###Manual Task 手工任务

####Description 描述

手工任务定义了 BPM 引擎外部的任务。 用来表示工作需要某人完成，而引擎不需要知道，也没有对应的系统和 UI 接口。 对于引擎，手工任务是直接通过的活动， 流程到达它之后会自动向下执行。

####Graphical notation 图形标记

手工任务显示为一个圆角矩形，左上角是一个手型小图标。

![](http://99btgc01.info/uploads/2014/12/bpmn.manual.task.png)

####XML representation 内容

	<manualTask id="myManualTask" name="Call client for more information" />

###Java Receive Task

####Description 描述

接收任务是一个简单任务，它会等待对应消息的到达。当前，我们只实现了这个任务的 java 语义。 当流程达到接收任务，流程状态会保存到存储里。 意味着流程会等待在这个等待状态， 直到引擎接收了一个特定的消息， 这会触发流程穿过接收任务继续执行。

####Graphical notation 图形标记

接收任务显示为一个任务（圆角矩形），右上角有一个消息小标记。消息是白色的（黑色图标表示发送语义）

![](http://99btgc01.info/uploads/2014/12/bpmn.receive.task.png)

####XML representation 内容

	<receiveTask id="waitState" name="wait" />   

要在接收任务等待的流程实例继续执行， 可以调用runtimeService.signal(executionId)，
传递接收任务上流程的 id。 下面的代码演示了实际是如何工作的：

	ProcessInstance pi = runtimeService.startProcessInstanceByKey("receiveTask");
	Execution execution = runtimeService.createExecutionQuery()
	  .processInstanceId(pi.getId())
	  .activityId("waitState")
	  .singleResult();
	assertNotNull(execution);
	    
	runtimeService.signal(execution.getId()); 
  
###Shell Task

####Description 描述

shell任务可以执行shell脚本和命令。 注意 shell 任务不是 BPMN 2.0 规范定义的官方任务。（它也没有对应的图标）。

####defining a shell task 定义

shell任务是一个专用的服务任务， 这个服务任务的type设置为'shell'

	<serviceTask id="shellEcho" activiti:type="shell"> 

shell 任务使用属性注入进行配置。 所有属性都可以包含 EL 表达式，会在流程执行过程中解析。可以配置以下属性： 

Table 8.7. Shell task parameter configuration

<table border="1" summary="Shell task parameter configuration"><colgroup><col><col><col></colgroup><thead><tr><th>属性</th><th>是否需要?</th><th>Type</th><th>描述</th><th>默认</th></tr></thead><tbody>
<tr><td>command</td><td>yes</td><td>String</td><td>执行的shell命令</td><td>&nbsp;</td></tr>
<tr><td>arg0-5</td><td>no</td><td>String</td><td>参数0至5</td><td>&nbsp;</td></tr>
<tr><td>wait</td><td>no</td><td>true/false</td><td>是否需要等待到shell进程结束</td><td>true</td></tr>
<tr><td>redirectError</td><td>no</td><td>true/false</td><td>标准错误与标准输出合并</td><td>false</td></tr>
<tr><td>cleanEnv</td><td>no</td><td>true/false</td><td>shell进行不继承当前环境</td><td>false</td></tr>
<tr><td>outputVariable</td><td>no</td><td>String</td><td>包含输出变量名称</td><td>输出不记录</td></tr>
<tr><td>errorCodeVariable</td><td>no</td><td>String</td><td>包含结果错误代码的变量名</td><td>不会注册错误级别</td></tr><tr><td>directory</td><td>no</td><td>String</td><td>shell进程的默认目录</td><td>当前目录</td></tr></tbody></table>

####Example usage 应用实例

下面的代码演示了使用 shell 任务的实例。它会执行 shell 脚本"cmd /c echo EchoTest"，等到它结束，再把输出结果保存到 resultVar 中。

	<serviceTask id="shellEcho" activiti:type="shell" >
	  <extensionElements>
	    <activiti:field name="command" stringValue="cmd" />  
	    <activiti:field name="arg1" stringValue="/c" />  
	    <activiti:field name="arg2" stringValue="echo" />  
	    <activiti:field name="arg3" stringValue="EchoTest" />  
	    <activiti:field name="wait" stringValue="true" />  
	    <activiti:field name="outputVariable" stringValue="resultVar" />  
	  </extensionElements>
	</serviceTask>                
                
###Execution listener 执行监听器

兼容性提醒：在发布 5.3 后，我们发现执行监听器， 任务监听器，表达式还是非公开 API。这些类在 org.activiti.engine.impl... 的子包， 包名中有一个 impl。
org.activiti.engine.impl.pvm.delegate.ExecutionListener,
org.activiti.engine.impl.pvm.delegate.TaskListener 和 org.activiti.engine.impl.pvm.el.Expression 已经废弃了。 从现在开始，应该使用 org.activiti.engine.delegate.ExecutionListener,
org.activiti.engine.delegate.TaskListener 和 org.activiti.engine.delegate.Expression。 在新的公开 API 中，删除了 ExecutionListenerExecution.getEventSource()。 因为已经设置了废弃编译警告，所以已存的代码应该可以正常运行。但是要考虑切换到新的公共API接口 （包名中没有.impl.）。

执行监听器可以执行外部 java 代码或执行表达式，当流程定义中发生了某个事件。 可以捕获的事件有：

* 流程实例的启动和结束。
* 选中一条连线。
* 节点的开始和结束。
* 网关的开始和结束。
* 中间事件的开始和结束。
* 开始时间结束或结束事件开始。

下面的流程定义包含了3个流程监听器：

	 <process id="executionListenersProcess">
	  
	    <extensionElements>
	      <activiti:executionListener class="org.activiti.examples.bpmn.executionlistener.ExampleExecutionListenerOne" event="start" />
	    </extensionElements>
	    
	    <startEvent id="theStart" />
	    <sequenceFlow sourceRef="theStart" targetRef="firstTask" />
	    
	    <userTask id="firstTask" />
	    <sequenceFlow sourceRef="firstTask" targetRef="secondTask">
	    <extensionElements>
	      <activiti:executionListener class="org.activiti.examples.bpmn.executionListener.ExampleExecutionListenerTwo" />
	    </extensionElements>
	    </sequenceFlow>
	    
	    <userTask id="secondTask" >
	    <extensionElements>
	      <activiti:executionListener expression="${myPojo.myMethod(execution.event)}" event="end" />
	    </extensionElements>
	    </userTask>
	    <sequenceFlow sourceRef="secondTask" targetRef="thirdTask" />
	       
	    <userTask id="thirdTask" />
	    <sequenceFlow sourceRef="thirdTask" targetRef="theEnd" />
	
	    <endEvent id="theEnd" />
	    
	  </process>

第一个流程监听器监听流程开始。监听器是一个外部 java 类（像
是 ExampleExecutionListenerOne）， 需要实现org.activiti.engine.delegate.ExecutionListener 接口。
当事件发生时（这里是 end 事件）， 会调用 notify(ExecutionListenerExecution execution) 方法。
	
	public class ExampleExecutionListenerOne implements ExecutionListener {
	
	  public void notify(ExecutionListenerExecution execution) throws Exception {
	    execution.setVariable("variableSetInExecutionListener", "firstValue");
	    execution.setVariable("eventReceived", execution.getEventName());
	  }
	}

也可以使用实现 org.activiti.engine.delegate.JavaDelegate 接口的代理类。 代理类可以在结构中重用，比如 serviceTask 的代理。

第二个流程监听器在连线执行时调用。注意这个 listener 元素不能定义event， 因为连线只能触发 take 事件。 为连线定义的监听器的event属性会被忽略。

最后一个流程监听器在节点 secondTask 结束时调用。这里使用expression 代替 class 来在事件触发时执行/调用。

	<activiti:executionListener expression="${myPojo.myMethod(execution.eventName)}" event="end" />

和其他表达式一样，流程变量可以处理和使用。因为流程实现对象有一个保存事件名称的属性， 可以在方法中使用 execution.eventName 获的事件名称。

流程监听器也支持使用 delegateExpression, 和服务任务相同。

	<activiti:executionListener event="start" delegateExpression="${myExecutionListenerBean}" />

在 activiti 5.12 中，我们也介绍了新的流程监听器，
org.activiti.engine.impl.bpmn.listener.ScriptExecutionListener。 这个脚本流程监听器可以为某个流程监听事件执行一段脚本。
	
	<activiti:executionListener event="start" class="org.activiti.engine.impl.bpmn.listener.ScriptExecutionListener" >
	  <activiti:field name="script">
	    <activiti:string>
	      def bar = "BAR";  // local variable
	      foo = "FOO"; // pushes variable to execution context
	      execution.setVariable("var1", "test"); // test access to execution instance
	      bar // implicit return value
	    </activiti:string>
	  </activiti:field>
	  <activiti:field name="language" stringValue="groovy" />
	  <activiti:field name="resultVariable" stringValue="myVar" />
	<activiti:executionListener>

####Field injection on execution listeners 流程监听器的属性注入

使用流程监听器时，可以配置class属性，可以使用属性注入。这和使用服务任务属性注入相同， 参考它可以获得属性注入的很多信息。

下面的代码演示了使用了属性注入的流程监听器的流程的简单例子。
	
	 <process id="executionListenersProcess">
	    <extensionElements>
	      <activiti:executionListener class="org.activiti.examples.bpmn.executionListener.ExampleFieldInjectedExecutionListener" event="start">
	        <activiti:field name="fixedValue" stringValue="Yes, I am " />
	        <activiti:field name="dynamicValue" expression="${myVar}" />
	      </activiti:executionListener>
	    </extensionElements>
	    
	    <startEvent id="theStart" />
	    <sequenceFlow sourceRef="theStart" targetRef="firstTask" />
	    
	    <userTask id="firstTask" />
	    <sequenceFlow sourceRef="firstTask" targetRef="theEnd" />
	    
	    <endEvent id="theEnd" />
	  </process>


```

	public class ExampleFieldInjectedExecutionListener implements ExecutionListener {
	
	  private Expression fixedValue;
	
	  private Expression dynamicValue;
	
	  public void notify(ExecutionListenerExecution execution) throws Exception {
	    execution.setVariable("var", fixedValue.getValue(execution).toString() + dynamicValue.getValue(execution).toString());
	  }
	}
```

ExampleFieldInjectedExecutionListener 类串联了两个注入的属性。 （一个是固定的，一个是动态的），把他们保存到流程变量'var'中。

	@Deployment(resources = {"org/activiti/examples/bpmn/executionListener/ExecutionListenersFieldInjectionProcess.bpmn20.xml"})
	public void testExecutionListenerFieldInjection() {
	  Map<String, Object> variables = new HashMap<String, Object>();
	  variables.put("myVar", "listening!");
	    
	  ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("executionListenersProcess", variables);
	    
	  Object varSetByListener = runtimeService.getVariable(processInstance.getId(), "var");
	  assertNotNull(varSetByListener);
	  assertTrue(varSetByListener instanceof String);
	    
	  // Result is a concatenation of fixed injected field and injected expression
	  assertEquals("Yes, I am listening!", varSetByListener);
	}

###Task listener 任务监听器

任务监听器可以在发生对应的任务相关事件时执行自定义 java 逻辑 或表达式。

任务监听器只能添加到流程定义中的用户任务中。 注意它必须定义在 BPMN 2.0 extensionElements 的子元素中， 并使用 activiti 命名空间，因为任务监听器是 activiti 独有的结构。

	<userTask id="myTask" name="My Task" >
	  <extensionElements>
	    <activiti:taskListener event="create" class="org.activiti.MyTaskCreateListener" />
	  </extensionElements>
	</userTask>

任务监听器支持以下属性：

* event（必选）：任务监听器会被调用的任务类型。 可能的类型为：
	* create：任务创建并设置所有属性后触发。
	* assignment：任务分配给一些人时触发。 当流程到达userTask， assignment事件 会在create事件之前发生。 这样的顺序似乎不自然，但是原因很简单：当获得create时间时， 我们想获得任务的所有属性，包括执行人。
	* complete：当任务完成，并尚未从运行数据中删除时触发。
	* delete：只在任务删除之前发生。 注意在通过completeTask正常完成时，也会执行。
* class：必须调用的代理类。 这个类必须实现org.activiti.engine.delegate.TaskListener 接口。

	public class MyTaskCreateListener implements TaskListener {

	  public void notify(DelegateTask delegateTask) {
	    // Custom logic goes here
	  }
	
	}

可以使用属性注入把流程变量或执行传递给代理类。注意代理类的实例是在部署时创建的（和 activiti 中其他类代理的情况一样），这意味着所有流程实例都会共享同一个实例。

* expression：（无法同时与 class 属性一起使用）： 指定事件发生时执行的表达式。 可以把 DelegateTask 对象和事件名称（使用 task.eventName ） 作为参数传递给调用的对象。

```
	<activiti:taskListener event="create" expression="${myObject.callMethod(task, task.eventName)}" />
```

* delegateExpression 可以指定一个表达式，解析一个实现了TaskListener 接口的对象， 这与服务任务一致。
	
```
	<activiti:taskListener event="create" delegateExpression="${myTaskListenerBean}" />
```

* 在 activiti 5.12 中，我们也介绍了新的任务监听
器，org.activiti.engine.impl.bpmn.listener.ScriptTaskListener。 脚本任务监听器可以为任务监听器事件执行脚本。

```
	<activiti:taskListener event="complete" class="org.activiti.engine.impl.bpmn.listener.ScriptTaskListener" >
	  <activiti:field name="script">
	    <activiti:string>
	      def bar = "BAR";  // local variable
	      foo = "FOO"; // pushes variable to execution context
	      task.setOwner("kermit"); // test access to task instance
	      bar // implicit return value
	    </activiti:string>
	  </activiti:field>
	  <activiti:field name="language" stringValue="groovy" />
	  <activiti:field name="resultVariable" stringValue="myVar" />
	<activiti:taskListener>
```
###Multi-instance (for each) 多实例（循环）

####Description 描述

多实例节点是在业务流程中定义重复环节的一个方法。 从开发角度讲，多实例和循环是一样的： 它可以根据给定的集合，为每个元素执行一个环节甚至一个完整的子流程， 既可以顺序依次执行也可以并发同步执行。

多实例是在一个普通的节点上添加了额外的属性定义 （所以叫做'多实例特性'），这样运行时节点就会执行多次。下面的节点都可以成为一个多实例节点

* User Task
* Script Task
* Java Service Task
* Web Service Task
* Business Rule Task
* Email Task
* Manual Task
* Receive Task
* (Embedded) Sub-Process
* Call Activity

网关和事件 不能设置多实例。

根据规范的要求，每个上级流程为每个实例创建分支时都要提供如下变量：

* nrOfInstances：实例总数
* nrOfActiveInstances：当前活动的，比如，还没完成的，实例数量。 对于顺序执行的多实例，值一直为1。
* nrOfCompletedInstances：已经完成实例的数目。
* 
可以通过 execution.getVariable(x) 方法获得这些变量。
另外，每个创建的分支都会有分支级别的本地变量（比如，其他实例不可见， 不会保存到流程实例级别）：

* loopCounter：表示特定实例的在循环的索引值。可以使用 activiti
的 elementIndexVariable 属性修改 loopCounter 的变量名。

####Graphical notation 图形标记

如果节点是多实例的，会在节点底部显示三条短线。 三条竖线表示实例会并行执行。 三条横线表示顺序执行。

![](http://99btgc01.info/uploads/2014/12/bpmn.multi.instance.png)

####XML representation 内容

要把一个节点设置为多实例，节点xml元素必须设置一个multiInstanceLoopCharacteristics 子元素

	<multiInstanceLoopCharacteristics isSequential="false|true">
	 ...
	</multiInstanceLoopCharacteristics>

isSequential 属性表示节点是进行 顺序执行还是并行执行。实例的数量会在进入节点时计算一次。 有一些方法配置它。一种方法是使用 loopCardinality 子元素直接指定一个数字。

	<multiInstanceLoopCharacteristics isSequential="false|true">
	  <loopCardinality>5</loopCardinality>
	</multiInstanceLoopCharacteristics>

也可以使用结果为整数的表达式：

	<multiInstanceLoopCharacteristics isSequential="false|true">
	  <loopCardinality>${nrOfOrders-nrOfCancellations}</loopCardinality>
	</multiInstanceLoopCharacteristics>

另一个定义实例数目的方法是，通过 loopDataInputRef 子元素，设置一个类型为集合的流程变量名。 对于集合中的每个元素，都会创建一个实例。 也可以通过 inputDataItem 子元素指定集合。 下面的代码演示了这些配置：

	<userTask id="miTasks" name="My Task ${loopCounter}" activiti:assignee="${assignee}">
	  <multiInstanceLoopCharacteristics isSequential="false">
	    <loopDataInputRef>assigneeList</loopDataInputRef>
	    <inputDataItem name="assignee" />
	  </multiInstanceLoopCharacteristics>
	</userTask>

假设 assigneeList 变量包含这些值[kermit, gonzo, foziee]。 在上面代码中，三个用户任务会同时创建。每个分支都会拥有一个用名为assignee 的流程变量， 这个变量会包含集合中的对应元素，在例子中会用来设置用户任务的分配者。

loopDataInputRef 和 inputDataItem 的缺点是1）名字不好记， 2）根据 BPMN 2.0 格式定义，它们不能包含表达式。activiti 通过在 multiInstanceCharacteristics 中设置 collection 和
elementVariable 属性解决了这个问题：
	
	<userTask id="miTasks" name="My Task" activiti:assignee="${assignee}">
	  <multiInstanceLoopCharacteristics isSequential="true" 
	     activiti:collection="${myService.resolveUsersForTask()}" activiti:elementVariable="assignee" >
	  </multiInstanceLoopCharacteristics>
	</userTask>

多实例节点在所有实例都完成时才会结束。也可以指定一个表达式在每个实例结束时执行。如果表达式返回true，所有其他的实例都会销毁，多实例节点也会结束，流程会继续执行。这个表达式必须定义在completionCondition 子元素中。

	<userTask id="miTasks" name="My Task" activiti:assignee="${assignee}">
	  <multiInstanceLoopCharacteristics isSequential="false" 
	     activiti:collection="assigneeList" activiti:elementVariable="assignee" >
	    <completionCondition>${nrOfCompletedInstances/nrOfInstances >= 0.6 }</completionCondition>
	  </multiInstanceLoopCharacteristics>
	</userTask>

在这里例子中，会为 assigneeList 集合的每个元素创建一个并行的实例。 当60%的任务完成时，其他任务就会删除，流程继续执行。

####Boundary events and multi-instance 边界事件和多实例

因为多实例是一个普通节点，它也可以在边缘使用边界事件。 对于中断型边界事件，当捕获事件时，所有激活的实例都会销毁。 参考以下多实例子流程

![](http://99btgc01.info/uploads/2014/12/bpmn.multi.instance.boundary.event.png)

这里，子流程的所有实例都会在定时器触发时销毁，无论有多少实例， 也不管内部哪个节点没有完成。

###Compensation Handlers 补偿处理器

####Description 描述

[试验]

如果一个节点用来补偿另一个节点的业务，它可以声明为一个补偿处理器。 补偿处理器不包含普通的流，只在补偿事件触发时执行。

补偿处理器不能包含进入和外出顺序流。

补偿处理器必须使用直接关联分配给一个补偿边界事件。

####Graphical notation 图形标记

如果节点是补偿处理器，补偿事件图标会显示在中间底部区域。下面的流程图显示了一个服务任务，附加了一个补偿边界事件， 并分配了一个补偿处理器。 注意 "cancel hotel reservation" 服务任务中间底部区域显示的补偿处理器图标。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.compensation.event%281%29.png)

####XML representation 内容

为了声明作为补偿处理器的节点，我们需要把 isForCompensation 设置为 true：

	<serviceTask id="undoBookHotel" isForCompensation="true" activiti:class="...">
	</serviceTask>
 