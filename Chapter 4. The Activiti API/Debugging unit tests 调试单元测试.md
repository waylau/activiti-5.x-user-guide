##Debugging unit tests 调试单元测试

当使用内存数据库 H2 进行单元测试时，下面的教程会告诉我们 如何在调试环境下更容易的监视 Activiti 的数据库。 这里的截图都是基于eclipse，这种机制很容易复用到其他 IDE 下。

假设我们已经在单元测试里设置了一个断点。 Ecilpse 里，在代码左侧双击：

![](http://99btgc01.info/uploads/2014/12/api.test.debug.breakpoint.png)

现在用调试模式运行单元测试（右击单元测试， 选择“运行为”和“单元测试”），测试会停在我们的断点上， 然后我们就可以监视测试的变量，它们显示在右侧面板里。

![](http://99btgc01.info/uploads/2014/12/api.test.debug.view.png)

要监视Activiti的数据，打开“显示”窗口 （如果找不到，打开“窗口”->“显示视图”->“其他”，选择显示。） 并点击（代码已完成）

org.h2.tools.Server.createWebServer("-web").start()

![](http://99btgc01.info/uploads/2014/12/api.test.debug.start.h2.server.png)

选择你点击的行，右击。然后选择“显示”（或者直接快捷方式就不用右击了）

![](http://99btgc01.info/uploads/2014/12/api.test.debug.start.h2.server.2.png)

现在打开一个浏览器，打开 http://localhost:8082 ， 输入内存数据库的 JDBC URL（默认为 jdbc:h2:mem:activiti ）， 点击连接按钮。

![](http://99btgc01.info/uploads/2014/12/api.test.debug.h2.login.png)

你现在可以看到 Activiti 的数据，通过它们可以了解单元测试时如何以及为什么这样运行的。

![](http://99btgc01.info/uploads/2014/12/api.test.debug.h2.tables.png)
