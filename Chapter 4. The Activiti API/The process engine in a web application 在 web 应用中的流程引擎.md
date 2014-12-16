##The process engine in a web application 在 web 应用中的流程引擎

ProcessEngine 是线程安全的， 可以在多线程下共享。在 web 应用中， 意味着可以在容器启动时创建流程引擎， 在容器关闭时关闭流程引擎。
下面代码演示了如何编写一个 ServletContextListener 在普通的Servlet 环境下初始化和销毁流程引擎：

	public class ProcessEnginesServletContextListener implements ServletContextListener {
	  
	  public void contextInitialized(ServletContextEvent servletContextEvent) {
	    ProcessEngines.init();
	  }
	
	  public void contextDestroyed(ServletContextEvent servletContextEvent) {
	    ProcessEngines.destroy();
	  }
	
	}

contextInitialized 方法会执行 ProcessEngines.init()。 这会查找 classpath 下的 activiti.cfg.xml 文
件， 根据配置文件创建一个 ProcessEngine（比如，多个 jar 中都包含配置文件）。 如果 classpath 中包含多个配置文件，确认它们有不同的名字。 当需要使用流程引擎时，可以通过

	ProcessEngines.getDefaultProcessEngine()

或者

	ProcessEngines.getProcessEngine("myName");

当然，也可以使用其他方式创建流程引擎，可以参考[配置章节](../Chapter 3. Configuration 配置/Creating a ProcessEngine 创建 ProcessEngine.md)中的描述。

ContextListener 中的 contextDestroyed 方法会执行ProcessEngines.destroy() ，这会关闭所有初始化的流程引擎。