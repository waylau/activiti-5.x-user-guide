##Annotation-based configuration 基于注解的配置

[试验] @EnableActiviti注解相对较新，未来可能会有变更。

除了基于 XML 的配置以外，还可以选择基于注解的方式来配置 Spring 环境。 这与使用 XML 的方法非常相似，除了要使用 @Bean 注解， 而且配置是使用 java 编写的。 它已经可以直接用于 Activiti-Spring 的集成了：

首先介绍（需要 Spring 3.0+ ）的是 @EnableActiviti 注解。 最简单的用法如下所示：

	@Configuration
	  @EnableActiviti
	  public static class SimplestConfiguration {
	    
	  }

它会创建一个 Spring 环境，并对 Activiti 流程引擎进行如下配置

* 默认的内存 H2 数据库，启用数据库自动升级。
* 一个简单的 DataSourceTransactionManager
* 一个默认的 SpringJobExecutor
* 自动扫描 processes/ 目录下的 bpmn20.xml 文件。


在这样一个环境里，可以直接通过注入操作 Activiti 引擎：

	  @Autowired
	  private ProcessEngine processEngine;
	  
	  @Autowired
	  private RuntimeService runtimeService;
	  
	  @Autowired
	  private TaskService taskService;
	  
	  @Autowired
	  private HistoryService historyService;
	  
	  @Autowired
	  private RepositoryService repositoryService;
	  
	  @Autowired
	  private ManagementService managementService;
	  
	  @Autowired
	  private FormService formService;

当然，默认值都可以自定义。比如，如果配置了 DataSource，它就会代替默认创建的数据库配置。 事务管理器，job 执行器和其他组件都与之相同。 比如如下配置：
	
	  @Configuration
	  @EnableActiviti
	  public static class Config {
	    
	    @Bean
	    public DataSource dataSource() {
	        BasicDataSource basicDataSource = new BasicDataSource();
	        basicDataSource.setUsername("sa");
	        basicDataSource.setUrl("jdbc:h2:mem:anotherDatabase");
	        basicDataSource.setDefaultAutoCommit(false);
	        basicDataSource.setDriverClassName(org.h2.Driver.class.getName());
	        basicDataSource.setPassword("");
	        return basicDataSource;
	    }
	    
	  }

其他数据库会代替默认的。下面介绍了更加复杂的配置。注意AbstractActivitiConfigurer 用法， 它暴露了流程引擎的
配置，可以用来对它的细节进行详细的配置。

	@Configuration
	@EnableActiviti
	@EnableTransactionManagement(proxyTargetClass = true)
	class JPAConfiguration {
	
	    @Bean
	    public OpenJpaVendorAdapter openJpaVendorAdapter() {
	        OpenJpaVendorAdapter openJpaVendorAdapter = new OpenJpaVendorAdapter();
	        openJpaVendorAdapter.setDatabasePlatform(H2Dictionary.class.getName());
	        return openJpaVendorAdapter;
	    }
	
	    @Bean
	    public DataSource dataSource() {
	        BasicDataSource basicDataSource = new BasicDataSource();
	        basicDataSource.setUsername("sa");
	        basicDataSource.setUrl("jdbc:h2:mem:activiti");
	        basicDataSource.setDefaultAutoCommit(false);
	        basicDataSource.setDriverClassName(org.h2.Driver.class.getName());
	        basicDataSource.setPassword("");
	        return basicDataSource;
	    }
	
	    @Bean
	    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(
	        OpenJpaVendorAdapter openJpaVendorAdapter, DataSource ds) {
	        LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
	        emf.setPersistenceXmlLocation("classpath:/org/activiti/spring/test/jpa/custom-persistence.xml");
	        emf.setJpaVendorAdapter(openJpaVendorAdapter);
	        emf.setDataSource(ds);
	        return emf;
	    }
	
	    @Bean
	    public PlatformTransactionManager jpaTransactionManager(
	        EntityManagerFactory entityManagerFactory) {
	        return new JpaTransactionManager(entityManagerFactory);
	    }
	
	    @Bean
	    public AbstractActivitiConfigurer abstractActivitiConfigurer(
	        final EntityManagerFactory emf,
	        final PlatformTransactionManager transactionManager) {
	
	        return new AbstractActivitiConfigurer() {
	
	            @Override
	            public void postProcessSpringProcessEngineConfiguration(SpringProcessEngineConfiguration engine) {
	                engine.setTransactionManager(transactionManager);
	                engine.setJpaEntityManagerFactory(emf);
	                engine.setJpaHandleTransaction(false);
	                engine.setJobExecutorActivate(false);
	                engine.setJpaCloseEntityManager(false);
	                engine.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
	            }
	        };
	    }
	
	    // A random bean
	    @Bean
	    public LoanRequestBean loanRequestBean() {
	        return new LoanRequestBean();
	    }
	}