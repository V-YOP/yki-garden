article:: true

- Hibernate Validator用于值校验，能够避免业务中过多地出现校验业务代码。Hibernate Validator支持对实体类和内置Java类型进行校验，同时支持定义校验组，即让类型约束从属于特定组，只在进行该组的校验时才发挥作用。通过下面的starter引入hibernate validator依赖：
- ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```
- Hibernate Validator利用**切面**完成自己的逻辑，它根据**类**上的`@Validated`（或`@Valid`）注解去进行切入，并根据**参数**上的注解去进行校验。总的来说，Hibernate Validator要发挥作用需要：
	- `spring-boot-starter-validation`依赖引入
	  logseq.order-list-type:: number
	- 类上标注`@Validated`或`@Valid`（**必须在类上标注！**）
	  logseq.order-list-type:: number
	- 方法的参数上标注相应注解
	  logseq.order-list-type:: number