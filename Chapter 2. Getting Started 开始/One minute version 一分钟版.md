##One minute version 一分钟版

从 [Activiti 网站](http://www.activiti.org)下载 Activiti Explorer 的 WAR 文件后，可以按照下列步骤以默认配置运行示例。 你需要安装 [Java 运行时](http://java.sun.com/javase/downloads/index.jsp) 和 [Apache Tomcat](http://tomcat.apache.org/download-70.cgi)（其实，任何提供了 servlet 功能的 web 容器都可以正常运行。但是我们主要是使用 Tomcat 进行的测试）。

* 把下载的 activiti-explorer.war 复制到 Tomcat 的 webapps 目录下。
* 执行 Tomcat 的 bin 目录下的 startup.bat 或 startup.sh 启动服务器。
* Tomcat 启动后，打开浏览器访问 [ http://localhost:8080/activiti-explorer]( http://localhost:8080/activiti-explorer)（译者注：8080 是你的 Tomcat 安装默认端口，当然你可以给 Tomcat 指定其他端口号）。 使用 kermit/kermit 账号登录。

这样就好了！Activiti Explorer 默认使用 H2 内存数据库，如果你想使用其他数据库 请参考这里[长版本-Activiti setup](https://github.com/waylau/activiti-5.x-user-guide/blob/master/Chapter%202.%20Getting%20Started%20%E5%BC%80%E5%A7%8B/Activiti%20setup%20%E5%AE%89%E8%A3%85.md)。同时确保系统变量JAVA_HOME 设置正确。具体方法看你是什么操作系统。

只需要将 WAR 拷贝进 Tomcat 的 webapps 就能运行 Activiti Explorer 和 REST 应用。默认，应用是运行在内存数据库的，已经包含了示例流程，用户和群组信息。

下面是示例中可以使用的用户：

Table 2.1. The demo users

<table border="1" summary="The demo users"><colgroup><col><col><col></colgroup><thead><tr><th>用户 Id</th><th>密码</th><th>角色</th></tr></thead><tbody><tr><td>kermit</td><td>kermit</td><td>admin</td></tr><tr><td>gonzo</td><td>gonzo</td><td>manager</td></tr><tr><td>fozzie</td><td>fozzie</td><td>user</td></tr></tbody></table>

现在可以用上面的账号访问应用

Table 2.2. The webapp tools

<table border="1" summary="The webapp tools"><colgroup><col><col><col><col></colgroup><thead><tr><th>应用名称</th><th>URL</th><th>描述</th><td class="auto-generated">&nbsp;</td></tr></thead><tbody><tr><td>Activiti Explorer</td><td><a target="_top" href="http://localhost:8080/activiti-explorer" class="ulink">http://localhost:8080/activiti-explorer</a> </td><td>用户控制台。使用此工具来启动新的流程，分配任务，查看和认领任务等，这个工具还允许对 Activiti 引擎进行管理。
</td><td class="auto-generated">&nbsp;</td></tr></tbody></table>

注意 Activiti Explorer 演示实例只是一种简单快速展示 Activiti 的功能的方式。 但是并**不是**说只能使用这种方式使用 Activiti。 Activiti 只是一个 jar， 可以内嵌到任何 Java 环境
中：swing 或者 Tomcat, JBoss, WebSphere 等等。 也可以把Activiti 作为一个典型的单独运行的 BPM 服务器运行。 只要 java 可以做的，Activiti也可以。