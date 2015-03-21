##10.3. Usage 用法

###10.3.1. Simple Example

使用 JPA 变量的例子可以在 JPAVariableTest 中找到。我们将会一步一步的解释 `JPAVariableTest.testUpdateJPAEntityValues`。

首先，我们需要创建一个基于 `META-INF/persistence.xml` 的EntityManagerFactory 作为我们的持久化单元。它包含持久化单元中所有的类和一些供应商特定的配置。

我们将使用一个简单的实体作为测试，其中包含有一个 id 和 `String` 类型的 value 属性，这也将会被持久化。在允许测试之前，我们创建一个实体并且保存它。
	
	@Entity(name = "JPA_ENTITY_FIELD")
	public class FieldAccessJPAEntity {
	
	  @Id
	  @Column(name = "ID_")
	  private Long id;
	
	  private String value;
	
	  public FieldAccessJPAEntity() {
	    // Empty constructor needed for JPA
	  }
	
	  public Long getId() {
	    return id;
	  }
	
	  public void setId(Long id) {
	    this.id = id;
	  }
	
	  public String getValue() {
	    return value;
	  }
	
	  public void setValue(String value) {
	    this.value = value;
	  }
	}

我们开始一个新的流程实例，添加实体作为变量。与其它的变量一样，它们存储在引擎的持久存储区。当下次这个变量被请求，它会从基于类和Id的存储的 `EntityManager` 中加载。

	Map<String, Object> variables = new HashMap<String, Object>();
	variables.put("entityToUpdate", entityToUpdate);
	
	ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("UpdateJPAValuesProcess", variables);

在我们的流程定义的第一个节点包含一个 `serviceTask` 将调用方法 在 `entityToUpdate` 上 `setValue`，它解析为 JPA 变量，我们启动流程实例并将从相关联的当前引擎的上下文“EntityManager”进行加载。

	<serviceTask id='theTask' name='updateJPAEntityTask'
	  activiti:expression="${entityToUpdate.setValue('updatedValue')}" />

当 service-task 完成后，流程实例等在流程定义中定义的 userTask，这使我们能够检查流程实例。在这一点上，EntityManager 已刷新并更改到实体已经被推到数据库。当我们得到变量 entityToUpdate 值时，它再次加载，我们得到实体，并将实体中的属性 `value` 设置到 `updatedValue` 。

	// Servicetask in process 'UpdateJPAValuesProcess' should have set value on entityToUpdate.
	Object updatedEntity = runtimeService.getVariable(processInstance.getId(), "entityToUpdate");
	assertTrue(updatedEntity instanceof FieldAccessJPAEntity);
	assertEquals("updatedValue", ((FieldAccessJPAEntity)updatedEntity).getValue());