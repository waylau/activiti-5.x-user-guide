##10.1. Requirements 要求

只支持符合以下要求的实体：

* 实体应该使用 JPA 注解进行配置，我们支持字段和属性访问两种方式。映射的超级累类也能够被使用。
* 实体中应该有一个使用 `@Id` 注解的主键，不支持复合主键 (`@EmbeddedId` 和 `@IdClass`)。Id 字段/属性能够使用 JPA 规范支持的任意类型： 原生态数据类型和他们的包装类型（boolean除外），`String`, `BigInteger`, `BigDecimal`, `java.util.Date` 和 `java.sql.Date`.