##Unit testing 单元测试

业务流程是软件项目的一部分，它也应该和普通的业务流程一样进行测试： 使用单元测试。因为 Activiti 是一个嵌入式的 java 引擎，为业务流程编写单元测试和写普通单元测试完全一样。

Activiti 支持 JUnit 3和4进行单元测试。使用 JUnit 3时， 必须集成 org.activiti.engine.test.ActivitiTestCase。它通过保护的成员变量提供 ProcessEngine 和服务，在测试的 setup() 中， 默认会使用classpath 下的 activiti.cfg.xml 初始化流程引擎。 想使用不同的配置文件，可以重写 getConfigurationResource() 方法。 如果配置文件相同的话，对应的流程引擎会被静态缓存， 就可以用于多个单元测试。

继承了 ActivitiTestCase,你可以在测试方法上使用 org.activiti.engine.test.Deployment 注解。测试执行前，与测试类在同一个包下的， 格式为 testClassName.testMethod.bpmn20.xml的资源文件，会被部署。 测试结束后，发布包也会被删除，包括所有相关的流程实例，任务，等等。Deployment 注解也可以直接设置资源的位置。 参考[Javadocs](http://www.activiti.org/javadocs/org/activiti/engine/test/Deployment.html)
获得更多信息。

把这些放在一起，JUnit 3 测试看起来像这样。

	public class MyBusinessProcessTest extends ActivitiTestCase {
	   
	  @Deployment
	  public void testSimpleProcess() {
	    runtimeService.startProcessInstanceByKey("simpleProcess");
	    
	    Task task = taskService.createTaskQuery().singleResult();
	    assertEquals("My Task", task.getName());
	    
	    taskService.complete(task.getId());
	    assertEquals(0, runtimeService.createProcessInstanceQuery().count());
	  }
	}      
     
要想在使用 JUnit 4 编写单元测试时获得同样的功能， 可以使
用 org.activiti.engine.test.ActivitiRule。 通过它，可以通过getter 方法获得流程引擎和各种服务。 和 ActivitiTestCase 一样（参考上面章节），使用这个 Rule 也会启用org.activiti.engine.test.Deployment 注解（参考上面章节使用和配置的介绍），它会在 classpath 下查找默认的配置文件。 如果配置文件相同的话，对应的流程引擎会被静态缓存， 就可以用于多个单元测试。

下面的代码演示了 JUnit 4 单元测试并使用了 ActivitiRule 的例子。
	
	public class MyBusinessProcessTest {
	  
	  @Rule
	  public ActivitiRule activitiRule = new ActivitiRule();
	  
	  @Test
	  @Deployment
	  public void ruleUsageExample() {
	    RuntimeService runtimeService = activitiRule.getRuntimeService();
	    runtimeService.startProcessInstanceByKey("ruleUsage");
	    
	    TaskService taskService = activitiRule.getTaskService();
	    Task task = taskService.createTaskQuery().singleResult();
	    assertEquals("My Task", task.getName());
	    
	    taskService.complete(task.getId());
	    assertEquals(0, runtimeService.createProcessInstanceQuery().count());
	  }
	}
	      