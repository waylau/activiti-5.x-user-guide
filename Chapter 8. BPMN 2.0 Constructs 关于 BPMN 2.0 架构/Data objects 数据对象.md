##Data objects 数据对象

[试验]

BPMN 提供了一种功能，可以在流程定义或子流程中定义数据对象。根据BPMN 规范，流程定义可以包含复杂 XML 结构， 可以导入 XSD 定义。对于 Activiti 来说，作为 Activiti 首次支持的数据对象，可以支持如下的 XSD 类型：

```
	<dataObject id="dObj1" name="StringTest" itemSubjectRef="xsd:string"/>
```

```
	<dataObject id="dObj2" name="BooleanTest" itemSubjectRef="xsd:boolean"/>
```	

```
	<dataObject id="dObj3" name="DateTest" itemSubjectRef="xsd:datetime"/>
```

```	
	<dataObject id="dObj4" name="DoubleTest" itemSubjectRef="xsd:double"/>
```	

```
	<dataObject id="dObj5" name="IntegerTest" itemSubjectRef="xsd:int"/>
```

```	
	<dataObject id="dObj6" name="LongTest" itemSubjectRef="xsd:long"/>
```

数据对象定义会自动转换为流程变量，名称与 'name' 属性对应。 除了数据对象的定义之外，Activiti 也支持使用扩展元素来为这个变量赋予默认值。下面的 BPMN 片段就是对应的例子：
	
	<process id="dataObjectScope" name="Data Object Scope" isExecutable="true">
	          <dataObject id="dObj123" name="StringTest123" itemSubjectRef="xsd:string">
	            <extensionElements>
	              <activiti:value>Testing123</activiti:value>
	            </extensionElements>
	          </dataObject>
	      
