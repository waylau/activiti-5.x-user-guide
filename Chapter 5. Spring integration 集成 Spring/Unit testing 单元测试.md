##Unit testing 单元测试

当集成 Spring 时，使用标准的 Activiti 测试工具类是非常容易的对业务流程进行测试。 下面的例子展示了如何在一个典型的基于Spring单元测试测试业务流程：

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration("classpath:org/activiti/spring/test/junit4/springTypicalUsageTest-context.xml")
	public class MyBusinessProcessTest {
	  
	  @Autowired
	  private RuntimeService runtimeService;
	  
	  @Autowired
	  private TaskService taskService;
	  
	  @Autowired
	  @Rule
	  public ActivitiRule activitiSpringRule;
	  
	  @Test
	  @Deployment
	  public void simpleProcessTest() {
	    runtimeService.startProcessInstanceByKey("simpleProcess");
	    Task task = taskService.createTaskQuery().singleResult();
	    assertEquals("My Task", task.getName());
	  
	    taskService.complete(task.getId());
	    assertEquals(0, runtimeService.createProcessInstanceQuery().count());
	   
	  }
	}      
	      
注意对于这种方式，你需要在 Spring 配置中（在上文的例子中它是自动注入的）定义一个 org.activiti.engine.test.ActivitiRulebean

	<bean id="activitiRule" class="org.activiti.engine.test.ActivitiRule">
	  <property name="processEngine" ref="processEngine" />
	</bean>        


      