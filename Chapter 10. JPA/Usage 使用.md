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