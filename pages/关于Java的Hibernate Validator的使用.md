article:: true

- Hibernate Validator用于值校验，能够避免业务中过多地出现校验业务代码。Hibernate Validator支持对实体类和内置Java类型进行校验，同时支持定义校验组，即让类型约束从属于特定组，只在进行该组的校验时才发挥作用。Spring MVC自动地包括了Hibernate Validator依赖。
- Hibernate Validator利用切面完成自己的逻辑，它根据`@Validated`（或`@Valid`，但`@Valid`不支持校验组）去切入**类**和**方法**上，然后