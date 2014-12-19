##One minute version 一分钟版

从 [Activiti 网站](http://www.activiti.org)下载 Activiti Explorer 的 WAR 文件后，可以按照下列步骤以默认配置运行示例。 你需要安装 [Java 运行时](http://java.sun.com/javase/downloads/index.jsp) 和 [Apache Tomcat](http://tomcat.apache.org/download-70.cgi)（其实，任何提供了 servlet 功能的 web 容器都可以正常运行。但是我们主要是使用 Tomcat 进行的测试）。

* 把下载的 activiti-explorer.war 复制到 Tomcat 的 webapps 目录下。
* 执行 Tomcat 的 bin 目录下的 startup.bat 或 startup.sh 启动服务器。
* Tomcat 启动后，打开浏览器访问 [ http://localhost:8080/activiti-explorer]( http://localhost:8080/activiti-explorer)（译者注：8080 是你的 Tomcat 安装默认端口，当然你可以给 Tomcat 指定其他端口号）。 使用 kermit/kermit 账号登录。

这样就好了！Activiti Explorer 默认使用 H2 内存数据库，如果你想使用其他数据库 请参考这里[长版本-Activiti setup](https://github.com/waylau/activiti-5.x-user-guide/blob/master/Chapter%202.%20Getting%20Started%20%E5%BC%80%E5%A7%8B/Activiti%20setup%20%E5%AE%89%E8%A3%85.md)。


