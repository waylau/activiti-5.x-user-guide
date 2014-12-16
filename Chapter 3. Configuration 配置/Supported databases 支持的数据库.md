##Supported databases 支持的数据库

下面是可以引用的类型

Table 3.1. Supported databases

<table border="1" summary="Supported databases"><colgroup><col><col><col></colgroup>
<thead>
<tr>
<th>Activiti 数据库类型</th><th>JDBC URL示例</th><th>备注</th>
</tr>
</thead>
<tbody><tr><td>h2</td><td>jdbc:h2:tcp://localhost/activiti</td><td>默认配置数据库</td></tr><tr><td>mysql</td><td>jdbc:mysql://localhost:3306/activiti?autoReconnect=true</td><td>使用 mysql-connector-java 驱动进行测试</td></tr><tr><td>oracle</td><td>jdbc:oracle:thin:@localhost:1521:xe</td><td>&nbsp;</td></tr><tr><td>postgres</td><td>jdbc:postgresql://localhost:5432/activiti</td><td>&nbsp;</td></tr><tr><td>db2</td><td>jdbc:db2://localhost:50000/activiti</td><td>&nbsp;</td></tr><tr><td>mssql</td><td>jdbc:sqlserver://localhost:1433/activiti</td><td>&nbsp;</td></tr></tbody></table>