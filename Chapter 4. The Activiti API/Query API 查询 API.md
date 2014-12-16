##Query API 查询 API

有两种方法可以从引擎中查询数据：查询 API 和原生查询。查询 API 提供了完全类型安全的 API。 你可以为自己的查询条件添加很多条件 （所有条件都以 AND 组合）和精确的排序条件。下面的代码展示了一个例子：

	List<Task> tasks = taskService.createTaskQuery()
	         .taskAssignee("kermit")
	         .processVariableValueEquals("orderId", "0815")
	         .orderByDueDate().asc()
	         .list();

有时，你需要更强大的查询，比如使用 OR 条件或不能使用查询 API 实现的条件。 这时，我们推荐原生查询，它让你可以编写自己的SQL查询。 返回类型由你使用的查询对象决定，数据会映射到正确的对象上。比如，任务，流程实例，执行，等等。 因为查询会作用在数据库上，你必须使用数据库中定义的表名和列名；这要求了解内部数据结构， 因此使用原生查询
时一定要注意。表名可以通过 API 获得，可以尽量减少对数据库的依赖。

	List<Task> tasks = taskService.createNativeTaskQuery()
        .sql("SELECT count(*) FROM " + managementService.getTableName(Task.class) + " T WHERE T.NAME_ = #{taskName}")
        .parameter("taskName", "gonzoTask")
        .list();

    long count = taskService.createNativeTaskQuery()
        .sql("SELECT count(*) FROM " + managementService.getTableName(Task.class) + " T1, "
               + managementService.getTableName(VariableInstanceEntity.class) + " V1 WHERE V1.TASK_ID_ = T1.ID_")
        .count();
      

