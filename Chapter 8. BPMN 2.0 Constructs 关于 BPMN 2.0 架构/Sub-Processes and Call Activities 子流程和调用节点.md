##Sub-Processes and Call Activities 子流程和调用节点

###Sub-Process

####Description 描述

子流程（Sub-process）是一个包含其他节点，网关，事件等等的节点。 它自己就是一个流程，同时是更大流程的一部分。 子流程是完全定义在父流程里的 （这就是为什么叫做内嵌子流程）。

子流程有两种主要场景：

* 子流程可以使用继承式建模。 很多建模工具的子流程可以折叠， 把子流程的内部细节隐藏，显示一个高级别的端对端的业务流程总览。
* 子流程会创建一个新的事件作用域。 子流程运行过程中抛出的事件，可以被子流程边缘定义的 边界事件捕获， 这样就可以创建一个仅限于这个子流程的事件作用范围。

使用子流程要考虑如下限制：

* 子流程只能包含一个空开始事件，不能使用其他类型的开始事件。子路程必须 至少有一个结束节点。注意，BPMN 2.0 规范允许忽略子流程的 开始和结束节点，但是当前 activiti 的实现并不支持。
* 顺序流不能跨越子流程的边界。

####Graphical notation 图形标记

子流程显示为标准的节点，圆角矩形。 这时子流程是折叠的，只显示名称和一个加号标记，展示了高级别的流程总览：

![](http://99btgc01.info/uploads/2014/12/bpmn.collapsed.subprocess.png)

这时子流程是展开的，子流程的步骤都显示在子流程边界内：

![](http://99btgc01.info/uploads/2014/12/bpmn.expanded.subprocess.png)

使用子流程的主要原因，是定义对应事件的作用域。下面流程模型演示了这个功能：调查软件/调查引荐任务需要同步执行，两个任务需要在同时完成，在二线支持解决之前。 这里，定时器的作用域（比如，节点需要及时完成）是由子流程限制的。

![](http://99btgc01.info/uploads/2014/12/bpmn.subprocess.with.boundary.timer.png)

####XML representation 内容

子流程定义为 subprocess 元素。 所有节点，网关，事件，等等。它是子流程的一部分，需要放在这个元素里。
	
	<subProcess id="subProcess">
	    
	  <startEvent id="subProcessStart" />
	  
	  ... other Sub-Process elements ...
	
	  <endEvent id="subProcessEnd" />
	    
	 </subProcess>     

###Event Sub-Process 事件子流程

####Description 描述

事件子流程是 BPMN 2.0 中的新元素。事件子流程是由事件触发的子流程。 事件子流程可以添加到流程级别或任意子流程级别。用于触发事件子流程的事件是使用开始事件配置的。为此，事件子流程是不支持空开始事件的。 事件子流程可以被消息事件，错误事件，信号事件，定时器事件，或补偿事件触发。开始事件的订阅在包含事件子流程的作用域（流程实例
或子流程）创建时就会创建。 当作用域销毁也会删除订阅。

事件子流程可以是中断的或非中断的。一个中断的子流程会取消当前作用域内的所有流程。非中断事件子流程会创建那一个新的同步分支。中断事件子流程只会被每个激活状态的宿主触发一次， 非中断事件子流程可以触发多次。子流程是否是终端的，配置使用事件子流程的开始事件配置。

事件子流程不能有任何进入和外出流程。当事件触发一个事件子流程时，输入顺序流是没有意义的。 当事件子流程结束时，无论当前作用域已经结束了（中断事件子流程的情况），或为非中断子流程生成同步分支会结束。

当前的限制：
* activiti只支持中断事件子流程。
* activiti只支持使用错误开始事件或消息开始事件的事件子流程。

####Graphical notation 图形标记

事件子流程可以显示为边框为虚线的内嵌子流程。

![](http://99btgc01.info/uploads/2014/12/bpmn.subprocess.eventSubprocess.png)

####XML representation 内容

事件子流程的 XML 内容与内嵌子流程是一样的。 另外，要把triggeredByEvent 属性设置为 true：

	<subProcess id="eventSubProcess" triggeredByEvent="true">
	        ...
	</subProcess>

####Example 实例

下面是一个使用错误开始事件触发的事件子流程的实例。事件子流程是放在“流程级别”的，意思是，作用于流程实例：

![](http://99btgc01.info/uploads/2014/12/bpmn.subprocess.eventSubprocess.example.1.png)

事件子流程的XML如下所示：
	
	<subProcess id="eventSubProcess" triggeredByEvent="true">
	        <startEvent id="catchError">
	                <errorEventDefinition errorRef="error" /> 
	        </startEvent>
	        <sequenceFlow id="flow2" sourceRef="catchError" targetRef="taskAfterErrorCatch" />
	        <userTask id="taskAfterErrorCatch" name="Provide additional data" />
	</subProcess>

如上面所述，事件子流程也可以添加成内嵌子流程。如果添加为内嵌子流程，它其实是边界事件的一种替代方案。 考虑下面两个流程图。两种情况内嵌子流程会抛出一个错误事件。两种情况错误都会被捕获并使用一个用户任务处理。

![](http://99btgc01.info/uploads/2014/12/bpmn.subprocess.eventSubprocess.example.2a.png)

相对于：
![](http://99btgc01.info/uploads/2014/12/bpmn.subprocess.eventSubprocess.example.2b.png)

两种场景都会执行相同的任务。然而，两种建模的方式是不同的：

* 内嵌子流程是使用与执行作用域宿主相同的流程执行的。意思是内嵌子流程可以访问它作用域内的内部变量。 当使用边界事件时，执行内嵌子流程的流程会删除，并生成一个流程根据边界事件的顺序流继续执行。 这意味着内嵌子流程创建的变量不再起作用了。
* 当使用事件子流程时，事件是完全由它添加的子流程处理的。 当使用边界事件时，事件由父流程处理。

这两个不同点可以帮助我们决定是使用边界事件还是内嵌事件子流程来解决特定的流程建模/实现问题。

###Transaction subprocess 事务子流程

[试验]

####Description 描述

事务子流程是内嵌子流程，可以用来把多个流程放到一个事务里。 事务是一个逻辑单元，可以把一些单独的节点放在一起，这样它们就可以一起成功或一起失败。事务可能的结果： 事务可以有三种可能的结果：

* 事务成功，如果没有取消也没有因为问题终结。如果事务子流程是成功的， 就会使用外出顺序流继续执行。 如果流程后来抛出了一个补偿事件，成功的事务可能被补偿。
注意：和普通内嵌子流程一样，事务可能在成功后， 使用中间补偿事件进行补偿。
* 事务取消，如果流程到达取消结束事件。这时，所有流程都会终结和删除。触发补偿的一个单独的流程，会通过取消边界事件继续执行。 在补偿完成之后，事务子流程会使用取消边界事务的外出顺序流向下执行。
* 事务被问题结束，如果跑出了一个错误事件， 而且没有在事务子流程中捕获。（如果错误被事务子流程的边界事件处理了，也会这样应用。） 这时，不会执行补偿。

下面的图形演示了三种不同的结果：

![](http://99btgc01.info/uploads/2014/12/bpmn.transaction.subprocess.example.1.png)

**与A CID 事务的关系**：一定不要把 bpmn 事务子流程与技术（ACID）事务相混淆。 bpmn事务子流程不是技术事务领域的东西。要理解 activiti 中的事务管理，请参考 [并发与事务](Transactions and Concurrency 事务与并发.md)。 bpmn 事务和技术事务有以下不同点：

* ACID事务一般是短期的，bpmn事务可能持续几小时，几天，甚至几个月才能完成。（考虑事务中包含的节点可能有用户任务，一般人员响应的时间比应用时间要长。或者，在其他情况下，bpmn 事务可能要等待发生一些事务事件，就像要根据某种次序执行。） 这种操作通常要相比更新数据库的一条数据，或把一条信息保存到事务性队列中，消耗更长的时间来完成。
*  因为不能在整个业务节点的过程中保持一个技术性的事务，所以 bpmn 事务一般要跨越多个 ACID 事务。
*  因为 bpmn 事务会跨越多个 ACID 事务，所以会丧失 ACID 的特性。比如，考虑上述例子。 假设“约定旅店”和“刷信用卡”操作在单独的ACID事务中执行。 也假设“预定旅店”节点已经成功了。现在我们处于一个中间不稳定状态，因为我们预定了酒店，但是还没有刷信用卡。 现在，在一个ACID事务中，我们要依次执行不同的操作，也会有一个中间不稳定状态。 不同的是，这个中间状态对事务的外部是可见的。 比如，如果通过外部预定服务进行了预定，其他使用相同预定服务的部分就可以看到旅店被预定了。 这意味着实现业务事务时，我们完全失去了隔离属性 （注：我们也经常放弃隔离性，来为ACID事务获得更高的并发，但是我们可以完全控制，中间不稳定状态也只持续很短的时间）。

* bpmn 业务事务也不能使用通常的方式回滚。因为它跨越了多个事务，bpmn 事务取消时一些 ACID 事务可能已经提交了。 这时，它们不能被回滚了。

因为 bpmn 事务实际上运行时间很长，缺乏隔离性和回滚机制都需要被区别对待。 实际上，这里也没有更好的办法在特定领域处理这些问题：

* 使用补偿执行回滚。如果事务范围抛出了取消事件，会影响已经执行成功的节点， 并使用补偿处理器执行补偿。
* 隔离性的缺乏通常使用特定领域的解决方法来解决。比如，上面的例子中， 一个旅店房间可能会展示给第二个客户，在我们确认第一个客户付费之前。 虽然这可能与业务预期不符，预定服务可能选择允许一些过度的预约。
* 另外，因为事务会因为风险而中断，预定服务必须处理这种情况，已经预定了旅店，但是一直没有付款的情况。 （因为事务被中断了）。这时预定服务需要选择一个策略，在旅店房间预定超过最大允许时间后， 如果还没有付款，预定就会取消。

综上所述：ACID 处理的是通常问题（回滚，隔离级别和启发式结果）， 在实现业务事务时，我们需要找到特定领域的解决方案来处理这些问题。

**目前的限制**：

* bpmn规范要求流程引擎能根据底层事务的协议处理事件，比如如果底层协议触发了取消事件，事务就会取消。 作为内嵌引擎，activiti目前不支持这项功能。（对此造成的后果，可以参考下面的一致性讨论）。

**ACID事务顶层的一致性和优化并发**： bpmn 事务保证一致性，要么所有节点都成功，或者一些节点成功，对其他成功的节点进行补偿。 无论哪种方式，都会有一致性的结果。不过要讨论一些 activiti 内部的情况，bpmn 事务的一致性模型是叠加在流程的一致性模型之上的。activiti 执行流程是事务性的。并发使用了乐观锁。在 activiti 中，bpmn 错误，取消和补偿事件都建立在同样的 acid 事务与乐观锁之上。 比如，取消结束事件只能触发它实际到达的补偿。如果之前服务任务抛出了未声明的异常。或者， 补偿处理器的效果无法提交，如果底层的 acid 事务的参与者把事务设置成必须回滚。 或者当两个并发流程到达了取消结束事件，可
能会触发两次补偿，并因为乐观锁异常失败。 所有这些都说明 activiti 中实现 bpmn 事务时，相同的规则也作用域普通的流程和子流程。 所以为了保证一致性，重要的是使用一种方式考虑实现乐观事务性的执行模型。

####Graphical notation 图形标记

事务子流程显示为内嵌子流程，使用双线边框。

![](http://99btgc01.info/uploads/2014/12/bpmn.transaction.subprocess.png)

####XML representation 内容

事务子流程使用 transaction 标签：
	
	<transaction id="myTransaction" >
	        ...
	</transaction>

####Example 实例

下面是事务子流程的实例：

![](http://99btgc01.info/uploads/2014/12/bpmn.transaction.subprocess.example.2.png)

###Call activity (subprocess) 调用活动（子流程）

####Description 描述

BPMN 2.0 区分了普通子流程， 也叫做内嵌子流程，和调用节点，看起来很相似。 从概念上讲，当流程抵达时，两者都会调用子流程。

不同点是调用节点引用流程定义外部的一个流程，子流程 会内嵌到原始的流程定义中。使用调用节点的主要场景是需要重用流程定义， 这个流程定义需要被很多其他流程定义调用的时候。

当流程执行到调用节点，会创建一个新分支，它是到达调用节点的流程的分支。 这个分支会用来执行子流程，默认创建并行子流程，就像一个普通的流程。 上级流程会等待子流程完成，然后才会继续向下执行

####Graphical notation 图形标记

调用节点显示与子流程相同， 不过是粗边框（无论是折叠和展开的）。 根据不同的建模工具，调用节点也可以展开，但是显示为折叠的子流程。

![](http://99btgc01.info/uploads/2014/12/bpmn.collapsed.call.activity.png)

####XML representation 内容

有种调用节点是经常性的节点，需要 calledElement 引用被 key 定义的流程。在实践中，这意味着流程的 ID 用于 calledElement。

	<callActivity id="callCheckCreditProcess" name="Check credit" calledElement="checkCreditProcess" />

注意，子流程的流程定义是在执行阶段解析的。 就是说子流程可以与调用的流程分开部署，如果需要的话。

####Passing variables 传递变量

可以把流程变量传递给子流程，反之亦然。数据会复制给子流程，当它启动的时候， 并在它结束的时候复制回主流程。

	<callActivity id="callSubProcess" calledElement="checkCreditProcess" >
	  <extensionElements>
	          <activiti:in source="someVariableInMainProcess" target="nameOfVariableInSubProcess" />
	          <activiti:out source="someVariableInSubProcss" target="nameOfVariableInMainProcess" />
	  </extensionElements>
	</callActivity>

我们使用 Activiti 扩展来简化 BPMN 标准元素调用dataInputAssociation 和 
dataOutputAssociation，这只在你使用 BPMN 2.0 标准方式声明流程变量才管用。

这里也可以使用表达式：

	<callActivity id="callSubProcess" calledElement="checkCreditProcess" >
	        <extensionElements>
	          <activiti:in sourceExpression="${x+5}"" target="y" />
	          <activiti:out source="${y+5}" target="z" />
	        </extensionElements>
	</callActivity>

最后 z = y + 5 = x + 5 + 5

####Example 实例

下面的流程图演示了简单订单处理。先判断客户端信用，这可能与很多其他流程相同。 检查信用阶段这里设计成调用节点。

![](http://99btgc01.info/uploads/2014/12/bpmn.call.activity.super.process.png)

流程看起来像这样的：

	<startEvent id="theStart" />
	<sequenceFlow id="flow1" sourceRef="theStart" targetRef="receiveOrder" />
	
	<manualTask id="receiveOrder" name="Receive Order" />
	<sequenceFlow id="flow2" sourceRef="receiveOrder" targetRef="callCheckCreditProcess" />
	    
	<callActivity id="callCheckCreditProcess" name="Check credit" calledElement="checkCreditProcess" />
	<sequenceFlow id="flow3" sourceRef="callCheckCreditProcess" targetRef="prepareAndShipTask" />
	   
	<userTask id="prepareAndShipTask" name="Prepare and Ship" />
	<sequenceFlow id="flow4" sourceRef="prepareAndShipTask" targetRef="end" />
	    
	<endEvent id="end" />

子流程看起来像这样的：

![](http://99btgc01.info/uploads/2014/12/bpmn.call.activity.sub.process.png)