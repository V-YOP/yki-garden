article:: true

- 之前学习过hibernate Validator，但当时做的笔记弄丢了，最近可能又要开始写Java，所以把这一部分学习一下，做个笔记，避免到处写丑陋的样板代码。
- Hibernate Validator用于值校验，能够避免业务中过多地出现校验业务代码。Hibernate Validator支持对自定义类型，集合类型和内置Java类型进行校验，同时支持定义校验组，即让类型约束从属于特定组，只在进行该组的校验时才发挥作用。
- 通过下面的starter引入hibernate validator依赖：
- ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```
- Hibernate Validator利用**切面**完成自己的逻辑，它根据**类**上的`@Validated`注解去进行切入，并根据**参数**上的注解去进行校验。总的来说，要让校验起效，需要：
	- `spring-boot-starter-validation`依赖引入
	  logseq.order-list-type:: number
	- 类上标注`@Validated`（**必须在类上标注！必须是`@Validated`而非`@Valid`**）
	  logseq.order-list-type:: number
	- 方法的参数上标注相应校验注解（其中，实体类参数上必须加`@Valid`或`@Validated`注解）
	  logseq.order-list-type:: number
	- 调用方法时不能从内部调用（切面实现的锅）
	  logseq.order-list-type:: number
- 注意，Hibernate Validator通过切面工作，因此**它不仅能切入Controller，也能切入Service**，但仅此而已，某些时候还是需要手动进行校验。
- 下面是一个极简例子，涉及到控制器和实体类参数的校验，它已经提出了许多要注意的部分：
- ```java
  @RestController
  @Validated
  public class SomeController {
  
      // 定义带校验的实体类（注意实体类上不需要加任何额外注解）
      @Data
      public static class SomeDto {
          @NotBlank // 对于实体类，注解加到字段上
          private String name;
  
          @NotBlank @Length(min=10) // 可以同时有多个校验 注解
          private String value;
  
          @Valid // 对实体类用 Valid 进行嵌套校验（注意Valid在字段为null的时候不检查！）
          private SomeDto next;
  
          private List<@Valid SomeDto> dtos; // 对集合进行校验（注意这里不会检查集合是否为null或空！同时也没检查集合元素是否为null！）
      }
      @PostMapping("/query")
      public SomeDto query(
              // 实体类参数用 @Valid 或 @Validated注解进行校验
              @Valid @RequestBody SomeDto dto,
              @NotBlank @Length(min=10) @RequestParam String param) {
          return dto;
      }
  }
  
  ```
- `@Valid`校验能进行嵌套校验，但**它会直接忽略掉null**，因此上面的`SomeDto`定义中，`next`字段为null，以及`dtos`字段为null或空集合，或集合中存在null，都是容忍的。
- # 关于Service的校验的注意点
- 一般而言，Spring项目中Service的接口和实现是分离的，如果要校验Service，
- # 常用校验注解
- # 拦截校验异常
- # 自定义校验
- # 分组校验
- # 手动校验