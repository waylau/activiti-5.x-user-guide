##Sequence Flow 顺序流

###Description 描述

顺序流是连接两个流程节点的连线。 流程执行完一个节点后，会沿着节点的所有外出顺序流继续执行。 就是说，BPMN 2.0 默认的行为就是并发的： 两个外出顺序流会创造两个单独的，并发流程分支。

###Graphical notation 图形标记

顺序流显示为从起点到终点的箭头。 箭头总是指向终点

![](http://99btgc01.info/uploads/2014/12/bpmn.sequence.flow.png)

###XML representation 内容

顺序流需要流程范围内唯一的id， 以及对起点与 终点元素的引用。

	<sequenceFlow id="flow1" sourceRef="theStart" targetRef="theTask" />

###Conditional sequence flow 条件顺序流

####Description 描述

可以为顺序流定义一个条件。离开一个BPMN 2.0节点时， 默认会计算外出顺序流的条件。如果条件结果为 true, 就会选择外出顺序流继续执行。当多条顺序流被选中时， 就会创建多条分支， 流程会继续以并行方式继续执行。

注意：上面的讨论仅涉及 BPMN 2.0 节点（和事件）， 不包括网关。网关会用特定的方式处理顺序流中的条件， 这与网关类型相关。

####Graphical notation 图形标记

条件顺序流显示为一个正常的顺序流，不过在起点有一个菱形。 条件表达式也会显示在顺序流上。

![](http://99btgc01.info/uploads/2014/12/bpmn.conditional.sequence.flow.png)

####XML representation 内容

条件顺序流定义为一个正常的顺序流， 包含 conditionExpression 子元素。 注意目前只支持 tFormalExpressions， 如果没有设置xsi:type="" , 就会默认值支持目前支持的表达式类型

	<sequenceFlow id="flow" sourceRef="theStart" targetRef="theTask">
	  <conditionExpression xsi:type="tFormalExpression">
	    <![CDATA[${order.price > 100 && order.price < 250}]]>
	  </conditionExpression>
	</sequenceFlow>

当前条件表达式**只能使用 UEL**， 可以参考表达式章节获取更多信息。 使用的表达式需要返回 boolean 值，否则会在解析表达式时抛出异常。

* 下面的例子引用了流程变量的数据， 通过 getter 调用 JavaBean。

	<conditionExpression xsi:type="tFormalExpression">
	  <![CDATA[${order.price > 100 && order.price < 250}]]>
	</conditionExpression>

* 这个例子通过调用方法返回一个 boolean 值。
 
	<conditionExpression xsi:type="tFormalExpression">
	  <![CDATA[${order.isStandardOrder()}]]>
	</conditionExpression>

在 activiti 发布包中，包含以下流程实例，使用了值和方法表达式 （参
考 org.activiti.examples.bpmn.expression )包）：

![](http://99btgc01.info/uploads/2014/12/bpmn.uel-expression.on.seq.flow.png)

###Default sequence flow 默认顺序流

####Description 描述

所有的 BPMN 2.0 任务和网关都可以设置一个默认顺序流。 只有在节点的其他外出顺序流不能被选中是，才会使用它作为外出顺序流继续执行。 默认顺序流的条件设置不会生效。

####Graphical notation 图形标记

默认顺序流显示为了普通顺序流，起点有一个“斜线”标记。

![](http://99btgc01.info/uploads/2014/12/bpmn.default.sequence.flow.png)

####XML representation 内容

默认顺序流通过对应节点的 default 属性定义。 下面的 XML 代码演示了排他网关设置了默认顺序流 flow 2 。 只有当 conditionA 和conditionB 都返回 false 时， 才会选择它作为外出连线继续执行。
	
	<exclusiveGateway id="exclusiveGw" name="Exclusive Gateway" default="flow2" />
	<sequenceFlow id="flow1" sourceRef="exclusiveGw" targetRef="task1">
	  <conditionExpression xsi:type="tFormalExpression">${conditionA}</conditionExpression>
	</sequenceFlow>
	<sequenceFlow id="flow2" sourceRef="exclusiveGw" targetRef="task2"/>
	<sequenceFlow id="flow3" sourceRef="exclusiveGw" targetRef="task3">
	  <conditionExpression xsi:type="tFormalExpression">${conditionB}</conditionExpression>
	</sequenceFlow>        

对应下面的图形显示：

![](http://99btgc01.info/uploads/2014/12/bpmn.default.sequence.flow.example.png)