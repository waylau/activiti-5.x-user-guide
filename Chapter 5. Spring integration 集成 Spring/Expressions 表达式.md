##Expressions 表达式

当使用 ProcessEngineFactoryBean 时候，默认情况下，在 BPMN 流程中的所有[表达式](http://www.activiti.org/userguide/index.html#apiExpressions)都将会'看见'所有的 Spring beans。 它可以限制你在表达式中暴露出的 beans 或者甚至可以在你的配置中使用一个 Map 不暴露任何 beans。下面的例子暴露了一个单例 bean（printer），可以把 "printer" 当作关键字使用。**想要不暴露任何beans，仅仅只需要在 SpringProcessEngineConfiguration 中传递一个空的 list 作为'beans'的属性。当不设置'beans'的属性时，在应用上下文中 Spring beans 都是可以使用的。**
	
	<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	  ...
	  <property name="beans">
	    <map>
	      <entry key="printer" value-ref="printer" />
	    </map>
	  </property>
	</bean>
	  
	  <bean id="printer" class="org.activiti.examples.spring.Printer" />

现在暴露出来的 beans 就可以在表达式中使用：例如，在
SpringTransactionIntegrationTest 中的 hello.bpmn20.xml 展示的是如何使用 UEL 方法表达式去调用 Spring bean 的方法：
	
	<definitions id="definitions" ...>
	  
	  <process id="helloProcess">
	  
	    <startEvent id="start" />
	    <sequenceFlow id="flow1" sourceRef="start" targetRef="print" />
	    
	    <serviceTask id="print" activiti:expression="#{printer.printMessage()}" />
	    <sequenceFlow id="flow2" sourceRef="print" targetRef="end" />
	    
	    <endEvent id="end" />
	    
	  </process>
	
	</definitions>

这里的 Printer 看起来像这样：

	public class Printer {
	
	  public void printMessage() {
	    System.out.println("hello world");
	  }
	}

并且 Spring bean 的配置（如上文所示）看起来像这样：
	
	<beans ...>
	  ...
	
	  <bean id="printer" class="org.activiti.examples.spring.Printer" />
	
	</beans>

