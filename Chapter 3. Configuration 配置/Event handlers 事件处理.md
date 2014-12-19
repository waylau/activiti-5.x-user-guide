##Event handlers 事件处理

Activiti 5.15 中实现了一种事件机制。它允许在引擎触发事件时获得提醒。 参考[所有支持的事件类型](#Supported event types 支持的事件类型)了解有效的事件。

可以为对应的事件类型注册监听器，在这个类型的任何时间触发时都会收到提醒。 你可以添加引擎范围的事件监听器通过[配置](#Configuration and setup 配置和安装)， 添加引擎范围的事件监听器在[运行阶段使用API](#Adding listeners at runtime 运行时添加监听)， 或添加 event-listener 到[特定流程定义的 BPMN XML 中](#Adding listeners to process definitions 添加监听到流程定义)。

所有分发的事件，都是org.activiti.engine.delegate.event.ActivitiEvent 的子类。事件包含（如果有效）type，executionId，processInstanceId 和processDefinitionId。 对应的事件会包含事件发生时对应上下文的额外信息， 这些额外的载荷可以在[所有支持的事件类型](#Supported event types 支持的事件类型)中找到。

###Event listener implementation 事件监听实现

实现事件监听器的唯一要求是实现org.activiti.engine.delegate.event.ActivitiEventListener。 西面是一个实现监听器的例子，它会把所有监听到的事件打印到标准输出中，包括job执行的事件异常：

	public class MyEventListener implements ActivitiEventListener {
	
	  @Override
	  public void onEvent(ActivitiEvent event) {
	    switch (event.getType()) {
	    
	      case JOB_EXECUTION_SUCCESS:
	        System.out.println("A job well done!");
	        break;
	  
	      case JOB_EXECUTION_FAILURE:
	        System.out.println("A job has failed...");
	        break;
	        
	      default:
	        System.out.println("Event received: " + event.getType());
	    }
	  }
	
	  @Override
	  public boolean isFailOnException() {
	    // The logic in the onEvent method of this listener is not critical, exceptions
	    // can be ignored if logging fails...
	    return false;
	  }
	}

isFailOnException() 方法决定了当事件分发时，onEvent(..) 方法抛出异常时的行为。 这里返回的是 false，会忽略异常。 当返回 true 时，异常不会忽略，继续向上传播，迅速导致当前命令失败。 当事件是一个 API 调用的一部分时（或其他事务性操作，比如 job 执行）， 事务就会回滚。当事件监听器中的行为不是业务性时，建议返回 false。 activiti 提供了一些基础的实现，实现了事件监听器的常用场景。可以用来作为基类或监听器实现的样例：

* org.activiti.engine.delegate.event.BaseEntityEventListener： 这个事件监听器的基
类可以用来监听实体相关的事件，可以针对某一类型实体，也可以是全部实体。 它隐藏了
类型检测，并提供了三个需要重写的方法：onCreate(..), onUpdate(..) 和 onDelete(..)，当实
体创建，更新，或删除时调用。对于其他实体相关的事件，会调用 onEntityEvent(..)。

###<a name='Configuration and setup 配置和安装'>Configuration and setup 配置和安装</a>

把事件监听器配置到流程引擎配置中时，会在流程引擎启动时激活，并在引擎启动启动中持续工作着。

eventListeners 属性需要org.activiti.engine.delegate.event.ActivitiEventListener 的队列。 通常，我们可以声明一个内部的bean定义，或使用 ref 引用已定义的 bean。 下面的代码，向配置添加了一个事件监听器，任何事件触发时都会提醒它，无论事件是什么类型：

	<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
	    ...
	    <property name="eventListeners">
	      <list>
	         <bean class="org.activiti.engine.example.MyEventListener" />
	      </list>
	    </property>
	</bean>

为了监听特定类型的事件，可以使用 typedEventListeners 属性，它需要一个 map 参数。 map
的 key 是逗号分隔的事件名（或单独的事件名）。 map 的 value
是 org.activiti.engine.delegate.event.ActivitiEventListener 队列。 下面的代码演示了向配置中添加一个事件监听器，可以监听 job 执行成功或失败：

	<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
	    ...
	    <property name="typedEventListeners">
	      <map>
	        <entry key="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" >
	          <list>
	            <bean class="org.activiti.engine.example.MyJobEventListener" />
	          </list>
	        </entry>
	      </map>
	    </property>
	</bean>

分发事件的顺序是由监听器添加时的顺序决定的。首先，会调用所有普通的事件监听器（eventListeners 属性），按照它们在 list 中的次序。 然后，会调用所有对应类型的监听器（ typedEventListeners 属性），如果对应类型的事件被触发了。

###<a name='Adding listeners at runtime 运行时添加监听'>Adding listeners at runtime 运行时添加监听</a>

可以通过API（RuntimeService）在运行阶段添加或删除额外的事件监听器

	/**
	 * Adds an event-listener which will be notified of ALL events by the dispatcher.
	 * @param listenerToAdd the listener to add
	 */
	void addEventListener(ActivitiEventListener listenerToAdd);
	
	/**
	 * Adds an event-listener which will only be notified when an event occurs, which type is in the given types.
	 * @param listenerToAdd the listener to add
	 * @param types types of events the listener should be notified for
	 */
	void addEventListener(ActivitiEventListener listenerToAdd, ActivitiEventType... types);
	
	/**
	 * Removes the given listener from this dispatcher. The listener will no longer be notified,
	 * regardless of the type(s) it was registered for in the first place.
	 * @param listenerToRemove listener to remove
	 */
	 void removeEventListener(ActivitiEventListener listenerToRemove);

注意运行期添加的监听器引擎重启后就消失了

###<a name='Adding listeners to process definitions 添加监听到流程定义'>Adding listeners to process definitions 添加监听到流程定义</a>

可以为特定流程定义添加监听器。监听器只会监听与这个流程定义相关的事件，以及这个流程定义上发起的所有流程实例的事件。 监听器实现可以使用，全类名定义，引用实现了监听器接口的表达式，或配置为抛出一个message/signal/error 的 BPMN 事件。

####Listeners executing user-defined logic 让监听器执行用户定义的逻辑

下面代码为一个流程定义添加了两个监听器。第一个监听器会接收所有类型的事件，它是通过全类名定义的。 第二个监听器只接收作业成功或失败的事件，它使用了定义在流程引擎配置中的 beans 属性中的一个 bean。

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener class="org.activiti.engine.test.MyEventListener" />
	    <activiti:eventListener delegateExpression="${testEventListener}" events="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" />
	  </extensionElements>
	        
	  ...
	
	</process>

对于实体相关的事件，也可以设置为针对某个流程定义的监听器，实现只监听发生在某个流程定义上的某个类型实体事件。 下面的代码演示了如何实现这种功能。可以用于所有实体事件（第一个例子），也可以只监听特定类型的事件（第二个例子）

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener class="org.activiti.engine.test.MyEventListener" entityType="task" />
	    <activiti:eventListener delegateExpression="${testEventListener}" events="ENTITY_CREATED" entityType="task" />
	  </extensionElements>
	        
	  ...
	
	</process>

entityType 支持的值有：attachment, comment, execution,identity-link, job, process-instance,
process-definition, task。

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener class="org.activiti.engine.test.MyEventListener" entityType="task" />
	    <activiti:eventListener delegateExpression="${testEventListener}" events="ENTITY_CREATED" entityType="task" />
	  </extensionElements>
	        
	  ...
	
	</process>
####Listeners throwing BPMN events

[试验阶段]

另一种处理事件的方法是抛出一个 BPMN 事件。请注意它只针对与抛出一个activiti 事件类型的 BPMN 事件。 比如，抛出一个 BPMN 事件，在流程实例删除时，会导致一个错误。 下面的代码演示了如何在流程实例中抛出一个 signal，把 signal 抛出到外部流程（全局），在流程实例中抛出一个消息事件， 在流程实例中抛出一个错误事件。除了使用 class 或delegateExpression， 还使用了 throwEvent 属性，通过额外属性，指定了抛出事件的类型。

```xml

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener throwEvent="signal" signalName="My signal" events="TASK_ASSIGNED" />
	  </extensionElements>
	</process>
```

```xml

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener throwEvent="globalSignal" signalName="My signal" events="TASK_ASSIGNED" />
	  </extensionElements>
	</process>
```

```xml

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener throwEvent="message" messageName="My message" events="TASK_ASSIGNED" />
	  </extensionElements>
	</process>
```

```xml

	<process id="testEventListeners">
	  <extensionElements>
	    <activiti:eventListener throwEvent="error" errorCode="123" events="TASK_ASSIGNED" />
	  </extensionElements>
	</process>
```

如果需要声明额外的逻辑，是否抛出 BPMN 事件，可以扩展 activiti 提供的监听器类。在子类中重写 isValidEvent(ActivitiEvent event)， 可以防止抛出 BPMN 事件。对应的类是
org.activiti.engine.test.api.event.SignalThrowingEventListenerTest,
org.activiti.engine.impl.bpmn.helper.MessageThrowingEventListener 和
org.activiti.engine.impl.bpmn.helper.ErrorThrowingEventListener.

####Notes on listeners on a process-definition 流程定义中监听器的注意事项

* 事件监听器只能声明在 process 元素中，作为 extensionElements 的子元素。 监听器不能定义在流程的单个 activity 下。
* delegateExpression 中的表达式无法访问 execution 上下文，这与其他表达式不同（比如 gateway ）。 它只能引用定义在流程引擎配置的beans 属性中声明的 bean，或者使用 spring（未使用 beans 属性）中所有实现了监听器接口的 spring-bean。
* 在使用监听器的 class 属性时，只会创建一个实例。记住监听器实现不会依赖成员变量，确认是多线程安全的。
* 当一个非法的事件类型用在 events 属性或 throwEvent 中时，流程定义发布时就会抛出异常。（会导致部署失败）。如果 class 或delegateExecution 由问题（类不存在，不存在的 bean 引用，或代理类没有实现监听器接口），会在流程启动时抛出异常（或在第一个有效的流程定义事件被监听器接收时）。所以要保证引用的类正确的放在 classpath 下，表达式也要引用一个有效的实例。

###Dispatching events through API 通过 API 分发事件

我们提供了通过 API 使用事件机制的方法，允许大家触发定义在引擎中的任何自定义事件。
建议（不强制）只触发类型为 CUSTOM 的 ActivitiEvents。可以通过RuntimeService 触发事件：

	/**
	 * Dispatches the given event to any listeners that are registered.
	 * @param event event to dispatch.
	 * 
	 * @throws ActivitiException if an exception occurs when dispatching the event or when the {@link ActivitiEventDispatcher}
	 * is disabled.
	 * @throws ActivitiIllegalArgumentException when the given event is not suitable for dispatching.
	 */
	 void dispatchEvent(ActivitiEvent event);

###<a name='Supported event types 支持的事件类型'>Supported event types 支持的事件类型</a>

下面是引擎中可能出现的所有事件类型。每个类型都对应 org.activiti.engine.delegate.event.ActivitiEventType 中的一个枚举值

Table 1. Supported events 

<table border="1" summary="Supported events"><colgroup><col><col><col></colgroup>
<thead>
<tr>
<th>Event 名称</th>
<th>描述</th>
<th>Event 类</th></tr></thead>
<tbody>
<tr><td>ENGINE_CREATED</td><td>监听器监听的流程引擎已经创建
完毕，并准备好接受API调用。</td><td>
<code class="literal">org.activiti...ActivitiEvent</code></td></tr>
<tr><td>ENGINE_CLOSED</td><td>监听器监听的流程引擎已经关
闭，不再接受API调用。</td><td><code class="literal">org.activiti...ActivitiEvent</code></td></tr>
<tr><td>ENTITY_CREATED</td><td>创建了一个新实体。实体包含在
事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>ENTITY_INITIALIZED</td><td>创建了一个新实体，初始化也完成了。如果这个实体的创建会包含子实体的创建，这个事件会在子实体都创建/初始化完成后被触发，这是与ENTITY_CREATED的区别。<code class="literal">ENTITY_CREATE</code> event.</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>ENTITY_UPDATED</td><td>更新了已存在的实体。实体包含
在事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>ENTITY_DELETED</td><td>删除了已存在的实体。实体包含
在事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>ENTITY_SUSPENDED</td><td>暂停了已存在的实体。实体包含
在事件中。会被 ProcessDefinitions,ProcessInstances 和 Tasks抛出。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>ENTITY_ACTIVATED</td><td>激活了已存在的实体，实体包含
在事件中。会被 ProcessDefinitions, ProcessInstances 和 Tasks抛出。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>JOB_EXECUTION_SUCCESS</td><td>作业执行成功。job包含在事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>JOB_EXECUTION_FAILURE</td><td>作业执行失败。作业和异常信息包含在事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code> and <code class="literal">org.activiti...ActivitiExceptionEvent</code></td></tr><tr><td>JOB_RETRIES_DECREMENTED</td><td>因为作业执行失败，导致重试次
数减少。作业包含在事件中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>TIMER_FIRED</td><td>触发了定时器。job包含在事件
中。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>JOB_CANCELED</td><td>取消了一个作业。事件包含取消
的作业。作业可以通过API调用取消， 任务完成后对应的边界定时器也会取消，在新流程定义发布时也会取消。
</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code>
</td></tr><tr><td>ACTIVITY_STARTED</td><td>一个节点开始执行</td><td><code class="literal">org.activiti...ActivitiActivityEvent</code></td></tr>
<tr><td>ACTIVITY_COMPLETED</td><td>一个节点成功结束</td><td><code class="literal">org.activiti...ActivitiActivityEvent</code></td></tr>
<tr><td>ACTIVITY_CANCELLED</td><td>要取消一个节点。取消是因为三个原因（MessageEventSubscriptionEntity, SignalEventSubscriptionEntity, TimerEntity）</td><td><code class="literal">org.activiti…​ActivitiActivityCancelledEvent</code></td></tr>
<tr><td>ACTIVITY_SIGNALED</td><td>一个节点收到了一个信号</td><td><code class="literal">org.activiti...ActivitiSignalEvent</code></td></tr>
<tr><td>ACTIVITY_MESSAGE_RECEIVED</td><td>一个节点收到了一个消息。在节点收到消息之前触发。收到后，会触发 ACTIVITY_SIGNAL 或ACTIVITY_STARTED，这会根据节点的类型（边界事件，事件子流程开始事件） </td></tr>
<tr><td>ACTIVITY_ERROR_RECEIVED</td><td>一个节点收到了一个错误事件。在节点实际处理错误之前触发。事件的activityId对应着处理错误的节点。 这个事件后续会是ACTIVITY_SIGNALLED或ACTIVITY_COMPLETE，如果错误发送成功的话。</td></tr>
<tr><td>UNCAUGHT_BPMN_ERROR</td><td>抛出了未捕获的BPMN错误。流程
没有提供针对这个错误的处理器。 事件的activityId为空。</td><td><code class="literal">org.activiti...ActivitiErrorEvent</code></td></tr>
<tr><td>ACTIVITY_COMPENSATE</td><td>一个节点将要被补偿。事件包含
了将要执行补偿的节点id。</td><td><code class="literal">org.activiti...ActivitiActivityEvent</code></td></tr>
<tr><td>VARIABLE_CREATED</td><td>创建了一个变量。事件包含变量名，变量值和对应的分支或任务（如果存在）。</td><td><code class="literal">org.activiti...ActivitiVariableEvent</code></td></tr><tr><td>VARIABLE_UPDATED</td><td>更新了一个变量。事件包含变量
名，变量值和对应的分支或任务（如果存在）。</td><td><code class="literal">org.activiti...ActivitiVariableEvent</code></td></tr>
<tr><td>VARIABLE_DELETED</td><td>删除了一个变量。事件包含变量名，变量值和对应的分支或任务（如果存在）</td><td><code class="literal">org.activiti...ActivitiVariableEvent</code></td></tr>
<tr><td>TASK_ASSIGNED</td><td任务被分配给了一个人员。事件
包含任务。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr><tr><td>TASK_CREATED</td><td>创建了新任务。它位于ENTITY_CREATE事件之后。当任务是由流程创建时， 这个事件会在TaskListener执行之前被执行。
</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>TASK_COMPLETED</td><td>任务被完成了。它会在ENTITY_DELETE事件之前触发。当任务是流程一部分时，事件会在流程继续运行之前， 后续事件将是ACTIVITY_COMPLETE，对应着完成任务的节点。</td><td><code class="literal">org.activiti...ActivitiEntityEvent</code></td></tr>
<tr><td>PROCESS_COMPLETED</td><td> 流程已结束。在最后一个节点
的 ACTIVITY_COMPLETED 事件之后触发。 当流程到达的状态，没有任何后续连线时， 流程就会结束。</td><td><code class="literal">org.activiti…​ActivitiEntityEvent</code></td></tr>
<tr><td>PROCESS_CANCELLED</td><td> 流程已取消。在流程实例删除前从运行时触发。流程实例被 API RuntimeService.deleteProcessInstance 调用</td><td><code class="literal">org.activiti…​ActivitiCancelledEvent</code></td></tr>
<tr><td>MEMBERSHIP_CREATED</td><td>用户被添加到一个组里。事件包
含了用户和组的id。</td><td><code class="literal">org.activiti...ActivitiMembershipEvent</code></td></tr>
<tr><td>MEMBERSHIP_DELETED</td><td>用户被从一个组中删除。事件包
含了用户和组的id。</td><td><code class="literal">org.activiti...ActivitiMembershipEvent</code></td></tr>
<tr><td>MEMBERSHIPS_DELETED</td><td>所有成员被从一个组中删除。在
成员删除之前触发这个事件，所以他们都是可以访问的。 因为性能方面的考虑，不会为每个成员触发单独的MEMBERSHIP_DELETED事件。</td></tr></tbody></table>

引擎内部所有 ENTITY_* 事件都是与实体相关的。下面的列表展示了实体事件与实体的对应关
系：

* ENTITY_CREATED, ENTITY_INITIALIZED, ENTITY_DELETED: Attachment, Comment, Deployment, Execution, Group, IdentityLink, Job, Model, ProcessDefinition, ProcessInstance, Task, User.
* ENTITY_UPDATED: Attachment, Deployment, Execution, Group, IdentityLink, Job, Model, ProcessDefinition, ProcessInstance, Task, User.
* ENTITY_SUSPENDED, ENTITY_ACTIVATED: ProcessDefinition, ProcessInstance/Execution, Task.

###Additional remarks 附加信息

**只有事件被发送，对应的引擎内的监听器才会被通知到**。你有很多引擎 - 在同一个数据库运行 - 事件只会发送给注册到对应引擎的监听器。其他引擎发生的事件不会发送给这个监听器，无论实际上它们运行在同一个或不同的JVM中。

对应的事件类型（对应实体）都包含对应的实体。根据类型或事件，这些实体不能再进行更新（比如，当实例以被删除）。可能的话，使用事件提供的 EngineServices 来以安全的方式来操作引擎。即使如此，你需要小心的对事件对应的实体进行更新/操作。

没有对应历史的实体事件，因为它们都有运行阶段的对应实体。