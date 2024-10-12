article:: true

- Hibernate Validator用于值校验，能够避免业务中过多地出现校验业务代码。Hibernate Validator支持对实体类和内置Java类型进行校验，同时支持定义校验组，即让类型约束从属于特定组，只在进行该组的校验时才发挥作用。通过下面的starter引入hibernate validator依赖：
- ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```
- Hibernate Validator利用**切面**完成自己的逻辑，它根据**类**和**方法**上的`@Validated`（或`@Valid`，但`@Valid`不支持校验组）注解去进行切入，并根据**参数**上的注解去进行校验。