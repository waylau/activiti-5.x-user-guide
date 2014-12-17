##Events 事件

事件用来表明流程的生命周期中发生了什么事。 事件总是画成一个圆圈。 在BPMN 2.0 中，事件有两大分类：捕获（catching） 或 触发（throwing） 事件。

* 捕获（Catching）：当流程执行到事件， 它会等待被触发。触发的类型是由内部图表或 XML 中的类型声明来决定的。 捕获事件与触发事件在显示方面是根据内部图表是否被填充来区分的（白色的）。
* 触发（Throwing）：当流程执行到事件， 会触发一个事件。触发的类型是由内部图表或 XML 中的类型声明来决定的。 触发事件与捕获事件在显示方面是根据内部图表是否被填充来区分的（被填充为黑色）。

###Event Definitions 事件定义

事件定义决定了事件的语义。如果没有事件定义，这个事件就不做什么特别的事情。 没有设置事件定义的开始事件不会在启动流程时做任何事情。如果给开始事件添加了一个事件定义（比如定时器事件定义）我们就声明了开始流程的事件 "类型 " （这时定时器事件监听器会在某个时间被触发）。

###<a name="Timer Event Definitions 定时器事件定义">Timer Event Definitions 定时器事件定义</a>

定时器事件是根据指定的时间触发的事件。可以用于 [开始事件](#Start Events 开始事件)，[中间事件](#Intermediate Catching Events 中间捕获事件) 或 [边界事件](#Boundary Events 边界事件)。

定时器定义必须包含下面介绍的一个元素：

* timeDate。使用 [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601#Dates) 格式指定一个确定的时间，触发事件的时间。示例：

	<timerEventDefinition>
	    <timeDate>2011-03-11T12:13:14</timeDate>
	</timerEventDefinition>

* timeDuration。指定定时器之前要等待多长时间， timeDuration 可以设置为 timerEventDefinition 的子元素。 使用[ISO 8601](http://en.wikipedia.org/wiki/ISO_8601#Dates) 规定的格式 （由 BPMN 2.0 规定）。示例（等待10天）。

	<timerEventDefinition>
	    <timeDuration>P10D</timeDuration>
	</timerEventDefinition>

* timeCycle。指定重复执行的间隔，可以用来定期启动流程实例，或为超时时间发送多个提醒。 timeCycl e元素可以使用两种格式。第一种是 [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601#Dates) 标准的格式。示例（重复3次，每次间隔10小时）：
 
	<timerEventDefinition>
	    <timeCycle>R3/PT10H</timeCycle>
	</timerEventDefinition>

另外，你可以使用 cron 表达式指定 timeCycle，下面的例子是从整点开始，每5分钟执行一次：
	
	0 0/5 * * * ?

请参考 cron 表达式 [教程](http://www.quartz-scheduler.org/docs/tutorials/crontrigger.html)来了解如何使用 

*注意： 第一个数字表示秒，而不是像通常 Unix cron 中那样表示分钟。
重复的时间周期能更好的处理相对时间，它可以计算一些特定的时间点（比如，用户任务的开始时间），而 cron 表达式可以处理绝对时间 - 这对[定时开始事件](#Timer Start Event 定时开始事件)特别有用。*

你可以在定时器事件定义中使用表达式，这样你就可以通过流程变量来影响那个定时器定义。 流程定义必须包含ISO 8601（或cron）格式的字符串，以匹配对应的时间类型。

	<boundaryEvent id="escalationTimer" cancelActivity="true" attachedToRef="firstLineSupport">
	     <timerEventDefinition>
	      <timeDuration>${duration}</timeDuration>
	    </timerEventDefinition>
	  </boundaryEvent>

注意： 只有启用 job 执行器之后，定时器才会被触发。 （activiti.cfg.xml 中的 jobExecutorActivate 需要设置为 true， 不过，默认 job 执行器是关闭的）。

###<a name="Error Event Definitions 错误事件定义">Error Event Definitions 错误事件定义</a>

错误事件是由指定错误触发的。

重要提醒：BPMN 错误与 Java 异常完全不一样。 实际上，他俩一点儿共同点都没有。BPMN 错误事件是为了对业务异常建模。Java 异常是要用[特定方式](file:///F:/%E3%80%90Activiti%E3%80%91/activiti-5.16.4/activiti-5.16.4/docs/userguide/index.html#serviceTaskExceptionHandling)处理。

错误事件定义会引用一个 error 元素。下面是一个 error 元素的例子，引用了一个错误声明：

	<endEvent id="myErrorEndEvent">
	  <errorEventDefinition errorRef="myError" />
	</endEvent>  

引用相同 error 元素的错误事件处理器会捕获这个错误。

###<a name="Signal Event Definitions 信号事件定义">Signal Event Definitions 信号事件定义</a>

信号事件会引用一个已命名的信号。信号全局范围的事件（广播语义）。 会发送给所有激活的处理器。

信号事件定义使用 signalEventDefinition 元素。 signalRef 属性会引用 definitions 根节点里定义的 signal 子元素。 下面是一个流程的实例，其中会抛出一个信号，并被中间事件捕获。

	<definitions... >
	        <!-- declaration of the signal -->
	        <signal id="alertSignal" name="alert" />
	        
	        <process id="catchSignal">
	                <intermediateThrowEvent id="throwSignalEvent" name="Alert">
	                        <!-- signal event definition -->
	                        <signalEventDefinition signalRef="alertSignal" />
	                </intermediateThrowEvent>
	                ...
	                <intermediateCatchEvent id="catchSignalEvent" name="On Alert">
	                        <!-- signal event definition -->
	                        <signalEventDefinition signalRef="alertSignal" />
	                </intermediateCatchEvent>
	                ...             
	        </process>
	</definitions>

signalEventDefinition 引用相同的 signal 元素。

####Throwing a Signal Event 触发信号事件

既可以通过 bpmn 节点由流程实例触发一个信号，也可以通过 API 触发。 下面的 org.activiti.engine.RuntimeService 中的方法 可以用来手工触发一个信号。

	RuntimeService.signalEventReceived(String signalName);
	RuntimeService.signalEventReceived(String signalName, String executionId);

signalEventReceived(String signalName); 和 signalEventReceived(String signalName, String
executionId); 之间的区别是 第一个方法会把信号发送给全局所有订阅的处理器（广播语义）， 第二个方法只把信息发送给指定的执行。

####Catching a Signal Event 捕获信号事件

信号事件可以被中间捕获信号事件或边界信息事件捕获。

####Querying for Signal Event subscriptions 查询信号事件的订阅

可以查询所有订阅了特定信号事件的执行：
	
	List<Execution> executions = runtimeService.createExecutionQuery()
	      .signalEventSubscriptionName("alert")
	      .list();

我们可以使用 signalEventReceived(String signalName, String executionId) 方法把信号发送给这些执行。

####Signal event scope 信号事件范围

默认，信号会在流程引擎范围内进行广播。就是说，你可以在一个流程实例中抛出一个信号事件，其他不同流程定义的流程实例都可以监听到这个事件。

然而，有时只希望在同一个流程实例中响应这个信号事件。 比如一个场景是，流程实例中的同步机制，如果两个或更多活动是互斥的。

如果想要限制信号事件的范围，可以使用信号事件定义的 scope 属性（不是 BPMN2.0 的标准属性）：

	<signal id="alertSignal" name="alert" activiti:scope="processInstance"/>

这个属性值默认是 "global"

####Signal Event example(s) 信号事件实例

下面是两个不同流程使用信号交互的例子。第一个流程在保险规则更新或改变时启动。 在修改被参与者处理时，会触发一个信息，通知规则改变：

![](http://99btgc01.info/uploads/2014/12/bpmn.signal.event.throw.png)

这个时间会被所有感兴趣的流程实例捕获。下面是一个订阅这个事件的流程实例。

![](http://99btgc01.info/uploads/2014/12/bpmn.signal.event.catch.png)

注意：要了解信号事件是广播给**所有**激活的处理器的。 这意味着在上面的例子中，所有流程实例都会接收到这个事件。 这就是我们想要的。然而，有的情况下并不想要这种广播行为。 考虑下面的流程

![](http://99btgc01.info/uploads/2014/12/bpmn.signal.event.warning.1.png)

上述流程描述的模式 activiti 并不支持。这种想法是执行“do something”任务时出现的错误，会被边界错误事件捕获，然后使用信号传播给并发路径上的分支，进而中断"do something inparallel"任务。 目前，activiti 实际运行的结果与期望一致。信号会传播给边界事件并中断任务。**但是，根据信号的广播含义，它也会传播给所有其他订阅了信号事件的流程实例。** 所以，这就不是我们想要的结果。

注意： 信号事件不会执行任何与特定流程实例的联系。 如果你只想把一个信息发给指定的流程实例，需要手工关联，再使用 signalEventReceived(String signalName, String executionId) 和对应的[查询机制](http://activiti.org/userguide/index.html#bpmnSignalEventDefinitionQuery)。

###<a name="Message Event Definitions 消息事件定义">Message Event Definitions 消息事件定义</a>

消息事件的事件消息引用命名。一个消息有一个名字和一个有效载荷。不像一个信号，一个消息事件总是指向一个接收器。

一个消息事件的定义是使用 messageeventdefinition 元素声明。属性messageref 引用一个消息元素声明为定义的根元素的子元素。以下是摘录的一个过程，其中两个消息事件声明的开始事件和中间捕捉事件消息引用。

	<definitions id="definitions" 
	  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	  xmlns:activiti="http://activiti.org/bpmn"
	  targetNamespace="Examples"
	  xmlns:tns="Examples">
	  
	  <message id="newInvoice" name="newInvoiceMessage" />
	  <message id="payment" name="paymentMessage" />
	  
	  <process id="invoiceProcess">  
	  
	    <startEvent id="messageStart" >
	        <messageEventDefinition messageRef="newInvoice" />
	    </startEvent>
	    ...    
	    <intermediateCatchEvent id="paymentEvt" >
	        <messageEventDefinition messageRef="payment" />
	    </intermediateCatchEvent>
	    ...
	  </process>
	
	</definitions>

####Throwing a Message Event 触发消息事件

作为一个嵌入式的流程引擎，activit i不能真正接收一个消息。这些环境相关，与平台相关的活动 比如连接到 JMS（Java消息服务）队列或主题或执行 WebService 或 REST 请求。 这个消息的接收是你要在应用或架构的一层实现的，流程引擎则内嵌其中。

在你的应用接收一个消息之后，你必须决定如何处理它。 如果消息应该触发启动一个新流程实例， 在下面的 RuntimeService 的两个方法中选择一个执行：
	
	ProcessInstance startProcessInstanceByMessage(String messageName);
	ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
	ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object> processVariables);           

这些方法允许使用对应的消息系统流程实例。

如果消息需要被运行中的流程实例处理，首先要根据消息找到对应的流程实例 （参考下一节）然后触发这个等待中的流程。 RuntimeService 提供了如下方法可以基于消息事件的订阅来触发流程继续执行

	void messageEventReceived(String messageName, String executionId);
	void messageEventReceived(String messageName, String executionId, HashMap<String, Object> processVariables);    

####Querying for Message Event subscriptions 查询消息事件的订阅

Activiti 支持消息开始事件和中间消息事件。

* 消息开始事件的情况，消息事件订阅分配给一个特定的 process definition 。这个消息订阅可以使用 ProcessDefinitionQuery 查询到：

	ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
	      .messageEventSubscription("newCallCenterBooking")
	      .singleResult();

因为同时只能有一个流程定义关联到消息的订阅点，查询总是返回0或一个结果。 如果流程定义更新了， 那么只有最新版本的流程定义会订阅到消息事件上。

* 中间捕获消息事件的情况，消息事件订阅会分配给特定的执行。 这个消息事件订阅可以使用 ExecutionQuery 查询到：

	Execution execution = runtimeService.createExecutionQuery()
	      .messageEventSubscriptionName("paymentReceived")
	      .variableValueEquals("orderId", message.getOrderId())
	      .singleResult();

这个查询可以调用对应的查询，通常是流程相关的信息 （这里，最多只能有一个流程实例对应着 orderId ）。

####Message Event example(s) 消息事件实例

下面是一个使用两个不同消息启动的流程实例：

![](http://99btgc01.info/uploads/2014/12/bpmn.start.message.event.example.1.png)

可以用在，流程需要不同的方式来区分开始事件，而后最终会进入同样的路径。

###<a name="Start Events 开始事件">Start Events 开始事件</a>

开始事件用来指明流程在哪里开始。开始事件的类型（流程在接收事件时启动， 还是在指定时间启动，等等），定义了流程如何启动， 这通过事件中不同的小图表来展示。 在 XML 中，这些类型是通过声明不同的子元素来区分的。

开始事件**都是捕获事件**： 最终这些事件都是（一直）等待着，直到对应的触发时机出现。

在开始事件中，可以设置下面的 activiti 特定属性：

* initiator：当流程启动时，把当前登录的用户保存到哪个变量名中。 示例如下：

	<startEvent id="request" activiti:initiator="initiator" />

登录的用户必须使用 IdentityService.setAuthenticatedUserId(String) 方法设置， 并像这样包含在 try-finally 代码中：

	try {
	  identityService.setAuthenticatedUserId("bono");
	  runtimeService.startProcessInstanceByKey("someProcessKey");
	} finally {
	  identityService.setAuthenticatedUserId(null);
	}

这段代码来自 Activiti Explorer，所以它可以和[Chapter 9. Forms 表单](../Chapter 9. Forms 表单/README.md)一起结合使用。

###None Start Event 空开始事件

####Description 描述

空开始事件技术上意味着没有指定启动流程实例的触发条件。 这就是说引擎不能预计什么时候流程实例会启动。 空开始事件用于，当流程实例要通过API 启动的场景， 通过调用 startProcessInstanceByXXX 方法。

	ProcessInstance processInstance = runtimeService.startProcessInstanceByXXX();

注意： 子流程都有一个空开始事件。

####Graphical notation 图形标记

空开始事件显示成一个圆圈，没有内部图表（没有触发类型）

![](http://99btgc01.info/uploads/2014/12/bpmn.none.start.event.png)

####XML representation 内容

空开始事件的XML结构是普通的开始事件定义，没有任何子元素 （其他开始事件类型都有一个子元素来声明自己的类型）

	<startEvent id="start" name="my start event" />

####Custom extensions for the none start event 空开始事件的自定义扩展

formKey：引用用户在启动新流程实例时需要填写的表单模板， 更多信息可以参考[Chapter 9. Forms 表单](../Chapter 9. Forms 表单/README.md)。 实例：

	<startEvent id="request" activiti:formKey="org/activiti/examples/taskforms/request.form" />

###<a name="Timer Start Event 定时开始事件">Timer Start Event 定时开始事件</a>

####Description 描述

定时开始事件用来在指定的时间创建流程实例。 它可以同时用于只启动一次的流程和应该在特定时间间隔启动多次的流程。

注意：子流程不能使用定时开始事件。

注意：定时开始事件在流程发布后就会开始计算时间。 不需要调用 startProcessInstanceByXXX ，虽然也而已调用启动流程的方法， 但是那会导致调用 startProcessInstanceByXXX 时启动过多的流程。

注意：当包含定时开始事件的新版本流程部署时，对应的上一个定时器就会被删除。这是因为通常不希望自动启动旧版本流程的流程实例。

####Graphical notation 图形标记

定时开始事件显示为了一个圆圈，内部是一个表。

![](http://99btgc01.info/uploads/2014/12/bpmn.clock.start.event.png)

####XML representation 内容

定时开始事件的XML内容是普通开始事件的声明，包含一个定时定义子元素。 请参考[Timer Event Definitions 定时器事件定义](#Timer Event Definitions 定时器事件定义) 查看配合细节。

示例：流程会启动4次，每次间隔5分钟，从2011年3月11日，12:13开始计时。

    <startEvent id="theStart">
            <timerEventDefinition>
                <timeCycle>R4/2011-03-11T12:13/PT5M</timeCycle>
            </timerEventDefinition>
     </startEvent>

示例：流程会根据选中的时间启动一次。

	<startEvent id="theStart">
            <timerEventDefinition>
                <timeDate>2011-03-11T12:13:14</timeDate>
            </timerEventDefinition>
    </startEvent>

###Message Start Event

####Description 描述

[消息](#Message Event Definitions 消息事件定义)开始事件可以用其使用一个命名的消息来启动流程实例。 这样可以帮助我们使用消息名称来选择正确的开始事件。

在**发布**包含一个或多个消息开始事件的流程定义时，需要考虑下面的条件：

* 消息开始事件的名称在给定流程定义中不能重复。流程定义不能包含多个名称相同的消息开始事件。 如果两个或以上消息开始事件应用了相同的事件，或两个或以上消息事件引用的消息名称相同，activiti 会在发布流程定义时抛出异常。
* 消息开始事件的名称在所有已发布的流程定义中不能重复。 如果一个或多个消息开始事件引用了相同名称的消息，而这个消息开始事件已经部署到不同的流程定义中， activiti 就会在发布时抛出一个异常。
* 流程版本：在发布新版本的流程定义时，之前订阅的消息订阅会被取消。 如果新版本中没有消息事件也会这样处理。

启动流程实例，消息开始事件可以使用 下列 RuntimeService 中的方法来触发：

	ProcessInstance startProcessInstanceByMessage(String messageName);
	ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
	ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object< processVariables); 
                                
这里的 messageName 是 messageEventDefinition 的 messageRef 属性引用的 message 元素的 name 属性。 **启动**流程实例时，要考虑一下因素：

* 消息开始事件只支持顶级流程。消息开始事件不支持内嵌子流程。
* 如果流程定义有多个消息开始事件，runtimeService.startProcessInstanceByMessage(...) 会选择
对应的开始事件。
* 如果流程定义有多个消息开始事件和一个空开始事件。
runtimeService.startProcessInstanceByKey(...) 和 runtimeService.startProcessInstanceById(...) 会使用空开始事件启动流程实例。
* 如果流程定义有多个消息开始事件，而且没有空开始事件，
runtimeService.startProcessInstanceByKey(...) 和 runtimeService.startProcessInstanceById(...) 会抛出异常。
* 如果流程定义只有一个消息开始事件， runtimeService.startProcessInstanceByKey(...) 和
runtimeService.startProcessInstanceById(...) 会使用这个消息开始事件启动流程实例。
* 如果流程被调用环节（callActivity）启动，消息开始事件只支持如下情况：
* 在消息开始事件以外，还有一个单独的空开始事件
* 流程只有一个消息开始事件，没有空开始事件。

####Graphical notation 图形标记

消息开始事件是一个圆圈，中间是一个消息事件图标。图标是白色未填充的，来表示捕获（接收）行为。

![](http://99btgc01.info/uploads/2014/12/bpmn.start.message.event.png)

####XML representation 内容

消息开始事件的XML内容时在普通开始事件申请中包含一个 messageEventDefinition 子元素：

	<definitions id="definitions" 
	  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	  xmlns:activiti="http://activiti.org/bpmn"
	  targetNamespace="Examples"
	  xmlns:tns="Examples">
	  
	  <message id="newInvoice" name="newInvoiceMessage" />
	  
	  <process id="invoiceProcess">  
	  
	    <startEvent id="messageStart" >
	        <messageEventDefinition messageRef="tns:newInvoice" />
	    </startEvent>
	    ...    
	  </process>
	
	</definitions>

###Signal Start Event

####Description 描述

[signal](#Signal Event Definitions 信号事件定义) 开始事件，可以用来通过一个已命名的信号（signal）来启动一个流程实例。 信号可以在流程实例内部使用“中间信号抛出事务”触发， 也可以通过 API (runtimService.signalEventReceivedXXX 方法) 触发。两种情况下， 所有流程实例中拥有相同名称的 signalStartEvent 都会启动。

注意，在两种情况下，都可以选择同步或异步的方式启动流程实例。

必须向 API 传入 signalName， 这是 signal 元素的 name 属性值， 它会被 signalEventDefinition 的 signalRef 属性引用。

####Graphical notation 图形标记

信号开始事件显示为一个中间包含信号事件图标的圆圈。标记是无填充的，表示捕获（接收）行为

![](http://99btgc01.info/uploads/2014/12/bpmn.start.signal.event.png)

####XML representation 内容

signalStartEvent 的 XML 格式是标准的 startEvent 声明，其中包含一个 signalEventDefinition 子元素：

 <signal id="theSignal" name="The Signal" />

    <process id="processWithSignalStart1">
        <startEvent id="theStart">
          <signalEventDefinition id="theSignalEventDefinition" signalRef="theSignal"  />
        </startEvent>
        <sequenceFlow id="flow1" sourceRef="theStart" targetRef="theTask" />
        <userTask id="theTask" name="Task in process A" />
        <sequenceFlow id="flow2" sourceRef="theTask" targetRef="theEnd" />
        <endEvent id="theEnd" />
    </process>

###Error Start Event

####Description 描述

[错误](#Error Event Definitions 错误事件定义)开始事件可以用来触发一个事件子流程。 错误开始事件不能用来启动流程实例。

错误开始事件都是中断事件。

####Graphical notation 图形标记

错误开始事件是一个圆圈，包含一个错误事件标记。标记是白色未填充的，来表示捕获（接收）行为。

![](http://99btgc01.info/uploads/2014/12/bpmn.start.error.event.png)

####XML representation 内容

错误开始事件的 XML 内容是普通开始事件定义中，包含一个 errorEventDefinition 子元素。

	<startEvent id="messageStart" >
	        <errorEventDefinition errorRef="someError" />
	</startEvent>

###End Events 结束事件

结束事件表示（子）流程（分支）的结束。 结束事件**都是触发事件**。 这是说当流程达到结束事件，会触发一个结果。 结果的类型是通过事件的内部黑色图标表示的。 在XML内容中，是通过包含的子元素声明的。

###None End Event 空结束事件

####Description 描述

空结束事件意味着到达事件时不会指定抛出的结果。 这样，引擎会直接结束当前执行的分支，不会做其他事情

####Graphical notation 图形标记

空结束事件是一个粗边圆圈，内部没有小图表（无结果类型）

![](http://99btgc01.info/uploads/2014/12/bpmn.none.end.event.png)

####XML representation 内容

空结束事件的XML内容是普通结束事件定义，不包含子元素(其他结束事件类型都会包含声明类型的子元素)

	<endEvent id="end" name="my end event" />

###<a name="Error End Event 错误结束事件">Error End Event 错误结束事件</a>

####Description 描述

当流程执行到**错误结束事件**，流程的当前分支就会结束，并抛出一个错误。 这个错误可以被对应的中间[边界错误](#"Boundary Events 边界事件)事件捕获。 如果找不到匹配的边界错误事件，就会抛出一个异常。

####Graphical notation 图形标记

错误结束事件是一个标准的结束事件（粗边圆圈），内部有错误图标。 错误图表是全黑的，表示触发语法。

![](http://99btgc01.info/uploads/2014/12/bpmn.error.end.event.png)

####XML representation 内容

错误结束事件的内容是一个错误事件， 子元素为errorEventDefinition。

	<endEvent id="myErrorEndEvent">
	  <errorEventDefinition errorRef="myError" />
	</endEvent>          

errorRef 属性引用定义在流程外部的 error 元素：

	<error id="myError" errorCode="123" />
	...
	<process id="myProcess">  
	...      

error 的 errorCode 用来查找 匹配的捕获边界错误事件。 如果 errorRef 与任何 error 都不匹配， 就会使用errorRef来作为errorCode 的缩写。 这是 activiti 特定的缩写。 更具体的
说，见如下代码：

	<error id="myError" errorCode="error123" />
	...
	<process id="myProcess">  
	...  
	  <endEvent id="myErrorEndEvent">
	    <errorEventDefinition errorRef="myError" />
	  </endEvent>

等价于

	<endEvent id="myErrorEndEvent">
	  <errorEventDefinition errorRef="error123" />
	</endEvent> 

注意 errorRef 必须与 BPMN 2.0 格式相符， 必须是一个合法的 QName。

###Cancel End Event 取消结束事件

[试验]

####Description 描述

取消结束事件只能与BPMN事务子流程结合使用。 当到达取消结束事件时，会抛出取消事件，它必须被取消边界事件捕获。 取消边界事件会取消事务，并触发补偿机制。

####Graphical notation 图形标记

取消结束事件显示为标准的结束事件（粗边圆圈），包含一个取消图标。 取消图标是全黑的，表示触发语法

![](http://99btgc01.info/uploads/2014/12/bpmn.cancel.end.event.png)

####XML representation 内容

取消结束事件内容是一个结束事件， 包含 cancelEventDefinition 子元素。

	<endEvent id="myCancelEndEvent">
	  <cancelEventDefinition />
	</endEvent>     

###<a name="Boundary Events 边界事件">Boundary Events 边界事件</a>

边界事件都是捕获事件，它会附在一个环节上。 （边界事件不可能触发事件）。这意味着，当节点运行时， 事件会监听对应的触发类型。 当事件被捕获，节点就会中断， 同时执行事件的后续连线。

所以边界事件的定义方式都一样：

	<boundaryEvent id="myBoundaryEvent" attachedToRef="theActivity">
	      <XXXEventDefinition/>
	</boundaryEvent>

边界事件使用如下方式进行定义：

* 唯一标识（流程范围）
* 使用 caught 属性 引用事件衣服的节点。 注意边界事件和它们附加的节点在同一级别上。（比如，边界事件不是包含在节点内的）。
* 格式为 XXXEventDefinition 的 XML 子元素 （比如，TimerEventDefinition，ErrorEventDefinition，等等） 定义了边界事件的类型。参考对应的边界事件类型， 获得更多细节。

###Timer Boundary Event 定时边界事件

####Description 描述

定时边界事件就是一个暂停等待警告的时钟。当流程执行到绑定了边界事件的环节， 会启动一个定时器。 当定时器触发时（比如，一定时间之后），环节就会中断， 并沿着定时边界事件的外出连线继续执行。

####Graphical notation 图形标记

定时边界事件是一个标准的边界事件（边界上的一个圆圈），内部是一个定时器小图标。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.timer.event.png)

####XML representation 内容

定时器边界任务定义是一个正规的[边界事件](#Boundary Events 边界事件)。 指定类型的子元素是 timerEventDefinition 元素。

	<boundaryEvent id="escalationTimer" cancelActivity="true" attachedToRef="firstLineSupport">
	   <timerEventDefinition>
	    <timeDuration>PT4H</timeDuration>
	  </timerEventDefinition>
	</boundaryEvent>    

请参考[定时事件定义](#Timer Event Definitions 定时器事件定义)获得更多定时其配置的细节。

在流程图中，可以看到上述例子中的圆圈边线是虚线：

![](http://99btgc01.info/uploads/2014/12/bpmn.non.interrupting.boundary.timer.event.png)

经典场景是发送一个升级邮件，但是不打断正常流程的执行。

因为 BPMN 2.0 中，中断和非中断的事件还是有区别的。默认是中断事件。 非中断事件的情况，不会中断原始环节，那个环节还停留在原地。 对应的，会创建一个新分支，并沿着事件的流向继续执行。 在XML内容中，要把cancelActivity 属性设置为 false：

	<boundaryEvent id="escalationTimer" cancelActivity="false" attachedToRef="firstLineSupport"/>

注意：边界定时事件只能在job执行器启用时使用。（比如，把activiti.cfg.xml 中的 jobExecutorActivate 设置为 true，因为默认 job 执行器默认是禁用的）

####Known issue with boundary events 边界事件的已知问题

使用边界事件有一个已知的同步问题。 目前，不能边界事件后面不能有多条外出连线 （参考[ACT-47](http://jira.codehaus.org/browse/ACT-47)。 解决这个问题的方法是在一个连线后使用并发网关

![](http://99btgc01.info/uploads/2014/12/bpmn.known.issue.boundary.event.png)

###Error Boundary Event 错误边界事件

####Description 描述

节点边界上的中间捕获错误事件， 或简写成**边界错误事件**，它会捕获节点范围内抛出的错误。

定义一个边界错误事件，大多用于内嵌[子流程](http://activiti.org/userguide/index.html#bpmnSubProcess)，或[调用节点](http://activiti.org/userguide/index.html#bpmnCallActivity)，对于子流程的情况，它会为所有内部的节点创建一个作用范围。 错误是由[错误结束事件](Error End Event 错误结束事件)抛出的。 这个错误会传递给上层作用域，直到找到一个错误事件定义向匹配的边界错误事件。

当捕获了错误事件时，边界任务绑定的节点就会销毁， 也会销毁内部所有的执行分支 （比如，同步节点，内嵌子流程，等等）。 流程执行会继续沿着边界事件的外出连线继续执行。

####Graphical notation 图形标记

边界错误事件显示成一个普通的中间事件（圆圈内部有一个小圆圈） 放在节点的标记上，内部有一个错误小图标。错误小图标是白色的，表示它是一个捕获事件。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.error.event.png)

####XML representation 内容

边界错误事件定义为普通的[边界事件](#Boundary Events 边界事件)：

	<boundaryEvent id="catchError" attachedToRef="mySubProcess">
	  <errorEventDefinition errorRef="myError"/>
	</boundaryEvent>

和错误结束事件一样， errorRef 引用了 process 元素外部的一个错误定义：

	<error id="myError" errorCode="123" />
	...
	<process id="myProcess">
	...

errorCode 用来匹配捕获的错误：

* 如果没有设置 errorRef，边界错误事件会捕获 所有错误事件，无论错误的 errorCode是什么。
* 如果设置了errorRef，并引用了一个已存的错误，边界事件就只捕获错误代码与之相同的错误。
* 如果设置了errorRef，但是BPMN 2.0中没有定义错误， errorRef 就会当做 errorCode 使用（和错误结束事件的用法类似）

####Example 示例

下面的流程实例演示了如何使用错误结束事件。 当完成'审核盈利'这个用户任务是，如果没有提供足够的信息， 就会抛出错误，错误会被子流程的边界任务捕获， 所有'回顾销售'子流程中的所有节点都会销毁。 （即使'审核客户比率'还没有完成），并创建一个'提供更多信息'的用户任务。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.error.example.png)

这个流程也放在demo中了。流程XML和单元测试可以在 org.activiti.examples.bpmn.event.error 包下找到

###Signal Boundary Event 信号边界事件

####Description 描述

节点边界的中间捕获[信号](#Signal Event Definitions 信号事件定义)， 或简称为边界信号事件，它会捕获信号定义引用的相同信号名的信号。

注意：与其他事件（比如边界错误事件）不同，边界信号事件不只捕获 它绑定方位的信号。

信号事件是一个全局的范围（广播语义），就是说信号可以在任何地方触发， 即便是不同的流程实例。

注意：和其他事件（比如边界错误事件）不同，捕获信号后，不会停止信号的传播。 如果你有两个信号边界事件，它们捕获相同的信号事件，两个边界事件都会被触发， 即使它们在不同的流程实例中。


####Graphical notation 图形标记

边界信号事件显示为普通的中间事件（圆圈里有个小圆圈），位置在节点的边缘， 内部有一个信号小图标。信号图标是白色的（未填充）， 来表示捕获的意思。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.signal.event.png)

####XML representation 内容

边界信号事件定义为普通的边界事件：
	
	<boundaryEvent id="boundary" attachedToRef="task" cancelActivity="true">       
	          <signalEventDefinition signalRef="alertSignal"/>
	</boundaryEvent>
                  
####Example 实例

参考[信号事件定义]((#Signal Event Definitions 信号事件定义))章节

###Message Boundary Event 消息边界事件

####Description 描述

节点边界上的中间捕获消息， 或简称边界消息事件，根据引用的消息定义捕获相同消息名称的消息。

####Graphical notation 图形标记

边界[消息](#Message Event Definitions 消息事件定义)事件显示成一个普通的中间事件（圆圈里有个小圆圈），位于节点边缘， 内部是一个消息小图标。消息图标是白色（无填充），表示捕获语义

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.message.event.png)

注意，边界消息事件可能是中断（右侧）或非中断（左侧）的。

####XML representation 内容

边界消息事件定义为标准的边界事件：

	<boundaryEvent id="boundary" attachedToRef="task" cancelActivity="true">       
	          <messageEventDefinition messageRef="newCustomerMessage"/>
	</boundaryEvent>

####Example 实例

参考[消息事件定义](#Message Event Definitions 消息事件定义)章节

###Cancel Boundary Event 取消边界事件

[试验]

####Description 描述

在事务性子流程的边界上的中间捕获取消，或简称为边界取消事件 cancel event， 当事务取消时触发。当取消边界事件触发时，首先中断当前作用域的所有执行。 然后开始补偿事务内的所有激活的补偿边界事件。 补偿是同步执行的。例如，离开事务钱，边界事务会等待补偿执行完毕。 当补偿完成后，事务子流程会沿着取消边界事务的外出连线继续执行。

注意：每个事务子流程只能有一个取消边界事件。

注意：如果事务子流程包含内嵌子流程，补偿只会触发已经成功完成的子流程。

注意：如果取消边界子流程对应的事务子流程配置为多实例， 如果一个实例触发了取消，就会取消所有实例。 

####Graphical notation 图形标记

取消边界事件显示为了一个普通的中间事件（圆圈里套小圆圈），在节点的边缘，内部是一个取消小图标。取消图标是白色（无填充），表明是捕获语义。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.cancel.event.png)

####XML representation 内容

取消边界事件定义为普通[边界事件](#Boundary Events 边界事件)：

	<boundaryEvent id="boundary" attachedToRef="transaction" >       
	          <cancelEventDefinition />
	</boundaryEvent>
                        
因为取消边界事件都是中断的，所以不需要使用 cancelActivity 属性。

###Compensation Boundary Event 补偿边界事件

[试验]

####Description 描述

节点边界的中间捕获补偿， 或简称为补偿边界事件， 可以用来设置一个节点的补偿处理器。

补偿边界事件必须使用直接引用设置唯一的补偿处理器。

补偿边界事件与其他边界事件的策略不同。 其他边界事件（比如信号边界事件）当到达关联的节点就会被激活。 离开节点时，就会挂起，对应的事件订阅也会取消。 补偿边界事件则不同。补偿边界事件在关联的节点成功完成时激活。 当补偿事件触发或对应流程实例结束时，事件订阅才会删除。 它遵循如下规则：

* 补偿触发时，补偿边界事件对应的补偿处理器会调用相同次数，根据它对应的节点的成功次数。
* 如果补偿边界事件关联到多实例节点， 补偿事件会订阅每个实例。
* 如果补偿边界事件关联的节点中包含循环， 补偿事件会在每次节点执行时进行订阅。
* 如果流程实例结束，订阅的补偿事件都会结束。

注意：补偿边界事件不支持内嵌子流程。

####Graphical notation 图形标记

补偿边界事件显示为标准中间事件（圆圈里套圆圈），位于节点边缘， 内部有一个补偿小图标。补偿图标是白色的（无填充），表示捕获语义。另外，下面的图形演示了使用无方向的关联，为边界事件设置补偿处理器。

![](http://99btgc01.info/uploads/2014/12/bpmn.boundary.compensation.event.png)

####XML representation 内容

补偿边界事件定义为标准[边界事件](#Boundary Events 边界事件)：

	<boundaryEvent id="compensateBookHotelEvt" attachedToRef="bookHotel" >       
	          <compensateEventDefinition />
	</boundaryEvent>
	
	<association associationDirection="One" id="a1"  sourceRef="compensateBookHotelEvt" targetRef="undoBookHotel" />
	
	<serviceTask id="undoBookHotel" isForCompensation="true" activiti:class="..." />

因为补偿边界事件在节点成功完成后激活， 所以不支持 cancelActivity属性。

###<a name="#Intermediate Catching Events 中间捕获事件">Intermediate Catching Events 中间捕获事件</a>
	
所有中间捕获事件都使用同样的方式定义：

	<intermediateCatchEvent id="myIntermediateCatchEvent" >
	      <XXXEventDefinition/>
	</intermediateCatchEvent>

中间捕获事件的定义包括

* 唯一标识（流程范围内）
* 一个结构为XXXEventDefinition的XML子元素 （比如 TimerEventDefinition等） 定义了中间捕获事件的类型。参考特定的捕获事件类型， 获得更多详情。

###Timer Intermediate Catching Event 定时中间捕获事件

####Description 描述

定时中间事件作为一个监听器。当执行到达捕获事件节点， 就会启动一个定时器。 当定时器触发（比如，一段时间之后），流程就会沿着定时中间事件的外出节点继续执行。

####Graphical notation 图形标记

定时器中间事件显示成标准中间捕获事件，内部是一个定时器小图标

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.timer.event.png)

####XML representation 内容

定时器中间事件定义为标准中间捕获事件。 指定类型的子元素为timerEventDefinition 元素

    <intermediateCatchEvent id="timer">
            <timerEventDefinition>
                <timeDuration>PT5M</timeDuration>
            </timerEventDefinition>
    </intermediateCatchEvent>

参考[定时器事件定义](#Timer Event Definitions 定时器事件定义)了解配置信息。

###Signal Intermediate Catching Event

####Description 描述

中间捕获[信号](#Signal Event Definitions 信号事件定义)事件 通过引用信号定义来捕获相同信号名称的信号。

注意：与其他事件（比如错误事件）不同，信号不会在捕获之后被消费。 如果你有两个激活的信号边界事件捕获相同的信号事件，两个边界事件都会被触发， 即便它们在不同的流程实例中。

####Graphical notation 图形标记

中间信号捕获事件显示为一个普通的中间事件（圆圈套圆圈）， 内部有一个信号小图标。信号小图标是白色的（无填充）， 表示捕获语义。

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.signal.catch.event.png)

####XML representation 内容

信号中间事件定义为普通的[中间捕获事件](#Intermediate Catching Events 中间捕获事件)。 对应类型的子元素是signalEventDefinition 元素。

	<intermediateCatchEvent id="signal">
	        <signalEventDefinition signalRef="newCustomerSignal" />
	</intermediateCatchEvent>

####Example 实例

参考[信号事件定义](#Signal Event Definitions 信号事件定义)章节。

###Message Intermediate Catching Event 消息中间捕获事件

####Description 描述

一个中间捕获[消息](#Message Event Definitions 消息事件定义)事件，捕获特定名称的消息

####Graphical notation 图形标记

中间捕获消息事件显示为普通中间事件（圆圈套圆圈），内部是一个消息小图标。消息图标是白色的（无填充）， 表示捕获语义。

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.message.catch.event.png)

####XML representation 内容

消息中间事件定义为标准[中间捕获事件](#Intermediate Catching Events 中间捕获事件)。 指定类型的子元素是messageEventDefinition 元素。

	<intermediateCatchEvent id="message">
	        <messageEventDefinition signalRef="newCustomerMessage" />
	</intermediateCatchEvent>

####Example 实例

参考[消息事件定义](#Message Event Definitions 消息事件定义)章节。

###Intermediate Throwing Event 内部触发事件

所有内部触发事件的定义都是同样

	<intermediateThrowEvent id="myIntermediateThrowEvent" >
	      <XXXEventDefinition/>
	</intermediateThrowEvent>

内部触发事件定义包含

* 唯一标识（流程范围）
* 使用格式为XXXEventDefinition的XML子元素 （比如signalEventDefinition 等） 定义中间触发事件的类型。 参考对应触发事件的类型，了解更多信息

###Intermediate Throwing None Event

下面的流程图演示了一个空中间触发事件的例子， 它通常用于表示流程中的某个状态。

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.none.event.png)

通过添加执行监听器，就可以很好地监控一些 KPI。

	<intermediateThrowEvent id="noneEvent">
	  <extensionElements>
	    <activiti:executionListener class="org.activiti.engine.test.bpmn.event.IntermediateNoneEventTest$MyExecutionListener" event="start" />
	  </extensionElements>
	</intermediateThrowEvent>

这里你可以添加自己的代码，把事件发送给BAM工具或DWH。引擎不会为这个事件做任何事情， 它直接径直通过

###Signal Intermediate Throwing Event 信号中间触发事件

####Description 描述

中间触发[信号](#Signal Event Definitions 信号事件定义)事件为定义的信号抛出一个信号事件。

在 activiti中，信号会广播到所有激活的处理器中（比如，所以捕获信号事件）。 信号可以通过同步和异步方式发布。

* 默认配置下，信号是同步发送的。就是说，抛出事件的流程实例会等到信号发送给所有捕获流程实例才继续执行。 捕获流程实例也会在触发流程实例的同一个事务中执行，意味着如果某个监听流程出现了技术问题（抛出异常），所有相关的实例都会失败。
* 信号也可以异步发送。这时它会在到达抛出信号事件后决定哪些处理器是激活的。 对这些激活的处理器，会保存一个异步提醒消息（任务），并发送给 jobExecutor。

####Graphical notation 图形标记

中间信号触发事件显示为普通中间事件（圆圈套圆圈），内部又一个信号小图标。信号图标是黑色的（有填充），表示触发语义。

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.signal.throw.event.png)

####XML representation 内容

消息中间事件定义为标准中间触发事件。 指定类型的子元素是signalEventDefinition 元素。

	<intermediateThrowEvent id="signal">
	        <signalEventDefinition signalRef="newCustomerSignal" />
	</intermediateThrowEvent>

异步信号事件如下所示：

	<intermediateThrowEvent id="signal">
	        <signalEventDefinition signalRef="newCustomerSignal" activiti:async="true" />
	</intermediateThrowEvent>

####Example 实例

参考[信号](#Signal Event Definitions 信号事件定义)事件定义章节。

###Compensation Intermediate Throwing Event

[试验]

####Description 描述

中间触发补偿事件 可以用来触发补偿。

触发补偿： 补偿可以由特定节点或包含补偿事件的作用域触发。 补偿是通过分配给节点的补偿处理器来完成的。

* 当补偿由节点触发，对应的补偿处理器会根据节点成功完成的次数执行相同次数。
* 如果补偿由当前作用域触发，当前作用域的所有节点都会执行补偿， 也包含并发分支。
* 补偿的触发是继承式的：如果执行补偿的节点是子流程，补偿会作用到子流程中包含的所有节点。 如果子流程是内嵌节点，补偿会递归触发。 然而，补偿不会传播到流程的上层： 如果补偿在子流程中触发，不会传播到子流程范围外。 bpmn规范定义，由节点触发的流程只会作用到“子流程同一级别”。
* activiti的补偿执行次序与流程执行顺序相反。 以为着最后完成的节点会最先执行补偿，诸如此类。
* 中间触发补偿事件可以用来补偿成功完成的事务性子流程。

注意： 如果补偿被一个包含子流程的作用域触发，子流程还包含了关联补偿处理器的节点，补偿只会传播到子流程，如果它已经成功完成了。 如果子流程中的节点也完成了，并关联了补偿处理器， 如果子流程包含的这些节点还没有完成，就不会执行补偿处理器。 参考下面实例：

![](http://99btgc01.info/uploads/2014/12/bpmn.throw.compensation.example1.png)

这个流程中，我们有两个并发分支，一些分支时内嵌子流程，一个是“使用信用卡”节点。假设两个分支都启动了，第一个分支等待用户完成“审核预定”任务。第二个分支执行“使用信用卡”节点， 并发生了一个错误，这导致“取消预定”事件，并触发补偿。 这时，并发子流程还没有结束，意味着补偿事件不会传播给子流程， 所以“取消旅店预定”这个补偿处理器不会执行。 如果用户任务（就是内嵌子流程）在“取消预定”之前完成了， 补偿就会传播给内嵌子流程。

流程变量： 当补偿内嵌子流程时，用来执行补偿处理器的分支可以访问子流程的本地流程实例， 因为这时它是子流程完成的分支。 为了实现这个功能，流程变量的快照会分配给分支（为执行子流程而创建的分支）。 为此，有以下限制条件：

* 补偿处理器无法访问子流程内部创建的，添加到同步分支的变量。
* 分配给分支的流程变量在继承关系上层的（分配给流程实例的流程变量没有包含在快照中）： 补偿触发时，补偿处理器通过它们所在的地方访问这些流程变量。
* 变量快照只用于内嵌子流程，不适用其他节点。

已知限制：

* waitForCompletion="false"还不支持。当补偿使用中间触发补偿事件触发时， 事件没有等待，
在补偿成功结束后。
* 补偿自己由并发分支执行。并发分支的执行顺序与被补偿的节点完成次序相反。 未来 activiti 可能支持选项来顺序执行补偿。
* 补偿不会传播给callActivity调用的子流程实例。

####Graphical notation 图形标记

中间补偿触发事件显示为标准中间事件（圆圈套圆圈），内部是一个补偿小图标。补偿图标是黑色的（有填充），表示触发语义。

![](http://99btgc01.info/uploads/2014/12/bpmn.intermediate.compensation.throw.event.png)

####XML representation 内容

补偿中间事件定义为普通的中间触发事件。 对应类型的子元素
是 compensateEventDefinition 元素。

	<intermediateThrowEvent id="throwCompensation">
	        <compensateEventDefinition />
	</intermediateThrowEvent>

另外，可选参数 activityRef 可以用来触发特定作用域/节点的补偿：

	<intermediateThrowEvent id="throwCompensation">
	        <compensateEventDefinition activityRef="bookHotel" />
	</intermediateThrowEvent>